---
title: TSN-Pytorch
date: 2021-01-07 09:23:06
tags:
  - deep learning
  - Pytorch
  - Action Recgnition
categories:
  - Pytorch
---

<https://github.com/yjxiong/tsn-pytorch>

---
### 0. Intro
TSN 是动作识别领域里面一个很老工作，也是一个很重要的工作。作者开源了代码，是用 Caffe 实现的，但是由于 Pytorch 在学界的流行，作者又重新实现了一个 Pytorch 版本的。代码写的很优雅，基本是可以直接用的，这个简单记录一下使用步骤。  
**TSN 的训练模式有 3 种, (1) RGB, (2) Flow, (3) RGB+Flow, 这里只介绍第一种**  
**下文需要用到的代码片段都已经上传到 fork 的 repo 中 <https://github.com/clover978/tsn-pytorch>**

<!-- more -->

---
### 1. Data-Prepare
  - 抽帧：  
    首先生成视频文件列表：  
    ```
    cd /home/dxx/UCF101/UCF101/
    find . -type f > trainvallist.txt
    ```
    然后抽帧：
    ```
    cd <tsn-path>
    python tools/extract_frames.py
    ```
  - 生成 lst 文件：  
    `tsn-pytorch` 用文本文件的形式作为输入，读取数据集。文本文件的格式如下：
    ```text
    # 视频帧路径  视频总帧数  视频标签
    <image_dir_path> <frames_cnt> <label_index>
    ```
    准备 `classInd.txt` 文件，用下面的脚本生成训练需要的文本文件：
    ```
    python gen_tsn_list.py
    ```
    
---
### 2. Train
  - ucf101 训练：  
    UCF101数据集 可以直接使用如下的命令训练：  
    ```
    # tools/train_tsn.sh
    
    CUDA_VISIBLE_DEVICES=0,1,2,3 \
            python -u main.py ucf101 RGB train_tsn.txt val_tsn.txt \
            --arch BNInception \
            --num_segments 5 --gd 20 \
            --lr 0.001 --lr_steps 70 130 --epochs 180 \
            -b 140 -j 8 --dropout 0.5 --snapshot_pref models/meitu_bninception \
            |& tee log.txt
    ```
  - 自定义数据集训练：  
    自定义的数据集，可以使用同样的命令训练，训练之前只需要修改 `main.py`, 将 num_class 替换为自己数据集的类别数
    ```
    # main.py:25
    if args.dataset == 'ucf101':
        num_class = <datasets classes number>
    ```
  - 使用 resume 参数从中间开始训练
    ```
    --resume models/tsn-pytorch/meitu_bninception_rgb_1_checkpoint.pth.tar
    ```
