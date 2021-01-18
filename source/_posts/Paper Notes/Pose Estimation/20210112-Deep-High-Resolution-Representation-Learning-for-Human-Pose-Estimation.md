---
title: Deep High-Resolution Representation Learning for Human Pose Estimation
date: 2021-01-12 13:55:08
tags:
  - deep learning
  - Pose Estimation
  - topdown
categories:
  - Paper Notes
  - Pose Estimation
---

# [Deep High-Resolution Representation Learning for Human Pose Estimation](https://arxiv.org/pdf/1902.09212.pdf)

> 1. 本文提出一种新型的网络结构 HRNet，与以往的网络结构不同，HRNet 中 high-to-low resolution subnetworks 是平行排列的，不同分辨率的特征通过 downsample 和 upsample 的形式相互融合。
> 2. HRNet 在 COCO test-dev 数据集上取得 75.5% 的准确率，在加入额外训练数据后准确率为 77.0%

<!-- more -->


## 1. Method
- Sequential multi-resolution subnetworks  
  现有网络的 backbone 大多为 Sequential 结构，每个 subnetwork 被称为一个 stage，相邻的 stage 通过 downsample 连接，网络的分辨率逐层减半。
    ```
    N11 → N22 → N33 → N44
    ```
- Parallel multi-resolution subnetworks  
  HRNet 第一个 stage 为 high-resolution subnetwork，然后以并行的方法，逐渐添加 high-to-low resolution subnetworks。因此，后续添加的 stage 既包含之前 stage 的 high resolution 信息，又包含 low resolution 信息。
    ```
    N11 → N21 → N31 → N41
          N22 → N32 → N42
                N33 → N43
                      N44
   ```
- Repeated multi-scale fusion  
  HRNet 中加入 *exchange units* 来交换不同 subnetworks 的信息。  
  downsample 过程通过 3x3 的卷积实现，卷积的 stride 为 2，图像分辨率下降一半；  
  upsample 过程分为两步，第一步是 nearest neighbor sampling，第二步是 1x1 卷积，用于改变通道数。
- Heatmap estimation  
  作者在最后一层 *exchange unit* 输出的 特征图上，预测最终的 heatmap。  


## 2. Result

- `COCO val`  

| Method                    | Backbone              | Pretrain  | Input size | #Params | GFLOPs | AP   | AP50 | AP75 | APM  | APL  | AR   |
|-                          |-                      |-          |-           |-        |-       |-     |-     |-     |-     |-     |-     |
| 8-stage Hourglass [40]    | 8-stage Hourglass     | N         | 256 × 192  | 25.1M   | 14.3   | 66.9 | −    | −    | −    | −    | −    |
| CPN [11]                  | ResNet-50             | Y         | 256 × 192  | 27.0M   | 6.20   | 68.6 | −    | −    | −    | −    | −    |
| CPN + OHKM [11]           | ResNet-50             | Y         | 256 × 192  | 27.0M   | 6.20   | 69.4 | −    | −    | −    | −    | −    |
| SimpleBaseline [72]       | ResNet-50             | Y         | 256 × 192  | 34.0M   | 8.90   | 70.4 | 88.6 | 78.3 | 67.1 | 77.2 | 76.3 |
| SimpleBaseline [72]       | ResNet-101            | Y         | 256 × 192  | 53.0M   | 12.4   | 71.4 | 89.3 | 79.3 | 68.1 | 78.1 | 77.1 |
| SimpleBaseline [72]       | ResNet-152            | Y         | 256 × 192  | 68.6M   | 15.7   | 72.0 | 89.3 | 79.8 | 68.7 | 78.9 | 77.8 |
| HRNet-W32                 | HRNet-W32             | N         | 256 × 192  | 28.5M   | 7.10   | 73.4 | 89.5 | 80.7 | 70.2 | 80.1 | 78.9 |
| HRNet-W32                 | HRNet-W32             | Y         | 256 × 192  | 28.5M   | 7.10   | 74.4 | 90.5 | 81.9 | 70.8 | 81.0 | 79.8 |
| HRNet-W48                 | HRNet-W48             | Y         | 256 × 192  | 63.6M   | 14.6   | 75.1 | 90.6 | 82.2 | 71.5 | 81.8 | 80.4 |
| SimpleBaseline [72]       | ResNet-152            | Y         | 384 × 288  | 68.6M   | 35.6   | 74.3 | 89.6 | 81.1 | 70.5 | 79.7 | 79.7 |
| HRNet-W32                 | HRNet-W32             | Y         | 384 × 288  | 28.5M   | 16.0   | 75.8 | 90.6 | 82.7 | 71.9 | 82.8 | 81.0 |
| **HRNet-W48**             | HRNet-W48             | Y         | 384 × 288  | 63.6M   | 32.9   | 76.3 | 90.8 | 82.9 | 72.3 | 83.4 | 81.2 |

- `COCO test-dev`  

| Method                        | Backbone         | Input size | #Params | GFLOPs |  AP  | AP50 | AP75 | APM  | APL  | AR   |
|-                              |-                 |-           |-        |-       |-     |-     |-     |-     |-     |-     |
| Mask-RCNN [21]                | ResNet-50-FPN    |            |         |        | 63.1 | 87.3 | 68.7 | 57.8 | 71.4 |      |
| G-RMI [47]                    | ResNet-101       | 353 × 257  | 42.6M   | 57.0   | 64.9 | 85.5 | 71.3 | 62.3 | 70.0 | 69.7 |
| Integral Pose Regression [60] | ResNet-101       | 256 × 256  | 45.0M   | 11.0   | 67.8 | 88.2 | 74.8 | 63.9 | 74.0 |      |
| G-RMI + extra data [47]       | ResNet-101       | 353 × 257  | 42.6M   | 57.0   | 68.5 | 87.1 | 75.5 | 65.8 | 73.3 | 73.3 |
| CPN [11]                      | ResNet-Inception | 384 × 288  |         |        | 72.1 | 91.4 | 80.0 | 68.7 | 77.2 | 78.5 |
| RMPE [17]                     | PyraNet [77]     | 320 × 256  | 28.1M   | 26.7   | 72.3 | 89.2 | 79.1 | 68.0 | 78.6 |      |
| CFN [25]                      |                  |            |         |        | 72.6 | 86.1 | 69.7 | 78.3 | 64.1 |      |
| CPN (ensemble) [11]           | ResNet-Inception | 384 × 288  |         |        | 73.0 | 91.7 | 80.9 | 69.5 | 78.1 | 79.0 |
| SimpleBaseline [72]           | ResNet-152       | 384 × 288  | 68.6M   | 35.6   | 73.7 | 91.9 | 81.1 | 70.3 | 80.0 | 79.0 |
| HRNet-W32                     | HRNet-W32        | 384 × 288  | 28.5M   | 16.0   | 74.9 | 92.5 | 82.8 | 71.3 | 80.9 | 80.1 |
| HRNet-W48                     | HRNet-W48        | 384 × 288  | 63.6M   | 32.9   | 75.5 | 92.5 | 83.3 | 71.9 | 81.5 | 80.5 |
| **HRNet-W48 + extra data**    | HRNet-W48        | 384 × 288  | 63.6M   | 32.9   | 77.0 | 92.7 | 84.5 | 73.4 | 83.1 | 82.0 |