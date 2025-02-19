# Chapter 2 图像分类 Image Classification

## 2.1 图像分类任务 Image Classification Task

图像分类任务是计算机视觉领域的基础任务，可作为构建很多其它任务（如对象检测、图像字幕、围棋决策等）的模块。

### 任务描述 Description

- 输入：一张图片（image）
- 输出：图片中物体的类别（category）

### 任务难点 Challenges

图像分类任务最大的问题在于**语义鸿沟（semantic gap）**。在计算机“眼”中，图像只是由许多像素值组成的网格，而不是人类眼中有直观意义的物体。

具体的难点包括：

1. **视角变化（Viewpoint Variation）**：同一物体在不同角度下的图片可能完全不同。
2. **类内差异（Intraclass Variation）**：同一类别物体之间可能存在明显差异。（如两个属于“猫”的样本可能长得非常不同）
3. **细粒度类别（Fine-grained Categories）**：同一大类下的细分类别之间可能非常相似。（如缅因猫、布偶猫、美国短毛猫等）
4. **背景杂乱（Background Clutter）**：图片背景可能干扰物体识别。
5. **光照变化（Illumination Changes）**：光照变化可能导致图片像素值变化。
6. **形变（Deformation）**：物体可能发生形变。（如猫可以站、坐、躺等）
7. **遮挡（Occlusion）**：物体可能被遮挡。

## 2.2 图像分类数据集 Image Classification Datasets

!!! note
    机器学习是一种**数据驱动方法（Data-Driven Approach）**。分类器的构建步骤主要包括：

    1. 收集图像和标签数据集；
    2. 使用机器学习算法训练分类器；
    3. 在新的图像数据上评估评估分类器。

    因而一般需要实现**训练（train）**和**预测（predict）**两个函数：

    ```python
    def train(images, labels):
        # Machine learning!
        return model
    
    def predict(model, test_images):
        # Use model to predict labels
        return test_labels
    ```

    可见数据集的质量和规模对模型性能有很大影响。

以下是一些常见的图像分类数据集：

!!! quote ""
    === "MNIST"
        ![MNIST](img/16.jpg){ align=left width=50% }

        - 手写数字数据集
        - 10个类别：数字 0 - 9
        - 60000张训练图片，10000张测试图片
        - 28 $\times$ 28的灰度图像
        ---
        - MNIST数据集小且简单，仅适合验证算法的正确性，不适合评估算法的性能。

    === "CIFAR-10"
        ![CIFAR-10](img/17.jpg){ align=left width=50% }

        - 10个类别：飞机、汽车、鸟、猫、鹿、狗、青蛙、马、船、卡车
        - 50000张训练图片，10000张测试图片
        - 32 $\times$ 32的RGB图像

    === "CIFAR-100"
        ![CIFAR-100](img/18.jpg){ align=left width=50% }

        - 100个类别：20个大类别，每个大类别包含5个小类别（如树木：枫树、橡树、棕榈树、松树、柳树）
        - 50000张训练图片，10000张测试图片
        - 32 $\times$ 32的RGB图像

    === "ImageNet"
        ![ImageNet](img/19.jpg){ align=left width=50% }

        - 1000个类别
        - 约1300000张训练图片，50000张验证图片，100000张测试图片
        - 图像尺寸不固定（训练时一般调整为 $256\times 256$ 或 $224\times 224$）
        - 2012年ILSVRC竞赛的数据集
        ---
        - 因为想要精确预测图片类别非常困难，一般采用**前5名准确率（Top 5 accuracy）**作为性能评价指标，即模型预测5个类别，其中包含正确类别的概率。
        - ImageNet是图像分类研究中最重要的数据集之一。

    === "MIT Places(Places365)"
        ![MIT Places](img/20.jpg){ align=left width=50% }

        - 场景数据集
        - 365个类别
        - 约8000000张训练图片，18250张验证图片（每个类别50张），328500张测试图片（每个类别900张）
        - 图像尺寸不固定（训练时一般调整为 $256\times 256$ 或 $224\times 224$）

    === "Omniglot"
        ![Omniglot](img/21.jpg){ align=left width=50% }

        - 1623个类别：50种语言
        - 每个类别仅包含20张图片
        --- 
        - 用于小样本学习（Few-shot Learning）研究


## 2.3 最近邻分类器 Nearest Neighbor Classifier

### 实现方法

#### 训练

只需存储所有训练图像和标签即可。

#### 预测

给出与预测图像最相似的训练图像的标签。

## 2.4 K-近邻分类器 K-Nearest Neighbor Classifier

### K值的影响

### 距离度量的影响

