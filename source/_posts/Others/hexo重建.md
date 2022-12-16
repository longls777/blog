---
title: hexo出现问题后重建流程
date: 2022-12-16 19:15:31
index_img: 
banner_img: 
categories: Others
math: true
---

## 一、初始化hexo

按照hexo建立博客流程

- 安装node.js
- 安装hexo
- 初始化hexo文件

建立一个初始化的hexo本地网站

## 二、安装fluid博客主题

安装流程可见fluid官网

[fluid github主页](https://github.com/fluid-dev/hexo-theme-fluid)

[fluid 用户手册](https://hexo.fluid-dev.com/docs/)

[fluid 配置指南](https://hexo.fluid-dev.com/docs/guide/)

## 三、复制原文章和配置文件

从原博客复制source文件夹以及_config.fluid.yml和config.yml配置文件，替换新hexo博客中的对应文件

## 四、替换fluid主题的图片

将原博客根目录下的图片复制到node_modules\hexo-theme-fluid\source\img文件夹下

## 五、生成并推送至GitPages

可能需要先安装hexo deploy相关的git组件

`npm install --save hexo-deployer-git`

然后hexo clean & hexo g & hexo d，生成并部署

## 六、将文件夹与github仓库关联，建立新的开发分支

`git remote add origin git@github.com:longls777/heart.github.io.git`

建立新的开发分支并推送至仓库