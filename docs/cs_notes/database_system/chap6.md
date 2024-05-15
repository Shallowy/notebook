# Chapter 6 ER模型 Entity-Relationship Model
## 6.0 数据库设计流程 Database Design Process
1. 需求分析 requirement analysis
2. 概念设计 Conceptual design
3. 逻辑设计 logical design
4. 物理设计 physical design

---

本章介绍的ER模型即为2. 概念设计的工具之一。

## 6.1 ER图概述 Overview of ER Diagram
<figure markdown="span">
    ![](img/29.png){width="600"}
</figure>

一些要点：

- 确定各个实体集。
- **花括号**表示多值属性。（如图中 $time\_slot$ 中属性）
- 建立关系，**箭头**表示多（包括零）对（指向）一，**单横线（无箭头）**表示多对多，**双横线**表示全参与（集合中每个元素都参与此关系），**双框**表示弱实体集（不能独立存在，有依赖性，如图中 $section$ 依赖于 $course$）。
- 关系也可以带有属性。（如图中 $grade$ 属性）

## ER模型概述 Outline of the ER Model
