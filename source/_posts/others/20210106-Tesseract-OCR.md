---
title: Tesseract-OCR
date: 2021-01-06 20:34:38
tags:
  - deep learning
  - OCR
categories:
  - others
---

[Tesseract ](https://github.com/tesseract-ocr/tesseract)是 Github 上一个 OCR 的开源项目，目前最新版已经更新到 v4.0.x，最近小小尝试了一下怎么使用这个开源库，记一下笔记。

<!-- more -->

## Install
`环境： Ubuntu16.04`  
Tesseract 可以使用 apt 安装，但是在 Ubuntu18.04 之前的版本，官方库里面的最新版本只有 3.x.x，作为 Ubuntu16.04 用户，需要先添加 PPA，其他系统版本的 PPA 可以在[这里](https://github.com/tesseract-ocr/tesseract/wiki#linux)找到。  
```
apt-get install python-software-properties software-properties-common
add-apt-repository ppa:alex-p/tesseract-ocr
apt-get update
```
添加完 PPA 之后就可以直接 apt 安装了
```
apt-get install tesseract-ocr libtesseract-dev
```

除了 apt 安装外，也可以直接从 github clone
代码，然后编译安装， 不再详述。

## Language support
Tesseract 的模型文件放在 `$TESSDATA_PREFIX` 指定的位置，当找不到这个环境变量是，默认路径是 `/usr/share/tesseract-ocr/4.00/tessdata`

Teseeract 支持的语言及对应的模型文件可以在 [这里](https://github.com/tesseract-ocr/tesseract/wiki/Data-Files#updated-data-files-for-version-400-september-15-2017) 找到

Tesseract 的中文支持有两种方式：
  - 直接 apt 安装  
    `apt install tesseract-ocr-chi-sim tesseract-ocr-chi-sim-vert`
  - 简体中文语言包模型  
    ```
    wget https://github.com/tesseract-ocr/tessdata/raw/4.00/chi_sim.traineddata
    mv chi_sim.traineddata /usr/share/tesseract-ocr/4.00/tessdata/
    ```
     
  `tesseract --list-langs` 查看目前支持的语言
    
## Command Line Usage
<https://github.com/tesseract-ocr/tesseract/wiki/Command-Line-Usage>

Tesseract 的命令行使用可以参考上面的链接
这里列举一个最简答的用法
```
tesseract input.png output -l eng --oem 3 --psm 3
# input.png： 输入的图片
# output： 输出文件名（指定 stdout 输出到标准输出）
# -l：指定语言，eng（英语，默认），chi_sim（简体中文）
# --oem[0,1,2,3]： OCR Engine Mode，4.x.x 版本默认为 3, 就是使用 available 的模型，即 LSTM 模型
# --psm[0-13]： Page Segement Mode，默认为 3，自动分页
# Tesseract 还有许多许多其他的参数可以设置，具体可以参照上方的链接
```

## Python API
<https://github.com/sirfz/tesserocr>  
Tesseract 官方给出了一个使用 Python API 的 [脚本](https://github.com/tesseract-ocr/tesseract/wiki/APIExample#c-api-in-python)，脚本的原理是使用 ctypes + .so/.dll文件，用 Python 调用 C 的接口，这种方法还需要写 platform specified 的代码，所以比较麻烦。我在网上找到有人写了一个 Python Wrapper， 在 [官方 repo](https://github.com/tesseract-ocr/tesseract/wiki/AddOns#tesseract-wrappers) 下还可以找到更多 Wrapper 的链接，下面介绍利用这个方法使用 Python 接口。

  - 安装
      ```
      apt-get install tesseract-ocr libtesseract-dev libleptonica-dev
      # 前两个在装 Tesseract 的时候已经安装过了，一定记得要装第三个东西
      CPPFLAGS=-I/usr/local/include pip3 install tesserocr
      ```
  - Usage
  官方 repo 里面给出了很多 接口示例，这里摘抄一个最基本的用法，tesserocr 支持输入 PIL Image 对象或者 图片文件。
    ```
    from tesserocr import PyTessBaseAPI
    
    images = ['sample.jpg', 'sample2.jpg', 'sample3.jpg']
    
    with PyTessBaseAPI() as api:
        for img in images:
            api.SetImageFile(img)
            print api.GetUTF8Text()
            print api.AllWordConfidences()
    # api is automatically finalized when used in a with-statement (context manager).
    # otherwise api.End() should be explicitly called when it's no longer needed.
    ```
  - **ERROR**  
  按照上面的步骤，确实可以安装成功 tesserocr， 然后执行 `python3 -c 'import tesserocr; print(tesserocr.tesseract_version())'`，发现版本号竟然是 3.4.0. 于是开始了漫长的升级之旅
    - 卸载 Tesseract3.4.0， tesserocr  
    首先将已经安装的版本卸载掉
    - 重新安装 tesserocr  
    安装之后又出错了， import 的时候找不着 tesseract.so.3。  
    这个就很烦了，无奈之下我只能从源码开始安装 Tesseract， 再从 源码安装 tesserocr， 最后终于成功了。
    - 从源码安装 Tesseract  
    <https://github.com/tesseract-ocr/tesseract/wiki/Compiling>
    - 从源码安装 tesserocr  
    ```
    git clone https://github.com/sirfz/tesserocr
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/:/usr/local/lib/
    pip3 install .
    ```
    > 最后问题是终于解决了，具体这两个东西到底哪个是必须要源码编译，哪个可以用 apt/pip 安装，我也不知道，反正两个全部都编译安装的话是可以成功的。注意安装 tesserocr 的时候，确保电脑上只有最新版的 Tesseract