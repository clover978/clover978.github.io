---
title: Embedding label structures for fine-grained feature representation
date: 2021-01-06 13:53:11
tags: 
  - deep learning
  - FGVC
categories:
  - Paper Notes
  - FGVC
---

## [Embedding label structures for fine-grained feature representation](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Zhang_Embedding_Label_Structures_CVPR_2016_paper.pdf)
>- **CVPR 2016**
>- 这篇文章提出两个创新点，一个是使用 "structured label" 改进 `triple loss`；另一个是同时使用 triple loss 和 softmax loss 进行训练。

 <!-- more -->

  - 同时使用 softmax loss 和 triple loss 约束训练：  
  这个没什么好说的，一个很简单的想法。使用三路的网络，提取特征，然后分为两支，一支过 fc+softmax 然后分类，一支对比 anchor, positive, negative 的特征距离，然后用 triple loss 进行约束。网络结果参照论文链接。
  - 结构化目标嵌入（embed label structures）:  
  这一部分是对 triple loss 的一个改进，在细粒度分类问题中，目标的分类可以看做层级的，以车辆分类为例，目标结构由粗到细可以是 品牌-模型-年份，因此做模型分类的时候，可以使用四元组进行训练：`r(reference), p+(same model), p-(same make, diffrent model), n(diffrent make)`，四元组的 loss 等价于 `L(r, p+, p-) + L(r, p-, n)`。
  - 实验结果：  
  作者在 `standFord Cars` 上面进行实验，实验结果略（笔记只记录 CUB 数据集的结果）
