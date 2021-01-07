---
title: Cascaded Pyramid Network for Multi-Person Pose Estimation
date: 2021-01-07 11:32:27
tags:
  - deep learning
  - Pose Estimation
categories:
  - Paper Notes
  - Pose Estimation
---


https://arxiv.org/abs/1711.07319

# Introduction:
1. 提出 `cascaded pyramid network（CPN）`，包含 `global pyramid network` 和 `pyramid refined network`
2. 探讨 多人目标检测中各种因素的影响
3. COCO 上 **test-dev AP： 73.0; test-challenge AP：72.1**

<!-- more -->

# Related work:
1. muti-pose:
    - bottom-up: DeepCut, OpenPose  
    - top-down: Mask-RCNN  

2. single pose:
    - CPM
    - hourglass
    - ...

3. detection:
    - FPN
    - mask-RCNN

# Method

  - network structure:  
    [Fig-1](https://arxiv.org/pdf/1711.07319.pdf)

  - Detector:
    这篇文章使用的 detetor 配置如下， FPN(res101) + ROIAlign + softNMS. AP = 0.411, AP(Person) = 0.533

  - GlobalNet  
    basemodel 提取不同 stage 的特征，底层的特征直接用插值的方法 upsample，然后通过 `1*1` 卷积将通道统一为 256，对特征做 `element-wise sum` 得到 feature pyramid，金字塔上每一层特征通过一个 1\*1 conv + 3\*3\*17 conv 得到 globalnet 的输出（ heatmap \* 17）

  - RefineNet  
    使用 bottleneck 结构对特征进行 upsample，特征图越小，通过的 bootleneck 越多，upsample 倍率越大，最后所有特征都具有相同的大小。将 4 个 stage 的特征在 channel wise concat 起来。

  - ohkm
    根据 loss 判断出 hard keypoint，然后只对部分关键点的 loss 进行反向传播。

# Experiment
  - Cropping Strategy  
    `bbox --(extend)--> 256:192  --(crop)--> block --(resize)--> 256:192`
  - Data Augmentation  
    random flip  
    random rotation  
    random scale
  - Test:  
    **Apply gussian filter to heatmap**  
    **Also test flipped image**  
    **最高的响应 R1，第二高的响应为 R2，最终结果 = R1+0.25\*R2**  
    **Rescore: S(pose) = S(bbox) \* avg(S(key-point))**


# Result

**Ablation experiment**
|| AP | FLOPs | Params |
|- |- | - | - |
| ResNet-50 + dilation(res4-5) | 66.5 | 17.71G | 92M |
| GlobalNet only | 66.6 | 3.90G | 94M |
| CPN w/o ohkm | 68.6 | 6.20G | 102M |
| CPN | 69.4 | 6.20G | 102M |
| GlobalNet + Concat | 68.5 | 5.87G | - |
| GlobalNet + 1 bottleneck +Concat | 69.2 | 6.92G | - |
| CPN (384 × 288) | 71.6 | 13.9G | - |



在 **refineNet** 中使用不同的 layer

| | AP | FLOPs |
| - | - | - | 
| C2 | 68.3 | 5.02G |
| C2-3 | 68.4 | 5.50G | 
| C2-4 | 69.1 | 5.88G | 
| C2-5 | 69.4 | 6.20G | 