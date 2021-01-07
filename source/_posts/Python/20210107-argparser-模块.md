---
title: argparser 模块
date: 2021-01-07 11:05:37
tags:
  - Python
  - argparser
categories:
  - Python
---

# Command line parse
argpaser 是 Python 官方推荐的 命令行参数解析器，习惯使用它可以极大地方便 Python程序的交互。
<!-- more -->

## Example
```
import argparse

args = argparse.ArgumentParser(descroption='Argpase example')
args.add_argument('first', type=str, help='first argument')

args = args.parse_args()
print(args.fisrt)
```

## 必选参数
```
args.add_argument('first', type=str, help='first argument')
```

## 可选参数
```
args.add_argument('--optional', type=str, help='first argument')
```

## action
```
args.add_argument('--action_1', action='store_true', help='action_1 is True')
```