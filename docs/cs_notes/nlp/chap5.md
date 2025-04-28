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

主要问题：

1. $h$ 的变化非常剧烈，记忆跳跃，表现得很随机

#### LSTM(Long Short-Term Memory)

解决RNN记忆变化剧烈的问题

<figure markdown="span">
    ![](img/29.jpg){width="500"}
</figure>

- 把输入分成长期记忆 $c$ 和短期记忆 $h$ 两部分