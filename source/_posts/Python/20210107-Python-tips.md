---
title: Python tips
date: 2021-01-07 10:52:07
tags:
  - Python
categories:
  - Python
---

收集一些短小的 Python 代码段
<!-- more -->

```Python
# 查看磁盘占用
import os
vfs = os.statvfs('/')
vfs
total = vfs.f_blocks * statvfs.f_bsize
used = vfs.f_bsize * (vfs.f_blocks - vfs.f_bfree)
```


```Python
# 输出保留 2 位小数
a = 3.1415926
print( round(a,2) )
```


```Python
# 查看文件类型
improt filetype

f = '1.txt'
kind = filetype.guess(f)
print(kind.mime)
```
