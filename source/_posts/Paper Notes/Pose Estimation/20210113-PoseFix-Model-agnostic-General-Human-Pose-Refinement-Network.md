---
title: 'PoseFix: Model-agnostic General Human Pose Refinement Network'
date: 2021-01-13 13:54:18
tags:
  - deep learning
  - Pose Estimation
  - poseplugin
categories:
  - Paper Notes
  - Pose Estimation
---

# [PoseFix: Model-agnostic General Human Pose Refinement Network](https://arxiv.org/abs/1812.03595.pdf)

> 1. 现有的 Pose Estimation 方法的 error 具有相似的分布，收集这些 error statistics 作为先验知识，结合标注数据，可以合成 Pose 数据。  
  利用合成 Pose 数据训练网络，就可以让网络学习 error statistics，进而 refine 其他网络的 Pose 结果。
> 2. PoseFix 可对任意模型进行 refine，实验结果表明对多种模型都有提升效果。
<!-- more -->


## 1. Method
  - Synthesizing poses for training
  Pose 关键点一共有 5 中类型：
    - Good： 预测结果完全正确  
    - Jitter： 预测结果与 GT 有小距离偏差  
    - Inversion： 预测结果将对称关键点预测错误  
    - Swap： 预测结果将关键点标注到旁边的人体目标上  
    - Miss： 预测结果与 GT 有大距离偏差  
  根据之前的研究，Pose Estimation 中这 5 种类型关键点服从一定的分布，依据不同类型关键点出现的频率，随机合成 Pose，然后输入 PoseFix 训练。

  - Architecture and learning of PoseFix
  PoseFix 将 原始图像 和 合成Pose 同时输入网络，网络输出 refined pose。PoseFix 采用和 simplebaseline 相同的骨干网络。  
  PoseFix 利用 Coarse-to-fine 的模式进行预测，网络输入的 合成Pose 为粗粒度结果，网络输出为 精细化结果。refined pose 被两个 Loss 约束。  
    - L(H): 分别在 x方向 和 y方向 上计算 交叉熵  
    - L(C): 利用 softmax 函数，计算出对应的 Pose 坐标 C，然后计算 C 与 GT 的 L1 距离。
  L(H) 约束网络只产生一个关键点，L(C) 约束网络输出的关键点坐标更精确。


## 2. Result
  `COCO test-dev`
  
  | Methods                     | AP    | AP.50 | AP.75 | APM   | APL   | AR    | AR.50 | AR.75 | ARM   | ARL   |
  |-                            |-      |-      |-      |-      |-      |-      |-      |-      |-      |-      |
  | AE [18]                     | 56.6  | 81.7  | 62.1  | 48.1  | 69.4  | 62.5  | 84.9  | 67.2  | 52.2  | 76.5  |
  | + PoseFix (Ours)            | 63.9  | 83.6  | 70.0  | 56.9  | 73.7  | 69.1  | 86.6  | 74.2  | 61.1  | 79.9  |
  | PAFs [4]                    | 61.7  | 84.9  | 67.4  | 57.1  | 68.1  | 66.5  | 87.2  | 71.7  | 60.5  | 74.6  |
  | + PoseFix (Ours)            | 66.7  | 85.7  | 72.9  | 62.9  | 72.3  | 71.3  | 88.0  | 76.7  | 66.3  | 78.1  |
  | Mask R-CNN (ResNet-50) [9]  | 62.9  | 87.1  | 68.9  | 57.6  | 71.3  | 69.7  | 91.3  | 75.1  | 63.9  | 77.6  |
  | + PoseFix (Ours)            | 67.2  | 88.0  | 73.5  | 62.5  | 75.1  | 74.0  | 92.2  | 79.6  | 68.8  | 81.1  |
  | Mask R-CNN (ResNet-101)     | 63.4  | 87.5  | 69.4  | 57.8  | 72.0  | 70.2  | 91.8  | 75.6  | 64.3  | 78.2  |
  | + PoseFix (Ours)            | 67.5  | 88.4  | 73.8  | 62.6  | 75.5  | 74.3  | 92.6  | 79.9  | 69.1  | 81.4  |
  | Mask R-CNN (ResNeXt-101-64) | 64.9  | 88.6  | 71.0  | 59.6  | 73.3  | 71.4  | 92.4  | 76.8  | 65.9  | 78.9  |
  | + PoseFix (Ours)            | 68.7  | 89.3  | 75.2  | 64.1  | 76.4  | 75.2  | 93.1  | 80.9  | 70.3  | 81.9  |
  | Mask R-CNN (ResNeXt-101-32) | 64.9  | 88.4  | 70.9  | 59.5  | 73.2  | 71.3  | 92.2  | 76.7  | 65.8  | 78.9  |
  | + PoseFix (Ours)            | 68.5  | 88.9  | 75.0  | 64.0  | 76.2  | 75.0  | 92.9  | 80.7  | 70.1  | 81.8  |
  | IntegralPose                | 66.3  | 87.6  | 72.9  | 62.7  | 72.7  | 73.2  | 91.8  | 79.1  | 68.3  | 79.8  |
  | + PoseFix (Ours)            | 69.5  | 88.3  | 75.9  | 65.7  | 76.1  | 75.9  | 92.4  | 81.8  | 71.1  | 82.5  |
  | CPN (ResNet-50) [6]         | 68.6  | 89.6  | 76.7  | 65.3  | 74.6  | 75.6  | 93.7  | 82.6  | 70.8  | 82.0  |
  | + PoseFix (Ours)            | 71.8  | 89.8  | 78.9  | 68.3  | 78.1  | 78.2  | 93.9  | 84.3  | 73.5  | 84.6  |
  | CPN (ResNet-101)            | 69.6  | 89.9  | 77.6  | 66.3  | 75.6  | 76.6  | 93.9  | 83.5  | 72.0  | 82.9  |
  | + PoseFix (Ours)            | 72.6  | 90.2  | 79.7  | 69.0  | 78.9  | 78.9  | 94.1  | 85.0  | 74.2  | 85.1  |
  | Simple (ResNet-50) [28]     | 69.4  | 90.1  | 77.4  | 66.2  | 75.5  | 75.1  | 93.9  | 82.4  | 70.8  | 81.0  |
  | + PoseFix (Ours)            | 72.5  | 90.5  | 79.6  | 68.9  | 79.0  | 78.0  | 94.1  | 84.4  | 73.4  | 84.1  |
  | Simple (ResNet-101)         | 70.5  | 90.7  | 78.8  | 67.5  | 76.3  | 76.2  | 94.3  | 83.7  | 72.1  | 81.9  |
  | + PoseFix (Ours)            | 73.3  | 90.8  | 80.7  | 69.8  | 79.8  | 78.7  | 94.4  | 85.3  | 74.3  | 84.8  |
  | Simple (ResNet-152)         | 71.1  | 90.7  | 79.4  | 68.0  | 76.9  | 76.8  | 94.4  | 84.3  | 72.6  | 82.4  |
  | + PoseFix (Ours)            | 73.6  | 90.8  | 81.0  | 70.3  | 79.8  | 79.0  | 94.4  | 85.7  | 74.8  | 84.9  |