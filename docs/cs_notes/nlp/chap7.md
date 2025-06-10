# Chapter 7. Transformers

传统RNN无法并行计算，传统CNN能够并行计算，但每次只能看到局部区域的信息。我们需要一种既能并行，又能很好地处理序列数据的方法。

## 7.1 Self-Attention

### 计算方法

给定一个序列 $X = [x^1, x^2, ..., x^n]$，我们需要计算每个元素对其他元素的注意力权重。具体步骤如下：

1. **词向量编码**：将每个元素 $x^i$ 转换为一个词向量表示。
    $$ a^i = Wx^i $$
2. **线性变换**：将每个词向量 $a^i$ 通过三个不同的线性变换，得到**查询（Query, to match others）、键（Key, to be matched）和值（Value, information to be extracted）**向量。

    $$ \begin{aligned}q^i &= W^qa^i \\ k^i &= W^ka^i \\ v^i &= W^va^i \end{aligned} $$

3. **计算注意力权重**：使用点积计算查询向量与键向量的相似度，然后通过 softmax 函数归一化得到注意力权重。（公式以计算第1个元素的注意力权重为例，其中 $d$ 为 $q$ 和 $k$ 的维度）
    $$ \hat{\alpha_{1, i}} = Softmax(\alpha_{1, i}) = Softmax(\frac{q^1 \cdot k^i}{\sqrt{d}}) $$
4. **加权求和**：将注意力权重与值向量相乘，得到最终的输出。（公式以计算第1个元素的输出为例）
    $$ b^1 = \sum_i \hat{\alpha_{1, i}} v^i $$

<figure markdown="span">
    ![](img/56.jpg){width="500"}
</figure>

<figure markdown="span">
    ![](img/57.jpg){width="500"}
</figure>

!!! note "对注意力层的理解"
    每过一个注意力层其实都是对各个词向量做一轮更新，得到新的一组词向量。新的词向量包含了更精准的上下文信息，相当于在词向量空间中更靠近了上下文的语义对应的那个词向量。

!!! note "对 $q, k, v$ 的举例理解"

    > 为什么 $q, k$ 的维度一般比词向量小？

    $q$: 查询向量，如意思为”我前面是否有形容词？”
    $k$: 键向量，如意思为“我是形容词！”
    $v$: 值向量，如“蓝色” token 的值向量意思为“把名词变蓝！”


### 矩阵加速

自注意力计算的过程都可以写成矩阵乘法，并用**GPU并行加速**：

令 $I = [a^1, a^2, ..., a^n]$ 为输入矩阵，$O = [b^1, b^2, ..., b^n]$ 为输出矩阵，有：

$$\begin{cases}Q &= W^qI \\ K &= W^kI \\ V &= W^vI \end{cases},\ \hat{A} = Softmax(A) = Softmax(\frac{K^TQ}{\sqrt{d}}),\ O = V\hat{A}$$

### 多头自注意力 Multi-Head Self-Attention

每个元素对应多个 Q, K, V 向量，分别计算注意力权重，然后将所有头的输出拼接起来。

实际实现与普通自注意力相同，只是在计算出 Q, K, V 后多一步分块，计算出 B 后再多一步拼接。

<figure markdown="span">
    ![](img/58.jpg){width="500"}
</figure>

<figure markdown="span">
    ![](img/59.jpg){width="350"}
</figure>

### 位置编码 Positional Encoding

因为自注意力机制不考虑元素的顺序信息（并行时每个词是对等的），我们需要添加位置编码来保留序列中元素的位置信息。

最简单的做法是给每个 $x_i$ 拼接一个 one-hot 位置向量 $p_i$：

<figure markdown="span">
    ![](img/60.jpg){width="150"}
</figure>

## 7.2 Transformer

<figure markdown="span">
    ![](img/61.jpg){width="400"}
</figure>

???+ note 
    其实现在大部分大语言模型都只用了 Transformer 的编码器部分，因为编码器支持并行计算，而解码器需要逐步生成，不能并行。

??? question "Transformer 和 MLP、CNN、RNN 三者中谁更像？"
    更像 MLP 和 RNN.
    
    - 更像 MLP 的原因：把 k, q, v 看作输入，输出实质上是 v 的加权和，与 MLP 的做法一致。
        - 只是 MLP 的权重是学出来的，而 Transformer 的注意力分数是根据输入计算出来的（这也表明 Transformer 比 MLP 更灵活）
    - 更像 RNN 的原因：后面会讲

### Add & Norm

Add 指残差连接时的加法，Norm 为 Layer Normalization.

???+ note "Transformers without Normalization"
    2025最新的工作，观察 Layer Normalization 的输入和输出，发现函数关系很像 $\tanh$，可以直接用 $\tanh$ 函数代替 Layer Normalization.

### Feed Forward

Transformer 中的前馈网络是一个中间宽、前后窄的 MLP**（真的是MLP吗？好像是 $1\times 1$ 卷积，也就是一一对应独立计算的？目前理解：对输出的每个向量独立做这个二层MLP，先升维再降维，而不同向量之间互不影响，后半句解释了什么是 $1 \times 1$ 卷积）**

- 实际上计算时因为硬件资源限制，将中间维度分块，割成多个小的 MLP 并行计算，最终相加。这样的做法称为 sharding.

大模型的参数量主要来自于前馈网络。


???+ note "对前馈网络MLP的理解"

    MLP由两个线性变换和一个激活函数组成。中间状态维度比输入和输出维度大。

    <figure markdown="span">
        ![](img/74.jpg){width="500"}
    </figure>

    在第一个线性变换中，权重矩阵的每一行代表了一个问题的特征（如“是迈克尔·乔丹”），如果对应的词向量也代表迈克尔·乔丹，则二者的点积，再加上偏置后约为1，表示是迈克尔·乔丹。

    <figure markdown="span">
        ![](img/72.jpg){width="500"}
    </figure>

    当然我们的问题的答案只是 是或否（二分类问题），所以接下来需要经过一个 ReLU 激活函数，使得负数结果变为0.

    接下来经过第二个线性变换，其权重矩阵的每一列绑定了前一个线性变换的每一行对应的特征（如“是迈克尔·乔丹”绑定了“篮球”“芝加哥公牛”等特征）。如果对应的二分类结果接近1，则这个神经元被激活，其特征的值被加到输出中。

    > 有个问题：第二个线性变换的偏置有什么用？

    <figure markdown="span">
        ![](img/73.jpg){width="500"}
    </figure>

    最后将输出的向量与一开始输入的词向量相加，形成残差连接。所以看起来的效果就是：如果输入词向量中包含迈克尔·乔丹，则MLP会将与迈克尔·乔丹相关的特征，如“篮球”“芝加哥公牛”等添加到该词向量中。

## 7.3 Transformer 变体与挑战者

### MLP-Mixer

MLP-Mixer 使用 MLP 代替自注意力机制做信息混合，主要用于图像领域。

其中 Mixer Layer 包含两个 MLP：

1. Channel-mixing MLP 对每个维度（通道）的特征进行融合
2. Token-mixing MLP 对每个token内部的特征进行融合

过程类似信号处理中的傅里叶变换，依次对两个维度做傅里叶变换，等同于同时对两个维度做傅里叶变换。

<figure markdown="span">
    ![](img/62.jpg){width="500"}
</figure>

### Linear Attention

把 RNN 的计算过程展开：

<figure markdown="span">
    ![](img/63.jpg){width="500"}
</figure>

发现无法并行计算的最大原因来自 $f_A$，于是我们直接去掉 $f_A$，即不让模型遗忘：

<figure markdown="span">
    ![](img/64.jpg){width="500"}
</figure>

用 $D$ 替换 $f_B$：

<figure markdown="span">
    ![](img/65.jpg){width="500"}
</figure>

假设 $D_t = v_tk_t^T$：

<figure markdown="span">
    ![](img/66.jpg){width="500"}
</figure>

**就是 attention 中 $k, q, v$ 的形式！** （self-attention without softmax, or RNN without forget gate, called Linear Attention）

> 感觉 RNN 的精髓就在于序列化，也就是 $f_A$ 的部分，而这样去掉顺序信息，再强行套上 attention 的形式，有些附会了...


### Retention Network

在 Linear Attention 的基础上，加入控制信息保留和遗忘的系数

> 事实上好像没那么简单，有一些复杂的推导

<figure markdown="span">
    ![](img/67.jpg){width="500"}
</figure>

### Gated Retention Network

引入门控机制来控制信息的保留和遗忘

<figure markdown="span">
    ![](img/68.jpg){width="500"}
</figure>

---

???+ note "Linear Attentinon Variants"
    有各种各样的设计：

    <figure markdown="span">
        ![](img/69.jpg){width="500"}
    </figure>

    - 其中 Mamba 是真正比较成功地挑战了 Transformer 的模型


