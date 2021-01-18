---
title: >-
  Fine-Grained Image Classification Using Modified DCNNs Trained by Cascaded
  Softmax and Generalized Large-Margin Losses
date: 2021-01-06 13:53:45
tags: 
  - deep learning
  - FGVC
categories:
  - Paper Notes
  - FGVC
---

# [Fine-Grained Image Classification Using Modified DCNNs Trained by Cascaded Softmax and Generalized Large-Margin Losses](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8419081)

> 1. **TNNLS 2018**
> 2. 这篇文章提出了两个创新点，一个是 级联softmax结构，另一个是 泛化 large-margin loss。主要的思想就是细粒度分类中的标签是具有层次结构的，large-margin loss 是度量学习中提出的概念，这里在细粒度分类问题中进行了改进。
> 3. 文章摆了大量的公式，所以我是没怎么仔细看的，并且它的结果并不很出色。使用 VGG 作为 backbone，在 CUB 上准确率是 77.0，结合 bilinear-CNN，从 84.1 提升到了 85.4
 <!-- more -->


## 1. Method
  - Cascaded Softmax Loss：  
  级联softmax，见文中的 `插图3`，其实是一个很简单的结构，假设识别任务中有 50 个粗分类，每个分类中有 4 个细分类，总共 200 分类。传统的网络，直接通过 fc8(200)+softmax 进行训练，这里改成了 fc8(200)+ softmax + fc9(50)+softmax 训练，并且在 fc7 和 fc(9) 之间添加了一个 skip connection. 网络的 loss 则变成了每一层 softmax 的 loss 相加。
  - Generalized Large-Margin Loss：  
  泛化 large-margin loss，这个用起来其实也很简单，就是在 fc7 添加一个 loss层 进行监督，large-margin loss 的作用就是增加类间距离，减小类内距离，在标签具有分层结构的情况下，对每个 level 的标签都进行这样的约束，这一部分文中使用了大量的公式，没有仔细看。


## 2. 实验结果：  
  作者的这个改动可以应用到任何 CNN 结构中，所以作者做了大量的实验：
  
  |                           | w/o bbox  | with bbox |
  |-                          |-          |-          |
  | googlenet+SM              | 73.6      | 77.4      |
  | googlenet+CSM             | 74.6      | 78.4      |
  | googlenet+SC+CSM          | 75.3      | 79.0      |
  | googlenet+SM+GLM          | 76.8      | 80.5      |
  | googlenet+CSM+GLM         | 77.1      | 81.3      |
  | googlenet+SC+CSM+GLM      | 77.6      | 82.0      |
  | VGG+SM                    | 72.5      | 78.6      |
  | VGG+SC+CSM+GLM            | 77.0      | 82.4      |
  | B-CNN                     | 84.1      | 84.8      |
  | B-CNN+SC+CSM+GLM          | 85.4      | 85.7      |
  | googlenet+SM+contrastive  | 74.1      | 77.8      |
  | googlenet+SM+triplet      | 74.1      | 78.0      |
  | googlenet+SM+center loss  | 74.5      | 78.4      |
  | googlenet+SM+min-max      | 75.1      | 78.9      |

    `SM: softmax`  
    `CSM: cascaded softmax`  
    `SC: skip connection`  
    `GLM: generlized large-margin loss`  
    
    通过对比可以看出：  
      + GLM 对结果的影响是最大的， 76.8
      + 单独 CSM 的效果并不明显，加上 SC 后还能提高一下。 74.6 -> 75.3
      + 在 VGG， googlenet 上有明显的提高，但是在 B-CNN 上的提高就比较小了。
      + GLM 相对于其他的 度量学习 方法效果也是最好的。