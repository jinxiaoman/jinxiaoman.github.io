---
layout:     post
title:      webstrom里的git操作
subtitle:   webstrom工具学习
date:       2017-10-17
author:     jinxm
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - webstrom
---
# 前言

git的使用方式很多种，王道就是用命令行。现在从新认识webstrom工具，先从常用的git开始。命令行熟练度可以的话那速度是无敌的。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fkl8wuuvebj30m80cv7ae.jpg)

git操作主要分类
1. 远程仓库类
2. 分支类
3. 查询类
4. 本地操作类

# git选项的含义
1. Commit File      提交到本地版本库
2. Add               添加进缓存区
3. Annotate
3. Show Current Revision    显示当前文件的最新版本信息
4. Compare with the Same Repository Version   与当前版本比较（最新版本，比较的是本地工作区和本地版本的差异）
5. Compare with Latest Repository Version     与上一版本比较。当前版本与本地工作区所做的更改共同与上一版本比较
6. Compare with        于任意历史版本作比较
7. Compare with Branch   与任意分支作比较，包括本地以及远程分支
8. Show History         展示关联文件的提交 信息历史
9. Show History for Selection   对指定的代码块，显示历史版本信息
10. Revert Change     还原代码，将本地工作区的代码还原为本地仓库中的最新版本代码
11. Repository       仓库的二级选项，相关于仓库类的

# 二级菜单
1. Branches  显示分支，包括本地分支和远程分支
1.1 checkout  检出至本地工作区。此时本地已经验出过
1.2 checkOut as new local branch   检出至本地工作区，并创建新的分支
1.3 compare    两个分支进行比较，
1.4 merge       进行合并操作，可能会有冲突
1.5 delete      删除当前分支
2. Tag  打 tag
3. Merge Changes   合并操作

# Version Control 面板
1. Local Change   工作区的更改与当前版本的差异列表，Unversioned File 表示没有加入到版本管理列表  Default中是版本管理中经过本地修稿的文件。
2. 合并冲突。

# 现在webstrom操作git更方便，结合修改远程仓库基本完美
	1. 修改命令
	git remote set-url origin url
	2. 删除在添加 一直用这个
	git remote rm origin
	git remote add origin 你的git地址

## 查看远程仓库地址
   `git remote -v`

>暂时就用这么多吧
