---
title: Prefix-Tuning - Optimizing Continuous Prompts for Generation
tags: fine tuning
categories: Paper Reading Notes
date: 2023-08-17 19:19:00
index_img: 
banner_img: 
math: true
hide: true
---

**标题：**《GPrefix-Tuning: Optimizing Continuous Prompts for Generation》

**论文来源：** arXiv:2101.00190

**原文链接：** https://arxiv.org/abs/2101.00190

**源码：** https://github.com/XiangLi1999/PrefixTuning



## 概述

全参数微调需要为每个下游任务微调specific model，本文提出了Prefix-tuning的方法，可以冻结LLM的参数，对于不同的下游任务只训练一段特殊的前缀即可，大大减少了要训练的参数量

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818134205878.png)

## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818134538365.png)

- 对于自回归模型——【prefix， X， Y】
- 对于Encoder-Decoder模型——【prefix， X， prefix'， Y】

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818134917380.png)



根据经验，直接优化$P_{\theta}$会导致不稳定的优化和轻微的效果下降，所以作者将$P_{\theta}$重新参数化：
$$
P_{\theta}[i,:]=MLP_{\theta}(P^{'}_{\theta}[i,:])
$$
其中$P^{'}_{\theta}[i,:]$是一个很小的矩阵，一旦训练完成，就可以只保留$P_{\theta}$了，换句话说，$P^{'}_{\theta}[i,:]$和$MLP_{\theta}(\cdot)$都只是为了稳定优化过程的辅助结构，不参与最终的推理过程



## Experiments

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818135650921.png)



## Intrinsic Evaluation

### Prefix Length

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818135857948.png)

越长的prefix代表着越多的可训练参数，根据实验，随着prefix长度的增加，模型效果先上升再下降

> Empirically, longer prefixes have a negligible impact on inference speed, because attention computation over the entire prefix is parallellized on GPUs.



### Full vs Embedding-only

仅仅调整Embedding层（EMB），效果也不咋地，作者称这种方式为离散prompt的上限，因为离散prompt的token也都是从vocab space中获取的

> Consequently, we have this chain of increasing expressive power: **discrete prompting < embedding-only ablation < prefix-tuning**.



###  Prefixing vs Infixing

- Prefixing ： 【prefix， X， Y】
- Infixing ： 【X，prefix， Y】

Infixing效果比Prefixing略差，作者解释是Prefixing影响了X和Y，而Infixing只能影响Y（aaron：合理）



### Initialization

作者发现prefix如何初始化在低数据量时影响很大，通过使用真实单词的激活来初始化前缀，可以显著改善生成效果

> We find that how the prefix is initialized has a large impact in low-data settings. Random initialization leads to low performance with high variance. Initializing the prefix with activations of real words significantly improves generation, as shown in Figure 5.

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818141103198.png)

> In particular, initializing with task relevant words such as “summarization” and “table-to-text” obtains slightly better performance than task irrelevant words such as “elephant” and “divide”, but using real words is still better than random.（aaron：true word好歹带点语义，random就是纯随机碰运气了）



## Discussion

优点：

- Personalization：例如为了保护用户隐私，每个用户都可以训练单独的prefix，用户个性化设置
- Batching Across Users：prefix-tuning允许批处理不同用户的查询，即使这些查询使用不同的前缀支持
- Inductive Bias of Prefix-tuning：保留LM参数可能有助于泛化到训练期间未见过的领域（外推能力好），这与直觉一致





> 和adapter-tuning的比较：
>
> Moreover, we observe that **prefix-tuning** requires vastly fewer parameters compared to **adapter-tuning** while maintaining comparable performance. We think this gain in parameter efficiency is **because prefix-tuning keeps the pretrained LM intact as much as possible**, and therefore exploits the LM more than adapter-tuning.





> LLM过参数化，主要是内在低维发挥作用，这也解释了为什么只用很小的参数微调（prefix）就能获得很好的效果：
>
> Concurrent work by Aghajanyan et al. (2020) uses intrinsic dimension to show that there exists a low dimension reparameterization that is as effective for fine-tuning as the full parameter space. This explains why good accuracy on downstream task can be obtained by updating only a small number of parameters. Our work echoes the finding by showing that good generation performance can be attained by updating a very small prefix.
>
> Aghajanyan et al. (2020)这篇文章就是LoRA获得灵感的文章之一，可以说这篇文章也为prefix-tuning提供了理论支持
>
> 这篇文章的链接如下：
>
> [Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning](https://arxiv.org/abs/2012.13255)

