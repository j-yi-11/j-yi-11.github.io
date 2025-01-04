这里主要记录一下对diffusion的认识

### 如何训练diffusion model
本质上是训练U-Net，让U-Net学会每一步如何去除噪声。

在每一轮的训练过程中，包含以下内容：

1. 每一个训练样本对应一个随机时刻向量time step，编码时刻向量t转化为对应的time step Embedding向量；
2. 将时刻向量t对应的true noise ε转化为噪声图；
3. 将成组的time step Embedding向量、Noisy image注入到U-Net训练；
4. U-Net输出预测噪声Predicted noise，与true noise构建损失。

#### 制作training dataset

一组训练集包括： **噪声强度** (人为预先设定)， **加噪后的图片** (原始没有噪声的图+噪声强度)，以及 **噪声图** （人为设定的噪声强度对应的纯噪声图，相当于 true noise）。

训练时U-Net只要在已知噪声强度的条件下，学习如何从加噪后的图片中计算出 **噪声图** 就可以了。我们并不直接输出无噪声的原图，而是让U-Net去 **预测原图上所加过的噪声** 。当需要生成图片的时候，我们用加噪图 **减掉** 噪声就能恢复出原图了。

### Stable Diffusion的training过程

大致思路和diffusion类似，只是多了文本的处理。

训练数据集的制作，和上面相比，加入 文本 信息，其他不变。

**文本语义信息怎么用？**：在内部的每一个的Resnet后面加上一个Attention模块处理语义信息，attention模块的注意力来源于CLIP对输入文本编码得到的text embedding。

### Stable Diffusion的inference过程

扩散模型的缺点：

1. *Diffusion Model*是在原图上完成前向、反向扩散过程，计算量巨大；
2. *Diffusion Model*只与时刻向量*t*产生作用，生成的结果不可控；

为改善扩散模型的缺点，Stable Diffusion引入图像压缩技术（VAE模型），在低维空间完成扩散过程；并添加CLIP模型，使文本-图像产生关联。

Stable Diffusion在实际应用中的过程：原图——经过编码器E变成低维编码图——DM的前向过程逐步添加噪声，变成噪声图——T轮U-Net网络完成DM的反向过程——经过解码器D变成新图。

1. Stable Diffusion会事先训练好一个编码器E、解码器D，来学习原始图像与低维数据之间的压缩、还原过程；
2. 首先通过训练好的编码器E ，将原始图像压缩成低维数据（$512\times 512 \rightarrow 4\times 64\times 64$），再经过多轮高斯噪声转化为低维噪声Latent data（不涉及DDIM inversion的时候可以一步加噪声到位）
3. 然后用低维噪声Latent data、时刻向量t、文本向量Text Embedding、在U-Net网络进行T轮去噪，完成反向扩散过程；
4. 将得到的低维去噪图通过训练好的解码器D，还原出原始图像，完成整个扩散生成过程。

CLIP模型可以粗浅理解为 求出图像和文本相似度 的大型预训练模型

#### 文生图

1. Stable Diffusion在潜空间里生成一个随机张量。我们通过设置随机种子seed来控制这个张量的生成。如果我们设置这个随机种子为一个特定的值，则会得到相同的随机张量。这就是我们在潜空间里的图片。但是当前还全是噪点。
2. U-Net将潜噪点图+文本提示词作为输入，并预测噪点，此噪点同样也在潜空间内（一个4 x 64 x 64的张量）
3. 从潜图片中抽取潜噪点，并生成了新的潜图片
4. Step 2 与 Step 3 重复特定采样次数，例如2000次。最后，VAE的decoder将潜图片转回像素空间，这便是我们最终得到的图片。

#### stable diffusion中的文本控制怎么做的

Tokenizer将每个输入的单词转为一个数，称为token。每个token然后转为一个768维的向量，称为词嵌入（embedding）。词嵌入然后由Text Transformer处理，并可以被U-Net处理。

Text Transformer的输出会被在U-Net中使用到多次。U-Net以一个叫做**cross-attention**机制的方式来使用它。这即是prompt适配图片的地方。

这里我们使用提示词“A man with blue eyes”作为例子。SD将单词blue与eyes组合到一起（self-attention within the prompt），这样便可以生成一个蓝眼睛的男人，而不是穿蓝衬衫的男人。然后它会使用这个信息引导反向扩散，使得最终生成的图片包含蓝色眼睛（cross-attention between 提示词与图片）

一个备注：Hypernetwork是一种fine-tune Stable Diffusion模型的技术，它会操纵cross-attention网络来注入风格。[LoRA](https://stable-diffusion-art.com/lora/)模型修改cross-attention模块的权重来修改风格。

#### 图生图（本质是：文图生图）

图生图的输入是一张输入**图片+文本提示词**。生成的图片会由输入图片以及文本提示词两者共同调节。

1. 输入图片编码到潜空间
2. 加入噪点到潜图片。Denoising strength控制加入多少噪点。如果为0，则不会加入噪点。如果为1，则会加入最多的噪点，这样潜图片则转为一张完全随机的张量（此时等价于文生图）。
3. U-Net使用潜噪点图以及文本提示词作为输入，并预测潜空间内的噪点（一个4 x 64 x 64 的张量）
4. 从潜图片中剔除潜噪点，并生成新的潜图片。 
5. Step 3 与 4 重复多次，直到特定采样步数，例如1000次。最后，VAE的解码器将潜图片转回像素空间，便得到了最终生成的图片。