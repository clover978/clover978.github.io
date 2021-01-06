---
title: Git commitzen
date: 2021-01-06 20:36:23
tags:
  - Linux
  - Git
categories:
  - others
---


# Git
## 0. Intro
`Git` 是一种非常常见的 **分布式版本管理工具**，学习使用 `git`应该算是程序员的一项基本功了，毕竟稍微有一些体量的公司，都会有自己的版本管理系统。  
`Git` 的子命令相当之多，比较复杂的那些，有相关需求的时候上网查就可以了，这篇笔记只记录最简单常用的命令。  

<!-- more -->


## 1. Why Git
***以下内容纯属个人理解，没有参考网上资料，不一定正确***  
Git 中引入了几个重要的概念：
  - 分支：  
  一个**代码仓库**（repo）至少拥有一个 **master** 分支，还可以拥有多个其他分支，在分支上的开发各自分开，然后合并（merge）到主分支中。
  - 暂存区：  
  在 `git` 中，磁盘上的代码可以叫做**本地代码**，被加入到 `git`仓库的代码处于**工作区**，工作区的代码的改动不会直接进入**分支**，而是被放在**暂存区**中。使用 `git diff` 命令，可以轻松查看开发迭代在上版本代码上的所有改动。  
  在实际使用过程中会对这些概念有更清晰的认识。
  - origin/master：  
  `git` 的代码管理是分布式的，同样一份代码，可以存储在各个不同的地方。这需要依赖一个 git 服务器和一个 **origin** 分支。因此 `git`仓库 中的代码总共存在于 5 个地方：本地磁盘、工作区、暂存区、本地分支、云端分支。前四个在本地存储，最后一个在服务器存储，没有 git 服务器同样可以使用 `git` 做版本管理，只是不支持分布式，本地删掉之后代码就没有了。  
  其中 origin 就是远程仓库的名字，master 是远程分支的名字，当仓库只有一个分支的时候，origin/master 即代表了远端仓库的代码。

## 2. Git in action
使用 `git`，首先需要配置一下开发者用户名
```bash
git config --global user.name "yourname"
git config --global user.email "your.email@gmail.com"
git config --list
```

下面用几个命令，简单介绍 `git` 的最常见用法  
```bash
# 创建一个 git 仓库
git init    # 一般这个命令会在空文件夹下执行，此时工作区为空

# 创建一个新文件
touch readme.txt    # 此时磁盘代码发生了变动，工作区依然为空

# 查看分支状态, 将代码加入 git 仓库
    ## --> 如果文件是 new file，使用 git add <file> 将其加入仓库，这样文件同时进入了工作区和暂存区  
    git status
    git add readme.txt
    ## --> 如果文件是 modified，代表文件本来处于工作区，但是进行了改动，使用 git add <file> 将其加入暂存区，或者使用 git checkout -- <file> 撤销工作区改动。git diff <file> 可以查看文件的改动  
    echo "new changes" > readme.txt     # 现在工作区和暂存区中的文件是一致的。对文件进行改动
    git status
    git diff
    git add readme.txt  |  git checkout -- readme.txt

# 提交改动
# 使用 git commit 将 暂存区 的修改提交到 git 分支上，每一次 commit 对应 git 分支上的 一个节点。
git commit -m "<massage>"
or
git commit  # edit commit massage in vim

# 查看 log
git log

# 提交到远端仓库
    ## --> 如果本地是一个新仓库，则需要配置远端仓库地址，如果本地仓库是从远端 clone/pull 下来的，则不需要配置
    git remote add origin <url>  # url 示例 https://github.com/clover978/tsn-pytorch
git push origin master  # 将本地分支 push 到远端的 master 分支 （origin 代表远端仓库的地址）

# 从远程仓库拉取。代码提交到远端之后，就可以在别的地方拉取代码了
    ## --> 如果没有从远端拉取过代码，则使用 git clone 克隆一个新的代码仓库
    git clone <url>
    or 
    git remote add origin <url>  # 配置远端仓库地址，然后拉取
git pull origin master  # 拉取远端的 master 分支到本地
```

# Git commitzen
## 0.. Intro
[用工具思路规范化 git commit massage](https://github.com/pigcan/blog/issues/15)  
**`Git commitzen`** 是一个 规范 commit massage 的工具。  
掌握了 `git` 之后，自然也就知道 commit massage 是什么东西了。commit massage 需不需要规范化属于个人喜好问题，这篇笔记推荐一种目前流行的 commit massage 规范以及配置方法。

## 1.. Angular Git Commit Guidelines
```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>

type: 本次 commit 的类型，诸如 bugfix docs style 等
scope: 本次 commit 波及的范围
subject: 简明扼要的阐述下本次 commit 的主旨，在原文中特意强调了几点 1. 使用祈使句 2. 首字母不要大写 3. 结尾无需添加标点
body: 同样使用祈使句，在主体内容中我们需要把本次 commit 详细的描述一下，比如此次变更的动机，如需换行，则使用 |
footer: 描述下与之关联的 issue 或 break change
```

## 2. commitizen 
看完上面规范之后会不会感觉很麻烦，每次都要回忆该怎么写 commit massage。[commitzen](https://github.com/commitizen/cz-cli) 就是一个帮助规范化 commit massage 的工具。可以在 github 上欣赏一下该项目的 commits，或者 clone 到本地用 git log 看一下。  
下面介绍怎么使用 commitzen
  - 命令行安装  
    安装 commitzen-cli 需要先安装 npm
    ```bash
    # windows
    下载安装 node.js. https://nodejs.org/en/
    # linux
    apt install npm
    ```
    
    安装 commitzen-cli 及 conventional change log
    ```
    npm install -g commitizen
    npm install -g cz-conventional-changelog
    echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
    ```
    使用 `git cz` 代替 `git commit` 提交代码
    
    
  - ***VSCode（推荐***  
  如果使用 vscode 的话，可以直接安装 `Visual Studio Code Commitizen Support` 插件