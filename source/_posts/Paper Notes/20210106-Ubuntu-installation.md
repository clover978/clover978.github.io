---
title: Ubuntu installation
date: 2021-01-06 19:51:30
tags:
  - Linux
categories:
  - Linux
---

## 0. 备份数据
  - bashrc 文件
  - docker
```
docker save -o images.tar $image_id1 $ image_id2
docker export -o container_1.tar container_id1
docker export -o container_2.tar container_id2
```
  - 记录重要文件路径

<!-- more -->

## 1. 制作系统盘
  - 下载  **镜像文件**
  - 下载安装 **大白菜**
  - 制作启动盘

## 2. 安装系统
  - 按照安装流程安装系统
  - 安装 net-tools, openssh
    ```
    sudo apt update && sudo apt install -y net-tools, openssh-server
    service ssh start
    ifconfig
    ```
  - 关闭图形显示，确认ip地址
    ```bash
    # 关闭图形化显示
    sudo systemctl set-default multi-user.target
    sudo reboot
    ifconfig
    ```

## 3. 挂载硬盘
  - 格式化硬盘，推荐 ext4
  - 创建挂载点
  - 挂载硬盘
    ```bash
    sudo mkfs.ext4 /dev/sda1
    sudo mkdir /data
    sudo mount /dev/sda1 /data
    ```

## 4. 更新国内源
```bash
sudo mv /etc/apt/sources.list /etc/apt/sourses.list.backup
sudo vi /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

sudo apt update
```

## 5. 安装显卡驱动
```
下载显卡驱动
https://www.nvidia.cn/Download/index.aspx

sudo apt install gcc make
sudo ./NVIDIA-Linux-x86_64-455.23.04.run –no-opengl-files -no-x-check -no-nouveau-check
```

## 6. 安装 docker
```
# 安装 docker-ce
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
sudo usermod -aG docker $USER
docker -v

# 安装 nvidia-docker2
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install nvidia-docker2

# 配置默认 nvidia runtime
sudo vi /etc/docker/daemon.json
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

service docker restart

docker run --rm hello-world nvidia-smi
```
