---
title: Python 中的 configure 设计
date: 2021-01-07 11:02:06
tags:
  - Python
categories:
  - Python
---

# Build configure file for you project
作为程序员，配置文件的作用和必要性无需赘言，很多优秀的开源项目也会配备一个 config 文件，下面介绍一些常见的 config 方法
<!-- more -->

---
## Python 脚本
Python 脚本是一种简单的配置方法。这种方法适用于一些自己写的小工程，一来它安全性不高，二来违背了配置与代码解耦的原则。
```python
############################################
#  databaseconfig.py
############################################
#!/usr/bin/env python
import preprocessing
mysql = {'host': 'localhost',
         'user': 'root',
         'passwd': 'my secret password',
         'db': 'write-math'}
preprocessing_queue = [preprocessing.scale_and_center,
                       preprocessing.dot_reduction,
                       preprocessing.connect_lines]
use_anonymous = True

############################################
#  main.py
############################################
#!/usr/bin/env python
import databaseconfig as cfg
connect(cfg.mysql['host'], cfg.mysql['user'], cfg.mysql['password'])

```

---
## YAML 文件
- yaml 文件也是一种很流行的配置方法，相比于使用 Python 脚本，部署的时候， yaml 文件可以做到与代码解耦，因此更友好。相比于 json 文件，yaml 文件更易于阅读修改，除非有特殊需求，一般情况使用 yaml 文件配置工程是很好的选择。
    ```python
    ######################
    #  databaseconfig.yml
    ######################
    mysql:
        host: localhost
        user: root
        passwd: my secret password
        db: write-math
    other:
        preprocessing_queue:
            - preprocessing.scale_and_center
            - preprocessing.dot_reduction
            - preprocessing.connect_lines
        use_anonymous: yes
    
    
    ######################
    #  main.py
    ######################
    import yaml
    
    with open("config.yml", 'r') as ymlfile:
        cfg = yaml.load(ymlfile)
    
    for section in cfg:
        print(section)
    print(cfg['mysql'])
    print(cfg['other'])
    
    ```

- yaml 文件里面还可以指定 item 的数据类型  
[数据类型对应表](http://huangro.iteye.com/blog/372976)
    ```yml
    video_size: !!python/tuple [1280, 720]  # 注意后面是中括号
    stride: 300
    history: 10
    mask_n: 10
    ```


---
## configparser
configparser 是 Python 的一个模块。
