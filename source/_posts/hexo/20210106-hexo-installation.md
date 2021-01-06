---
title: hexo installation
date: 2021-01-06 09:32:59
tags: 
  - hexo
categories:
  - hexo
---

---
## 1. 安装 git, node.js, Hexo
  参考[ Hexo 官方教程](!https://hexo.io/zh-cn/docs/)


---
## 2. 添加 github pages
- 新建 github repository  
- 新建 hexo 分支，并设置为默认分支
- 添加 ssh hey
    ```
    ssh-keygen -t rsa
    cat .ssh\id_rsa.pub
    ```
  在 `settings -> SSH and GPG keys -> New SSH key` 里面添加公钥。
- 新建 hexo 博客
    ```
    git clone clover978.github.io
    hexo init clover978.github.io
    npm install hexo-deployer-git --save
    ```
  在 `clover978.github.io/_config.yml` 里设置如下字段：  
    ```
    deploy:
    type: 'git'
    repo: https://github.com/clover978/clover978.github.io
    branch: master
    ```
<!-- more -->



---
## 3. 安装 theme
```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```


## 4. 其他设置
  参考 `_config.yml` 和 `themes/next/_config.yml` 里的各种字段，设置页面布局。


## 5. 发布 blog
  - 编写 blog
    ```
    hexo new <title>
    # 编辑 _post 里面生成 md 文件
    hexo g
    hexo d
    ```
- 同步 hexo
    ```
    git add .
    git commit -m "add new blog"
    git push
    ```