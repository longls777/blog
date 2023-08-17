---
title: LORA - LOW-RANK ADAPTATION OF LARGE LANGUAGE MODELS
tags: fine tuning
categories: Paper Reading Notes
date: 2023-08-17 19:19:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812164721483.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812164721483.png
math: true
---

**标题：**《LORA: LOW-RANK ADAPTATION OF LARGE LANGUAGE MODELS》

**论文来源：** arXiv:2106.09685

**原文链接：** https://arxiv.org/pdf/2106.09685.pdf

**源码：** https://github.com/microsoft/LoRA



## 参考链接

> https://huggingface.co/blog/lora
>
> 大模型微调（finetune）方法总结-LoRA,Adapter,Prefix-tuning，P-tuning，Prompt-tuning - YBH的文章 - 知乎 https://zhuanlan.zhihu.com/p/636481171



## 概述

具有超大参数的LLM如何应用到下游任务呢？

可以通过添加一些额外参数，冻结LLM参数，训练这些额外参数来适配下游任务，但是由于增加了模型深度或者减少了模型可用序列长度（soft prompt），模型推理会有延迟，而且一般没有finetuning的效果好



有研究表明学习到的过参数化的模型实际上建立在低秩维度上，因此LORA假设模型权重在微调的过程中也有一个较低的内在维度

LORA是在神经网络中直接训练一些dense层来拟合LLM微调过程中的**秩分解矩阵**，在这个过程中冻结pre-trained LLM的参数

LORA在推理的时候和LLM的参数一同推理，与全参数微调相比不会带来太多的推理延迟

> Our simple linear design allows us to merge the trainable matrices with the frozen weights when deployed, introducing no inference latency compared to a fully fine-tuned model, by construction.



## Method

![Lora](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812164721483.png)

对于pre-trained权重矩阵$W_0 \in \mathbb{R}^{d \times k}$，微调过程中存在$W_0 + \Delta W=W_0 + BA$，其中$B \in \mathbb{R}^{d \times r}$ ，$A \in \mathbb{R}^{r \times k}$，并且$r \ll min(d,k)$
$$
h = W_0x + \Delta Wx=W_0x + BAx
$$
在初始化的过程中，对于$A$采用随机高斯初始化，对于$B$采用0初始化，所以在刚开始$\Delta W = BA$为0矩阵

LORA还通过$\frac{\alpha}{r}$来缩放$\Delta Wx$，其中$\alpha$是$r$的一个常数，当使用Adam优化器时，微调$\alpha$相当于微调学习率，所以LORA直接将$\alpha$设为尝试的第一个$r$，并且不再微调它。这个缩放有助于减少在改变 $r$ 时重新调整超参数的需要

- 当把$r$设置为pre-trained参数矩阵的秩时，LORA可以实现和全参数微调基本一致的效果
- 推理时，通过$W=W_0+BA$进行推理，与构造微调的模型（添加额外训练层）相比，LORA不会引入额外推理延迟



## Results

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817170459131.png)



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817170525889.png)



LORA效果甚至比全参数微调效果更好，而且也优于其他的微调方法



## WHICH WEIGHT MATRICES IN TRANSFORMER SHOULD WE APPLY LORA TO？

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817170820836.png)

- 单独对$W_q$或者$W_k$使用LORA效果不好
- 对$W_q$和$W_v$一起使用LORA效果好
- 即使$r=4$，也能捕获足够的信息
- 对多个权重矩阵使用较小的$r$进行LORA微调，优于对单独的权重矩阵使用较大的$r$进行LORA微调要好



## WHAT IS THE OPTIMAL RANK r FOR LORA?

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817171214092.png)



非常小的$r$效果已经很不错了，$r$的增加并不能覆盖更有意义的子空间

