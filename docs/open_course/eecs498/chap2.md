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
        

    === "CIFAR-10"

    === "CIFAR-100"

    === "ImageNet"

    === "MIT Places"

    === "Omniglot"


## 2.3 最近邻分类器 Nearest Neighbor Classifier

## 2.4 K-近邻分类器 K-Nearest Neighbor Classifier

### K值的影响

### 距离度量的影响

