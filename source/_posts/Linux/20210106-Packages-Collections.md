---
title: Packages Collections
date: 2021-01-06 19:59:23
tags:
  - Linux
categories:
  - Linux
---

收集一些比较冷门的 lib, packages 安装命令

<!-- more -->

## Ubuntu:
```
libgthread-2.0.so.0         apt install libglib2.0-0
libSM.so.6                  apt install libsm6
libXrender.so.1             apt install libxrender1
libXext.so.6                apt install libxext6
```

## Python3
```
pycocotools                 pip install "git+https://github.com/philferriere/cocoapi.git#egg=pycocotools&subdirectory=PythonAPI"
skbuild                     pip install scikit-build
```

## Caffe
```
apt install -y \
    protobuf-compiler \
    libprotobuf-dev \
    libboost-all-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    libatlas-base-dev \
    libopencv-dev \
    libhdf5-dev \
    libleveldb-dev \
    liblmdb-dev \
    libsnappy-dev \
    python-numpy \
    python-setuptools \
    python-opencv \
    ipython \

pip install scikit-image protobuf
```
