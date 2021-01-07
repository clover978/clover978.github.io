---
title: >-
  Weakly Supervised Local Attention Network for Fine-Grained Visual
  Classification
date: 2021-01-06 13:53:32
tags:
  - deep learning
  - FGVC
categories:
  - Paper notes
  - FGVC
---

## [Weakly Supervised Local Attention Network for Fine-Grained Visual Classification](https://arxiv.org/pdf/1808.02152.pdf)
>- **arxiv (MSRA)**
>- 这篇文章提出一种 LAP(Local Attention Pooling) 的机制，利用 attention map 去提取更具区分性的特征图；文中还提出一种弱监督的方式去训练网络的方法，在训练过程中加入了 attention dropout 和 attention center loss。
>- 文中提出的网络 WS-LAN(Weakly Supervised Local Attention Network)在 CUB 数据集上准确率达到了 87.9

 <!-- more -->

  - Local Attention Pooling:  
  这一部分通过文中的 `插图1` 很容易就能看懂。  
  作者首先通过 CNN 分别提取到图片的 特征图 和 注意力图，其中每一个注意力图用于聚焦目标的一个 part 上。然后将特征图与 k 个注意力图分别点乘，得到 k 个 part 的特征图。对 k 个特征图进行卷积核池化操作，得到每一个 part 的特征，将 k 个 part 特征合并到一起形成最终的特征。  
  这一部分实际上与 SE-Net 很相似。SE-Net 将特征经过卷积之后，再经过一个 SE 结构，等同于在卷积过程中对卷积核的不同 channel 赋予不同的权重，上一层的特征会分别与卷积核的不同 channel 进行卷积操作。假设将上一层特征看做 LAN 中的特征图，channel间 attention 最大的 topk 个卷积核看做 LAN 中的注意力图，那么两个网络的区别就是一个是进行 卷积操作，一个是进行 点乘操作。
  所以这一部分实际上可以看做是一个稍加改动，更为复杂的 SE-Net。但是这种改动是必要的，因为下面的 WS-LAN 需要在这个网络结构上进行训练。
  - WS-LAN：  
    这一部分，作者将两个传统网络训练中的 trick 迁移到了上面的 LAN 中，这应该是文章最大的两个提升点。  
    这部分的两个 trick 实际上想解决的问题只有一个，就是 提取到的 ***attention map 很容易只聚焦到目标的一两个最具区分度的区域***。
    + attention dropout：  
    作者将 dropout 加入到 LAP 操作中。attention map 和 feature map 进行点乘的时候，attention map 以 概率(1-p) 被 drop 掉，以概率 p 被保留，并乘以 1/p，这个操作与传统的 dropout 如出一辙。
    + attention center loss：  
    人脸识别文章中提出过一个 center loss，作者同样将其迁移到 WS-LAN 中。  
    attention map 与 feature map 点乘之后得到 k 个 part feature，作者为这 k 个 feature 维护 k 个 center，然后每次训练的时候，计算每个 feature 和 center 的距离，最小化同一个 part 的 feature distance，最大化不同 part 的 feature distance，作为 attention center loss 的约束条件。
  - 实验结果：  
    |        | acc  |
    |-       |-     |
    | LAN    | 85.5 |
    | WS-LAN | 87.9 |
