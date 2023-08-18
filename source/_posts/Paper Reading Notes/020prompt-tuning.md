---
title: The Power of Scale for Parameter-Efficient Prompt Tuning
tags: fine tuning
categories: Paper Reading Notes
date: 2023-08-18 22:54:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818214122657.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818214122657.png
math: true
---

**标题：**《The Power of Scale for Parameter-Efficient Prompt Tuning》

**论文来源：** arXiv:2104.08691

**原文链接：** https://arxiv.org/abs/2104.08691



## 参考链接

> https://www.cnblogs.com/gogoSandy/p/17202169.html



## 概述

soft prompt，是prefix-tuning的改进，随着模型参数增加，能够“close the gap”，获得和finetune相似的效果

> 在SuperGLUE任务上，随着模型参数的上升，Prompt-Tunning快速拉近和模型微调的效果，110亿的T5模型（Prefix-tuning使用的是15亿的GPT2），已经可以打平在下游多任务联合微调的LM模型，并且远远的甩开了Prompt Design（GPT3 few-shot）

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818214122657.png)

## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818215002160.png)



给定token序列$\{ x_1,x_2,...,x_n \}$，LLM通过矩阵$X_e \in \mathbb{R}^{n \times e}$编码这些token，其中$e$是embedding dimension，soft prompt就是可训练的矩阵$P_e \in \mathbb{R}^{p \times e}$，其中$p$表示prompt的长度，两者拼接成$[P_e;X_e] \in \mathbb{R}^{(p+n) \times e}$，随后作为input输入model，训练时只训练$P_e$即可

> 与Prefix-tuning的差异在于prompt-Tunning只对输入层（Embedding）进行微调，而Prefix-tuning是对虚拟Token对应的上游layer全部进行微调，因此Prompt-Tunning的微调参数量级要更小，且不需要修改原始模型结构，这就是“简化”的来源

<img src="http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818221453512.png" alt="" style="zoom:70%;" />

相同的prefix长度，Prompt-Tunning(<0.01%)微调的参数量级要比Prefix-Tunning(0.1%~1%)小10倍以上





### Design Decisions

- 使用相关的token embedding初始化，比如分类任务就用类别的枚举
- prompt的长度，越小意味着要训练的参数越少，目标是寻找最短效果最好的length



## Experiments



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818220830523.png)

> 作者也做了全面的消融实验，包括以下4个方面，最核心的感受就是**只要模型足够够大一切都好说**
>
> - prompt长度（a）：固定其他参数，作者尝试了{1，5，20，100，150}，当模型规模到百亿后，只要prompt长度大于1，更长的prompt并不能带来效果提升
> - Prompt初始化（b）：作者尝试了随机uniform初始化，用标签文本空间初始化，和用Top5K高频词采样初始化，在10^8规模，标签词初始化效果最好。作者发现预测label也会在对应prompt空间内。不过到百亿规模后，初始化带来的影响就会消失
> - T5继续预训练（c）：作者认为T5本身的Span Corruption预训练目标和掩码词，并不适合冻结LM的场景，因为在微调中模型可以调整预训练目标和下游目标的差异，而只使用prompt可能无法弥合差异。其实这里已经能看出Encoder-Decoder框架在生成场景下没有GPT这样的Decoder来的自然。因此作者基于LM目标对T5进行继续预训练
> - 继续预训练step（d）：以上的继续预训练steps，继续预训练步数越高，模型效果在不同模型规模上越单调
>
> 考虑Prompt-Tunning使用Embedding来表征指令，可解释性较差。作者使用cosine距离来搜索prompt embedding对应的Top5近邻，发现：
>
> - embedding的近邻出现语义相似的cluster，例如{ Technology / technology / Technologies/ technological / technologies }，说明**连续prompt实际可能是相关离散prompt词的聚合语义**
> - 当连续prompt较长（len=100），存在多个prompt token的KNN相同：个人认为这和prefix-tuning使用MLP那里我的猜测相似，**prompt应该是一个整体**
> - 使用标签词初始化，微调后标签词也大概率会出现在prompt的KNN中，说明初始化可以提供更好的prior信息加速收敛
>
> 
