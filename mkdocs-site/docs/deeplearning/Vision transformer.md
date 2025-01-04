https://paddlepedia.readthedocs.io/en/latest/tutorials/computer_vision/classification/ViT.html
https://blog.csdn.net/qq_42410605/article/details/142899935
ViT 的核心思想是将传统的（CNN）的卷积操作替换为 Transformer 的注意力机制，借鉴 Transformer 模型在自然语言处理（NLP）中的成功经验，用于图像分类任务。

### 结构详解

主要由三个模块组成：

- Linear Projection (Patch + Position 的Embedding层)
- Transformer Encoder
- MLP Head (分类层)

#### embedding
Transformer Encoder读入的是一小块一小块这样的图片。这样做的原因是：将**一个个小块图像视为token（token可以理解为NLP中的一个字或词），在Transformer中计算每个token之间的相关性**。CNN以卷积+池化的方式不断下采样，这样理论上模型可以通过加深模型深度，达到增大感受野的目的。不过实际结果显示，CNN**对边缘的响应很弱**。这也非常好理解，越靠边缘的像素，因为被卷积次数少，自然在梯度更新时，贡献更少。CNN只能和临近像素计算相关性。由于其滑窗卷积的特性，**无法对非邻域的像素共同计算**，例如左上角的像素无法和右下角的像素联合卷积。这就导致了某些空间信息是无法利用的。同时根据MAE论文中所说的，**自然图像具有冗余性，即相邻像素点代表的信息是差不多的**，所以只计算邻域像素无法最大化利用图像特征。
回到ViT中，仅仅把图像拆分成小块（patch）是不够的，Transformer Encoder需要的是一个向量，shape为[num_token, token_dim]。对于图片数据来说，shape为[H,W,C]是不符合要求的，所以就需要转换，要将图片数据通过这个Embedding层转换成token。
以ViT-B/16为例：假设输入图像为224x224x3，一个token原始图像shape为16x16x3，那这样就可以将图像拆分成(224/16)^2 = 196个patch，然后将每个patch线性映射至一维向量中，那么这个一维向量的长度即为16∗16∗3=768 维。将196个token叠加在一起最后维度就是[196, 768]。

在代码实现中，patch的裁剪是用一个patch_size大小的卷积同时以patch_size的步长进行卷积实现。

#### Positional Encoding

因为 Transformer 的注意力机制不依赖于输入的顺序，而图像中的空间信息是重要的，因此需要给每个图像块添加位置编码（Positional Encoding），以保留图像块的位置信息。这样，Transformer 可以理解图像块之间的相对位置关系。位置编码的方式与 NLP 中的 Transformer 类似，ViT 默认使用 **可学习的1D位置编码**，将二维图像的分割图块按照固定顺序展平成一维序列后，为序列中的每个位置分配一个可学习的编码向量。
token可以添加**Positional Encoding（可以学习的参数）**。由于self-attention的机制，在没有字符位置的情况下，“我爱你”和“你爱我”的计算结果是一样的。所以当添加了Positional Encoding，分类准确率将会更高。所以迁移到ViT的Embeding层中，位置信息是以**Add**的方式融合在输入tensor中的。

#### Classification Token
另外，输入tensor还需要有一个分类层，数据格式和其他token一样，长度为768的向量，与位置编码的融合方式不一样，这里做的是**Concat**，这样做是因为**分类信息是在后面需要取出来单独做预测**，所以不能以Add方式融合，shape也就从[196, 768]变为[197, 768]。

ViT 模型在输入图像块之前，通常会添加一个分类标记（[CLS] Token）。这个分类标记类似于 BERT 模型中的 [CLS] 标记，用来**代表整个图像的全局特征**。最终，经过 Transformer 编码器的处理后，CLS 标记的输出被用于进行图像分类。CLS 标记的输出经过一个**全连接层**，将其映射到目标类别空间中，得到最终的分类结果。CLS 是 “classification” 的缩写，表示分类。它是一个附加到图像块序列之前的向量，类似于 BERT 模型中处理文本任务时添加的 [CLS] 标记。CLS 标记没有直接对应于任何特定的图像块，它只是一个特殊的向量，用于捕获整个图像的全局信息。

> 例如：[0.9, 0.05, 0.05]表示 90% 的概率是“猫”，5% 的概率是“狗”，5% 的概率是其他类别。


#### Transformer Encoder
Transformer Encoder也就是堆叠encoder block几次，和常规的CNN类似，主要由几个部分组成：
- Layer Normalization
相比于Batch Normalization（BN）针对**所有**的样本，对某一个特征图计算均值和方差，然后然后对这个特征图神经元做归一化；LN是对**某一个**样本，计算该样本所有特征图的均值和方差，然后对这个样本做归一化。
BN适用于不同mini batch数据分布差异不大的情况，而且BN需要开辟变量存每个节点的均值和方差，空间消耗略大；而且BN适用于有mini_batch的场景。LN只需要一个样本就可以做归一化，可以避免 BN 中受 mini-batch 数据分布影响的问题，也不需要开辟空间存每个节点的均值和方差。
- Multi-Head Attention
这个结构是self-attention中的一种。每个图像块与其他所有图像块之间的相似性通过自注意力机制计算，模型通过这种机制捕捉全局的特征表示。
- DropOut/DropPath
论文中描述是使用了DropOut，但实际别人复现的代码中使用的是DropPath。两者对最后的结果影响不大，这里也不展开详细说了。
- MLP Block
由一个全连接层 + GELU激活函数 + DropOut组成，采用的是倒瓶颈结构，输入特征层经过一次全连接之后，通道膨胀为原来的4倍，后一个全连接层再恢复成原来的数目。所以经过MLP Block之后tensor的shape是不变的。



### 如何训练得到？

利用图像分类的数据集进行训练。
https://zhuanlan.zhihu.com/p/382549255
https://arxiv.org/pdf/2106.10270