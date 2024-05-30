---
title: LLMs fine tuning方法总结
tags: fine tuning
categories: NLP
date: 2023-8-17 20:17:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817203959034.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817203959034.png
math: true
---

> 大模型微调（finetune）方法总结-LoRA,Adapter,Prefix-tuning，P-tuning，Prompt-tuning - YBH的文章 - 知乎 https://zhuanlan.zhihu.com/p/636481171



# LoRA

自然语言处理目前存在一个重要范式：一般领域数据的大规模预训练，对特定任务或领域的适应（finetune）。但是随着预训练语言模型越来越大，这个范式存在以下问题：

- 当我们finetune大模型时，由于训练成本太高，不太可能重新训练所有模型参数
- 以前的方法都或多或少有其它性能问题，如adapter增加了模型层数，引入了额外的推理延迟；prefix-tuning比较难训练，效果不如直接finetune

基于上述背景，论文作者得益于前人的一些关于内在维度（intrinsic dimension）的发现：模型是过参数化的，它们有更小的内在维度，模型主要依赖于这个低的内在维度（low intrinsic dimension）去做任务适配。假设模型在任务适配过程中权重的改变量是低秩（low rank）的，由此提出低秩自适应（LoRA）方法，LoRA允许我们通过优化适应过程中dense层变化的秩分解矩阵来间接训练神经网络中的一些密集层，同时保持预先训练的权重不变

![Lora](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812164721483.png)

LoRA的实现思想很简单，就是冻结一个prLM的矩阵参数，并选择用A和B矩阵来替代，下游任务微调时只训练A和B，推理时prLM和LoRA矩阵一起推理



## Adapter

> [Parameter-Efficient Transfer Learning for NLP](https://arxiv.org/pdf/1902.00751.pdf)
>
> [MAD-X: An Adapter-Based Framework for Multi-Task Cross-Lingual Transfer](https://arxiv.org/pdf/2005.00052.pdf)

2019年，Houlsby N等人将Adapter引入NLP领域，作为全模型微调的一种替代方案。Adapter主体架构下图所示

![Adapter](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817203959034.png)

在预训练模型每一层（或某些层）中添加Adapter模块（如上图左侧结构所示），微调时冻结预训练模型主体，由Adapter模块学习特定下游任务的知识。每个Adapter模块由两个前馈子层组成，第一个前馈子层将Transformer块的输出作为输入，将原始输入维度d投影到m，通过控制m的大小来限制Adapter模块的参数量，通常情况下m<<d。在输出阶段，通过第二个前馈子层还原输入维度，将m重新投影到d，作为Adapter模块的输出（如上图右侧结构）。通过添加Adapter模块来产生一个易于扩展的下游模型，每当出现新的下游任务，通过添加Adapter模块来避免全模型微调与灾难性遗忘的问题。Adapter方法不需要微调预训练模型的全部参数，通过引入少量针对特定任务的参数，来存储有关该任务的知识，降低对模型微调的算力要求



##  Prefix-tuning

> [Prefix-Tuning: Optimizing Continuous Prompts for Generation](https://arxiv.org/pdf/2101.00190.pdf)
>
> [GitHub - XiangLi1999/PrefixTuning: Prefix-Tuning: Optimizing Continuous Prompts for Generation](https://github.com/XiangLi1999/PrefixTuning)

前缀微调（prefix-tunning），用于生成任务的轻量微调。前缀微调将一个连续的特定于任务的向量序列添加到输入，称之为前缀，如下图中的红色块所示。与提示（prompt）不同的是，前缀完全由自由参数组成，与真正的token不对应。相比于传统的微调，前缀微调只优化了前缀。因此，我们只需要存储一个大型Transformer和已知任务特定前缀的副本，对每个额外任务产生非常小的开销

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817210525153.png)



## P-tuning

>  [GPT Understands, Too](https://arxiv.org/abs/2103.10385)
>
> [GitHub - THUDM/P-tuning: A novel method to tune language models. Codes and datasets for paper ``GPT understands, too''.](https://github.com/THUDM/P-tuning)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817211156195.png)

P-tuning是稍晚些的工作，主要针对NLU任务

- 对于BERT类双向语言模型采用模版（P1, x, P2, [MASK], P3）
- 对于单向语言模型采用（P1, x, P2, [MASK]）

同时加了两个改动：

- 考虑到预训练模型本身的embedding就比较离散了（随机初始化+梯度传回来小，最后只是小范围优化），同时prompt本身也是互相关联的，所以作者先用LSTM对prompt进行编码
- 在输入上加入了anchor，比如对于RTE任务，加上一个问号变成[ PRE ] [  prompt tokens  ] [  HYP  ] ? [  prompt tokens  ] [ MASK ]后效果会更好

P-tuning的效果很好，之前的Prompt模型都是主打小样本效果，而P-tuning终于在整个数据集上超越了精调的效果：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817212600189.png)



## Prompt-tuning

> [The Power of Scale for Parameter-Efficient Prompt Tuning](https://arxiv.org/abs/2104.08691)

Prompt-tuning给每个任务定义了自己的Prompt，拼接到数据上作为输入，同时freeze预训练模型进行训练，**在没有加额外层的情况下**，可以看到随着模型体积增大效果越来越好，最终追上了精调的效果：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817212740069.png)

同时，Prompt-tuning还提出了Prompt-ensembling，也就是在一个batch里同时训练同一个任务的不同prompt，这样相当于训练了不同「模型」，比模型集成的成本小多了