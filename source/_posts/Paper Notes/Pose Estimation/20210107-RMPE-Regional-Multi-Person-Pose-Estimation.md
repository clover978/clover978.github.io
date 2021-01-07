---
title: RMPE  Regional Multi-Person Pose Estimation
date: 2021-01-07 11:34:28
tags:
  - deep learning
  - Pose Estimation
categories:
  - Paper Notes
  - Pose Estimation
---

https://arxiv.org/pdf/1612.00137.pdf

# Introduction
1. top-down 的 muti-person PE 方法有两个缺点：a 有的框(IOU>0.5)实际上框的不准，b 有一些冗余框
2. 提出 Regional Muti-person estimation 框架。
3. 提出 symmetric spatial transformer network (SSTN), 对 SPPE 方法进行改进，在不准确的 bbox 上仍然能够准确地预测姿态。
4. 提出 parametric pose NMS (PP-NMS), 解决冗余框的问题
5. 提出 pose-guided human proposal generator (PGPG), 对训练数据进行数据增强

<!-- more -->


# Related Work
  - Single Person PE：
     - tree models
     - random forrest models
     - CNN based models
  - Muti person PE:
    - part based framework
      - 一大堆没听过的算法
      - 缺点是只利用了局部区域的信息，part 的检测效果可能不好
    - two-step framework
      - 又是一大堆没听过的算法
      - 本文的框架基于 two-step framework，解决人体 bbox 检测不准的问题。

# Regional Multi-person Pose Estimation
fig3
  - STN & SDTN （SSTN）：  
    作者认为，目标检测得到的框很有可能是不准确的，这是制约 top-down 方法效果的一个关键点。因此，作者将 detection 的结果输入 SPPE 模块之前，会先经过一个 STN 模块，得到 PE 结果后，再经过一个对称的 SDTN 模块。STN 的目标是从 detection 结果中 warp 出更精确的框，STDN 的目标则相反。STN 和 STDN 都包含 3 个参数，通过推到可以得到两者的关系（参考论文）。
  - Parallel SPPE：  
    为了使 STN 更准确地定位到 detection proposal 的 domainate person，作者还加入了一个 Parallel SPPE，这个模块是一个固定参数的 SPPE 模块，输入由 STN 生成的 grid 之后，使用预训练好的 SPPE 对 grid 中的 person 进行预测，然后与 groundtruth 进行比较，这里的 groundtruth 是对原始 groundtruth 进行 center-located 操作之后得到的。  
    > 我的理解是，假设图片中有 A,B 两个人，目标检测得到一个不太准的框 bbox(A)，Parallel SPPE 的 groundtruth 就是以 A 的 pose groundtruth 为中心的 图片（SPPE 的 gt 就是直接将 gt 坐标转化到 bbox(A) 里面图片），如果 STN 生成的 grid 离 A 很远，Parallel SPPE 的误差就会非常大。
  - Parametric Pose NMS：  
    为了消除冗余，作者提出了对 Pose 进行 NMS 的方法，NMS 的 metric 是 两个 Pose 的距离，作者提出了两种度量 Pose distance 的方法，NMS 的时候会取两种距离的加权和。
      - H_sim  
        公式参照论文，这个距离很好理解，就是两个 pose 各个关键点的 L2 距离
      - K_sim  
        公式参照论文，这个距离的含义是，如果两个 Pose 的某个关键点重合（重合的概念被定义为 一个关键点处于另一个关键点为中心的矩形区域中），就会计算两个关键点置信度的乘积，因此，重叠的关键点越多，并且置信度越高的情况下，计算的结果越大。
    > 这个地方有个很奇怪的点。按照公式，越相似的 Pose 计算出来的 distance 会越大，但 NMS 公式里面却是距离小才会被抑制，不知道哪里看错了。
  - Poseguided Proposals Generator：  
  为了让 STN+SPPE 模块适应多样的 bbox proposal 输入，作者提出了一种数据增广的方案。作者会统计 detection 的 bbox 和 gt 的偏移量的分布。训练的时候，根据偏移量的分布，随机对 detection 的结果进行 shift。  
  由于不同的 Pose 对应的偏移量分布可能各不相同，作者定义了 4 个 原子姿态，首先对图片的 Pose 用 kmeans 进行聚类，每种原子姿态会对应不同的偏移量分布。



