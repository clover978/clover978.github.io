---
title: Scale-Aware Representation Learning for Bottom-Up Human Pose Estimation
date: 2021-01-12 08:41:07
tags:
  - deep learning
  - Pose Estimation
  - bottomup
categories:
  - Paper Notes
  - Pose Estimation
---

# [Scale-Aware Representation Learning for Bottom-Up Human Pose Estimation](https://arxiv.org/pdf/1908.10357.pdf)

> 1. 本文以多人姿态估计中的尺度差异为研究点，提出 HigherHRNet 结构，生成不同尺度的特征图，同时在多尺度的特征图上进行监督学习，实现自底向上的多人姿态估计。
> 2. 文章提出的方法在 COCO-test-dev 数据集上取得 70.5% 的 mAP
<!-- more -->


## 1. Method
- HigherHRNet 网络结构  
  HRNet 生成原图 1/4 大小的特征图，然后经过一层卷积生成 K 通道的 heatmap。以此为基础，HigherHRNet 网络结构进行下面一系列改动：
  - ***multi-resolution supervision (MRS)***  
  1. 将 `1/4特征图` 经过 `deconv` 模块，生成 `1/2特征图`；  
  2. `1/2特征图` 进过一层卷积得到 K 通道 `1/2heatmap`；  
  3. 同时在 `1/4heatmap` 和 `1/2heatmap` 两个特征图上进行监督学习；  
  4. 预测时使用 `1/2heatmap` 进行预测。  
  - ***feature concat***  
  步骤1 中的 `1/4特征图` 先与 `1/4heatmap` concat，再进行 deconv 操作。  
  - ***heatmap aggregation***  
  步骤4 时，同时使用 `1/4heatmap` 和 `1/2heatmap` 进行预测。  
  - ***extra res. blocks***  
  步骤2 中，`1/2特征图` 经过 4 个 residual block 之后生成 `1/2heatmap`

  COCO-val ablation 实验结果  
| Network | w/ MRS | feature concat. | w/ heatmap aggregation | extra res. blocks | AP | APM | APL |
| - | - | - | - | - | - | - | - |
| HRNet       |   |   |   |   | 64.4 | 57.1 | 75.6 |
| HigherHRNet | y |   |   |   | 66.0 | 60.7 | 74.2 |
| HigherHRNet | y | y |   |   | 66.3 | 60.8 | 74.0 |
| HigherHRNet | y | y | y |   | 66.9 | 61.0 | 75.7 |
| HigherHRNet | y | y | y | y | 67.1 | 61.5 | 76.1 |

- Grouping  
  文章使用了 associative embedding 实现 grouping，将 tags 相近的关键点连接成单人姿态。


## 2. Result
  `COCO test-dev` 实验结果
  
| Method | AP | AP50 | AP75 | APM | APL | AR |
|- |- |- |- |- |- |- |
| OpenPose∗ [3]         | 61.8 | 84.9 | 67.5 | 57.1 | 68.2 |66.5 |
| Hourglass∗+ [30]      | 65.5 | 86.8 | 72.3 | 60.6 | 72.6 |70.2 |
| PifPaf [22]           | 66.7 | -    |-     | 62.4 | 72.9 | -   |
| SPM [32]              | 66.9 | 88.5 | 72.9 | 62.6 | 73.1 | -   |
| PersonLab+ [33]       | 68.7 | 89.0 | 75.4 | 64.1 | 75.5 |75.4 |
| **HigherHRNet-W48+**  | 70.5 | 89.3 | 77.2 | 66.6 | 75.8 |74.9 |



