---
title: GLM && chatGLM
tags: prLMs
categories: Paper Reading Notes
date: 2023-08-11 14:06:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230811142624851.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230811142624851.png
math: true
---

**标题：**《GLM: General Language Model Pretraining with Autoregressive Blank Infilling》

**论文来源：** ACL 2022 

**原文链接：** https://arxiv.org/abs/2103.10360

**源码：** https://github.com/THUDM/GLM



**标题：**《GLM-130B: An Open Bilingual Pre-trained Model》

**论文来源：** ICLR 2023

**原文链接：** https://arxiv.org/abs/2210.02414 

**源码：** https://github.com/THUDM/GLM-130B





## 参考链接

> 【报告笔记】 大规模语言模型系列技术：以GLM-130B为例 - 严昕的文章 - 知乎 https://zhuanlan.zhihu.com/p/636329188
>
> https://keg.cs.tsinghua.edu.cn/glm-130b/zh/posts/glm-130b/



## GLM

使用自回归填空作为预训练目标，综合GPT，BERT，T5的优点，更好地解决NLG和NLU问题

![GLM架构](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230811142624851.png)

### 预训练目标

#### Autoregressive Blank Infilling

目标是训练模型的NLU能力

- 随机将一段token替换为一个[MASK]，替换的长度服从$\lambda=3$的Poisson分布
- 重复采样span，直到15%的tokens被mask
- 模型自回归地预测被mask的tokens
- span的预测顺序会被随机排布，类似排列语言模型（应该是通过位置编码实现的）



#### Multi-Task Pretraining

目标是训练模型的NLG能力

- 文档层面：对单个span进行采样，其长度是从超过原始长度 50%–100% 的均匀分布中采样的，目标是生成长文本
- 句子层面：限制mask span必须是完整的句子，同样覆盖tokens的15%。 该目标针对 seq2seq 任务，其预测通常是完整的句子或段落



### 模型架构

Transformer改型

- 重新安排了LN和残差连接的顺序（preNorm），参考https://arxiv.org/pdf/1909.08053.pdf中的模型结构
- 使用单个线性层输出token预测
- 使用GELU替换RELU

![左边是GLM的连接顺序，右边是原始Transformer](http://longls777.oss-cn-beijing.aliyuncs.com/img/12343545987543.png)

### 2D Positional Encoding

- 每个token有两个位置编码
- 一个表示了在原文本中的位置，第二个表示在span中的位置
- 通过可学习的embedding实现

>  The two positional ids are projected into two vectors via learnable embedding tables, which are both added to the input token embeddings.
>
> Our encoding method ensures that the model is not aware of the length of the masked span when reconstructing them. ... Our design fits downstream tasks as usually the length of the generated text is unknown beforehand.

优点是自回归的预测span时无需知道span的总长度，更符合实际的生成任务



### Finetuning GLM

对于NLU任务，遵循PET（PatternExploiting Training，https://aclanthology.org/2021.eacl-main.20/），使用完形填空任务来微调

对于NLG任务，给定的上下文构成了输入的A部分，末尾附加了一个掩码标记。模型自回归地生成B部分的文本



### Experiments

![NLU能力](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230811151017255.png)

![NLG能力](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230811151127770.png)



NLU上效果超越BERT，NLG上效果超越T5，基本和BART持平



## GLM-130B

