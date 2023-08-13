---
title: SwiGLU激活函数
tags: Activation Function
categories: NLP
date: 2023-8-13 14:28:00
index_img: 
banner_img: 
math: true
---

> https://mltalks.medium.com/swiglu%E8%AE%BA%E6%96%87%E9%98%85%E8%AF%BB-4a9e2d55e250
>
> 谷歌大脑提出新型激活函数Swish惹争议：可直接替换并优于ReLU？（附机器之心测试） - 机器之心的文章 - 知乎 https://zhuanlan.zhihu.com/p/30332306



## SWISH激活函数

定义：
$$
f(x) = x \cdot \sigma(x)
$$

$$
\sigma(x)=(1+exp(-x))^{-1}
$$

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230813134234053.png)

SWISH激活函数的一次求导结果为：
$$
f'(x) = f(x) + \sigma(x)(1-f(x))
$$
![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230813134336831.png)

比ReLU好



## GLU（Gated Linear Unit）

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230813135619287.png)

GLU的定义：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230813135727628.png)



## SwiGLU

SwiGLU主要是为了提升transformer中的FFN层的实现

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230813135908336.png)