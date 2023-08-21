---
title: Hexo一键部署脚本
tags: 
categories: Others
date: 2023-7-20 16:38:00
index_img: 
banner_img: 
math: true
---



一遍一遍敲命令太麻烦了，我怎么之前就没想到写个脚本



```sh
cd z: && cd hexo_ && hexo clean && hexo g && hexo d && git add . && git commit -m "7" && git push origin hexo_
```



把这个保存为一个.sh文件，然后放在桌面直接点击即可



效果是自动更新生成hexo文件并部署到服务器，之后保存源文件到git仓库（通常仓库master分支为hexo生成的静态页面文件，另外新建一个分支放源文件，比如我的hexo_）



注意需要安装git bash，以及目录和git分支需要改成自己的