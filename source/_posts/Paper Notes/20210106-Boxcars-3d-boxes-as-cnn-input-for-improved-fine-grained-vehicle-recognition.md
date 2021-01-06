---
title: 'Boxcars: 3d boxes as cnn input for improved fine-grained vehicle recognition'
date: 2021-01-06 14:10:48
tags: 
  - deep learning
  - FGVC
categories:
  - Paper notes
  - FGVC
---

## [Boxcars: 3d boxes as cnn input for improved fine-grained vehicle recognition](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Sochor_BoxCars_3D_Boxes_CVPR_2016_paper.pdf)
>- **CVPR 2016**
>- 这篇文章提出了一种使用车辆 3D box 作为网络输入，来识别车型的方法。首先检测出车辆的 3D包围框，然后展开成 2D 图形，最后加入视角信息作为额外输入进行训练。同时还发布了一个新的数据集 `BoxCars`

<!-- more -->

  - 3D 包围框：  
  文中没有介绍获得车辆的 3D 包围框 的方法。这是作者之前的一个工作。[Automatic camera  calibration for traffic understanding](http://www.bmva.org/bmvc/2014/files/paper013.pdf)
  - 车辆图片平展：  
  获得 3D 包围框后，将其展开成 2D 图像，具体方法类似于将立体的纸盒子在平面上铺开。很简单的一个思路。
  - 车辆的视角信息：  
  作者用三个向量来对车辆的视角信息进行编码。分别是 `车尾-车头向量、车内-车外向量，车底-车顶向量`。通过这三个向量，可以完整表示出车辆的视角。
  - 栅格化包围框：  
  作者同时提出另外一种方法描述车辆的视角。将车辆的矩形包围框中用四种颜色进行表示。其中 车顶用黄色，车侧用红色，车头/车尾同蓝色，其他位置用白色。这样一幅新的栅格化图像同样可以确定出车辆的视角。  
  - 平铺图 + 视角辅助信息 训练：  
  平铺图直接进入 CNN 进行卷积操作；  
  视角编码通过 6x6 的矩阵表示，使用三向量表示的情况下，矩阵的第一行表示 3 个二维向量，其他行用 0 填充；使用栅格化包围框表示的情况下，直接将 bbox rescale 成 6x6 的矩阵。  
  文中只说 view encoding 是加在卷积操作之后，但是具体怎么加没有说明。
  - Bbox Cars 数据集：  
  作者同时还公布了一个新的数据集，数据集中的图片是从道路车辆监控视频中获取的，主要特点就是包含了车辆的 3D box 信息。
  - 实验结果：  
  作者提出的方法需要数据集标注了车辆的 3D box 信息，具体怎么在 CompCars 数据集上做的实验阐述的不太清楚，下面是实验结果：
  
      |          | top1  | top5  |
      |-         |-      |-      |
      | baseline | 0.767 | 0.917 |
      | ours     | 0.848 | 0.954 |