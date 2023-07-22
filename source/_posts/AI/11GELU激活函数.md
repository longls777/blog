---
title: GELU激活函数
tags: 
categories: AI
date: 2023-7-22 20:51:00
index_img:
banner_img:
math: true
---

> BERT中的激活函数GELU：高斯误差线性单元 - 清欢鱼的文章 - 知乎 https://zhuanlan.zhihu.com/p/349492378
>
> 为什么GELU激活函数越来越常见？ - 利先生的文章 - 知乎 https://zhuanlan.zhihu.com/p/609897428
>
> 深度学习中激活函数的导数在不连续可导时的处理 - puppy的文章 - 知乎 https://zhuanlan.zhihu.com/p/101321997

## 定义

GELU（Gaussian Error Linear Unit，高斯误差线性单元）于2016年由Hendrycks和Gimpel在论文《Gaussian Error Linear Units (GELUs)》中提出。

与其他常用的激活函数（如ReLU和sigmoid）相比，GELU具有更平滑的非线性特征，这有助于提高模型的性能。GELU可以看作是一种sigmoid和ReLU的混合体。



## 优点

- 在处理负数时不会像ReLU一样将输入裁剪到0，这可能导致梯度消失的问题
- 在大多数情况下比sigmoid和tanh表现更好，因为它在处理较大的输入时，会比sigmoid和tanh产生更强的非线性响应
- 在处理较小的输入时，又比ReLU表现更好，因为它有一个非零的梯度
- GELU函数的导数是连续的，这使得在训练深度神经网络时可以更容易地传播梯度，避免了ReLU函数在 $x=0$ 处的导数不连续的问题，从而减少了训练过程中出现的梯度消失问题



## 延伸问题：激活函数的导数在不可导处怎么处理？

Q: 深度学习中激活函数在不连续可导时的导数怎么处理呢？

**A: 激活函数不要求处处连续可导，在不连续可导处定义好该处的导数即可。**

sigmoid函数是处处连续可导的。其他如ReLU，在0处不连续可导。

**以caffe中的ReLU为例：**

在caffe中，给定输入x，ReLU层可以表述为：

$f(x) = x$, if $x>0$;

$f(x) = negative\_slope * x$, if $x <=0$.

当$negative\_slope>0$时，ReLU是leaky ReLU

$negative\_slope$默认为0， 即标准ReLU

输入$x=0$时，导数为$negative\_slope$（标准ReLU在$x=0$处的导数即为0）