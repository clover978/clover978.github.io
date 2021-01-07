---
title: Simple Baselines for Human Pose Estimation and Tracking
date: 2021-01-07 11:34:40
tags:
  - deep learning
  - Pose Estimation
categories:
  - Paper Notes
  - Pose Estimation
---

https://arxiv.org/abs/1711.07319

# Introduction:
1. 提出姿态估计 和 姿态跟踪 的一个简单 baseline

<!-- more -->

# Pose Estimation Using A Deconvolution Head Network
作者提出一个简单的网络结构作为 Baseline，网络结构如下：  
  参考Fig1
  
作者对比了之前的两种方法 Stacked hourglass 和 CPN，两者都是对特征图进行上采样，生成 heatmap，而作者提出的网络结构，直接通过三层反卷积生成了heatmap，结构十分简单。

## Pose Tracking Based on Optical Flow
作者对 ICCV'17 PoseTrack Challenge 的方法（*Detect-and-Track*）进行改进，提出 多人姿态跟踪 的 Baseline。作者的改进点有一下两个：  
  - **Joint Propagation using Optical Flow**  
    利用前一帧的关键点预测 J(k) 已经两帧的光流 F(k, k+1)，投影出当前帧的关键点 J(k+1)，计算关键点的包围框并扩大一定比例，投影出当前帧的包围框。  
    这样可以解决由于 **模糊、遮挡** 造成的误检
  - **Flow-based Pose Similarity**  
    Pose Track 的时候需要计算两帧之间的 similarity，作者认为，用 包围框的 IOU 作为 metric 不能处理运动太快的情况；直接用 OKS 作为 metric 不能处理人物动作幅度比较大的情况。因此，作者首先利用光流投影出上一帧的关键点在当前帧的位置，然后再计算 OKS。

## Flow-based Pose Tracking Algorithm  
  参考Fig2
  
# Experiments
## 1. Pose Estimation on COCO  
  - Train：
    - pretrain on ImageNet
    - 将 GT 框扩展成 4:3
    - crop 出 GT 框并 resize 成固定大小 256x192
    - 数据增强： scale（0.7~1.3）， rotation （-40~40），flip
  - Test：  
    - faster-rcnn 检测
    - origin + flip
    - quarter offset from highest response to the second highest response


| Method | Backbone | Input Size | OHKM | AP |
| - | - | - | - | - |
| 8-stage Hourglass | - | 256x192 | n | 66.9 | 
| 8-stage Hourglass | - | 256x256 | n | 67.1 |
| CPN | ResNet-50 | 256x192 | n | 68.6 |
| CPN | ResNet-50 | 384x288 | n | 70.6 |
| CPN | ResNet-50 | 256x192 | y | 69.4 |
| CPN | ResNet-50 | 384x288 | y | 71.6 | 
| Ours | ResNet-50 | 256x192 | n | 70.4 | 
| Ours | ResNet-50 | 384x288 | n | 72.2 |
| Ours | ResNet-101 | 256x192 | n | 71.4 | 
| Ours | ResNet-152 | 256x192 | n | 72.0 | 

## Pose Estimation and Tracking on PoseTrack
  - Train:  
    - pretrain on COCO
    - 根据 keypoint 画出 bbox，外扩 15%
    - 数据增强
  - Test：  
    - 检测框（ R-FCN、 FPN-DCN ）
    - 光流估计 （ FlowNet2S )
    - Propagation 
    - 丢弃低置信度框（<0.5）
    - 丢弃低置信度关键点 （<0.4）

| Method | Backbone | Detector | Joint Propagation | Similarity Metric | mAP | MOTA |
| - | - | - | - | - | - | - |
| a6 | ResNet-50 | R-FCN | y | SMulti_flow | 70.3 | 62.2 |
| b1 | ResNet-50 | FPN-DCN | n | SBbox | 69.3 | 59.8 |
| b2 | ResNet-50 | FPN-DCN | n | SPose | 69.3 | 59.7 |
| b3 | ResNet-50 | FPN-DCN | y | SBbox | 72.4 | 62.1 |
| b4 | ResNet-50 | FPN-DCN | y | SPose | 72.4 | 61.8 |
| b5 | ResNet-50 | FPN-DCN | y | SFlow | 72.4 | 62.4 |
| b6 | ResNet-50 | FPN-DCN | y | SMulti_Flow | 72.4 | 62.9 |
| c6 | ResNet-152 |  FPN-DCN | y | SMulti_Flow | 76.7 | 65.4 |
