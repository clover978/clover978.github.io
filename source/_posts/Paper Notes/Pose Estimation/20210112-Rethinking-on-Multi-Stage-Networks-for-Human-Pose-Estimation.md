---
title: Rethinking on Multi-Stage Networks for Human Pose Estimation
date: 2021-01-12 21:33:24
tags:
  - deep learning
  - Pose Estimation
  - topdown
categories:
  - Paper Notes
  - Pose Estimation
---

# [Rethinking on Multi-Stage Networks for Human Pose Estimation](https://arxiv.org/pdf/1901.00148.pdf)

> 1. 作者将现有的 Pose Estimation 方法分为 single-stage 和 multi-stage 两类。并且认为现有的 multi-stage 方法的不足之处在于模型设计缺陷。  
  文章在经典的 stack hourglass 模型的基础上，提出三个方向的改进：1) 单阶段模型设计，2) 跨阶段特征聚合，3) coarse-to-fine 监督学习。  
> 2. 本文设计的 Multi Stage Pose Network 在 COCO test-dev 上取得 78.1% 的准确率，在 MPII test 上取得 92.6% 的准确率。
<!-- more -->


## 1. Introduction
  **单阶段模型**从图像分类领域迁移而来，一般基于单个骨干网络。  
  **多阶段模型**由若干阶段组成，每个阶段是一个轻量级的网络，网络包含上采样和下采样通路。
  根据现有工作。单阶段模型和多阶段模型的优劣无法区分。
  - improvements
    - 多阶段模型中，每个阶段的 module 设计不够好。
    - 提出跨阶段的特征聚合方法，加强 information flow。
    - 采用由粗到细的监督学习方法。


## 2. Method
  - Analysis of a Single-Stage Module
  Hourglass 设计中，网络下采样和上采样过程中，channel 数一致为常数，导致大量信息丢失。  
  现有改进通常会在 下采样和上采样过程中增加 channel 数，下采样过程 [256, 386, 512, 768]，上采样过程 [768, 512, 386, 256]。  
  MSPN 提出，上采样过程中不需要增加 channel 数，维持较少的 channel，即下采样过程 [256, 386, 512, 768]，上采样过程 [256, 256, 256, 256]。  

| Method      | Res-50    | Res-101   | Res-152   | Res-254   |
|-            |-          |-          |-          |-          |
| AP          | 71.5      | 73.1      | 73.6      | 74.0      |
| FLOPs(G)    | 4.4       | 7.5       | 11.2      | 18.0      |
  > 单阶段模型的上限大约为 74.0。

| Stages    | Hourglass |       | Stages    | MSPN      |       |
|-          |-          |-      |-          |-          |-      |
|           | FLOPs(G)  | AP    |           | FLOPs(G)  | AP    |
| 1         | 3.9       | 65.4  | 1         | 4.4       | 71.5  |
| 2         | 6.2       | 70.9  | 2         | 9.6       | 74.5  |
| 4         | 10.6      | 71.3  | 3         | 14.7      | 75.2  |
| 8         | 19.5      | 71.6  | 4         | 19.9      | 75.9  |
  > MSPN 设计的 Single-Stage Module，模型上限能达到 75.9

| Method    | Res-50    | 2×Res-18  | L-XCP | 4× S-XCP  |
|-          |-          |-          |-      |-          |
| AP        | 71.5      | 71.6      | 73.7  | 74.7      |
| FLOPs     | 4.4G      | 4.0G      | 6.1G  | 5.7G      |
  > FLOPs 相近的情况下，多阶段模型的效果好于单阶段模型。

  - Cross Stage Feature Aggregation
  将 stage(t-1) 的  downsampling 和 upsampling units 特征图，通过 1x1 卷积，连接到 stage(t) 的 downsampling unit，形成 residual 结构，实现 Cross Stage Feature Aggregation。  
  - Coarse-to-fine Supervision
  MSPN 在靠前的阶段，heatmap 使用较大的高斯核，在靠后的阶段，heatmap 使用较小的高斯核，用这种方法构造 GT，监督网络学习。

| BaseNet   | CTF   | CSFA  | Hourglass | MSPN  |
|-          |-      |-      |-          |-      |
| √         |       |       | 71.3      | 73.3  |
| √         | √     |       | 72.5      | 74.2  |
| √         | √     | √     | 73.0      | 74.5  |
  > 加入 Coarse-to-fine Supervision(CTF) 和 Cross Stage Feature Aggregation(CSFA) 模型的精度提高。

## 3. Result
  `COCO test-dev`

  | Method              | Backbone          | Input Size    | AP    | AP50  | AP75  | APM   | APL   | AR    | AR50  | AR75  | ARM   | ARL   |
  |-                    |-                  |-              |-      |-      |-      |-      |-      |-      |-      |-      |-      |-      |
  | CMU Pose [5]        | -                 | -             | 61.8  | 84.9  | 67.5  | 57.1  | 68.2  | 66.5  | 87.2  | 71.8  | 60.6  | 74.6  |
  | Mask R-CNN [16]     | Res-50-FPN        | -             | 63.1  | 87.3  | 68.7  | 57.8  | 71.4  | -     | -     | -     | -     | -     |
  | G-RMI [31]          | Res-152           | 353×257       | 64.9  | 85.5  | 71.3  | 62.3  | 70.0  | 69.7  | 88.7  | 75.5  | 64.4  | 77.1  |
  | AE [28]             | -                 | 512×512       | 65.5  | 86.8  | 72.3  | 60.6  | 72.6  | 70.2  | 89.5  | 76.0  | 64.6  | 78.1  |
  | CPN [9]             | Res-Inception     | 384×288       | 72.1  | 91.4  | 80.0  | 68.7  | 77.2  | 78.5  | 95.1  | 85.3  | 74.2  | 84.3  |
  | Simple Base [46]    | Res-152           | 384×288       | 73.7  | 91.9  | 81.1  | 70.3  | 80.0  | 79.0  | -     | -     | -     | -     |
  | HRNet [39]          | HRNet-W48         | 384×288       | 75.5  | 92.5  | 83.3  | 71.9  | 81.5  | 80.5  | -     | -     | -     | -     |
  | Ours (MSPN)         | 4×Res-50          | 384×288       | 76.1  | 93.4  | 83.8  | 72.3  | 81.5  | 81.6  | 96.3  | 88.1  | 77.5  | 87.1  |
  | CPN+ [9]            | Res-Inception     | 384×288       | 73.0  | 91.7  | 80.9  | 69.5  | 78.1  | 79.0  | 95.1  | 85.9  | 74.8  | 84.6  |
  | Simple Base+* [46]  | Res-152           | 384×288       | 76.5  | 92.4  | 84.0  | 73.0  | 82.7  | 81.5  | 95.8  | 88.2  | 77.4  | 87.2  |
  | HRNet* [39]         | HRNet-W48         | 384×288       | 77.0  | 92.7  | 84.5  | 73.4  | 83.1  | 82.0  | -     | -     | -     | -     |
  | Ours (MSPN*)        | 4×Res-50          | 384×288       | 77.1  | 93.8  | 84.6  | 73.4  | 82.3  | 82.3  | 96.5  | 88.9  | 78.4  | 87.7  |
  | Ours (MSPN+*)       | 4×Res-50          | 384×288       | 78.1  | 94.1  | 85.9  | 74.5  | 83.3  | 83.1  | 96.7  | 89.8  | 79.3  | 88.2  |

  `COCO test-challenge`

  | Method              | Backbone          | Input Size    | AP    | AP50  | AP75  | APM   | APL   | AR    | AR50  | AR75  | ARM   | ARL   |
  |-                    |-                  |-              |-      |-      |-      |-      |-      |-      |-      |-      |-      |-      |
  | Mask R-CNN* [16]    | ResX-101-FPN      | -             | 68.9  | 89.2  | 75.2  | 63.7  | 76.8  | 75.4  | 93.2  | 81.2  | 70.2  | 82.6  |
  | G-RMI* [31]         | Res-152           | 353×257       | 69.1  | 85.9  | 75.2  | 66.0  | 74.5  | 75.1  | 90.7  | 80.7  | 69.7  | 82.4  |
  | CPN+ [9]            | Res-Inception     | 384×288       | 72.1  | 90.5  | 78.9  | 67.9  | 78.1  | 78.7  | 94.7  | 84.8  | 74.3  | 84.7  |
  | Sea Monsters+*      | -                 | -             | 74.1  | 90.6  | 80.4  | 68.5  | 82.1  | 79.5  | 94.4  | 85.1  | 74.1  | 86.8  |
  | Simple Base+* [46]  | Res-152           | 384×288       | 74.5  | 90.9  | 80.8  | 69.5  | 82.9  | 80.5  | 95.1  | 86.3  | 75.3  | 87.5  |
  | Ours (MSPN+*)       | 4×Res-50          | 384×288       | 76.4  | 92.9  | 82.6  | 71.4  | 83.2  | 82.2  | 96.0  | 87.7  | 77.5  | 88.6  |
  




