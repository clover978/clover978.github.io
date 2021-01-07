---
title: A Large-Scale Car Dataset for Fine-grained Categotozation and Verification
date: 2021-01-06 14:10:34
tags: 
  - deep learning
  - FGVC
categories:
  - Paper Notes
  - FGVC
---

## [A Large-Scale Car Dataset for Fine-grained Categotozation and Verification](http://arxiv.org/abs/1506.08959)  
>- **CVPR 2015**
>- 这篇文章提出了一个大型的数据集 CompCars，里面包含了从网络上收集的 13.6w 张整车图片， 2.7w 张车辆局部图片，还有从监控场景下拍摄的 5k 张车辆正脸图片。  
>- 实验部分使用这个数据集分别用于 **车辆细粒度分类、车辆属性所识别、车辆验证** 三个任务。

<!-- more -->

  - CompCars 数据集：
    - 品牌（Make）：163
    - 车型（Model）：1716
    - 年份（Year）：4455
    - 整车图片数：136,727
      + F ：18431
      + R ：13513
      + S ：23551
      + FS：49301
      + RS：31150
    - 局部图片数：27,618
      + outter part: `headlight, rearlight, fog light, air intake`
      + inter part: `console, steering wheel, dashboard, gear lever`
  - **Fine-grained classification 数据集：**
    - 车型： 431  

        |               | Train | Test | Total |
        |-              |-      |-     |-      |
        | 图片数（整车）| 16016 | 14939| 30955 |
  - **Baseline：**  
    + model: overfeat  
    + dataset: Fine-grained classification  
    + result:  

        | view | F     | R     | S     | FS    | RS    | all   |
        |-     |-      |-      |-      |-      |-      |-      |
        | top1 | 0.524 | 0.431 | 0.428 | 0.563 | 0.598 | 0.767 |
        | top5 | 0.748 | 0.647 | 0.602 | 0.769 | 0.777 | 0.917 |
        | Make | 0.701 | 0.521 | 0.507 | 0.680 | 0.656 | 0.829 |
      单视角识别车型： `RS` 效果最好；  
      单视角识别品牌： `F ` 效果最好；  
      多视角 Baseline： **`model: 0.767,  make: 0.829`**
  - Supplementary experiment:   
    在整个 `CompCars` 数据集上补充一个实验，比较三种模型的效果

        | model | AlexNet | Overfeat | GoogLeNet |
        |-      |-        |-         |-          |
        | top1  | 0.819   | 0.879    | 0.912     |
        | top5  | 0.949   | 0.969    | 0.981     |
         