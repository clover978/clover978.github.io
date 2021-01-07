---
title: Fine-grained Image Classification by Visual-Semantic Embedding
date: 2021-01-06 13:53:57
tags: 
  - deep learning
  - FGVC
categories:
  - Paper Notes
  - FGVC
---

## [Fine-grained Image Classification by Visual-Semantic Embedding](https://www.ijcai.org/proceedings/2018/0145.pdf)
>- **IJCAI 2018**
>- 这篇文章的创新点是利用到了细粒度分类中子类别的语义信息。文中提到了两种语义信息，一种是 **text context**，以 CUB 数据集为例，wiki 上对某一子类鸟的描述就是 text context；另一种是 **knowledge based context**，收集子类鸟的各种属性，建立知识库，有点属性学习的意思。
>- 这篇文章的想法很有创新，并且也有很好的效果，在 CUB 数据集上达到 86.2 的准确率。但是没有验证在其他数据集上是否同时有效。

 <!-- more -->

  - Two leval CNN:  
  作者设计了一个双层的网络结构，F(a) 是定位网络，F(b) 是回归排序网络，网络结构图参考论文。
    + Localization Network：  
    这一部分是传统的目标检测网络模型。提取 定位网络的特征，与回归网络提取的特征点乘，起到 Attention 的作用。
    + Regression Ranking Network：  
    这一部分通过 CNN 提取网络的特征，然后加入定位网络提取的特征作为 Attention，得到视觉特征，接下来通过 FC 层，将视觉特征映射到语义空间，网络的约束条件就是特征在语义空间中的距离，优化网络，减小学习到的语义特征与 gt 在语义空间中的特征距离。作者使用了两种语义空间，因此网络的视觉特征同时平行通过了两个 FC 层。
      - `Knowledge Base Embedding`：  
      这里参考了 [Learning Entity and Relation Embeddings for Knowledge Graph Completion](https://www.aaai.org/ocs/index.php/AAAI/AAAI15/paper/view/9571/9523) 中提出的 `TransR` 方法。并在此基础上针对细粒度分类问题提出了 `Attribute Base Embedding`。这一部分属于 知识图谱 的领域，看得不是很明白。  
      首先，传统的 知识库嵌入 中，知识库由 三元组(h, r, t) 组成，其中 h，t 表示两个实体，h 是起点，t 是终点；r 表示是实体之间的关系。实体(h,t) 由 d 维数组表示，关系(r) 由 r 维数组表示，映射矩阵 M(r) 是一个 d*r 的矩阵，将实体空间映射到关系空间。知识图谱的约束条件定义为：f(h, t) = ||h(r) + r - t(r) ||。  
      基于 知识库嵌入 改进的 属性知识库嵌入，三元组(h, r, t)中 实体h 是样本标签y， 关系r 是 has_property_of， 实体t 是样本的属性。通过优化映射矩阵，将样本标签映射到 属性知识库空间。
      - `Text Embedding`：  
      文本嵌入部分通过 word2vec 实现。作者首先 finetune 了一个 word2vec 模型，然后利用模型将 类别名称 映射到 文本语义空间。  
    + 实验结果：  
    作者在 CUB 数据集上做的实验，按照论文所述，训练的过程中既没有使用 bbox 信息，也没有使用 part annotation 信息，这一点不是很明白，和我理解的训练过程不太一样。  
    **CUB** 准确率： **0.862**