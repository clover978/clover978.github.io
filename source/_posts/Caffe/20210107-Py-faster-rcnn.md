---
title: Py-faster-rcnn
date: 2021-01-07 10:25:06
tags:
  - deep learning
  - Caffe
  - Faster RCNN
  - Object Detection
categories:
  - Caffe
---

# 使用 faster-rcnn 检测游戏中的血条
<https://github.com/rbgirshick/py-faster-rcnn>  
这一周使用 rbg 大神的 py-faster-rcnn 训练王者荣耀视频中的人物血条，因为之前已经有不少经验，以为可以直接了当得到结果，但是事实却并不那么如意，现在将实验过程简单总结一下。

<!-- more -->

## 环境准备
- 实验环境：  
  实验的环境配置是 `Python-2.7, OpenCV-2.4.9, Caffe-1.0, cudnn-v6`
- 安装依赖项：  
  faster-rcnn 使用的是 Caffe 框架，首先需要安装 Caffe 的依赖，另外还需要一些 Python 环境，通过 pip 即可安装。
  ```
  pip install cpython easydict
  ```
  
## 编译 faster-rcnn
- 从 github pull 代码  
  ```
  git clone --recursive https://github.com/rbgirshick/py-faster-rcnn.git
  ```
- 更新 Caffe  
  <http://blog.csdn.net/rzjmpb/article/details/52373012>  
  github 中的代码使用的 Caffe 版本很古老，因此不支持 cudnnv5，为了使用 cudnn，可以升级 Caffe 或者安装低版本的 cudnn，这里选择升级 Caffe.
  ```
  cd py-faster-rcnn/caffe-fast-rcnn/
  git remote add caffe https://github.com/BVLC/caffe.git 
  # git config --list 
  git fetch caffe
  git merge -X theirs caffe/master
  ```
  ***merge 之后需要注释掉 python_layer.hpp 中的一行代码***
  ```shell
  $ vi caffe-fast-rcnn/include/caffe/layers/python_layer.hpp +29
  // self_.attr("phase") = static_cast<int>(this->phase_);              // <================
  ```
  
  
- 配置 Makefile.config  
  注意打开 `WITH_PYTHON_LAYER=1`

- 编译Caffe 和 faster-rcnn
  ```
  cd py-faster-rcnn/caffe-fater-rcnn/
  make -j
  make pycaffe
  cd py-faster-rcnn/lib/
  make
  ```
  > Caffe 默认使用的是 Python2, OpenCV2; faster-rcnn 使用系统环境变量中的 Python 解释器。一定要注意两者保持一致，特别是由于不可抗原因必须使用 Python3 的场景，确保 Caffe 和 faster-rcnn 的编译环境保持一致（*faster-rcnn 中有大量的 Python2 接口不被 Python3兼容，使用 Python3 需要做很多修改，或者在 Github 上寻找其他 fork 分支*）

## 数据准备  
一般按照 pascal_voc 的格式准备数据，需要准备三个文件夹  
  - `my_data/VOC2007/Annotations/`  
  `Annotation` 文件夹下面是 XML 文件，具体格式如下：
    ```HTML
    <annotation>
        <folder>VOC2007</folder>
        <filename>img.jpg</filename>
        <size>
            <width>1280</width>
            <height>720</height>
            <depth>3</depth>
        </size>
        <object>
            <bndbox>
                <xmin>548</xmin>
                <xmax>711</xmax>
                <ymin>255</ymin>
                <ymax>287</ymax>
            </bndbox>
            <name>health_blue</name>
            <truncated>0</truncated>
            <difficult>0</difficult>
        </object>
        <object>
            <bndbox>
                <xmin>675</xmin>
                <xmax>842</xmax>
                <ymin>373</ymin>
                <ymax>407</ymax>
            </bndbox>
            <name>health_red</name>
            <truncated>0</truncated>
            <difficult>0</difficult>
        </object>
        <segmented>0</segmented>
    </annotation>
    ```
  - `my_data/VOC2007/ImageSets/`  
  `ImageSets/Main` 文件夹下面是 TXT 文件，总共有 4 个，分别是 `trainval.txt`, `train.txt`, `val.txt`, `test.txt`， 文件中每一行代表一张图片；  
  - `my_data/VOC2007/JPEGImages/`  
  `JPEGImages` 文件夹下面保存的是图片。  

**这里有几个小细节：   
（1）图片的文件名后缀是 .jpg；  
（2）ImageSets 文件夹下面的 txt 是没有后缀的；  
（3）xml 中矩形框字段的 tag 是 bndbox，不是常用的 bbox；  
（4）cls 的名字最好全部使用小写英文字母和下划线，因为 faster-rcnn 会将字符全部转换为小写；  
（5）注意检查 xml 中是否每一张图片都含有 object 字段，没有 object 字段的 xml 进入网络训练不会报错，但是对训练结果会有影响。**  

  - 制作完数据集后，在 `data/` 目录下，建立软连接，指向 `my_data/` 文件夹
    ```
    cd py-faster-rcnn/data/
    ln -s my_data/ VOCdevkit2007
    ```

## 更改网络设置
faster-rcnn 的网络结构由两部分组成，一部分在 `models/` 目录下，这里的主要包含 prototxt， 用来初始化 Solver 和 Network；另一部分在 `lib/` 目录下，这里包含 rpn, roi_layer, nms 等模块，以及用来 读取和解析 xml 文件的代码。  
- `models/`  
py-faster-rcnn 提供了两种训练方法，一种是 alt-opt 训练，一种是 end2end 训练，关于两种训练的差别可以参考 Github 上的 readme，另外我们还可以选择使用三种不同的 Backbone Network： {VGG16, VGG-M, ZF}，这里我们选择性能最好的 VGG16，训练方式为 end2end 训练。对应目录为 `models/pascal_voc/VGG16/faster_rcnn_end2end/`  
  -- 修改 train.prototxt
    ```
    $ vi models/pascal_voc/VGG16/faster_rcnn_end2end/train.prototxt
    # input_data 层（第 11 行）， 修改 num_classes 为 cls+1
    layer {
        name: 'input-data'
        type: 'Python'
        top: 'data'
        top: 'im_info'
        top: 'gt_boxes'
        python_param {
            module: 'roi_data_layer.layer'
            layer: 'RoIDataLayer'
            # param_str: "'num_classes': 21"
            param_str: "'num_classes': 3"              # <================
        }
    }
    # roi_data 层（第 530 行）， 修改 num_classes 为 cls+1
    layer {
        name: 'roi-data'
        type: 'Python'
        bottom: 'rpn_rois'
        bottom: 'gt_boxes'
        top: 'rois'
        top: 'labels'
        top: 'bbox_targets'
        top: 'bbox_inside_weights'
        top: 'bbox_outside_weights'
        python_param {
            module: 'rpn.proposal_target_layer'
            layer: 'ProposalTargetLayer'
            # param_str: "'num_classes': 21"
            param_str: "'num_classes': 3"              # <================
        }
    }
    # cls_score 层（第 620 行）， 修改 num_output 为 cls+1
    layer {
        name: "cls_score"
        type: "InnerProduct"
        bottom: "fc7"
        top: "cls_score"
        param {
            lr_mult: 1
        }
        param {
            lr_mult: 2
        }
        inner_product_param {
            # num_output: 21
            num_output: 3              # <================
            weight_filler {
                type: "gaussian"
                std: 0.01
            }
            bias_filler {
                type: "constant"
                value: 0
            }
        }
    }
    # box_pred 层（第 643 行）， 修改 num_output 为 4*(cls+1)
    layer {
        name: "bbox_pred"
        type: "InnerProduct"
        bottom: "fc7"
        top: "bbox_pred"
        param {
            lr_mult: 1
        }
        param {
            lr_mult: 2
        }
        inner_product_param {
            # num_output: 84
            num_output: 12              # <================
            weight_filler {
                type: "gaussian"
                std: 0.001
            }
            bias_filler {
                type: "constant"
                value: 0
            }
        }
    }
    ```
    -- 修改 test.prototxt
    ```
    $ vi models/pascal_voc/VGG16/faster_rcnn_end2end/test.prototxt
    # cls_score 层（第 567 行）， 修改 num_output 为 cls+1
    layer {
        name: "cls_score"
        type: "InnerProduct"
        bottom: "fc7"
        top: "cls_score"
        param {
            lr_mult: 1
        }
        param {
            lr_mult: 2
        }
        inner_product_param {
            # num_output: 21
            num_output: 3              # <================
            weight_filler {
                type: "gaussian"
                std: 0.01
            }
            bias_filler {
                type: "constant"
                value: 0
            }
        }
    }
    # box_pred 层（第 592 行）， 修改 num_output 为 4*(cls+1)
    layer {
        name: "bbox_pred"
        type: "InnerProduct"
        bottom: "fc7"
        top: "bbox_pred"
        param {
            lr_mult: 1
        }
        param {
            lr_mult: 2
        }
        inner_product_param {
            # num_output: 84
            num_output: 12              # <================
            weight_filler {
                type: "gaussian"
                std: 0.001
            }
            bias_filler {
                type: "constant"
                value: 0
            }
        }
    }
    ```
    -- 修改 solver.prototxt  
    修改 solver 中的参数，使之适合训练自己的数据；   
    faster-rcnn 的另外大多数网络参数在 `lib/fast-rcnn/config.py` 中，附有详细的注释，可以参考注释进行修改。
    
- `lib/`  
    -- 修改 pascal_voc.py  
    由于自己训练数据的类别数与 pascal_voc 不同，因此需要稍微修改一下
    ```Python
    vi lib/datasets/pascal_voc.py
    # 修改 self._classes 为自己的类别：
    # self._classes = ('__background__', # always index 0
    #             'aeroplane', 'bicycle', 'bird', 'boat',
    #             'bottle', 'bus', 'car', 'cat', 'chair',
    #             'cow', 'diningtable', 'dog', 'horse',
    #             'motorbike', 'person', 'pottedplant',
    #             'sheep', 'sofa', 'train', 'tvmonitor')
    self._classes = ('__background__', # always index 0
                 'health_blue', 'health_red')
    ```
    
## 开始训练
1. 建立测试结果的文件夹：  
    ```shell
    mkdir -p data/VOCdevkit2007/results/VOC2007/Main/
    ```
    训练完成后，会进行一次测试，如果不手动新建文件夹，测试的时候会报错说找不到这个目录。  

2. 直接使用 faster-rcnn 的 脚本进行训练：
    ```
    ./experiments/scripts/faster_rcnn_end2end.sh 0 VGG16 pascal_voc
    ```
    最大迭代次数在 `./experiments/scripts/faster_rcnn_end2end.sh` 中进行设置；  
    训练的 log 存储在 `experiments/logs/` 中， 也可以自己重定向出来。

## 测试模型
1. faster-rcnn 提供的训练脚本，在使用训练集验证集训练完成后，会在测试集上进行测试，输出模型的 AP 和 AR；  

2. 如果想使用单张图片进行测试，并且看到可视化效果，可以参考 `tools/demo.py`， `demo.py` 使用 matplotlib 进行可视化，对脚本稍加修改，可以将测试结果保存成图片存储下来。  

3. 修改脚本代码：
    ```
    #!/usr/bin/env python

    # --------------------------------------------------------
    # Faster R-CNN
    # Copyright (c) 2015 Microsoft
    # Licensed under The MIT License [see LICENSE for details]
    # Written by Ross Girshick
    # --------------------------------------------------------

    """
    Demo script showing detections in sample images.
    See README.md for installation instructions before running.
    """

    import _init_paths
    from fast_rcnn.config import cfg
    from fast_rcnn.test import im_detect
    from fast_rcnn.nms_wrapper import nms
    from utils.timer import Timer
    import matplotlib.pyplot as plt
    import numpy as np
    import scipy.io as sio
    import caffe, os, sys, cv2
    import argparse
    import glob

    CLASSES=('__background__','health_blue','health_red')

    def vis_detections(im, class_name, dets, thresh=0.5):
        inds = np.where(dets[:, -1] >= thresh)[0]
        boxes, scores = ([], [])
        for i in inds:
            bbox = dets[i, :4]
            score = dets[i, -1]
            boxes.append( dets[i, :4] )
            scores.append(score)
            print(bbox, score)
        return boxes, scores

    def demo(net, im, filename, out_dir):
        timer = Timer()
        timer.tic()
        scores, boxes = im_detect(net, im)
        timer.toc()
        print('-----------------------------------------------------')
        print (('Detection {} took {:.3f}s for {:d} object proposals').format(filename, timer.total_time, boxes.shape[0]))

        CONF_THRESH = 0.8
        NMS_THRESH = 0.1
        for cls_ind, cls in enumerate(CLASSES[1:]):
            cls_ind += 1 # because we skipped background
            cls_boxes = boxes[:, 4*cls_ind:4*(cls_ind + 1)]
            cls_scores = scores[:, cls_ind]
            dets = np.hstack((cls_boxes,
                            cls_scores[:, np.newaxis])).astype(np.float32)
            keep = nms(dets, NMS_THRESH)
            dets = dets[keep, :]

            _boxes, _scores = vis_detections(im, cls, dets, thresh=CONF_THRESH)
            for box, score in zip( _boxes, _scores):
                display_str = cls + ' ' + str(score)
                cv2.rectangle(im, tuple(box[:2]), tuple(box[2:]), (255,0,0), 2)
                cv2.putText(im, display_str, tuple(box[:2]), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)

        cv2.imwrite( os.path.join( out_dir, filename), im)

    def parse_args():
        """Parse input arguments."""
        parser = argparse.ArgumentParser(description='Faster R-CNN demo')
        parser.add_argument('model_name', help='model name',
                            type=str)
        parser.add_argument('--gpu', dest='gpu_id', help='GPU device id to use [0]',
                            default=1, type=int)
        parser.add_argument('--cpu', dest='cpu_mode',
                            help='Use CPU mode (overrides --gpu)',
                            action='store_true')
        parser.add_argument('--data_dir', dest='data_dir', help='data path',default='/bigdata/dxx/py-faster-rcnn/demo/',
                            type=str)
        parser.add_argument('--video_name', dest='video_name', help='video name',default='test.mp4',
                            type=str)
        parser.add_argument('--image_name', dest='image_name', help='image name',default='test.mp4',
                            type=str)
        parser.add_argument('--model_dir', dest='model_dir', help='model path',default='/bigdata/dxx/py-faster-rcnn/models/',
                            type=str)
        args = parser.parse_args()
        return args

    if __name__ == '__main__':
        cfg.TEST.HAS_RPN = True  # Use RPN for proposals
        args = parse_args()

        out_dir = os.path.join(args.data_dir, args.model_name)
        if not os.path.exists(out_dir):
            os.mkdir(out_dir)
            os.system('{} {} {}'.format('chmod', '777', out_dir))
        print('output images to {}'.format(out_dir))
        prototxt = 'models/pascal_voc/VGG16/faster_rcnn_end2end/test.prototxt'
        caffemodel = os.path.join(args.model_dir, args.model_name, 'vgg16_faster_rcnn_final.caffemodel')

        if args.cpu_mode:
            caffe.set_mode_cpu()
        else:
            caffe.set_mode_gpu()
            caffe.set_device(args.gpu_id)
            cfg.GPU_ID = args.gpu_id

        net = caffe.Net(prototxt, caffemodel, caffe.TEST)
        print ('\n\nLoaded network {:s}'.format(caffemodel))

        image_f = os.path.join(args.data_dir, 'image', args.image_name, '*.jpg')
        for im_f in sorted(glob.glob(image_f)):
            im = cv2.imread(im_f)
            demo(net, im, os.path.basename(im_f), out_dir)
        '''
        video_f = os.path.join(args.data_dir, 'video', args.video_name)
        video_cap = cv2.VideoCapture(video_f)
        frame_cnt = int(video_cap.get(cv2.CAP_PROP_FRAME_COUNT))
        error_frame = max(20, frame_cnt / 50)
        for i, frame_index in enumerate(range(0, frame_cnt, 10)):
            video_cap.set(cv2.CAP_PROP_POS_FRAMES, frame_index)
            status, frame = video_cap.read()
            if not status:
                    error_frame -= 1
                    if error_frame > 0:
                        continue
                    video_cap.release()
                    raise Exception('Bad video file')

            demo(net, frame, frame_index+'.jpg', out_dir)
        '''
    ```

4. 测试图片的路径格式如下：  
    ```shell
    + /bigdata/dxx/py-faster-rcnn/
        + models/
            + <model_name>/
                - vgg16_faster_rcnn_final.caffemodel
        + demo/
            + video/
                - test.mp4
            + image/
                + test.mp4/
                    - 001.jpg
                    - 002.jpg
            + <model_name>/  # detection results
                - 001.jpg
                - 002.jpg
    ```

5. `python tools/demo.py <model_name>`

6. 这次使用了迭代 5000 次的模型进行测试，在两个视频上进行了简单的测试，在测试的 100 张图片上，完全没有出现 漏检或错检，准确率达到 1.0

## 错误列表
- 注意事项：  
***1. 编译过程中，每次更新环境，最好重新编译 Caffe、pycaffe、faster-rcnn/lib***  
***2. 运行过程中，每次重新开始训练/测试，最好删除掉 `data/cache/` 和 `data/VOCdevkit2007/annotationcache/`***

- **`AttributeError: can't set attribute`**  
    这是由于 git merge 之后没有修改 `caffe-fast-rcnn/include/caffe/layers/python_layer.hpp` 导致的。

- **`AttributeError: 'module' object has no attribute 'text_format'`**  
    这个和 protobuf 的版本有关
    ```
    $ vi lib/fast-rcnn/train.py
    + import google.protobuf.text_format
    ```
    
- **`TypeError: slice indices must be integers or None or have an __index__ method`**  
    ```Python
    # $ vi lib/rpn/proposal_target_layer.py
    def _get_bbox_regression_labels(bbox_target_data, num_classes):
        clss = bbox_target_data[:, 0]
        bbox_targets = np.zeros((clss.size, 4 * num_classes), dtype=np.float32)
        bbox_inside_weights = np.zeros(bbox_targets.shape, dtype=np.float32)
        inds = np.where(clss > 0)[0]
        for ind in inds:
            ind = int(ind)                  <=================              
            cls = clss[ind]
            start = int(4 * cls)            <=================
            end = int(start + 4)            <=================
            bbox_targets[ind, start:end] = bbox_target_data[ind, 1:]
            bbox_inside_weights[ind, start:end] = cfg.TRAIN.BBOX_INSIDE_WEIGHTS
        return bbox_targets, bbox_inside_weights
    
    ```
    
- **`TypeError: 'numpy.float64' object cannot be interpreted as an index`**  
    <https://github.com/rbgirshick/py-faster-rcnn/issues/481>  
    numpy1.12 版本以后，不再支持 1.0，2.0 这样的浮点数作为索引，因此需要安装低版本的numpy  
    ```
    pip install -U numpy==1.11.0
    ```
    另外还可以找到所有出问题的地方，使用 astype() 函数进行类型转换  
    ```Python
    $ vi lib/roi_data_layer/minibatch.py +26
    fg_rois_per_image = np.round(cfg.TRAIN.FG_FRACTION *rois_per_image).astype(np.int)
    
    $ vi lib/datasets/ds_utils.py +12
    hashes = np.round(boxes * scale).dot(v).astype(np.int)
    
    $ vi lib/fast_rcnn/test.py line +129
    hashes = np.round(blobs['rois'] * cfg.DEDUP_BOXES).dot(v).astype(np.int)
    
    $ vi lib/rpn/proposal_target_layer.py +60
    fg_rois_per_image = np.round(cfg.TRAIN.FG_FRACTION * rois_per_image).astype(np.int)
    ```
