---
title: 查看 caffemodel 内容
date: 2021-01-07 10:37:29
tags:
categories:
---

## 使用 Python 接口，查看 .caffemodel 文件内容
训练好一个网络后，使用 Python 接口进行部署的时候，需要两个文件 `trian.prototxt` 和 `xx.caffemodel`。  
初始化网络的时候，首先会通过 `train.prototxt` 生成网络的拓扑结构，我们可以在  [netscope](https://ethereon.github.io/) 预览网络拓扑结构；  
网络搭建好了之后，Caffe 会从预训练模型中载入参数，对网络参数进行初始化，这个时候有三种情况：
  - `prototxt` 中出现的层，`caffemodel` 中有对应的层：  
  这是最常见的情况，网络参数会使用 `caffemodel`  中存储的参数进行初始化。如果两者的结构参数不一样，就会抛出异常（shape mismatch）。比较典型的有：`conv层/pool层` 的 `kernel-size`，`channel` 不一样；`fc层` 的输入/输出 维度不一样。
  - `prototxt` 中出现的层，`caffemodel` 中没有对应的层：  
  这种情况下，网络参数会使用 `prototxt` 中指定的 `filler` 方法进行初始化，等同于这一层 train from scratch；使用 ImageNet 预训练模型训练其他图片分类模型，修改最后一个 `fc层` 的名字，就是这种场景。
  - `caffemodel` 中出现的层，`prototxt` 中没有对应的层：  
  这种情况下，`caffemodel` 中的参数就不会使用，在 Caffe 中会打印出 `Ignore layer **` 这样的 log 信息。

    > 举例来说：使用 下面的 pretrain.prototxt 训练出来 caffemodel，然后初始化 train.prototxt 的参数时：layer-A 会被 ignore；layer-B 会使用 pretrain-model 的参数；layer-C 会使用 高斯分布初始化   
    ```shell
    # pretrain.prototxt
    layer{
        name: layer-A   # param in this layer will be ignored when training train.prototxt
        ...
    }
    layer{
        name: layer-B   # param in this layer will be used to initalize training prototxt
        ...
    }
    ```
    ```shell
    # train.prototxt
    layer{
        name: layer-B
        ...
    }
    layer{
        name: layer-C   # this layer does not have cordnate layer in pretrained model, will be initialize by weight-filler
        weiget-filler:{
            type: guassian
            std: 0.1
        }
    }
    ```

下面言归正传，prototxt 是文本文件，很容易看到 待训练模型 的网络结构，那么，如何看到预训练模型的网络结构呢？直接上代码
    ```Python
    import caffe.proto.caffe_pb2 as caffe_pb2
    model_f = 'pretrain.caffemodel'
    model = caffe_pb2.NetParameter()
    with open(model_f, 'rb') as f:
        string = f.read()
        model.ParseFromString(string)  # this instruction will cost much time
    layers = model.layer

    layers[0]
    '''
    name: "input-data"
    type: "Python"
    top: "data"
    top: "im_info"
    top: "gt_boxes"
    phase: TRAIN
    python_param {
    module: "roi_data_layer.layer"
    layer: "RoIDataLayer"
    param_str: "\'num_classes\': 3"
    }
    '''

    for layer in layers:
        print(layer.name)
    '''
    input-data
    data_input-data_0_split
    im_info_input-data_1_split
    gt_boxes_input-data_2_split
    conv1_1
    relu1_1
    conv1_2
    relu1_2
    pool1
    conv2_1
    relu2_1
    conv2_2
    relu2_2
    pool2
    conv3_1
    relu3_1
    conv3_2
    relu3_2
    conv3_3
    relu3_3
    pool3
    conv4_1
    relu4_1
    conv4_2
    relu4_2
    conv4_3
    relu4_3
    pool4
    conv5_1
    relu5_1
    conv5_2
    relu5_2
    conv5_3
    relu5_3
    conv5_3_relu5_3_0_split
    rpn_conv/3x3
    rpn_relu/3x3
    rpn/output_rpn_relu/3x3_0_split
    rpn_cls_score
    rpn_cls_score_rpn_cls_score_0_split
    rpn_bbox_pred
    rpn_bbox_pred_rpn_bbox_pred_0_split
    rpn_cls_score_reshape
    rpn_cls_score_reshape_rpn_cls_score_reshape_0_split
    rpn-data
    rpn_loss_cls
    rpn_loss_bbox
    rpn_cls_prob
    rpn_cls_prob_reshape
    proposal
    roi-data
    roi_pool5
    fc6
    relu6
    drop6
    fc7
    relu7
    drop7
    fc7_drop7_0_split
    cls_score6
    bbox_pred6
    loss_cls
    loss_bbox
    '''
    ```

## 使用 Python 接口，修改 caffemodel 的内容
在上面的接口中，我们可以看到，读取 caffemodel 的过程实际上是 load 一串二进制方式存储的字符串，相对地，我们也可以将 字符串序列化成为二进制文件，成为一个 caffemodel，实际上，使用 Caffe 训练模型的时候，就是通过这样一个函数保存模型的，当保存路径不存在或没有权限的时候，这个接口还会抛出异常，我们调用这个函数的 Python 接口，就可以修改预训练 caffemodel 中的参数了。
