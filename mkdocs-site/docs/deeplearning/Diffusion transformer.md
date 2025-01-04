Scalable Diffusion Models with Transformers.
https://blog.csdn.net/weixin_44934783/article/details/136263914

DiT是**基于Transformer架构的扩散模型**。用于各种图像（SD3、FLUX等）和视频（Sora等）视觉生成任务。

DiT证明了Transformer思想与扩散模型结合的有效性，并且还验证了Transformer架构在扩散模型上具备较强的Scaling能力，在稳步增大DiT模型参数量与增强数据质量时，DiT的生成性能稳步提升。

### 与传统(U-Net)扩散模型区别
- 架构
DiT将扩散模型中经典的U-Net架构**完全替换成了Transformer**架构。能够高效地捕获数据中的依赖关系并生成高质量的结果。
- 噪声调度策略
DiT扩散过程的采用简单的Linear scheduler（timesteps=1000，beta_start=0.0001，beta_end=0.02）。在传统的U-Net扩散模型（SD）中，所采用的noise scheduler通常是Scaled Linear scheduler。
### 与传统扩散的相同
DiT的整体框架使用和Stable Diffusion相同的Latent Diffusion架构，使用了和SD一样的VAE模型将像素级图像压缩到低维Latent特征。这极大地降低了扩散模型的计算复杂度（减少Transformer的token的数量）。
                  
### 输入图像的Patch化（Patchify）和位置编码
和ViT一样，DiT也需要经过Patch和位置编码。DiT和ViT一样，首先采用一个Patch Embedding来将输入图像Patch化，主要作用是将VAE编码后的**二维特征转化为一维序列**，从而得到一系列的图像tokens。DiT在这个图像Patch化的过程中，设计了**patch size超参数**，它直接决定了图像tokens的大小和数量，从而影响DiT模型的整体计算量。DiT论文中共设置了三种patch size，分别是 2, 4, 8。patch size 为 2*2 是最理想的。

Latent Diffusion Transformer结构中，输入的图像在经过VAE编码器处理后，生成一个Latent特征，Patchify的目的是将Latent特征转换成一系列 T 个 token（将Latent特征进行Patch化），每个 token 的维度为 d。Patchify 创建的 token 数量 T 由超参数 p 决定。将 p 减半会使 T 增加四倍，因此至少使整个 transformer Gflops 增加四倍。

### 位置编码
在执行 patchify 之后，我们对所有输入 token 应用标准的 ViT 基于频率的位置嵌入（正弦-余弦版本）。图像tokens后，还要加上Positional Embeddings进行位置标记，DiT中采用经典的**非学习sin&cosine位置编码**。

### DiT Block模块详细信息
DiT在完成输入图像的预处理后，就要将**Latent特征输入到DiT Block中进行特征的提取**，与ViT不同的是，DiT作为扩散模型还需要在Backbone网络中嵌入额外的condition（不同模态的条件信息等，包括了Timesteps以及类别标签（文本信息））。

#### DiT中的Backbone

- 常规的特征提取抽象；
- 对图像和特征额外的多模态条件特征进行融合。

额外信息都可以**采用Embedding来进行编码，从而注入DiT中**。DiT论文中为了增强特征融合的性能，一共设计3种方案来实现两个额外Embeddings的嵌入(说白了，就是**怎么加condition**)。

#### 上下文条件化

在上下文条件化中，条件信息 (t)（时间步嵌入） 和 (c)（其他条件，如类别或文本嵌入） 被表示为两个独立的嵌入向量。这些向量被**附加到输入图像 token 序列的开头**，形成一个扩展后的输入序列。Transformer 模块把条件化token通过**多头自注意力机制**与图像token一起信息交换。
这与 ViT 中的 cls tokens 类似，它允许我们**无需修改**就使用标准的 ViT 模块。在最后一个模块之后，我们从序列中**移除条件化token**。

#### 交叉注意力模块
条件信息 (t)（时间步嵌入） 和 (c)（其他条件，如类别或文本嵌入）被**concat**为一个长度为 2 的序列，与图像 token 序列分开。Transformer 模块被修改为在**多头自注意力模块后添加一个多头交叉注意力层**，专门用于让图像 token与条件token进行交互，从而将条件信息显式注入到图像特征中。
**图像特征作为Cross Attention机制的Query。条件信息的Embeddings作为Cross Attention机制的键（Key）和值（Value）**。

#### AdaLN-Zero
AdaLN-Zero 在 AdaLN 的基础上新增了 残差缩放参数 $\alpha$，用于动态控制残差路径的影响。通过将 $\alpha$ 初始化为零，模型的初始状态被设置为恒等函数，从而确保输出在训练初期的稳定性。这种设计显著提升了模型的训练稳定性和收敛速度。

AdaLN 有 4 个参数：$\gamma_1, \beta_1, \gamma_2, \beta_2$用于自注意力和 MLP 模块的归一化操作，没有残差缩放参数。AdaLN-Zero 增加了 2 个参数：$\alpha_1, \alpha_2$ 用于控制残差路径的输出，显著提升了训练稳定性和适应性，因此总共 6 个参数。如果任务需要**更强的稳定性**（如深层 Transformer 模型或大规模扩散模型训练），AdaLN-Zero 是更优的选择。


##### DiT中具体的初始化
对DiT Block中的AdaLN和Linear层均采用参数0初始化。对于其它网络层参数，使用正态分布初始化和xavier初始化。

原始ViT中，在patchify之后，输入tokens会直接由一系列Transformer块处理。但DiT的输入除了noise图像外，有时还会处理额外的条件信息，如noise时间步t，类标签c，自然语言等。为了处理条件信息，本文探索了四种Transformer块的变体，其会对标准ViT模块进行修改。

- In-context conditioning
简单地将t和c的embedding合并到输入tokens序列中，类似于ViT中的cls token。并在最后一个Transformer块之后，从tokens序列中删除这些条件token。

- Cross-attention block
将t和c的embedding拼接为长度为2的token序列，通过修改Transformer模块，在多头自注意力模块之后添加一个额外的多头交叉注意力层，并使用多头自注意力的输出作为query，条件embeddings作为key和value来引入条件。

- Adaptive layer norm（adaLN）block
采用自适应归一化层（adaLN）替换Transformer块中的标准归一化层（LN）。跟标准adaLN不同的是，本文不直接学习维度缩放和移位参数$\gamma$和$\beta$，而是从t和c的embedding的和中回归（MLP）计算$\gamma$和$\beta$，以作用于主特征。

- adaLN-Zero block
adaLN DiT块的一种修改。adaLN-Zero除了回归$\gamma$和$\beta$，还会每个DiT模块结束之前回归一个维度缩放参数$\alpha$。并且adaLN的MLP会被初始化为零向量，这样初始化时DiT的残差模块就是一个identity函数。 与普通的adaLN块一样，adaLN-Zero给模型增加的Gflops可以忽略不计。
