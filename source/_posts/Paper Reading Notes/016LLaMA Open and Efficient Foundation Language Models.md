---
title: LLaMA - Open and Efficient Foundation Language Models
tags: LLMs
categories: Paper Reading Notes
date: 2023-08-09 15:34:00
index_img: 
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/Tanganyika_Alpaca_2021_CS2-683x1024.jpg
math: true
---

**标题：**《LLaMA: Open and Efficient Foundation Language Models》

**论文来源：** arXiv:2302.13971

**原文链接：**  https://arxiv.org/pdf/2302.13971.pdf

**源码：** https://github.com/facebookresearch/llama



## 参考链接

> https://blog.csdn.net/weixin_44826203/article/details/129255185
>
> [为什么Pre Norm的效果不如Post Norm？](https://kexue.fm/archives/9009)
>
> [Transformer升级之路：2、博采众长的旋转式位置编码](https://kexue.fm/archives/9009)



## 概述

用的Transformer的decoder，和GPT很类似

## 模型细节

### RMS Pre-Norm

> https://arxiv.org/pdf/1910.07467.pdf



首先LLama采用了RMS Norm，是LayerNorm的一种变体，作用是可以在梯度下降时令损失更加平滑，具体细节不讲



Pre-Norm和Post-Norm哪个更好呢？[为什么Pre Norm的效果不如Post Norm？](https://kexue.fm/archives/9009)这篇文章给出了解释：

重点是在这个式子

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230809161116042.png)

- 同一设置之下，Pre Norm结构往往更容易训练，但最终效果通常不如Post Norm
- Pre Norm结构无形地增加了模型的宽度而降低了模型的深度，而我们知道深度通常比宽度更重要，所以是无形之中的降低深度导致最终效果变差了

<img src="http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230809160712945.png" style="zoom:15%;" />



但是为什么LLaMA中却采用了Pre-Norm，这个还没有找到合理的解释



### SwiGLU激活函数

> https://arxiv.org/pdf/2002.05202.pdf



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230809161453203.png)



### RoPE旋转位置编码

RoPE（Rotary Position Embedding）旋转位置编码，是苏剑林老师提出的一种旋转位置编码方法[Transformer升级之路：2、博采众长的旋转式位置编码](https://kexue.fm/archives/8265)，其思想是采用绝对位置编码的形式，实现相对位置编码

