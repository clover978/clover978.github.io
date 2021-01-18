---
title: Distribution-Aware Coordinate Representation for Human Pose Estimation
date: 2021-01-13 16:32:34
tags:
  - deep learning
  - Pose Estimation
  - poseplugin
categories:
  - Paper Notes
  - Pose Estimation
---

# [Distribution-Aware Coordinate Representation for Human Pose Estimation](https://arxiv.org/pdf/1910.06278v1.pdf)

> 1. 这篇文章从 **关节坐标表征** 方向研究，发现传统方法中的 coordinate encoding （关键点转化为高斯热力图）过程存在量化误差； coordinate decoding（网络输出热力图转化为关键点坐标）过程对 shift 操作的依赖很高。  
  作者提出 Distribution-Aware coordinate Representation of Keypoints (DARK) 方法，在 关节坐标表征（coordinate Representation）层面提升现有方法的效果。
> 2. DARK 方法可以不改变现有算法的条件下，无缝提升算法效果。
<!-- more -->


## 1. Method  
  - Coordinate Decoding  
  现有方法： 坐标极值点，向第二极值点偏移1/4像素。  
  DARK：  
    - 假设网络输出服从高斯分布，通过求解一阶导数零点计算中心点，根据论文公式9，中心点坐标可以表示为极值点坐标及其附近梯度值的函数。
    - Heatmap distribution modulation： 网络输出的 heatmap 通常不够平滑，文章提出 Heatmap distribution modulation 的方法，使用一个 Gaussian kernel 在原始特征图上进行卷积操作，Modulated heatmap 更有利于 Coordinate Decoding  
  - Coordinate Encoding
  现有方法：将 keypoint 坐标量化，然后以坐标点为中心生成高斯分布的 heatmap
  DARK：  取消量化操作，计算 heatmap 时，heatmap 中心点的坐标以浮点数表示。

## 2. Result
  `COCO test-dev`
  
  | Method              | Backbone          | Input size| #Params   | GFLOPs    | AP    | AP50  | AP75  | AP M  | AP L  | AR    |
  |-                    |-                  |-          |-          |-          |-      |-      |-      |-      |-      |-      |
  | G-RMI[23]           | ResNet-101        | 353 × 257 | 42.6M     | 57.0      | 64.9  | 85.5  | 71.3  | 62.3  | 70.0  | 69.7  |
  | IPR [27]            | ResNet-101        | 256 × 256 | 45.1M     | 11.0      | 67.8  | 88.2  | 74.8  | 63.9  | 74.0  | -     |
  | CPN [6]             | ResNet-Inception  | 384 × 288 | -         | -         | 72.1  | 91.4  | 80.0  | 68.7  | 77.2  | 78.5  |
  | RMPE [11]           | PyraNet           | 320 × 256 | 28.1M     | 26.7      | 72.3  | 89.2  | 79.1  | 68.0  | 78.6  | -     |
  | CFN [13]            | -                 | -         | -         | -         | 72.6  | 86.1  | 69.7  | 78.3  | 64.1  | -     |
  | CPN (ensemble) [6]  | ResNet-Inception  | 384 × 288 | -         | -         | 73.0  | 91.7  | 80.9  | 69.5  | 78.1  | 79.0  |
  | SimpleBaseline[33]  | ResNet-152        | 384 × 288 | 68.6M     | 35.6      | 73.7  | 91.9  | 81.1  | 70.3  | 80.0  | 79.0  |
  | HRNet[25]           | HRNet-W32         | 384 × 288 | 28.5M     | 16.0      | 74.9  | 92.5  | 82.8  | 71.3  | 80.9  | 80.1  |
  | HRNet[25]           | HRNet-W48         | 384 × 288 | 63.6M     | 32.9      | 75.5  | 92.5  | 83.3  | 71.9  | 81.5  | 80.5  |
  | DARK                | HRNet-W32         | 128 × 96  | 28.5M     | 1.8       | 70.0  | 90.9  | 78.5  | 67.4  | 75.0  | 75.9  |
  | DARK                | HRNet-W48         | 384 × 288 | 63.6M     | 32.9      | 76.2  | 92.5  | 83.6  | 72.5  | 82.4  | 81.1  |
  | G-RMI (extra data)  | ResNet-101        | 353 × 257 | 42.6M     | 57.0      | 68.5  | 87.1  | 75.5  | 65.8  | 73.3  | 73.3  |
  | HRNet (extra data)  | HRNet-W48         | 384 × 288 | 63.6M     | 32.9      | 77.0  | 92.7  | 84.5  | 73.4  | 83.1  | 82.0  |
  | DARK (extra data)   | HRNet-W48         | 384 × 288 | 63.6M     | 32.9      | 77.4  | 92.6  | 84.6  | 73.6  | 83.7  | 82.3  |

  `MPII`

  | Method  | Head  | Sho.  | Elb.  | Wri. | Hip    | Kne.  | Ank.  | Mean  |
  |-        |-      |-      |-      |-     |-       |-      |-      |-      |
  | HRN32   | 97.1  | 95.9  | 90.3  | 86.5 | 89.1   | 87.1  | 83.3  | 90.3  |
  | DARK    | 97.2  | 95.9  | 91.2  | 86.7 | 89.7   | 86.7  | 84.0  | 90.6  |