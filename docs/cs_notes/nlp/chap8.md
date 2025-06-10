# Chapter 8. 预训练模型：BERT及其变体 Pre-trained Model: BERT and its Variants

**预训练**指的是先在大规模通用数据集（往往成本低、无标注）上训练模型，学习通用的知识或特征，为后续特定任务的微调打下基础。

## 8.1 预训练模型 Pre-trained Model

词向量是非常适合预训练模型的一个例子。

传统的一些生成词向量的方式有一个很大的问题：**无法处理多义词**（不带上下文语义，如“宠物狗”的“狗”和“单身狗”的“狗”会被编码为同一向量）。

于是使用 **语境词向量（Contextualized Word Embedding）**：

<figure markdown="span">
    ![](img/70.jpg){width="400"}
</figure>

- 其中的模型可以是 LSTM、Self-attention layers 等


训练出来效果不错，下图是对“苹果”这个词的语境词向量的相似度可视化（黄色相似度高，蓝色相似度低）：

<figure markdown="span">
    ![](img/71.jpg){width="500"}
</figure>

???+ note "缩小模型的方法"
    因为数据成本低，预训练模型参数量越做越大，也就相应地产生了许多缩小模型的方法：

    - 网络剪枝（Network Pruning）：去掉一些不重要的参数
    - 知识蒸馏（Knowledge Distillation）：用大模型为小模型的训练数据标注
    - 参数量化（Parameter Quantization）：降低浮点数精度，用更少的比特数表示参数
    - 架构设计（Architecture Design）：设计更小的模型架构

## 8.2 微调 Fine-tuning

