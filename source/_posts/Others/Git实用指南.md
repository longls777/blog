---
title: Git实用指南
tags: 
categories: Others
date: 2023-9-22 16:47:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/62a6360f03a870b920cbfd41.png
banner_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230922174528787.png
math: true
---

## Git常用命令

克隆仓库到本地，注意如果没有在仓库配置SSH密钥，没有办法鉴权，无法通过`git@xxxxx`链接clone，只能使用`http....`链接

`git clone xxxxxxxx`

获取当前分支关联的远程分支的最新更改：

`git pull`

提交修改文件到git的暂存区：

`git add xxx`

通常提交所有文件：

`git add .`

提交暂存区文件到本地仓库，`xxx`一般是自己工作的简短描述，类似`add xxx feature`，意味添加`xxx`功能：

`git commit -m "xxx"`

推送到远程仓库，`remote_dev_branch`为远程分支名：

`git push origin remote_dev_branch`

推送到远程关联分支：

`git push`

merge分支，在合并后的分支下执行，意为将其他分支merge进本分支：

`git merge branch_name`



## Git一般开发流程

首先从远程**clone**开发仓库：

`git clone xxxxx`

此时本地分支一般是主分支**main**或**master**，新建自己的开发分支，例如**temp**：

`git checkout -b temp`

开发代码后，先提交到本地仓库，再push到远程的对应仓库：

`git add .`

`git commit -m "xxxxxx(comment)"`

`git push origin temp`

第一次push之后，本地**temp**分支和远程**temp**分支已关联，再次开发可直接执行：

`git push`

## Git合并分支

当需要和copartner同步进度时，可以选择把对方的分支merge进自己的分支（或者相反）

首先保证两个分支**temp**和**temp_c**（copartner的开发分支）在本地都更新到最新（在对应的分支下执行git pull）

切换到自己的分支**temp**：

`git checkout temp`

执行merge操作：

`git merge temp_c`

这时一般有很多合并冲突，需要在代码里解冲突（冲突文件在IDE里一般会标红），冲突的一般格式是：

```python
<<<<<<< HEAD
// 这里是当前分支的内容
=======
// 这里是被合并分支的内容
>>>>>>> temp_c
```

需要自己选择保留或修改哪部分，解完冲突之后再次提交：

`git add .`

`git commit -m "xxxxx"`

提交到自己的远程分支**temp**：

`git push`

或者可以提交到别的分支，这里只描述一种使用流程，更多的用法请自行探索