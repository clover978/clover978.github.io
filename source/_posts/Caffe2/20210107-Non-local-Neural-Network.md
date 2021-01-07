---
title: Non-local Neural Network
date: 2021-01-07 09:47:01
tags:
  - deep learning
  - Caffe2
  - Action Recgnition
categories:
  - Caffe2
---

Facebook 的 Non-local 开源了，上周简单试验了一下怎么用。记录一下步骤  
<https://github.com/facebookresearch/video-nonlocal-net>

<!-- more -->

---
## Caffe2 Install
<https://github.com/facebookresearch/video-nonlocal-net/blob/master/INSTALL.md>
  - 依赖库
    ```bash
    git clone --recrusive http://github.com/caffe2/caffe2.git

    apt-get update
    apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        git \
        libgoogle-glog-dev \
        libgtest-dev \
        libiomp-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libopenmpi-dev \
        libsnappy-dev \
        libprotobuf-dev \
        openmpi-bin \
        openmpi-doc \
        protobuf-compiler \
        python-dev \
        python-pip                          
    pip install \
        future \
        numpy \
        protobuf
    ```

  - 中间可能需要安装 `setuptools`
    ```bash
    apt install wget
    wget http://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz
    tar -xvf setuptools-0.6c11.tar.gz
    cd setuptools-0.6c11
    python setup.py build
    python setup.py install
    ```

  - Clone Non-local 代码
    ```bash
    cd caffe2
    git clone --recursive https://github.com/facebookresearch/video-nonlocal-net.git
    rm -rf caffe2/video/
    cp -r video-nonlocal-net/caffe2_customized_ops/video/ caffe2/video/
    ```

  - Make  
    ```bash
    # use_ffmpeg ON
    make -j
    python caffe2_setup.py install      # (not sure how it works)
    ```

  - 配置环境变量
    ```bash
    export PYTHONPATH=<caffe2-dir>/video-nonlocal-net/lib:$PYTHONPATH
    ```

---
## Data Preparation 
<https://github.com/facebookresearch/video-nonlocal-net/blob/master/DATASET.md>

我是用 UCF-101 数据集进行 finetune 的，主要参照了官方的链接制作 LMDB

  - 图片准备  
  首先下载解压 UCF-101 数据集到 `/data` 目录下
    ```text
    root@dxx# ls /data/
    UCF-101/  ucfTrainTestlist/
    ```

  - 生成 list 文件  
  官方文档里面使用 gen_py_list.py 生成 trainlist.txt  
  这里我们直接用 ucf101 提供的 trainlist01.txt  
  我们还需要做另外两个工作 （1）ucf101 的标签是从 1 开始，需要改成 0； （2）ucf101 中的 testlist01.txt 里面没有标签信息，需要自己加上
    ```bash
    cd <caffe2-dir>/video-nonlocal-net/process_data/
    cp -r kinetics/ ucf101/
    cd ucf101/

    rm -f *.txt
    cp /data/ucfTrainTestlist/classInd.txt .
    cp /data/ucfTrainTestlist/trainlist01.txt .
    cp /data/ucfTrainTestlist/testlist01.txt .


    python gen_py_list_football.py
    '''
    >>>python
    d = {}
    for line in open('classInd.txt'):
        ind, label = line.strip().split()
        d[label] = ind
        
    with open('trainlist.txt', 'w') as f:
        for line in open('trainlist01.txt'):
            img, ind = line.strip().split()
            if ind == '101':
                ind = '0'
            f.write('/data/UCF-101/' + img + ' ' + ind + '\n')

    with open('vallist.txt', 'w') as f:
        for line in open('testlist01.txt'):
            img = line.strip()
            label = img.split('/')[0]
            ind = d[label]
            if ind == '101':
                ind = '0'
            f.write('/data/UCF-101/' + img + ' ' + ind + '\n')
    '''
    ```

  - resize 数据集  
  为了加快训练时候的 IO 速度，我们可以提前把数据集 resize 好，官方直接写好了一个脚本
    ```bash
    # 修改 downscale_video_joblib.py:21,22
    YOUR_DATASET_FOLDER/train/  --> /data/UCF-101/
    YOUR_DATASET_FOLDER/train_256/  --> /data/UCF-101_s256/      # 注意不能少了路径最后的 '/'

    pip install joblib pandas
    mkdir -p /data/UCF-101_s256/
    python downscale_video_joblib.py
    # 修改 downscale_video_joblib.py:20
    trainlist.txt  -->  vallist.txt
    python downscale_video_joblib.py
    ```

  - 生成 LMDB
    ```bash
    sed -i 's/UCF-101/UCF-101_s256/g' trainlist.txt
    sed -i 's/UCF-101/UCF-101_s256/g' vallist.txt
    python shuffle_list_rep.py trainlist_shuffle_rep.txt

    pip install lmdb
    bash run_createdb.sh
    bash run_createdb_test.sh
    bash run_createdb_test_multicrop.sh
    ```

---
## Train
<https://github.com/facebookresearch/video-nonlocal-net/blob/master/README.md>

训练过程可以有很多种设置，这里我选择的是 `run_i3d_nlnet_400k.sh` 这个脚本，具体区别看官方说明。
```
cd <caffe2-dir>/video-nonlocal-net/scripts/
pip install networkx
apt install python-yaml

# 设置一下 yaml 文件:  ../configs/DBG_kinetics_resnet_8gpu_c2d_nonlocal_400k.yaml
# GPU_NUMS
# MODEL.NUM_CLASSES
# TRAIN.BATCH_SIZE
# TEST.BATCH_SIZE
# TEST.DATASET_SIZE

#  修改 solver 超参数


# 修改  run_i3d_nlnet_400k.sh
# 6: TRAIN.PARAMS_FILE ../data/pretrained_model/i3d_nonlocal_8x8_IN_pretrain_400k_clean.pkl \
# 14: FILENAME_GT ../process_data/ucf101/vallist.txt \
#
#
chmod 777 run_i3d_nlnet_400k.sh
./run_i3d_nlnet_400k.sh
```

> 按照上面的步骤基本可以开始训练 ucf101了，这里的预训练模型有两种选择，一种是在 Kinetics 上训练时选择的 ImageNet 预训练模型，脚本里面名字是 [`../data/pretrained_model/r50_pretrain_c2_model_iter450450_clean.pkl`](https://s3.amazonaws.com/video-nonlocal/pretrained_model.tar.gz); 一种是在 Kinetics 上训练好的模型 [`i3d_nonlocal_8x8_IN_pretrain_400k`](https://s3.amazonaws.com/video-nonlocal/i3d_nonlocal_8x8_IN_pretrain_400k.pkl)。  
我选择的是后者，但是刚开始训练的时候就会认为已经进行到了 400k 次迭代，所以直接结束了训练。我最后在 `../tools/train_net_video.py:131` 加上 `start_model_iter = 0`训练了起来（小伙伴可以尝试其他更好的解决办法）。一旦网络保存过一次 checkpoint，下次训练就会从 checkpoint 里选择最新的模型继续训练，这个时候就可以删掉 `start_model_iter = 0` 了。

---
## Test
视频分类这一块的模型测试，通常都会有两层意思，  
一个是对测试视频进行上述的数据准备操作，对于 C3D (caffe版) 来说，是生成 lst 文件；对于 Non-local I3D (caffe2版) 来说，是生成 lmdb 文件，这两个文件的作用都是定义测试 clip 在 video/image-squence 中对应的帧号，网络读取数据准备的结果文件后，会组织数据送入后面的卷积层。（当然，数据准备过程只与输入层的实现相关，与网络结构没有关系），这一部分工作与训练几乎没有区别，并且可以方便进行测试，因此很容易跑通。  
另一个意思就是对视频输入进行分类测试，也就是落地部署，一般的开源项目只为了让人复现实验结果，并不会实现这一部分。视频输入 与 文本/lmdb 输入具有不小的 gap，为了能够直接在视频上测试，需要对网络的输入模块进行改动，下面是 caffe2 版 Non-local I3D 部署的步骤。
  - Input layer modification  
    <https://github.com/clover978/video-nonlocal-net/blob/master/tools/deploy_net_video_local.py>
  - Image propocess  
    `caffe2_customized_ops/video/customized_video_input_op.h:200-221` summarize 了 Non-local I3D 的 video_input_op 对视频帧进行的预处理过程。更改后的网络结构需要实现对应的功能。
    ```Python
    # customized_video_input_op.h:206,  scale
    def _scale(frame, size=(320, 256)):
        return cv2.resize(frame, size)
    
    # customized_video_input_op.h:208,210,  crop
    def _crop(frame, crop_size=224, type='random'):
        # type: {'random', 'center'}
        h, w  = frame.shape[:2]
        if type == 'random':
            x = random.randint(crop_size//2, w-crop_size//2-1)
            y = random.randint(crop_size//2, h-crop_size//2-1)
        elif type == 'center':
            x = w//2
            y = h//2
        else:
            logger.warning('unknow type, use center crop instead')
            x = w//2
            y = h//2
        return frame[y-crop_size//2:y+crop_size//2, x-crop_size//2:x+crop_size//2, :]
    
    # customized_video_input_op.h:211,212,  sampling
    # @sa: deploy_net_video_local:collect_clip()
    
    # customized_video_input_op.h:213,  normalize
    def _normalize(clip, mean=128, std=1):
        # just do it over clip
        return (clip-mean)/std
        
    # customized_video_input_op.h:219,   channel_swap
    def _channel_swap(frame, use_bgr=True):
        if use_bgr:
            return cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        else:
            return frame

    ```
  

---
## Q&A
1. 训练过程中，会周期性地进行 test，我在 test 的时候报错
    ```
    Traceback:
        File "lib/utils/misc.py:200":
            used_gpu_memory = out_dict['Used GPU Memory']
    KeyError: Used GPU Memory
    ```
    查了半天不知道正确的 key 是什么，直接注释掉了 `lib/utils/metrics.py:343` 
    ```
        # json_stats['used_gpu_memory'] = misc.get_gpu_stats()
    ```
    
2. 关于训练时使用 `i3d_nonlocal_8x8_IN_pretrain_400k` 会直接结束训练的问题，作者提供了一个新的脚本，重置了 lr, iteration, momentum 等信息。
    ```
    cd <caffe2-dir>/video-nonlocal-net/process_data/convert_models/
    # 修改 modify_blob_rm.py: 5,6 行
    input_model_file  --> ../../data/pretrained_model/i3d_nonlocal_8x8_IN_pretrain_400k.pkl
    output_model_file  -->  ../../data/pretrianed_model/i3d_nonlocal_8x8_IN_pretrain_400k_clean.pkl
    ```

3. finetune 的时候，log 中显示的进度是根据 Kinetic 数据集的规模计算，有很大的出入，具体的设置在 `<caffe2-dir>/video-nonlocal-net/lib/core/config.py:78`, 配置 yaml 文件可以修改，添加 `TRAIN.DATASET_SIZE = 9537` 

4. 训练是输入的是视频，具体按照怎样的规则得到 clip ？  
github issue 中有两个是与这个问题相关的，可以参考 [#12](https://github.com/facebookresearch/video-nonlocal-net/issues/12) [#15](https://github.com/facebookresearch/video-nonlocal-net/issues/15)  
简单来说，第一步会 decode 出输入视频的所有帧，此时如果遇到太短的视频会抛弃掉，从源代码看，最小是 32K 字节。`==> video/customized_video_decoder.cc:79`  
第二步从解码出来的帧中，随机选择 clip_length 长度的图片进行训练。 `==> video/customized_video_io.cc:674,687`

5. 训练和测试的时候视频经过怎样的预处理？  
[#16](https://github.com/facebookresearch/video-nonlocal-net/issues/16)
[#10](https://github.com/facebookresearch/video-nonlocal-net/issues/10)