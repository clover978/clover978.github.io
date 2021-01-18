---
title: Associative Embedding  End-to-End Learning for Joint Detection and Grouping
date: 2021-01-11 17:05:40
tags:
  - deep learning
  - Pose Estimation
  - bottomup
categories:
  - Paper Notes
  - Pose Estimation
---

# [Associative Embedding  End-to-End Learning for Joint Detection and Grouping](https://proceedings.neurips.cc/paper/2017/file/8edd72158ccd2a879f79cb2538568fdc-Paper.pdf)

> 1. 这篇文章提出利用 Associative Embedding 实现 关键点分组 的方法，作者认为计算机视觉中的很多任务可以看做 Detection + Grouping 的操作，首先检测出关键点，然后给关键点打上 tag ，根据 tag 对关键点进行分组。作者用这种思路实现了一种多人姿态估计的算法。
> 2. 文章提出的方法在 COCO-test-dev 数据集上取得 65.5% 的 mAP，在 MPII 数据集上取得 77.5 的 mAP

<!-- more -->

## 1. Method 

- Network Architecture  
作者使用 StackHourglass 的网络结构，在最终预测时，同时预测 detection heatmap 和 associative embedding，associative embedding 是一个长度为 1 的向量。  
预测得到关键点的最强响应之后，将 associative embedding 相近的关键点连接起来，就得到了最终的多人姿态估计结果。  
作者对 StackHourglass 进行了一点小改动，提高了 resolution 部分的 特征图通道数，为了提高网络的参数量，使其能够学习到 associative embedding 信息。

- Detection and Grouping  
Detection 部分，本文采用和 Hourglass 相同的方法，使用 Heatmap 进行监督，利用 MSE Loss 学习关节点的位置。  
Grouping 部分，作者定义 grouping loss，定义图像中 N 个 Person， K 个 Keypoint，缩小相同 Person 关键点的 embedding 距离，增大不同 Person 关键点的 embedding 距离。具体定义见论文公式。  

- Multiscale Evaluation  
作者使用 Multiscale Evaluation 的方式进行测试。假设使用 4 种 scale 的图像作为输入，得到 4 种 heatmap 和 embedding，那么 新的 heatmap
由 4 个 heatmap 进行平均得到，embedding vector 由 4 个 embedding concat 而成，成为长度为 4 的 vector。  
  `COCO-test-dev` 实验结果
| | AP | AP50 | AP75 | APM | APL |
|- |- |- |- |- |- |
| single scale          | 0.566 | 0.818 | 0.618 | 0.498 | 0.670 |
| single scale + refine | 0.628 | 0.846 | 0.692 | 0.575 | 0.706 |
| multi scale           | 0.650 | 0.867 | 0.713 | 0.597 | 0.725 |
| multi scale + refine  | 0.655 | 0.868 | 0.723 | 0.606 | 0.726 |


# 2. Result

- `COCO-test-dev`  

| method |  AP |
|- |- |
| OpenPose | 0.618 |
| Mask RCNN | 0.627 |
| G-RMI | 0.649 |
| **AE** | **0.655** |

- `COCO-test-std`  

| method |  AP |
|- |- |
| OpenPose | 0.611 |
| G-RMI | 0.643 |
| **AE** | **0.663** |

- `MPII`  

| method |  AP |
|- |- |
| OpenPose | 0.756 |
| AlphaPose | 0.767 |
| **AE** | **0.775** |