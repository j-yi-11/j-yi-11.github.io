https://zhouyifan.net/2024/07/14/20240703-SD3/


Scaling Rectified Flow Transformers for High-Resolution Image Synthesis 用整流 (rectified flow) 生成模型、Transformer 神经网络做了模型参数扩增实验，以实现高质量文生图大模型。除了将去噪网络从 U-Net 改成 DiT 外，SD3 还在模型结构与训练策略上做了很多小改进：
- 改变训练时噪声采样方法
- 将一维位置编码改成二维位置编码
- 提升 VAE 隐空间通道数
- 对注意力 QK 做归一化以确保高分辨率下训练稳定

扩散模型 LDM 会使用一个神经网络模型来对噪声图像去噪。为了实现文生图，该去噪网络会以输入文本为额外约束。相比之前多数扩散模型，SD3 的主要改进是把**去噪模型的结构从 U-Net 变为了 DiT**。
### 整流模型介绍
所谓图像生成，其实就是让神经网络模型学习一个图像数据集所表示的分布，之后从分布里随机采样。比如我们想让模型生成人脸图像，就是要让模型学习一个人脸图像集的分布。我们很难表示出一个适合采样的复杂分布。因此，我们会把学习一个分布的问题转换成学习一个简单好采样的分布到复杂分布的映射。一般这个简单分布都是标准正态分布。

我们可以用简单的算法采样在原点附近的来自标准正态分布的蓝点，我们要想办法得到蓝点到红点的映射方法。学习这种映射依然是很困难的。而近年来包括扩散模型在内的几类生成模型用一种巧妙的方法来学习这种映射：从纯噪声（标准正态分布里的数据）到真实数据的映射很难表示，但从真实数据到纯噪声的逆映射很容易表示。所以，我们先人工定义从图像数据集到噪声的变换路线（红线），再让模型学习逆路线（蓝线）。让噪声数据沿着逆路线走，就实现了图像生成。

我们又可以用一种巧妙的方法间接学习图像生成路线。知道了预定义的数据到噪声的路线后，我们其实就知道了数据在路线上每一位置的速度（红箭头）。那么，我们可以以每一位置的反向速度（蓝箭头）为真值，学习噪声到真实数据的速度场。这样的学习目标被称为流匹配。

对于不同的扩散模型及流匹配模型，其本质区别在于图像到噪声的路线的定义方式。在扩散模型中，图像到噪声的路线是由一个复杂的公式表示的。而整流模型将图像到噪声的路线定义为了直线。比如根据论文的介绍，整流中$t$时刻数据$z_t$由真实图像$x_0$变换成纯噪声$\epsilon$的位置为:$z_t=(1-t)x_0+t\epsilon$

而较先进的扩散模型 EDM 提出的路线公式为（$b_t$是一个形式较为复杂的变量）：$z_t=x_0+b_t\epsilon$

由于整流最后学习出来的生成路线近乎是直线，这种模型在设计上就支持少步数生成。

### 非均匀训练噪声采样
在学习这样一种生成模型时，会先随机采样一个时刻$t\in [0,1]$根据公式获取此时刻对应位置在生成路线上的速度，再让神经网络学习这个速度。直观上看，刚开始和快到终点的路线很好学，而路线的中间处比较难学。因此，在采样时刻$t$时，SD3 使用了一种非均匀采样分布。

SD3 主要考虑了两种公式: mode和logit-norm。二者的共同点是中间多，两边少。mode 相比 logit-norm，在开始和结束时概率不会过分接近 0。

### 提升自编码器通道数
在当时设计整套自编码器 + LDM 的生成架构时，SD 的用了一个能把图像下采样 8 倍，通道数变为 4 的隐空间图像。比如输入[512, 512, 3]的图像会被自编码器编码成[64, 64, 4]。而近期有些工作发现，这个自编码器不够好，提升隐空间的通道数能够提升自编码器的重建效果。因此，SD3 把隐空间图像的通道数从4改为16。

### 多模态 DiT (MM-DiT)
SD3 的去噪模型是一个 Diffusion Transformer (DiT)。如果去噪模型只有带噪图像这一种输入的话，DiT 则会是一个结构非常简单的模型：图像过图块化层 (Patching) 并与位置编码相加，得到序列化的数据。这些数据会像标准 Transformer 一样，经过若干个子模块，再过反图块层得到模型输出。DiT 的每个子模块 DiT-Block 和标准 Transformer 块一样，由 LayerNorm, Self-Attention, 一对一线性层 (Pointwise Feedforward, FF) 等模块构成。

然而，扩散模型中的去噪网络一定得支持带约束生成。这是因为扩散模型约束于去噪时刻$t$。此外，作为文生图模型，SD3 还得支持文本约束。DiT 及本文的 MM-DiT 把模型设计的重点都放在**处理额外约束**上。

- 时刻约束: SD3的模块保留DiT的设计，用自适应 LayerNorm (Adaptive LayerNorm, AdaLN) 来引入额外约束。具体来说，过了 LayerNorm 后，数据的均值、方差会根据时刻约束做调整。另外，过完 Attention 层或 FF 层后，数据也会乘上一个和约束相关的系数。

- 文本约束。文本约束以2种方式输入进模型：**与时刻编码拼接**、**在注意力层中融合**。为了提高 SD3 的文本理解能力，描述文本 (“Caption”) 经由3种编码器编码，得到两组数据。一组较短的数据会经**由 MLP 与文本编码加到一起**；另一组数据会经过**线性层，输入进 Transformer 的主模块**中。

看输入的图像编码$x$和文本编码$c$。二者以相同的方式做了 DiT 里的 LayerNorm, FF 等操作。不过，相比此前多数基于 DiT 的模型，此模块用了**融合注意力层**。具体来说，在过注意力层之前，$x$和$c$对应的$Q, K, V$会分别拼接到一起，而不是像之前的模型一样，$Q$来自图像，$K, V$来自文本。过完注意力层，输出的数据会再次拆开，回到原本的独立分支里。

### 比例可变的位置编码
此前多数方法在使用类 ViT 架构时，都会把图像的图块从左上到右下编号，把二维图块拆成一维序列，再用这种一维位置编码来对待图块。这样做有一个很大的坏处：生成的图像的分辨率是无法修改的。解决此问题的方法很简单，只需要将一维的编码改为二维编码(可以理解为二维数组)。这样 Transformer 就不会搞混二维图块间的关系了。
