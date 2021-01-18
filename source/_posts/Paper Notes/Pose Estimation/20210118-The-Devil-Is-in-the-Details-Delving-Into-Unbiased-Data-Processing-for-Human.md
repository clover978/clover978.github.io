---
title: The Devil Is in the Details-Delving Into Unbiased Data Processing for Human
date: 2021-01-13 20:27:05
tags:
  - deep learning
  - Pose Estimation
  - poseplugin
categories:
  - Paper Notes
  - Pose Estimation
mathjax: true
---

# [The Devil Is in the Details-Delving Into Unbiased Data Processing for Human](https://arxiv.org/pdf/1911.07524.pdf)

> 1. 本文用数学方法描述了 Pose Estimation 中的 Coordinate System Transformation（坐标系转换）和  Keypoint Format Transformation（关键点格式转换）过程，并分析其中的误差。文章发现：  
  Coordinate System Transformation 过程中，传统方法中的 flip test 操作是 Biased Coordinate System Transformation，会导致误差。
  Keypoint Format Transformation 过程中，传统方法由于使用的量化坐标，也会导致关键点格式转换过程出现 bais。
  作者针对上述问题，提出的 Unbiased Data Processing 方法，消除误差。
> 2. UDP 一种模型无关的算法，可以用在任意现有的算法上，提升算法效果。
<!-- more -->


## 1. Method
 
### 1.1 连续空间 与 离散空间  
  现实中的图像处于连续空间中，图像由 2维空间 中无数个点构成。一个 R*R 的图像中关键点的坐标由一个实数对(x,y) 表示。  
  计算机中的图像处于离散空间中，图像由 若干像素点组成，一个 M*M 的图像中的关键点的坐标由一个 整数对(i,j) 表示。  
  **连续数据通过采样得到离散数据**：现实中的图像通过采样（感知），形成像素点，变成离散数据，存储在计算机中。  
  **离散数据通过插值得到连续数据**：计算机中的图像矩阵，可以将通过插值，将整数域的数据表示为实数域的数据，变成连续数据。  
#### 1.1.1 连续图像 与 离散图像 的转换  
  ![连续数据与离散数据](连续数据与离散数据.jpg)
  - 连续数据 到 离散数据  
  假设2维连续空间中有一幅图像，大小为 8cmx8cm，现在对其进行离散采样，我们考虑 x 轴方向上的采样情况。  
  在连续空间中，图像的 x 轴坐标范围为 [0,8]，以 x=1 处的数据为例，解释采样的过程。传感器感知 x=1 位置的数据，存储为 1 个像素，数据被转化为离散数据。可以发现，**在连续空间中长度为 8 的数据，在离散空间中占据 9 个像素**，离散空间中，图像的 x 轴坐标范围为 {0, 1, 2, 3, 4, 5, 6, 7, 8}。  
  - 离散数据 到 连续数据  
  将上面的例子反过来，假设计算机中存储了一幅图像，大小为 9x9，我们通过插值的方式还原图像在现实连续空间中的样子，只考虑 x 轴方向上的情况。  
  每个像素的中心点，在连续空间中为 x=0,1,2...,7,8 的位置，连续空间中非整数部分的数据，通过插值算法生成，假设采用最邻近插值算法，图像在连续空间中的表示如图所示。可以发现，**9x9 大小的离散图像，在连续空间中表示时，图像的宽度为 8 个单位长度**。
  注意，上面的例子中，图像在连续空间中的范围不是 [-0.5, 8.5]，我们在转换的过程中，坐标轴是始终对齐的，或者说，连续图像 与 离散图像 所在的坐标空间是一致的，只是数据的表示形式不同。  

### 1.2 坐标系统转换偏差分析（Coordinate System Transformation）  
  通过数学推导，UDP 得出，在姿态估计中，如果使用 flip test，关键点坐标会出现偏移，在网络输出的 heatmap 上，x 轴方向会出现一段 的偏移 $ \frac{(1-s)}{s} $ （ s 为输入输出像素比）；在原始图像的最终结果上，x 轴方向会出现 $ \frac{bw_{s}(s-1)}{W_{i}^{p}} $ （ $ bw_{s} $为 ROI 的宽像素数，$ W_{i}^{p} $为网络输入大小的宽像素数）的偏移。下面用实际例子验证上述结论。  
  ![坐标系转换偏差](坐标系转换偏差.jpg)
  假设：  
  - 原始图像大小为 ：18x18 像素；  
  - 网络输入图像大小为： 12x12像素；  
  - 关键点位置为：x 轴第 10 个像素点，x=9。  

  flip test 的流程如下：  
  1. 将原始图像 resize 为 12x12 大小；  
  2. 将 12x12 图像进行 flip；  
  3. 将 flip 后的图像输入网络，得到heatmap，在heatmap 上定位关键点输出坐标；  
  4. 将 heatmap 进行 flip back 操作，得到heatmap上关键点最终坐标；  
  5. 将关键点映射到原始图像上。  

  逐步分析最终结果（分别假设网络的输入输出像素比 s 为 2:1 和 3:1）：  
  1. 原始图像：大小 18x18，关键点位置 x=9；  
  2. Resize，得到网络输入图像：大小 12x12，关键点位置 x=6；  
  3. flip，得到翻转图像：大小 12x12，关键点位置 x=5；  
  4. 推理，得到 heatmap：大小为 6x6，关键点位置 x=5/2；（或者 大小为 4x4，关键点位置 x=5/3）；  
  5. flip back，得到未翻转图像对应的heatmap：大小为 6x6，关键点位置 x=5/2；（或者 大小为 4x4，关键点位置 x=4/3）；  
  6. 映射到原始图像：关键点位置 x=15/2；或者（关键点位置 x=6）  

  首先分析 heatmap 上关键点的位置偏差，在没有 flip test 的情况下，即去掉步骤3和步骤5，可以得到，heatmap：大小为 6x6，关键点位置 x=3；（或者大小为 4x4，关键点位置 x=2），这种情况下heatmap 上关键点的位置与原始图像是对齐的。  
  按照理论推导，s=2 时，flip test heatmap上关键点偏差为 -1/2；s=3时，flip test 在heatmap 上引入的关键点偏差为 -2/3。观察步骤5的计算结果，偏差分别为 {5/2 vs. 3} 和 {4/3 vs. 2}，与理论推导一致。  
  继续分析在原始图像上关键点的位置偏差，根据论文给出的公式，s=2时，flip test 在原始图像上引入的关键点偏差为 ；s=3时，flip test 在原始图像上引入的关键点偏差为 。观察步骤6的计算结果，偏差分别为 {15/2 vs. 9} 和 {6 vs 9}，与理论推导一致。  

### 1.3 关键点格式转换偏差分析（Keypoint Format Transformation）  
  人体姿态估计方法中，一般我们无法直接得出网络输出heatmap上，关键点的精确坐标，现有的方法一般是，首先找到最大值坐标，然后找到最大值领域的次大值坐标，关键点坐标为最大值向次大值偏移 0.25 像素的结果。这个过程显然存在量化误差。  
  一般情况下，heatmap 与原图的比例为 1：4，heatmap上一个像素对应原图4个像素，通过上面的操作，输出结果的最小间隔由1像素变成了0.5像素，等价于heatmap上一个像素对应原图2个像素，因此可以消除一定的误差。  
  论文分析结果，现有方法在输出的heatmap上，产生关键点位置的误差分布服从 E=0.125，V=1/192。映射到原图后，误差与heatmap上的误差线性相关，系数为 $ \frac{bw_{s}}{W_{o}} $ （ $ bw_{s} $为 ROI 的宽像素数，$ W_{o} $为heatmap的宽像素数）  
  结合上一节 flip test 中的坐标系转换偏差分析，最终分析得到，坐标系转换和关键点格式转换两部分引入的偏差，分布服从 E=0.125，V=1/48。  

### 1.4 无偏数据处理  
  作者发现姿态估计中这两种误差后，分别提出了消除误差的无偏数据处理方法，推荐看原论文中的公式。  
#### 1.4.1 无偏坐标系统转换  
  坐标系转换的误差主要来源于，传统方法没有注意到连续数据和离散数据的表示区别，在原图坐标系、网络输入图像坐标系、heatmap图像坐标系三者之间的转换过程，图像并没有对齐。论文提出的方法是将传统方法中缩放时的因数，由之前的 像素数比 改成 像素长度比 ，也就是分子分母同时减1。（例如上面提到的例子中的第一步，关键点的坐标应当乘以 11/17，而不是 12/18）  
#### 1.4.2无偏关键点格式转换  
  关键点格式转换的误差主要来源于量化操作，作者提出的方法是，在现有的基础上，引入回归loss，回归关键点在heatmap附近的offset。  


## 2. Result
  `COCO val`

  | Method              | Backbone          | Input size    | IPS/PPS     | AP          | AP50  | AP75  | APM   | APL   | AR    |
  |-                    |-                  |-              |-            |-            |-      |-      |-      |-      |-      |
  | **Bottom-up methods**|
  | HigherHRNet [26]    | HRNet-W32         | 512x512   | 0.8         | 64.4        | -     | -     | 57.1  | 75.6  | -     |
  | +UDP                | HRNet-W32         | 512x512   | 4.9 (×6.1)  | 67.0 (+2.6) | 86.2  | 72.0  | 60.7  | 76.7  | 71.6  |
  | HigherHRNet [26]    | HigherHRNet-W32   | 512x512   | 1.1         | 67.1        | 86.2  | 73.0  | 61.5  | 76.1  | 718   |
  | +UDP                | HigherHRNet-W32   | 512x512   | 2.9 (×2.6)  | 67.8 (+0.7) | 86.2  | 72.9  | 62.2  | 76.4  | 72.4  |
  | HigherHRNet [26]†   | HRNet-W48         | 640x640   | 0.6         | 67.9        | 86.7  | 74.4  | 62.5  | 76.2  | 73.0  |
  | +UDP                | HRNet-W48         | 640x640   | 4.1 (×6.8)  | 68.9 (+1.0) | 87.3  | 74.9  | 64.1  | 76.1  | 73.5  |
  | HigherHRNet [26]    | HigherHRNet-W48   | 640x640   | 0.75        | 69.9        | 87.2  | 76.1  | 65.4  | 76.4  | -     |
  | +UDP                | HigherHRNet-W48   | 640x640   | 2.7 (×3.6)  | 69.9        | 87.3  | 76.2  | 65.9  | 76.2  | 74.4  |
  | **Bottom-up methods with multi-scale** ([×2, ×1,×0.5]) test as in HigherHRNet [26]|
  | UDP                 | HRNet-W32         | 512x512   | -           | 70.4        | 88.2  | 75.8  | 65.3  | 77.6  | 74.7  |
  | HigherHRNet [26]    | HigherHRNet-W32   | 512x512   | -           | 69.9        | 87.1  | 76.0  | 65.3  | 77.0  | -     |
  | +UDP                | HigherHRNet-W32   | 512x512   | -           | 70.2 (+0.3) | 88.1  | 76.2  | 65.4  | 77.4  | 74.5  |
  | HigherHRNet [26]†   | HRNet-W48         | 640x640   | -           | 71.6        | 88.6  | 77.9  | 67.5  | 77.8  | 76.3  |
  | +UDP                | HRNet-W48         | 640x640   | -           | 71.3 (-0.3) | 89.0  | 77.1  | 66.9  | 77.7  | 75.7  |
  | HigherHRNet [26]    | HigherHRNet-W48   | 640x640   | -           | 72.1        | 88.4  | 78.2  | 67.8  | 78.3  | -     |
  | +UDP                | HigherHRNet-W48   | 640x640   | -           | 71.5 (-0.6) | 88.3  | 77.3  | 67.9  | 77.2  | 75.9  |
  | **Top-down methods**|
  | Hourglass [40]      | Hourglass         | 256x192   | -           | 66.9        | -     | -     | -     | -     | -     |
  | CPN [27]            | ResNet-50         | 256x192   | -           | 69.4        | -     | -     | -     | -     | -     |
  | CPN [27]            | ResNet-50         | 384x288   | -           | 71.6        | -     | -     | -     | -     | -     |
  | MSPN [28]           | MSPN              | 256x192   | -           | 75.9        | -     | -     | -     | -     | -     |
  | SimpleBaseline [24] | ResNet-50         | 256x192   | 23.0        | 71.3        | 89.9  | 78.9  | 68.3  | 77.4  | 76.9  |
  | +UDP                | ResNet-50         | 256x192   | 23.0        | 72.9(+1.6)  | 90.0  | 80.2  | 69.7  | 79.3  | 78.2  |
  | SimpleBaseline [24] | ResNet-152        | 256x192   | 11.5        | 72.9        | 90.6  | 80.8  | 69.9  | 79.0  | 78.3  |
  | +UDP                | ResNet-152        | 256x192   | 11.5        | 74.3(+1.4)  | 90.9  | 81.6  | 71.2  | 80.6  | 79.6  |
  | SimpleBaseline [24] | ResNet-50         | 384x288   | 20.3        | 73.2        | 90.7  | 79.9  | 69.4  | 80.1  | 78.2  |
  | +UDP                | ResNet-50         | 384x288   | 20.3        | 74.0(+0.8)  | 90.3  | 80.0  | 70.2  | 81.0  | 79.0  |
  | SimpleBaseline [24] | ResNet-152        | 384x288   | 11.1        | 75.3        | 91.0  | 82.3  | 71.9  | 82.0  | 80.4  |
  | +UDP                | ResNet-152        | 384x288   | 11.1        | 76.2(+0.9)  | 90.8  | 83.0  | 72.8  | 82.9  | 81.2  |
  | HRNet [25]          | HRNet-W32         | 256x192   | 6.9         | 75.6        | 91.9  | 83.0  | 72.2  | 81.6  | 80.5  |
  | +UDP                | HRNet-W32         | 256x192   | 6.9         | 76.8(+1.2)  | 91.9  | 83.7  | 73.1  | 83.3  | 81.6  |
  | HRNet [25]          | HRNet-W48         | 256x192   | 6.3         | 75.9        | 91.9  | 83.5  | 72.6  | 82.1  | 80.9  | 
  | +UDP                | HRNet-W48         | 256x192   | 6.3         | 77.2(+1.3)  | 91.8  | 83.7  | 73.8  | 83.7  | 82.0  |
  | HRNet [25]          | HRNet-W32         | 384x288   | 6.2         | 76.7        | 91.9  | 83.6  | 73.2  | 83.2  | 81.6  | 
  | +UDP                | HRNet-W32         | 384x288   | 6.2         | 77.8(+1.1)  | 91.7  | 84.5  | 74.2  | 84.3  | 82.4  |
  | HRNet [25]          | HRNet-W48         | 384x288   | 5.3         | 77.1        | 91.8  | 83.8  | 73.5  | 83.5  | 81.8  | 
  | +UDP                | HRNet-W48         | 384x288   | 5.3         | 77.8(+0.7)  | 92.0  | 84.3  | 74.2  | 84.5  | 82.5  |

  `COCO test-dev`

  | Method                          | Backbone              | Input size    | AP            | AP50  | AP75  | APM   | APL   | AR    |
  | -                               | -                     | -             | -             | -     | -     | -     | -     | -     |
  | **Bottom-up methods**|
  | AE [49]                         | Hourglass [40]        | 512x512   | 56.6          | 81.8  | 61.8  | 49.8  | 67.0  |-      |
  | G-RMI [47]                      | ResNet-101            | 353x257   | 64.9          | 85.5  | 71.3  | 62.3  | 70.0  |69.7   |
  | PersonLab [51]                  | ResNet-152            | 1401x140  | 66.5          | 88.0  | 72.6  | 62.4  | 72.3  |-      |
  | PifPaf [61]                     | -                     | -             | 66.7          | -     | -     | -     | -     | -     |
  | HigherHRNet [26]                | HRNet-W32             | 512x512   | 64.1          | 86.3  | 70.4  | 57.4  | 73.9  | -     |
  | +UDP                            | HRNet-W32             | 512x512   | 66.8 (+2.7)   | 88.2  | 73.0  | 61.1  | 75.0  | 71.5  |
  | HigherHRNet [26]                | HigherHRNet-W32       | 512x512   | 66.4          | 87.5  | 72.8  | 61.2  | 74.2  | -     |
  | +UDP                            | HigherHRNet-W32       | 512x512   | 67.2 (+0.8)   | 88.1  | 73.6  | 62.0  | 74.3  | 72.0  |
  | HigherHRNet [26]†               | HRNet-W48             | 640x640   | 67.4          | 88.6  | 74.2  | 62.6  | 74.3  | 72.8  |
  | +UDP                            | HRNet-W48             | 640x640   | 68.1 (+0.2)   | 88.3  | 74.6  | 63.9  | 74.1  | 73.1  |
  | HigherHRNet [26]                | HigherHRNet-W48       | 640x640   | 68.4          | 88.2  | 75.1  | 64.4  | 74.2  | -     |
  | +UDP                            | HigherHRNet-W48       | 640x640   | 68.6 (+0.2)   | 88.2  | 75.5  | 65.0  | 74.0  | 73.5  |
  | **Bottom-up methods with multi-scale** ([×2, ×1,×0.5]) test as in HigherHRNet [26]|
  | UDP                             | HRNet-W32             | 512x512   | 69.3          | 89.2  | 76.0  | 64.8  | 76.0  | 74.1  |
  | HigherHRNet [26]†               | HRNet-W32             | 512x512   | 68.8          | 88.8  | 75.7  | 64.4  | 75.0  | 73.5  |
  | UDP                             | HigherHRNet-W32       | 512x512   | 69.1          | 89.1  | 75.8  | 64.4  | 75.5  | 73.8  |
  | HigherHRNet [26]†               | HRNet-W48             | 640x640   | 70.4          | 89.7  | 77.4  | 66.4  | 75.7  | 75.2  |
  | +UDP                            | HRNet-W48             | 640x640   | 70.3          | 90.1  | 76.7  | 66.6  | 75.3  | 75.1  |
  | HigherHRNet [26]                | HigherHRNet-W48       | 640x640   | 70.5          | 89.3  | 77.2  | 66.6  | 75.8  | -     |
  | +UDP                            | HigherHRNet-W48       | 640x640   | 70.5          | 89.4  | 77.0  | 66.8  | 75.4  | 75.1  |
  | **Top-down methods**            |
  | Mask-RCNN [53]                  | ResNet-50-FPN [62]    | -             | 63.1          | 87.3  | 68.7  | 57.8  | 71.4 | -      |
  | Integral Pose Regression [63]   | ResNet-101 [64]       | 256x256   | 67.8          | 88.2  | 74.8  | 63.9  | 74.0 | -      |
  | SCN [56]                        | Hourglass [40]        | -             | 70.5          | 88.0  | 76.9  | 66.0  | 77.0 | -      |
  | CPN [27]                        | ResNet-Inception      | 384x288   | 72.1          | 91.4  | 80.0  | 68.7  | 77.2 | 78.5   |
  | RMPE [65]                       | PyraNet [66]          | 320x256   | 72.3          | 89.2  | 79.1  | 68.0  | 78.6 | -      |
  | CFN [67]                        | -                     | -             | 72.6          | 86.1  | 69.7  | 78.3  | 64.1 | -      |
  | CPN(ensemble) [27]              | ResNet-Inception      | 384x288   | 73.0          | 91.7  | 80.9  | 69.5  | 78.1 | 79.0   |
  | Posefix [68]                    | ResNet-152            | 384x288   | 73.6          | 90.8  | 81.0  | 70.3  | 79.8 | 79.0   |
  | CSANet [69]                     | ResNet-152            | 384x288   | 74.5          | 91.7  | 82.1  | 71.2  | 80.2 | 80.7   |
  | MSPN [28]                       | MSPN [28]             | 384x288   | 76.1          | 93.4  | 83.8  | 72.3  | 81.5 | 81.6   |
  | SimpleBaseline [27]             | ResNet-50             | 256x192   | 70.2          | 90.9  | 78.3  | 67.1  | 75.9 | 75.8   |
  | +UDP                            | ResNet-50             | 256x192   | 71.7 (+1.5)   | 91.1  | 79.6  | 68.6  | 77.5 | 77.2   |
  | SimpleBaseline [27]             | ResNet-50             | 384x288   | 71.3          | 91.0  | 78.5  | 67.3  | 77.9 | 76.6   |
  | +UDP                            | ResNet-50             | 384x288   | 72.5 (+1.2)   | 91.1  | 79.7  | 68.8  | 79.1 | 77.9   |
  | SimpleBaseline [27]             | ResNet-152            | 256x192   | 71.9          | 91.4  | 80.1  | 68.9  | 77.4 | 77.5   |
  | +UDP                            | ResNet-152            | 256x192   | 72.9 (+1.0)   | 91.6  | 80.9  | 70.0  | 78.5 | 78.4   |
  | SimpleBaseline [27]             | ResNet-152            | 384x288   | 73.8          | 91.7  | 81.2  | 70.3  | 80.0 | 79.1   |
  | +UDP                            | ResNet-152            | 384x288   | 74.7 (+0.9)   | 91.8  | 82.1  | 71.5  | 80.8 | 80.0   |
  | HRNet [25]                      | HRNet-W32             | 256x192   | 73.5          | 92.2  | 82.0  | 70.4  | 79.0 | 79.0   |
  | +UDP                            | HRNet-W32             | 256x192   | 75.2 (+1.7)   | 92.4  | 82.9  | 72.0  | 80.8 | 80.4   |
  | HRNet [25]                      | HRNet-W32             | 384x288   | 74.9          | 92.5  | 82.8  | 71.3  | 80.9 | 80.1   |
  | +UDP                            | HRNet-W32             | 384x288   | 76.1 (+1.2)   | 92.5  | 83.5  | 72.8  | 82.0 | 81.3   |
  | HRNet [25]                      | HRNet-W48             | 256x192   | 74.3          | 92.4  | 82.6  | 71.2  | 79.6 | 79.7   |
  | +UDP                            | HRNet-W48             | 256x192   | 75.7 (+1.4)   | 92.4  | 83.3  | 72.5  | 81.4 | 80.9   |
  | HRNet [25]                      | HRNet-W48             | 384x288   | 75.5          | 92.5  | 83.3  | 71.9  | 81.5 | 80.5   |
  | +UDP                            | HRNet-W48             | 384x288   | 76.5 (+1.0)   | 92.7  | 84.0  | 73.0  | 82.4 | 81.6   |