---
title: 使用 Python 接口制作 lmdb
date: 2021-01-07 10:33:48
tags:
  - Caffe
  - LMDB
  - Python
categories:
  - Caffe
---

# 使用 lmdb 的 Python 接口制作数据集

## Caffe 输入格式：
一般来说，使用 Caffe 训练的过程，网络结构的第一层是都用来处理输入数据的，输入数据的形式大多数为图片（NLP 工作中，输入的一般是文本的特征，不过这种情况不太常见）。Caffe 的数据输入层有以下几种：
<!-- more -->

```shell
root@p3d-container:/bigdata/dxx/pseudo-3d-residual-networks/caffe/src/caffe/layers# ll *data_layer.cpp
-rw-r--r-- 1 root root  4118 Mar 22 09:25 base_data_layer.cpp
-rw-r--r-- 1 root root  4313 Mar 22 09:25 data_layer.cpp
-rw-r--r-- 1 root root  4826 Mar 22 09:25 dummy_data_layer.cpp
-rw-r--r-- 1 root root  6136 Mar 22 09:25 hdf5_data_layer.cpp
-rw-r--r-- 1 root root  6946 Mar 22 09:25 image_data_layer.cpp
-rw-r--r-- 1 root root  4414 Mar 22 09:25 memory_data_layer.cpp
-rw-r--r-- 1 root root 17544 Mar 22 09:25 window_data_layer.cpp

################################################################
## 其中，base_data_layer.cpp 定义了数据层的基类；
## data_layer.cpp 支持 **lmdb/leveldb** 格式的输入
## hdf5_data_layer.cpp 支持 **hdf5** 格式的输入
## image_data_layer.cpp 支持 **图片** 直接输入
## 其他的三个 layer 我不太了解，感兴趣可以查阅相关资料
################################################################
```
我们平时最常见的情况就是使用 lmdb/leveldb 作为输入，Caffe 的 mnist example 就是使用的这两个格式。因为这样比直接使用 image 进行输入要更快一些（原因可能是因为 lmdb/leveldb 支持 prefetch 操作，可以节省 IO 时间），其中 lmdb 比 leveldb 更快，体积更小。  
目前 Caffe 最常见的输入就是 lmdb 格式。那么怎么制作 lmdb 呢？ Caffe 提供了一个 convert_imageset 程序，按照指定的格式整理好 image 和 label，直接调用命令可以将图片转换为 lmdb。具体操作参考官网，这里不详细介绍。


## lmdb 的 Python 接口
简单的情况下，我们可以直接利用官方提供的工具，将图片数据转换为 lmdb，然后利用 data_layer 的 transform_param 对数据进行简单的预处理或者数据增强操作。但是某些特殊情况下，transform_param 可能无法实现我们想要的预处理。这种情况下，我们可以修改 data_layer 的源码，实现想要的功能，当然这样就比较麻烦了，更为简单的方法就是，我们可以在制作 lmdb 之前先实现想要的预处理操作，然后将图片制作成 lmdb，这就是下面要介绍的内容。同理，如果想用这种方法实现比较复杂的数据增强功能也是可以的，不过，这种情况下，lmdb 占用的空间会相应的增加。  
我这次的任务是让 Caffe 实现视频输入功能，因此需要将 16 个连续视频帧 stack 起来，制作成一个 clip，后续的网络会对 clip 进行卷积操作。
```
import lmdb
import numpy as np
import cv2
import caffe
from caffe.proto import caffe_pb2

#basic setting
# 这个设置用来存放lmdb数据的目录
lmdb_file = 'lmdb_data'
batch_size = 256

# create the lmdb file
# map_size指的是数据库的最大容量，根据需求设置
lmdb_env = lmdb.open(lmdb_file, map_size=int(1e12))
lmdb_txn = lmdb_env.begin(write=True)
# 因为caffe中经常采用datum这种数据结构存储数据
datum = caffe_pb2.Datum()

item_id = -1
for x in range(1000):
    item_id += 1

    #prepare the data and label
    
    #data = np.ones((3,64,64), np.uint8) * (item_id%128 + 64) #CxHxW array, uint8 or float
    # pic_path设置成图像目录, 0表示读入灰度图
    data = cv2.imread(pic_path, 0)
    # label 设置图像的label就行
    label = item_id%128 + 64

    # save in datum
    datum = caffe.io.array_to_datum(data, label)
    keystr = '{:0>8d}'.format(item_id)
    lmdb_txn.put( keystr, datum.SerializeToString() )

    # write batch
    if(item_id + 1) % batch_size == 0:
        lmdb_txn.commit()
        lmdb_txn = lmdb_env.begin(write=True)
        print (item_id + 1)

# write last batch
if (item_id+1) % batch_size != 0:
    lmdb_txn.commit()
    print 'last batch'
    print (item_id + 1)
```
