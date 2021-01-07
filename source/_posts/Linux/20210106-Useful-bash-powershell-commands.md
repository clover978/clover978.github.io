---
title: Useful bash/powershell commands
date: 2021-01-06 20:02:46
tags:
  - Linux
categories:
  - Linux
---

收集一些常用的 Windows Powershell 和 Linux bash 命令

<!-- more -->

## 1. Powershell
```bash
# 创建软连接
New-Item -Path images -ItemType SymbolicLink -Value E:\coco\images\
```


```bash
# windows 命令行配置: 设置codepage为 936，GUI的设置经常不能生效
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Console\%SystemRoot%_system32_cmd.exe]
"CodePage"=dword:0000fde9
"FontFamily"=dword:00000036
"FontWeight"=dword:00000190
"FaceName"="Consolas"
"ScreenBufferSize"=dword:232900d2
"WindowSize"=dword:002b00d2
```

## 2. Bash

```bash
# 同时在 命令行 和 文件中输出
<cmd> | tee <log-file>
# echo "hello" 2>&1 | tee output.txt
```

```bash
# 查看终端大小
stty size
```

```bash
# 查看程序占用swap
awk '/^Swap:/ {SWAP+=$2}END{print SWAP" KB"}' /proc/$(pid)/smaps  
```

```bash
# 重启 kde
kquitapp5 plasmashell && kstart plasmashell
```

```bash
# 设置系统语言
apt install locales
locale-gen zh_CN.utf8
locale-gen zh_CN
export LANG=zh_CN.utf8
export LC_ALL=zh_CN.utf8
export LANGUAGE=zh_CN.utf8
update-locale LANG=zh_CN.utf8 LC_ALL=zh_CN.utf8 LANGUAGE=zh_CN.utf8
locale -a 
```

```bash
#  sed 命令操作文本
sed [-i] 's/^/HEAD/g' test.file
sed [-i] 's/$/TAIL/g' test.file
sed '10,20i new_line_between_10_and_20' test.file
```

```bash
# vnc
start: vncserver -geometry 1920x1080 
exec:  ip:<port>
stop: vncserver -kill :<port>
```

```bash
# 批量重命名文件
rename .jpeg .jpg ./*
```

```bash
# nohup 后台运行程序
nohup ./run.sh > log.txt &

# 其中 & 表示将命令放到后台执行; nohup 表示让进程忽略 HUP 信号。这样，关闭当前窗口后，程序仍然会执行，如果想停止程序，只能通过 ps 找到进程号，然后 kill 掉
```


```bash
# `screen` 命令提供了一种更强大的后台执行命令方法

# 新建一个 会话窗口（Session）
screen -S <会话名称>

# 列出所有会话
screen -ls

# 进入一个会话
screen -r <会话名称>

# 进入会话窗口后，就可以执行需要执行的命令了，在执行过程中退出会话不会中断程序

# 在会话窗口中退出会话
Ctrl + a + d
# 在会话窗口外退出会话
screen -d <会话名称>


# 可以看到 `screen` 将任务放到后台后，支持随时随地查看任务状态，或者与任务进行交互。不需要像 `nohup` 那样必须重定向出 log 文件（当然实际情况下重定向更为方便）。
```

```bash
# 将已经执行的前台任务放到后台

# 执行命令
./run.sh

# 将任务放到后台，此时任务状态是 Stopped
Ctrl + z

# 使用 `jobs` 可以查看任务
jobs

# 将任务放到后台执行
bg <job-id>
# 将任务放到前台执行
fg <job-id>


# 任务放到后台之后，一旦关闭当前窗口，这个窗口下的所有任务都会收到 HUP 信号，从而中断运行，因此还需要另外一个命令。  
# 使用 disown 命令，让任务不再附属于当前窗口，这样，使用 jobs 命令会发现任务不见了，关闭当前窗口也不会中断任务。
disown -h <job-id>
disown -ah  # 将所有任务移出作业列表
```

```bash
# 批量删除进程
ps -ef | grep firefox | awk '{print $2}' | xargs kill -9
```

```bash
# github content DNS 污染
199.232.68.133 raw.githubusercontent.com
199.232.68.133 userimage.githubusercontent.com
```

## 3. 图像处理

```bash
# resize 视频，保持较短边大于 256  

ffmpeg -i input.mp4 -vf scale=256:256:force_original_aspect_ratio=increase output.mp4
# 这条命令计算出来的 w/h 可能是奇数，对于 libx264 编码，无法生成视频

ffmpeg -i input.mp4 -vf scale=if(lt(iw\,ih)\,256\,-2):if(lt(iw\,ih)\,-2\,256) output.mp4
# 这个命令可以完美实现功能，并且只需要读取一次视频
```

```bash
# 提取视频帧， 设置 stride

ffmpeg -i input.mp4 -vf "scale=256:256:force_original_aspect_ratio-increase, select=not(mod(n\,4))" -vsync vfr images/img-%06d.jpg
```

```bash
# ffmpeg 抽帧
ffmpeg -i test.mp4 -r 1 -ss 0:0:0 -t 0:0:3 -s 1280x720 -f image2 %4d.jpg 
```

```bash
# ffmpeg 剪切视频
ffmpeg -ss 00:00:15 -t 00:00:05 -i input.mp4 -vcodec copy -acodec copy output.mp4
```

```bash
# imagemagick 处理图片
convert in.jpg -rotate 90 out.jpg
convert in.jpg -resize 640x360 out.jpg
```

## 4. pip 镜像源

```bash
# 临时用法：  
pip install example.whl -i https://pypi.douban.com/simple --trusted-host=pypi.douban.com
```

```bash
#  永久配置：

# vi  ~/.pip/pip.conf 
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```