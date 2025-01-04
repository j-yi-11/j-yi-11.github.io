之前尝试做过，现在记录一下对ControlNet的认识。

ControlNet是一种想法，在公开出来的版本中是在stable diffusion模型上应用的。它的基本思想如下：

1. 对于原来的大的文生图模型，先冻结它的参数
2. 把encode层权重参数进行copy，后面要**train的就是encode这部分**
3. copy后的encoder和原始模型通过零卷积层进行连接

The trainable copy is connected to the locked model with
zero convolution layers, denoted Z(·; ·). Specifically, Z(·; ·)
is a 1 × 1 convolution layer with both weight and bias initialized
to zeros. To build up a ControlNet, we use two
instances of zero convolutions with parameters Θz1 and Θz2
respectively. The complete ControlNet then computes
yc = F(x; Θ) + Z(F(x + Z(c;Θz1);Θc);Θz2), (2)
where yc is the output of the ControlNet block. In the first
training step, since both the weight and bias parameters of
a zero convolution layer are initialized to zero, both of the
Z(·; ·) terms in Equation (2) evaluate to zero, and



stable diffusion结构

Both the encoder and decoder contain 12 blocks,
and the full model contains 25 blocks, including the middle
block. Of the 25 blocks, 8 blocks are down-sampling or
up-sampling convolution layers, while the other 17 blocks
are main blocks that each contain 4 resnet layers and 2 Vision
Transformers (ViTs). Each ViT contains several crossattention
and self-attention mechanisms.



controlnet是如何被加到stable diffusion的：

The ControlNet structure is applied to each encoder level
of the U-net (Figure 3b). In particular, we use ControlNet
to create a trainable copy of the 12 encoding blocks and 1
middle block of Stable Diffusion. The 12 encoding blocks
are in 4 resolutions (64 × 64, 32 × 32, 16 × 16, 8 × 8) with
each one replicated 3 times. The outputs are added to the
12 skip-connections and 1 middle block of the U-net. Since
Stable Diffusion is a typical U-net structure, this ControlNet
architecture is likely to be applicable with other models.