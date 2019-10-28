---
title: Hexo博客同步管理
author: GopherTy
top: true
cover: false
toc: true
date: 2019-10-27 18:18:32
coverImg:
password:
summary: 
tags: 
    - Hexo博客教程
    - 同步管理
categories: 教程
reprintPolicy:
---
# Hexo 博客同步管理

## Hexo 部署到 Github 的过程

我们通过 hexo init 命令会生成 Hexo 的源文件，将编写的 .md 文件部署 Github 时，实际上是通过 hexo g 命令将 .md 文件生成的对应的 .html 文件（自动创建 public 的文件夹下），然后 hexo d 时会在本目录下创建 .deploy_git 文件夹里面的内容也就是 public 文件夹下的静态文件，在 Github 上显示的内容也就是该文件夹下的静态文件。

## 实现思路

利用 Github 的分支管理可以完成同步操作，在 Github 上创建两个分支，一个分支存放我们要部署的静态文件（也就是我们可以展示的内容），另一个分支存放 Hexo 生成的源文件。Hexo 源文件通过 git 来管理，而我们要展示的部分通过 Hexo 来管理。

![](https://pic3.zhimg.com/80/v2-fac8f8564c4f1de0c54e3c142ae1f81d_hd.jpg)

## 操作过程

在 Github 上新建一个 hexo 分支，将该分支设为默认分支（目的是通过 git 管理 hexo 的源文件），然后在本地任意目录下进行 git clone 操作（也就是克隆 Hexo 的源文件），把除了.git 文件夹外的所有文件都删掉把之前我们写的博客源文件全部复制过来（除了 `.deploy_git` ）复制过来的源文件中有一个 `.gitignore`，用来忽略一些不需要的文件，如果没有的话，自己新建一个，在里面写上如下，表示这些类型文件不需要上传到 git 仓库中：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

**注意：**如果你之前克隆过 theme 中的主题文件，那么应该把主题文件中的`.git`文件夹删掉，因为 git 不能嵌套上传，最好是显示隐藏文件，检查一下有没有，否则上传的时候会出错，导致你的主题文件无法上传，这样你的配置在别的电脑上就用不了了。

然后就可以通过 git 将 Hexo 的源文件上传到 Github 中了。其中`node_modules`、`public`、`db.json`已经被忽略掉了，没有关系，不需要上传的，因为在别的电脑上需要重新输入命令安装 。

最后在其他电脑上只需要安装相应的环境后，在任意文件夹下将 Hexo 的源文件进行克隆（不需要 hexo init）然后执行如下命令即可:

```
安装 .gitignore 忽略上传的 Hexo 源文件

npm install
npm install hexo-deployer-git --save

编译，部署

hexo g
hexo d

然后就可以开始此电脑上进行工作了。
```

## Tips:

１．写完后最好吧源文件都上传一下。

２．如果在已经编辑过的电脑上，只需要和远端的仓库同步一下即可（`git pull`）。