---
title: Pose Estimation resources
date: 2021-01-12 15:19:47
tags:
  - deep learning
  - Pose Estimation
  - benchmark
categories:
  - Paper Notes
  - Pose Estimation
---

# Awsome Pose Estimation

## 1. Pose Estimation Papers  
  [awesome-human-pose-estimation](https://github.com/wangzheallen/awesome-human-pose-estimation): a github repo which collects update-to-date **Pose Estimation Papers**.
<!-- more -->


## 2. Pose Estimation SOTA  
  [Paper With Code](https://paperswithcode.com/task/pose-estimation): Summary of SOTA over `MPII, COCO-val, COCO-test_dev, PoseTrack` datasets.  


## 3. Pose Estimation Codebase  
  [MMPose](https://github.com/open-mmlab/mmpose): an open-source toolbox for pose estimation based on PyTorch.  
  [MMPose Document](https://mmpose.readthedocs.io)


## 4. Pose Estimation benchmark
- `COCO test-challenge`  

| Method          | backbone          | input-size    | AP    | remark    |
|-                |-                  |-              |-      |-          |
| **bottomup** |
|                 |                   |               |       |           |
| **topdown** |
| G-RMI           | -                 |               | 69.1  | extra data    |
| CPN             | ResNet-Inception  |               | 72.1  | ensemble  |
| DARK            | HRNet-W48         | 384x288       | 76.4  | extra data & ensemble |

- `COCO test-dev`  

| Method          | backbone          | input-size    | AP    | remark    |
|-                |-                  |-              |-      |-          |
| **bottomup** |
| OpenPose        | -                 |               | 61.8  |           |
| AE              | StackHourglass    |               | 65.0  | multi scale   |
| AE              | StackHourglass    |               | 65.5  | refine    |
| HigherHRNet     | HRNet-W32         | 512x512       | 66.4  |           |
| HigherHRNet     | HRNet-W48         | 640x640       | 68.4  |           |
| HigherHRNet     | HRNet-W48         | 640x640       | 70.5  | multi scale   |
| **topdown** |
| CPN             | ResNet-Inception  |               | 72.1  |           |
| CPN             | ResNet-Inception  |               | 73.0  | ensemble  |
| RMPE            | PyraNet           | 320x256       | 72.3  |           |
| SimpleBaseline  | ResNet-152        | 384x288       | 73.7  |           |
| HRNet-W32       | HRNet-W32         | 384x288       | 74.9  |           |
| HRNet-W48       | HRNet-W48         | 384x288       | 75.5  |           |
| HRNet-W48       | HRNet-W48         | 384x288       | 77.0  | extra data    |
| EvoPose2D-L     |                   | 512x384       | 75.7  |           |
| MSPN            | 4×Res-50          | 384x288       | 76.1  |           |
| MSPN            | 4×Res-50          | 384x288       | 77.1  | extra data    |
| MSPN            | 4×Res-50          | 384x288       | 78.1  | extra data & ensemble |
| DARK            | HRNet-W48         | 384x288       | 76.2  | 75.5(+0.7)    |
| DARK            | HRNet-W48         | 384x288       | 77.4  | 77.0(+0.4) & extra data   |
| DARK            | HRNet-W48         | 384x288       | 78.9  | extra data & ensemble |
| UDP             | HRNet-W48         | 384x288       | 76.5  | 75.5(+1.0)    |
| PoseFix         | HRNet-W48         | 384x288       | 76.7  | 75.5(+1.2)    |
| PoseFix         | EvoPose2D-L       | 512x384       | 76.8  | 75.7(+1.1)    |


- `COCO-val`  

| Method          | backbone          | input-size    | AP    | remark    |
|-                |-                  |-              |-      |-          |
| **bottomup** |
| OpenPose        | -                 |               | 58.4  |           |
| OpenPose        | -                 |               | 61.0  | CPM refine    |
| HigherHRNet     | HRNet-W32         | 512x512       | 67.1  |           |
| HigherHRNet     | HRNet-W32         | 640x640       | 68.5  |           |
| HigherHRNet     | HRNet-W48         | 640x640       | 69.9  |           |
| UDP             | HigherHRNet-W32   | 512x512       | 67.8  | 67.1(+0.7)    |
| UDP             | HigherHRNet-W48   | 640x640       | 69.9  | 69.9(+0.0)    |
| **topdown** |
| CPN             | ResNet-Inception  |               | 72.7  |           |
| CPN             | ResNet-Inception  |               | 74.5  | ensemble  |
| MSPN            | 4×Res-50          | 384x288       | 76.4  | extra data & ensemble |
| HRNet-W32       | HRNet-W32         | 384x288       | 75.8  |           |
| HRNet-W48       | HRNet-W48         | 384x288       | 76.3  |           |
| EvoPose2D-L     |                   | 512x384       | 76.6  |           |
| DARK            | HRNet-W32         | 384x288       | 76.6  | 75.8(+0.8)    |
| PoseFix         | HRNet-W48         | 384x288       | 77.3  | 76.3(+1.0)    |
| UDP             | HRNet-W48         | 384x288       | 77.8  | 77.1(+0.7) **???**    |

- `MPII test`  

| Method          | backbone          | input-size    | AP    | remark    |
|-                |-                  |-              |-      |-          |
| **bottomup** |
|                 |                   |               |       |           |
| **topdown** |
| Stack Hourglass |                   |               | 90.9  |           |
| SimpleBaseline  | ResNet-152        |               | 91.5  |           |
| HRNet-W32       | HRNet-W32         |               | 92.3  |           |

- `MPII multi-person test`  

| Method          | backbone          | input-size    | AP    | remark    |
|-                |-                  |-              |-      |-          |
| **bottomup** |
| OpenPose        | -                 |               | 72.5  |           |
| OpenPose        | -                 |               | 75.6  | multi-scale   |
| AE              | StackHourglass    |               | 77.5  |           |
| **topdown** |
| RMPE            | PyraNet           | 320x256       | 82.1  |           |








  
