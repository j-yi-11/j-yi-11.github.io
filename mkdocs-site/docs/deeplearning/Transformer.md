Transformer模型中采用 encoder-decoder 架构。论文中encoder层由6个encoder堆叠在一起，decoder层也一样。每一个encoder和decoder的结构如下：
![](../images/attention-encoder-decoder.png)

- Encoder层：先是有一个multi-head attention（是self attention），后面接了一个feed forward网络
- Decoder层：先是有一个masked multi-head attention，后面接入一个multi-head attention，最后接一个feed forward网络

### Encoder

#### embedding

这里面以Word为例子说明一下，embedding包含word embedding和positional embedding：

- word embedding可以根据一些已有的方法（如Word2Vec）得到。

- positional embedding是为了解释输入序列中word的顺序，这个embedding和word embedding的维度一样，可以通过训练/某个公式得到，在原文中采用公式得到，公式为：
  $$
  \text{PE}(\text{pos}, 2i)=\sin (\text{pos}/10000^{2i/d_{\text{model}}}),\\
  \text{PE}(\text{pos}, 2i+1)=\cos (\text{pos}/10000^{2i/d_{\text{model}}}),
  $$

  - pos是指当前词在句子中的位置，i是指向量中每个值的index：在**偶数位置，使用正弦编码，在奇数位置，使用余弦编码**。

最后是需要把word embedding和positional embedding相加（**add**）后的结果输入到attention里面。

#### attention

##### self attention

先看self attention（scaled dot-product attention: **通过query和key的相似性程度来确定value的权重分布**）：

Encoder里面的self attention结构可以用公式表明为：
$$
\text{Attention}(Q,K,V)=\text{softmax}(\frac{QK^T}{\sqrt{d_k}})V
$$
由于真正的encoder是多个encoder block拼接在一起，self attention的输入是上一个block的输出或者最原始的输入。

1. 假设我们输入的embedding为$X$，我们在最开始的时候，用三个矩阵$W^Q,W^K,W^V$（64*512维，这些矩阵的第二个维度需要和embedding的维度一致，这些矩阵的参数就是通过transformer的训练得到的），然后把$X$左乘上去，得到最开始的Q, K, V（512维）。
2. 计算attention score。假设计算第一个词 “Thinking” 的自注意力。我们需要根据 “Thinking” ，对sentence中的每个word都计算一个score。这些score决定了我们在编码 “Thinking” 这个词时，需要对句子中其他位置的每个词放置多少的注意力。这些score，是通过计算 “Thinking” 的 Query 向量和需要评分的词的 Key 向量的点积得到的。如果我们计算句子中第一个位置词的注意力分数，则第一个分数是 $\boldsymbol{q_1}\cdot\boldsymbol{k_1}$ 第二个分数是 $\boldsymbol{q_1}\cdot\boldsymbol{k_2}$。然后将每个分数除以 $\sqrt{d_k}$（$d_k$是Key向量的维度）,目的是在反向传播时，求梯度更加稳定。最后将这些score进行 Softmax 将分数进行归一化处理，使得它们都为正数并且和为 1。
3. 将每个 Softmax 分数分别与每个 Value 向量相乘。这种做法背后的直觉理解是：对于分数高的位置，相乘后的值就越大，我们把更多的注意力放在它们身上；对于分数低的位置，相乘后的值就越小，这些位置的词可能是相关性不大，我们就可以忽略这些位置的词。将加权 Value 向量（即上一步求得的向量）求和。这样就得到了自注意力层在这个位置的输出。

https://www.cnblogs.com/mantch/p/11591937.html

##### multi-head

multi-head self attention**不仅只初始化一组Q、K、V的矩阵，而是初始化多组，tranformer使用了8组**，所以最后得到的是8个矩阵。具体做法：首先，通过8个不同的线性变换对Query、Key 和 Value 进行映射；然后，将不同的 Attention **拼接**起来；最后进行一次线性变换。每一组注意力用于将输入映射到不同的子表示空间，这使得模型可以在不同子表示空间中关注不同的位置。整个计算过程可表示为：
$$
\text{MultiHead}(Q,K,V)=\text{Concat}(\text{head}_1,⋯,\text{head}_h)W^O (h=8)\\
\text{head}_i=\text{Attention}(QW_i^Q,KW_i^K,VW_i^V)
$$
在多头注意力下，我们为每组注意力单独维护不同的 Query、Key 和 Value 权重矩阵，从而得到不同的 Query、Key 和 Value 矩阵。按照上面的方法，使用不同的权重矩阵进行 8 次自注意力计算，就可以得到 8 个不同的 $\boldsymbol{Z}$ 矩阵。因为前馈神经网络层接收的是 1 个矩阵（每个词的词向量），而不是上面的 8 个矩阵。因此，我们需要一种方法将这 8 个矩阵整合为一个矩阵：把矩阵$\boldsymbol{Z_0},\boldsymbol{Z_1},\cdots,\boldsymbol{Z_7}$拼接后的矩阵和权重矩阵$\boldsymbol{W^O}$相乘。得到最终的矩阵 $\boldsymbol{Z}$，这个矩阵包含了所有注意力头的信息。这个矩阵会输入到 FFN 层。

##### 位置前馈网络（Position-wise Feed-Forward Networks）

   位置前馈网络就是一个全连接前馈网络，每个位置的词都单独经过这个完全相同的前馈神经网络。其由两个线性变换组成，即两个全连接层组成，第一个全连接层的激活函数为 ReLU 激活函数。可以表示为：
$$
\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2
$$
在每个编码器和解码器中，虽然这个全连接前馈网络结构相同，但是**不共享参数**。整个前馈网络的输入和输出维度都是 $d_{\text{model}}=512$，第一个全连接层的输出和第二个全连接层的输入维度为$d_{\text{ff}}=2048$。

##### 残差连接和层归一化

  编码器结构中有一个需要注意的细节：每个编码器的每个子层（Self-Attention 层和 FFN 层）都有一个残差连接，再执行一个层标准化操作，整个计算过程可以表示为：
$$
\text{SubLayerOutput} = \text{LayerNorm}(x + \text{SubLayer}(x))
$$
为了方便进行残差连接，编码器和解码器中的所有子层和嵌入层的输出维度需要保持一致，在 Transformer 论文中 $d_{\text{model}}=512$。

##### Mask（掩码）

  Mask对某些值进行掩盖，使其在参数更新时不产生效果。Transformer涉及两种mask，分别是Padding Mask 和 Sequence Mask。其中，Padding Mask 在所有的 scaled dot-product attention 里面都需要用到，而 Sequence Mask 只有在 Decoder 的 Self-Attention 里面用到。

##### Padding Mask

  因为每个batch输入sequence的长度是不一样的，所以我们要对输入序列进行对齐。具体来说，就是在较短的序列后面填充 0（但是如果输入的序列太长，则是截断，把多余的直接舍弃）。因为这些填充的位置，其实是没有什么意义的，所以我们的 Attention 机制不应该把注意力放在这些位置上，所以我们需要进行一些处理。

  具体的做法：把这些位置的值加上一个非常大的负数（负无穷），这样的话，经过 Softmax 后，这些位置的概率就会接近 0。

##### Sequence Mask

  Sequence Mask 是为了使得 Decoder 不能看见未来的信息。也就是对于一个序列，在$t$时刻，我们的解码输出应该只能依赖于$t$时刻之前的输出。因此我们需要想一个办法，把$t$时刻之后的信息给隐藏起来。

  具体的做法：产生一个上三角矩阵，上三角的值全为 0。把这个矩阵作用在每个序列上，就可以达到我们的目的。

  总结：对于 Decoder 的 Self-Attention，里面使用到的 scaled dot-product attention，同时需要 Padding Mask 和 Sequence Mask，具体实现就是两个 Mask 相加。其他情况下，只需要 Padding Mask。

https://blog.csdn.net/benzhujie1245com/article/details/117173090