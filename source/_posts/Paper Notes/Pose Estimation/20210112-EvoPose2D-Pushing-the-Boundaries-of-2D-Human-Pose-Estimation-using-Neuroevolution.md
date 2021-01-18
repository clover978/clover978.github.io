---
title: >-
  EvoPose2D-Pushing the Boundaries of 2D Human Pose Estimation using
  Neuroevolution
date: 2021-01-12 15:15:37
tags:
  - deep learning
  - Pose Estimation
  - topdown
categories:
  - Paper Notes
  - Pose Estimation
---

# EvoPose2D-Pushing the Boundaries of 2D Human Pose Estimation using Neuroevolution

> 1. 本文使用 神经网络进化（neuroevolution） 的方法搜索最优的 2D Human Pose Estimation 模型。文章提出 weight transfer scheme 来加速神经网络进化的速度。
> 2. 通过神经网络进化生成的模型，可以比现有的 SOTA 具有更优的效果。其中最优的模型 `EvoPose2D-L` 相比于 HRNet-W48 只有 1/2.0 的 operations， 1/4.3 的 parameters。
> 3. `EvoPose2D-L` 在 COCO test-dev 数据集上准确率为 75.7%
<!-- more -->


## 1. Method
  - Weight transfer  
  文章提出 Weight transfer 来加速神经网络进化的过程。  
  为了保证突变（mutated）后的 child 网络能够快速收敛，很自然的想法就是使用 parent 的参数对其进行初始化。  
  作者根据 child 和 parent 的 kernel-size 和 channels 大小，定义了四种情况，具有 transfer 的方法参见论文公式。  
  - Search space  
  作者根据网络的各种超参，定义了 10e14大小的搜索空间，具有定义方法见论文描述。
  - Fitness  
  神经网络进化中，需要考虑 performance 和 effciency 的平衡，文章使用 Pareto optimizations 的优化方法，最小化适应函数 `fitness function`。函数表达式见论文。
  - Evolutionary strategy  
  神经网络进化的过程类似生物进化的流程。从 0 号祖先开始，生成若干个 child，然后找到 m 个 fit 比较好的 child，成为新的祖先，完成一次进化，重复多轮进化，直到 适应函数 收敛时，手动结束网络进化流程。  
  - Large-batch training  
  为了使神经网路进化的进程尽量快，文章使用了 large-batchsize 训练，训练过程中参照之前研究的 large-batchsize 训练方法，保证模型可以收敛。  
  - Compound scaling
  同样根据现有的研究结果，网络的 resolution, channels, depth 同步的缩放可以使神经网络进化过程更加高效，作者根据搜索空间中的 resolution 直接计算出 channels 和 depth，实现 Compound scaling。  


## 2. Result
- `COCO val`

| Method                | Backbone      | Pretrain  | Input size  | Params (M)  | FLOPs (G) |  AP   | AP50  | AP75  | APM   | APL   | AR    |
|-                      |-              |-          |-            |-            |-          |-      |-      |-      |-      |-      |-      |
| CPN [9]               | ResNet-50     | Y         | 256 × 192   | 27.0        | 6.20      | 68.6  | −     | −     | −     | −     | −     |
| SimpleBaseline [49]   | ResNet-50     | Y         | 256 × 192   | 34.0        | 5.21†     | 70.4  | 88.6  | 78.3  | 67.1  | 77.2  | 76.3  |
| SimpleBaseline [49]   | ResNet-101    | Y         | 256 × 192   | 53.0        | 8.84†     | 71.4  | 89.3  | 79.3  | 68.1  | 78.1  | 77.1  |
| SimpleBaseline [49]   | ResNet-152    | Y         | 256 × 192   | 68.6        | 12.5†     | 72.0  | 89.3  | 79.8  | 68.7  | 78.9  | 77.8  |
| HRNet-W32 [40]        | -             | N         | 256 × 192   | 28.5        | 7.65†     | 73.4  | 89.5  | 80.7  | 70.2  | 80.1  | 78.9  |
| HRNet-W32 [40]        | -             | Y         | 256 × 192   | 28.5        | 7.65†     | 74.4  | 90.5  | 81.9  | 70.8  | 81.0  | 79.8  |
| HRNet-W48 [40]        | -             | Y         | 256 × 192   | 63.6        | 15.7†     | 75.1  | 90.6  | 82.2  | 71.5  | 81.8  | 80.4  |
| MSPN [25]             | 4xResNet-50   | Y         | 256 × 192   | 120         | 19.9      | 75.9  | −     | −     | −     | −     | −     |
| SimpleBaseline [49]   | ResNet-152    | Y         | 384 × 288   | 68.6        | 28.1†     | 74.3  | 89.6  | 81.1  | 70.5  | 79.7  | 79.7  |
| HRNet-W32 [40]        | -             | Y         | 384 × 288   | 28.5        | 16.0†     | 75.8  | 90.6  | 82.7  | 71.9  | 82.8  | 81.0  |
| HRNet-W48 [40]        | -             | Y         | 384 × 288   | 63.6        | 35.3†     | 76.3  | 90.8  | 82.9  | 72.3  | 83.4  | 81.2  |
| HRNet-W48 + PF [33]   | -             | Y         | 384 × 288   | 63.6        | 35.3†     | 77.3  | 90.9  | 83.5  | 73.5  | 84.4  | 82.0  |
| SimpleBaseline        | ResNet-50     | N         | 256 × 192   | 34.1        | 5.21      | 70.6  | 89.0  | 78.4  | 66.9  | 77.1  | 77.3  |
| SimpleBaseline        | ResNet-50     | Y         | 256 × 192   | 34.1        | 5.21      | 71.0  | 89.2  | 78.5  | 67.4  | 77.4  | 78.0  |
| HRNet-W32             | -             | N         | 256 × 192   | 28.6        | 7.65      | 73.6  | 89.9  | 80.5  | 70.1  | 80.0  | 80.0  |
| EvoPose2D-XS          | -             | N         | 256 × 192   | 2.53        | 0.47      | 67.7  | 87.8  | 75.8  | 64.5  | 73.7  | 74.7  |
| EvoPose2D-XS          | -             | WT        | 256 × 192   | 2.53        | 0.47      | 68.0  | 87.9  | 76.1  | 64.5  | 74.3  | 75.0  |
| EvoPose2D-S           | -             | N         | 256 × 192   | 2.53        | 1.07      | 69.8  | 88.6  | 77.3  | 66.3  | 76.2  | 76.4  |
| EvoPose2D-S           | -             | WT        | 256 × 192   | 2.53        | 1.07      | 70.2  | 88.9  | 77.8  | 66.5  | 76.8  | 76.9  |
| EvoPose2D-M           | -             | N         | 384 × 288   | 7.34        | 5.59      | 75.1  | 90.2  | 81.9  | 71.5  | 81.7  | 81.0  |
| EvoPose2D-L           | -             | N         | 512 × 384   | 14.7        | 17.7      | 76.6  | 90.5  | 83.0  | 72.7  | 83.4  | 82.3  |
| EvoPose2D-L + PF      | -             | N         | 512 × 384   | 14.7        | 17.7      | 77.5  | 90.9  | 83.6  | 74.0  | 84.2  | 82.5  |

- `COCO test-dev`

| Method              | Backbone      | Pretrain  | Input size  | Params (M)  | FLOPs (G)   | AP    | AP50  | AP75  | APM   | APL   | AR    |
|-                    |-              |-          |-            |-            |-            |-      |-      |-      |-      |-      |-      |
| CPN [9]             | Res-Inception | Y         | 384 × 288   | -           | -           | 72.1  | 91.4  | 80.0  | 68.7  | 77.2  | 78.5  |
| SimpleBaseline [49] | ResNet-152    | Y         | 384 × 288   | 68.6        | 35.6        | 73.7  | 91.9  | 81.1  | 70.3  | 80.0  | 79.0  |
| HRNet-W48 [40]      | -             | Y         | 384 × 288   | 63.6        | 32.9        | 75.5  | 92.5  | 83.3  | 71.9  | 81.5  | 80.5  |
| HRNet-W48 + PF [33] | -             | Y         | 384 × 288   | 63.6        | 32.9        | 76.7  | 92.6  | 84.1  | 73.1  | 82.6  | 81.5  |
| EvoPose2D-L         | -             | N         | 512 × 384   | 14.7        | 17.7        | 75.7  | 91.9  | 83.1  | 72.2  | 81.5  | 81.7  |
| EvoPose2D-L + PF    | -             | N         | 512 × 384   | 14.7        | 17.7        | 76.8  | 92.5  | 84.3  | 73.5  | 82.5  | 81.7  |


