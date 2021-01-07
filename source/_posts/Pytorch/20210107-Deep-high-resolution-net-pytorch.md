---
title: Deep-high-resolution-net.pytorch
date: 2021-01-07 09:15:54
tags:
  - deep learning
  - Pytorch
  - Pose Estimation
categories:
  - Pytorch
---

https://github.com/leoxiaobin/deep-high-resolution-net.pytorch

<!-- more -->

## 1. 配置 HRNET 环境
```
FROM nvidia/cuda:11.1-cudnn8-devel-ubuntu18.04

LABEL auther=clover978

# Basic toolchain
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        git \
        ffmpeg \
        python3-pip \
        python3-dev \
        libsm6 \
        libxext6 \
    && cd /usr/bin \
    && ln -s /usr/bin/python3 python \
    && pip3 install --upgrade pip \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* 

# clone deep-high-resolution-net
ARG POSE_ROOT=/workspace/deep-high-resolution-net
RUN git clone https://gitee.com/clover978/deep-high-resolution-net.pytorch.git $POSE_ROOT
WORKDIR $POSE_ROOT

RUN pip3 install --no-cache-dir -r requirements.txt && \
    pip3 install --no-cache-dir pycocotools numpy opencv-python tqdm tensorboard tensorboardX pyyaml webcolors \
    pip3 install --no-cache-dir torch==1.7.0+cu110 torchvision==0.8.1+cu110 -f https://download.pytorch.org/whl/torch_stable.html 

# build deep-high-resolution-net lib
WORKDIR $POSE_ROOT/lib
RUN make

# install COCO API
ARG COCOAPI=/cocoapi
RUN git clone https://gitee.com/clover978/cocoapi $COCOAPI
WORKDIR $COCOAPI/PythonAPI
RUN make install

# clone 
ARG DET_ROOT=/workspace/Yet-Another-EfficientDet-Pytorch
RUN git clone https://gitee.com/clover978/Yet-Another-EfficientDet-Pytorch $DET_ROOT

WORKDIR $POSE_ROOT
```

> clover fork 的 HRNET(https://gitee.com/clover978/deep-high-resolution-net.pytorch.git) 进行了一些改动：
>  - 支持 QDTM 数据集，数据集包含 18 个关键点，不同于 COCO 的 17 个点；
>  - 添加代码保存中间模型，每 10 个 epoch 保存一次；
>  - 添加 inference 代码，可以对图片文件夹进行遍历，并将结果生成视频。

> clover fork 的 cocoapi(https://gitee.com/clover978/cocoapi) 进行了一个改动：
>  - 更改 PythonAPI/pycocotools/cocoeval.py:523，以适应 QDTM 数据集的 18 个关键点。 


## 2. 训练、测试
```
docker run -itd --name hrnet --ipc host -v /home/dxx:/home/dxx -v /data:/data --net host dxx/hrnet:v1

$ deep-high-resolution-net.pytorch
python tools/train.py --cfg experiments/qdtm/hrnet/w48_384x288_adam_lr1e-3_vault_v3.yaml 
python tools/inference.py --cfg experiments/qdtm/hrnet/w48_384x288_adam_lr1e-3_vault_v3.yaml TEST.MODEL_FILE output/output/qdtm/pose_hrnet/w48_384x288_adam_lr1e-3_vault_v3/model_best.pth
```
