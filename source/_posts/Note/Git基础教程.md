---
title: Git基础教程
date: 2018-11-20
tags: [Note,git]
categories:
- [Note,Git]
comments: false
---

# Git基础教程

Git 是一个分布式版本控制工具，最初由Linux之父Linus Torvalds创作，后托付滨野纯作为软件维护者。

详细历史见维基 [git wiki](https://zh.wikipedia.org/wiki/Git)

<!-- more -->

## Git的基本功能

[![Git脑图](https://study-resources-1256288674.cos.ap-chengdu.myqcloud.com/Git-mindMapping.png)](https://study-resources-1256288674.cos.ap-chengdu.myqcloud.com/Git-mindMapping.png)

### 项目管理

#### 初始化项目

```
git init
```

在项目根目录内使用此命令即将项目初始化为一个git仓库

#### 下载远程项目

```
git clone [option] URL [dirname]
```

- `[dirname]` 设置新建目录的名字
- option:
  - `-b <branch>` 克隆指定分支

### 提交修改

Git 有一个用于存储修改的暂存区，提交一次修改的过程首先需要添加文件到暂存区，然后再进行提交（commit），并填写提交记录

#### 跟踪文件 && 添加修改到暂存区

```
git add FILE
```

- `FILE` 需要添加到暂存区的文件，使用`.`来指代工作区全部文件

第一次添加的文件在之前是未跟踪（untracked）的状态，使用命令后会跟踪文件并将文件添加到暂存区；

已存在但是发生了修改的文件会添加进暂存区。

#### 查看暂存区状态

```
git status # 查看暂存区状态
git diff # 查看工作区与暂存区的区别
```

git diff若暂存区无修改则显示与当前版本的区别；也可使用`git diff --cached`来查看。

#### 提交暂存区到版本

```
git commit [options]
```

options:

- `-a` 提交全部未暂存的修改
- `-m MESSAGE` 使用短字符串作为提交记录，不适用此参数则会进入系统编辑器编写commit记录
- `--amend` 重新提交最近一次的修改记录

### 分支

#### 分支管理

```
git branch [options] [NAME]
```

options:

- `-a` 显示所有分支（本地和远程）
- `-d` 删除分支（已合并）
- `-D` 强制删除分支
- `-v` 显示最近一次提交

#### 检出分支

```
git checkout [options]
```

检出到指定分支（切换分支），同时会撤销当前分支上未提交的所有修改（回退到当前分支的最新版本）

options:

- `<file>` 撤销某个文件的修改
- `[branch_name]` 切换分支
- `-b [branch_name]`新建并切换分支
- `-b [branchname] [tagname]` 在特定标签下创建分支

可以使用checkout来快速撤销当前分支所有修改

```
git checkout .
```

#### 合并分支

```
git merge NAME # 将指定分支合并到当前分支
```

#### 常用分支名称

- master 主分支，稳定分支，默认分支
- dev 开发版本分支
- fix#4 修复问题时的临时分支

### 标签

Git可以给历史中的某一个提交打上标签，以示重要

比较有代表性的是用来标记发表结点（版本号）

```
git tag # 列出标签
git tag -l 'pattern' # 用特定模式查找标签
git show TAG # 查看标签信息与对应的提交信息
```

git标签分为**轻量标签**和**附注标签**两种

#### 轻量标签

像一个不会改变的分支（branch）- 它只是一个特定提交的引用。

#### 附注标签

是存储在Git数据库中的一个完整对象。

其中包含：

- 标签者的名字
- 电子邮件地址
- 日期时间
- 标签信息

它们可以使用GPG签名与验证

------

通常建议创建附注标签（信息量大），但如果你只是想用一个临时的标签，，或者因为某些原因不要保存那些信息，就使用轻量标签

#### 创建标签

`git tag -a <tagname> -m 'comment'` 创建一个附注标签

- `-m` 选项指定了将会存储在标签中的信息，如果没有指定，git将会运行编辑器要求你输入信息

`git tag <tagname>` 创建一个轻量标签

不需要使用`-a`、`-s`或`-m`选项，只需要提供标签名字

为之前的提交补充标签

```
git tag -a <tagname> <id>
```

id为要补充标签的提交的校验和（或部分校验和）

#### 推送标签

```
git push <remote> <tagname>
```

默认情况下，`git push`命令并不会推送标 签到远程仓库。在创建完标签后你必须显式地推送标签，过程和推送远程分支一样

如果想要一次推送很多标签，也可以使用带有`--tags`选项的`git push`命令。这将会把所有不在远程仓库的标签全部推送到远程仓库

#### 检出标签

Git 中并不能真的检出一个标签，因为它们并不能像分支一样来回移动。如果你想要工作目录与仓库中特定的标签版本完全一样，可以使用

```
git checkout -b <branchname> <tagname>
```

这将会在特定的标签上创建一个新分支

但是如果在这个分支上又进行了新提交，那么这个分支就会因为改动而向前移动导致和标签有所不同。

### 远程管理

#### 远程仓库

```
git remore add origin URL # 添加远程仓库为origin
git remote rm origin # 删除远程仓库origin

git branch -r # 查看远程仓库的分支
```

#### 推送

```
git push origin dev:dev # 将本地dev分支推送到origin/dev
git push REMOTE :BRANCH # 将空分支提交到远程分支，即删除远程分支
git push --upstream origin master # 使当前分支跟踪远程分支
```

#### 拉取

```
git fetch origin master # 拉取远程仓库origin的master分支并存到分支origin/master中
git merge origin/master # 将指定远程分支合并到当前分支中

git pull origin master # 合并fetch与merge两个操作
```

### 配置文件

```
git config --list # 查看配置文件
git config --list --global #
git config add
```

## 项目管理流程

[![Git脑图](https://study-resources-1256288674.cos.ap-chengdu.myqcloud.com/Git-mindMapping.png)](https://study-resources-1256288674.cos.ap-chengdu.myqcloud.com/Git-mindMapping.png)

### 项目提交

1. 初始化/下载项目

   ```
   git init
   git clone url
   ```

2. 创建修改

3. 添加修改到暂存

   ```
   git add filename
   ```

4. 提交暂存

   ```
   git commit -m "log"
   ```

### 远程协作

1. 拉取 & 合并远程分支

   ```
   git fetch origin master # origin/master
   git merge origin/master
   
   git pull origin master
   ```

2. 推送到远程仓库

   ```
   git push origin master
   ```

3. 合并提交 & 变基

   ```
   git fetch origin master
   git rebase origin/master
   ```

### 版本管理

1. 常驻分支

   master

   development

   \* fix#1000

2. 为版本打标签

3. [版本号规范](https://github.com/1000Delta/A11N0tes/blob/master/软件工程/软件版本号.md)

4. [优雅的使用commit](https://juejin.im/post/5afc5242f265da0b7f44bee4)