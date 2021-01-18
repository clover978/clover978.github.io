---
title: Multi-Attention Multi-Class Constraint for Fine-grained Image Recognition
date: 2021-01-06 13:54:07
tags: 
  - deep learning
  - FGVC
categories:
  - Paper Notes
  - FGVC
---

# [Multi-Attention Multi-Class Constraint for Fine-grained Image Recognition](https://arxiv.org/pdf/1806.05372.pdf)

> 1. **arxiv(Baidu)**
> 2. 这篇文章的思路是 attention + 度量学习。其中，attention 部分跟 SENet 如出一辙；度量学习比较像 contrastive loss。文章中定义了四种类型的特征，sasc, sadc, dasc, dadc, （s: same, a: attention, d: different, c: class），进行度量学习的时候，定义只有 sasc 是正样本，约束相同类别在相同attention区域学习到相似的特征。
> 3.s 在 CUB 数据集的准确率达到了 86.5。从 这篇文章 和 MSRA的WS-LAN 来看，attention + 度量学习 的方法是细粒度分类里面比较好的一个研究方向。
 <!-- more -->


## 1. Method
  - OSME (One-Squeeze Multi-Excitation Attention Module)：  
  这一部分，按照我的理解，可以说是完全照搬的 SENet，没有任何创新点。作者说明了 SENet 中 SE module 的原理。文中提出的网络结构，将提取到的特征，分别经过两个 SE module，得到的两个 re-weighted feature map，称为 attention1， attention2.  
  我理解的 OSME module 与 SE module 的区别就是一个是单路的，一个是双路的。
  - Multi-Attention Multi-Class Constraint：  
  这一部分，作者使用了度量学习的方法，每次训练的时候，网络会输入 2N 对图片，其中每一对图片都来自于同一类别。然后一对图片分别经过 OSME 的上半支和下半支，这样可以得到四种类别的特征。sasc, sadc, dasc, dadc，在 softmax 的基础上，作者对 hinge loss 进行改进，提出 MAMC Constraint，四种特征。挑选出一个特征作为 anchor，然后分三种情况：
    + 正样本是 sasc， 负样本是 {sadc, dasc, dadc};
    + 正样本是 sadc， 负样本是 {dadc};
    + 正样本是 dasc， 负样本是 {dadc};  
  在每种情况下，定义出 MAMC loss，最小化与正样本的特征距离，最大化与负样本的特征距离。


## 2. 实验结果：

  |                           | acc   |
  |-                          |-      |
  | VGG19                     | 79.0  |
  | ResNet50                  | 81.7  |
  | ResNet101                 | 82.5  |
  | ResNet-50 + OSME          | 84.9  |
  | ResNet-50 + OSME + MAMC_1 | 85.4  |
  | ResNet-50 + OSME + MAMC   | 86.2  |
  | ResNet-50 + OSME_3 + MAMC | 86.3  |
  | ResNet-101 + OSME + MAMC  | 86.5  |
  | RACNN                     | 85.3  |
  | MACNN                     | 86.5  |

    OSME_3: 使用 3 个 attention  
    MAMC_1：定义 MAMC loss 的时候，只区分第一种情况。  
    
    使用 ResNet50 作为 backbone， OSME 提升了 3.2%， MAMC 提升了 1.3% 的准确率。值得一提的是，相比于 MACNN，文中提出的网络结构要简单的多，在效率上具有很大的优势。