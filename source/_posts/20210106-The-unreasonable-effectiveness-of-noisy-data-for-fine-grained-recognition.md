---
title: The unreasonable effectiveness of noisy data for fine-grained recognition
date: 2021-01-06 13:46:06
tags: 
  - deep learning
  - FGVC
categories:
  - Paper notes
  - FGVC
---

## [The unreasonable effectiveness of noisy data for fine-grained recognition](http://cn.arxiv.org/pdf/1511.06789.pdf)  
>- **ECCV 2016**
>- 这篇文章提出使用网络爬取的数据进行细粒度分类任务。与其说是提出一种新的方法，不如说是新建了一个数据集。  
>- 网络爬取的图片并不完全准确，因此讨论了对这些噪声的处理方法。  
>- 加入大量网络数据后，训练的效果得到了大幅度的提升，目前在 CUB 数据集上准确率达到了 92.3  

 <!-- more -->

  - cross-domain noise:  
  `cross-domain noise` 指的是不属于这一大类的图片，例如搜索某种鸟，出来的结果是昆虫的图片。通过人工标注量化了这种噪声，发现这种情况比较少，并且当图片数目增多的时候，噪声占的比例也会减少。  
  这种噪声对结果的影响也比较小。
  - cross-category noise:  
  `cross-category noise` 指的是将其他子类的图片混入搜索结果的情况。这种噪声的比例难以量化，并且对结果的影响比较大，作者将搜索结果中重复的图片去除掉来减少这些噪声。
  - active learning：  
  作者还提出两种标注方法辅助去除噪声。1）使用预训练好的模型，挑选搜索结果中置信度高的结果；2）人工筛选搜索结果。
  - 实验结果：
  
    |     | CUB  | web-raw | web-filter | L-bird | L-bird(MC) | L-bird+CUB | L-bird+CUB(MC) |
    |-    |-     |-        |-           |-       |-           |-           |-               |
    | acc | 84.4 | 87.7    | 89.0       | 91.9   | 92.3       | 92.2       | 92.8           |
  
        `CUB`：CUB 数据集  
        `web-raw`：web 爬取数据集  
        `web-filter`： 去除 `cross-category noise`  
        `L-Bird`: 爬取所有鸟类的图片，进行预训练，然后在 web-filter 上 finetune  
        `MC`：测试的时候使用 multi-crop