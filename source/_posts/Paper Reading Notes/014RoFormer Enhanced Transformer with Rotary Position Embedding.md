---
title: RoFormer - Enhanced Transformer with Rotary Position Embedding
tags: Position Encoding
categories: Paper Reading Notes
date: 2023-08-09 16:46:00
index_img: 
banner_img: 
math: true
---

**标题：**《RoFormer: Enhanced Transformer with Rotary Position Embedding》

**论文来源：**  arXiv:2104.09864

**原文链接：** https://arxiv.org/pdf/2104.09864.pdf

**源码：** https://github.com/ZhuiyiTechnology/roformer



## 参考链接（建议按顺序阅读）

> [让研究人员绞尽脑汁的Transformer位置编码](https://kexue.fm/archives/8130)
>
> [Transformer升级之路：2、博采众长的旋转式位置编码](https://kexue.fm/archives/8265)
>
> [Transformer升级之路：4、二维位置的旋转式位置编码](https://kexue.fm/archives/8397)
>
> [Transformer升级之路：6、旋转位置编码的完备性分析](https://kexue.fm/archives/9403)
>
> [Transformer升级之路：10、RoPE是一种β进制编码](https://kexue.fm/archives/9675)
>
> [Transformer升级之路：12、无限外推的ReRoPE？](https://kexue.fm/archives/9708)



## 概述

RoPE：Rotary Position Embedding，旋转位置编码

目的是$Q,K$通过Attention后携带相对位置信息，也可以在$V$中添加，使得最终结果带有绝对位置信息，是Transformer式的Sinusoidal编码的一种优化，目前唯一一种可用于线性Attention的相对位置编码



## 定义

二维情况：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810164152570.png)

多维偶数维情况：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810164221414.png)

推荐实现方式：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810164311152.png)

可以看到，RoPE形式上和Sinusoidal位置编码有点相似，只不过Sinusoidal位置编码是加性的，而RoPE可以视为乘性的，其中在$\theta^i$的选择上，RoPE沿用了Sinusoidal位置编码的方案，即$\theta^i=10000^{-2i/d}$，它可以带来一定的远程衰减性

![RoPE的远程衰减性（d=128）](http://longls777.oss-cn-beijing.aliyuncs.com/img/1347893165.png)

最后，文章指出，RoPE是目前唯一一种可以用于线性Attention的相对位置编码。这是因为其他的相对位置编码，都是直接基于Attention矩阵进行操作的，但是线性Attention并没有事先算出Attention矩阵，因此也就不存在操作Attention矩阵的做法，所以其他的方案无法应用到线性Attention中。而对于RoPE来说，它是用绝对位置编码的方式来实现相对位置编码，不需要操作Attention矩阵，因此有了应用到线性Attention的可能性



尽管理论上RoFormer能处理任意长度的序列，但目前RoFormer还是具有平方复杂度的



未完待续...