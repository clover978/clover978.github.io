---
title: tensorflow object-detecct-api
date: 2021-01-07 10:19:14
tags:
  - deep learning
  - TensorFlow
  - Object Detection
categories:
  - TensorFlow
---

## 0. 安装
```
apt-get install protobuf-compiler
pip install pillow
pip install lxml
pip install jupyter
pip install matplotlib

# From tensorflow/models/research/
protoc object_detection/protos/*.proto --python_out=.

# From tensorflow/models/research/
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```
---

<!-- more -->


## 1. 训练命令
<https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_locally.md>
```shell
# From the tensorflow/models/research/ directory
python object_detection/train.py \
    --logtostderr \
    --pipeline_config_path=${PATH_TO_YOUR_PIPELINE_CONFIG} \
    --train_dir=${PATH_TO_TRAIN_DIR}
```
```PATH_TO_YOUR_PIPELINE_CONFIG```：网络设置文件 [pipeline_config.pbtxt]  
```PATH_TO_TRAIN_DIR```：训练模型保存位置

---
## 2. PIPELINE_CONFIG
<https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/configuring_jobs.md>
```proto
model {
(... Add model config here...)
}

train_config : {
(... Add train_config here...)
}

train_input_reader: {
(... Add train_input configuration here...)
}

eval_config: {
}

eval_input_reader: {
(... Add eval_input configuration here...)
}
```
pipeline_config 文件分为以下五个部分
  - model: ```object_detection/samples/model_configs```
  - train_config: 
      ```
    batch_size: 1
    optimizer {
    momentum_optimizer: {
    learning_rate: {
      manual_step_learning_rate {
        initial_learning_rate: 0.0002
        schedule {
          step: 0
          learning_rate: .0002
        }
        schedule {
          step: 900000
          learning_rate: .00002
        }
        schedule {
          step: 1200000
          learning_rate: .000002
        }
      }
    }
    momentum_optimizer_value: 0.9
    }
    use_moving_average: false
    }
    fine_tune_checkpoint: "/usr/home/username/tmp/model.ckpt-#####"
    from_detection_checkpoint: true
    gradient_clipping_by_norm: 10.0
    data_augmentation_options {
        random_horizontal_flip {
        }
    }
    ``` 
  - train_input_reader:
    ```
    tf_record_input_reader {
    input_path{
        "/usr/home/username/data/train.record"
    }
    label_map_path: "/usr/home/username/data/label_map.pbtxt"
    ```
    
---
## 3. TF-record
<https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/preparing_inputs.md>
```
# From tensorflow/models/research/
wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar
tar -xvf VOCtrainval_11-May-2012.tar
python object_detection/create_pascal_tf_record.py \
    --label_map_path=object_detection/data/pascal_label_map.pbtxt \
    --data_dir=VOCdevkit --year=VOC2012 --set=train \
    --output_path=pascal_train.record
python object_detection/create_pascal_tf_record.py \
    --label_map_path=object_detection/data/pascal_label_map.pbtxt \
    --data_dir=VOCdevkit --year=VOC2012 --set=val \
    --output_path=pascal_val.record
```

---
## 4. export trained model
<https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/exporting_models.md>
```
# From tensorflow/models/research/
python object_detection/export_inference_graph.py \
    --input_type image_tensor \
    --pipeline_config_path  object_detection/dxx_models/inception_resnet_v2/pipeline_config.pbtxt \ #${PIPELINE_CONFIG_PATH} i
    --trained_checkpoint_prefix object_detection/dxx_models/inception_resnet_v2/train/model.ckpt-381545 \ #${MODEL_PATH} 
    --output_directory object_detection/dxx_models/inception_resnet_v2/train/output_inference_graph.pb2
```