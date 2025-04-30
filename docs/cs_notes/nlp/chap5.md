# Chapter 5. 循环神经网络 Recurrent Neural Networks

RNN是一种“有记忆”的神经网络，适合处理序列数据（如文本、语音、视频等）。

## 5.1 循环神经网络结构

### RNN基本结构

<figure markdown="span">
    ![](img/26.jpg){width="500"}
</figure>

- 预测函数 $f$ 保持不变，最早的 $f$ 其实就是一个MLP

???+ note "双向RNN(Bidirectional RNN)"

    RNN的一个变种，思路为收集前后双向的信息。

    <figure markdown="span">
        ![](img/27.jpg){width="400"}
    </figure>

### 预测函数设计

#### Naive RNN

<figure markdown="span">
    ![](img/28.jpg){width="500"}
</figure>

**主要问题：**

1. $h$ 的变化非常剧烈，记忆跳跃，表现得很随机
2. 只能串行处理，计算速度慢
3. 梯度消失或爆炸（因为反向传播时，深度大且每一层的参数都相同，损失函数类似幂函数，1附近有跳变）

#### LSTM(Long Short-Term Memory)

为解决RNN记忆变化剧烈的问题，把输入分成长期记忆 $c$ 和短期记忆 $h$ 两部分：

<figure markdown="span">
    ![](img/29.jpg){width="500"}
</figure>

计算方法：

- 当前输入和短期记忆拼接，作为输入信息；
- 对输入信息做归一化，取值 $\in (-1, 1)$
- 将输入信息转换成三个阀门，取值 $\in (0, 1)$，分别为：
    - 输入阀门：决定多少当前输入信息进入长期记忆
    - 遗忘阀门：决定保留多少长期记忆
    - 输出阀门：决定多少长期记忆进入短期记忆

<figure markdown="span">
    ![](img/30.jpg){width="250"}
</figure>
<figure markdown="span">
    ![](img/31.jpg){width="500"}
</figure>


??? note "LSTM: A Search Space Odyssey"
    对LSTM做消融实验：
    <figure markdown="span">
        ![](img/32.jpg){width="500"}
    </figure>

    结论：

    1. 标准的LSTM工作得很好
    2. 遗忘门非常重要
    3. 输出门的激活函数非常重要
    4. 如果要简化LSTM，可以把输入门和遗忘门合并成一个门，且不使用peephole

但LSTM没有解决RNN只能串行计算的问题。

#### GRU(Gated Recurrent Unit)

将LSTM的input gate和forget gate合并成一个reset gate（记住和遗忘的占比相加为1）

GRU性能和LSTM相差不大，但计算速度更快。

<figure markdown="span">
    ![](img/33.jpg){width="500"}
</figure>


## 5.2 用于NLP的循环神经网络 RNN for NLP

### RNN用于NLP的优势

1. 可以捕捉长距离的依赖关系（long-distance dependencies）（而CNN是基于窗口（window-based）的）

    <figure markdown="span">
        ![](img/34.jpg){width="500"}
    </figure>

    ???+ example
        **He** does not have very much confidence in **himself**.

        The **reign** has lasted as long as the life of the **queen**.

        指代消解：

        - The **trophy** would not fit in the brown suitcase because it was too **big**.
        - The trophy would not fit in the brown **suitcase** because it was too **small**.
        
2. 既可以做编码器，又可以做解码器（用来生成，但传统CNN做不了）

    <figure markdown="span">
        ![](img/35.jpg){width="500"}
    </figure>


### Encoder-Decoder Models

又称为**Seq2Seq模型**，主要用于机器翻译任务

<figure markdown="span">
    ![](img/36.jpg){width="500"}
</figure>

- encoder提取语义并编码，decoder解码成目标语言

## 5.3 注意力机制 Attention Mechanism

<figure markdown="span">
    ![](img/37.jpg){width="500"}
</figure>

- 选择性地关注感受到的信息

???+ example
    用眼动仪定量分析，热力图显示了人们在阅读时的注意力分布（注视时间）。
    <figure markdown="span">
        ![](img/38.jpg){width="500"}
    </figure>