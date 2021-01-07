---
title: openpose_train
date: 2021-01-07 10:22:03
tags:
  - deep learning
  - Caffe
  - OpenPose
  - Pose Estimation
categories:
  - Caffe
---

https://github.com/CMU-Perceptual-Computing-Lab/openpose  
https://github.com/CMU-Perceptual-Computing-Lab/openpose_train

<!-- more -->

---
### 1. 拉取代码  
```
git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose_train.git
```
---

### 2. 下载数据集

#### 2.1. cocoapi
```
mkdir -p openpose_train/dataset/
mkdir -p openpose_train/dataset/COCO/
cd openpose_train/dataset/COCO/
git clone --recurse https://github.com/gineshidalgo99/cocoapi.git

# 安装 mingw, matlab PCT
# 编译 gasonMex, maskApiMex
cd cocoapi/MatlabAPI/
mex('CXXFLAGS=$CXXFLAGS -std=c++11 -Wall','-largeArrayDims','private/gasonMex.cpp','../common/gason.cpp','-I../common/','-outdir','private');
mex('COMPFLAGS=\$CFLAGS -Wall -std=c++11','-largeArrayDims','private/maskApiMex.c','../common/maskApi.c','-I../common/','-outdir','private');
```

#### 2.2. coco数据集
```
# annotations_trainval2017, image_info_test2017 --> dataset/cocoapi/annotations/
# train2017, val2017, test2017                  --> dataset/cocoapi/images/


http://images.cocodataset.org/zips/train2017.zip
http://images.cocodataset.org/zips/val2017.zip
http://images.cocodataset.org/zips/test2017.zip
http://images.cocodataset.org/annotations/annotations_trainval2017.zip
http://images.cocodataset.org/annotations/image_info_test2017.zip
http://images.cocodataset.org/annotations/stuff_annotations_trainval2017.zip
```

#### 2.3. 生成LMDB
```
# 生成 body keypoints LMDB
# 依次执行 a1_coco_jsonToNegativesJson, a2_coco_jsonToMat, a3_coco_matToMasks, a4_coco_matToRefinedJson
python c_generateLmdbs.py

# 生成 foot keypoints LMDB
# 依次执行 a2_coco_jsonToMat, a4_coco_matToRefinedJson
python c_generateLmdbs.py
```
---

### 3. 模型训练

#### 3.1. 拉取 OpenPose Caffe 和 预训练模型
```
cd /openpose_train/
git clone --recurse github.com/CMU-Perceptual-Computing-Lab/openpose_caffe_train.git

cd /openpose_train/dataset/
mkdir vgg
wget -c http://www.robots.ox.ac.uk/~vgg/software/very_deep/caffe/VGG_ILSVRC_19_layers.caffemodel
wget -c https://gist.githubusercontent.com/ksimonyan/3785162f95cd2d5fee77/raw/bb2b4fe0a9bb0669211cf3d0bc949dfdda173e9e/VGG_ILSVRC_19_layers_deploy.prototxt
```

#### 3.2. 生成 prototxt
```
cd /openpose_train/training
python d_setLayers.py
```

#### 3.3. 训练
```
cd /openpose_train/training_result/pose/
./train_pose.sh 0
```

---

### 4.测试

#### 4.1. 速度测试
```
# Test using LMDB:
/openpose_train/openpose_caffe_train/build/tools/caffe time -gpu 0 -model pose_training.prototxt

# Test using input layer:  368*368
/openpose_train/openpose_caffe_train/build/tools/caffe time -gpu 0 -model pose_deploy.prototxt -phase TEST
```

    | batchsize | 1     | 8     |16     | **deploy**|
    |---        |---    |---    |---    |---        |
    |time(ms)   |19     |103    |199    |17.3       |
    

#### 4.2. Evaluate
```
cd /openpose
./scripts/tests/pose_accuracy_coco_val.sh 
# https://github.com/gineshidalgo99/cocoapi/blob/master/PythonAPI/pycocoEvalDemo.ipynb  --> evaluate.py
python scripts/tests/evaluate.py


Body25 baseline:
 5000samples/119s = 42FPS
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.523
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.763
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.568
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.466
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.604
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.576
 Average Recall     (AR) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.790
 Average Recall     (AR) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.613
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.483
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.709

COCO baseline:
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.490
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.742
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.520
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.426
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.588
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.544
 Average Recall     (AR) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.766
 Average Recall     (AR) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.571
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.442
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.688

body23:
 5000samples/120s = 41FPS
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.433
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.692
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.450
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.360
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.535
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.484
 Average Recall     (AR) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.719
 Average Recall     (AR) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.502
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.378
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.632
 
 90 degree
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.324
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.602
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.304
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.259
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.416
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.376
 Average Recall     (AR) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.633
 Average Recall     (AR) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.366
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.283
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.505

 60 degree
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.345
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.630
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.327
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.290
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.425
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 20 ] = 0.399
 Average Recall     (AR) @[ IoU=0.50      | area=   all | maxDets= 20 ] = 0.662
 Average Recall     (AR) @[ IoU=0.75      | area=   all | maxDets= 20 ] = 0.392
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets= 20 ] = 0.313
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets= 20 ] = 0.520

```
