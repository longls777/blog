---
title: Training language models to follow instructions with human feedback
tags: LLMs
categories: Paper Reading Notes
date: 2023-08-18 14:29:00
index_img: 
banner_img: 
math: true
hide: true
---

**标题：**《Training language models to follow instructions with human feedback》

**论文来源：** arXiv:2203.02155

**原文链接：** https://arxiv.org/abs/2203.02155



## 参考链接

> https://cloud.tencent.com/developer/article/2216036



## 概述

InstructGPT和后面的ChatGPT，都是OpenAI在**大模型alignment问题**上的研究成果，什么是模型的alignment呢？在InstructGPT论文中作者是这么说的：

> “For example, large language models can **generate outputs that are untruthful, toxic, or simply not helpful to the user**. In other words, these models are not **aligned** with their users. 
>
> ChatGPT翻译：大型语言模型可以生成不真实、有毒、或者对用户没有帮助的输出。换句话说，这些模型与用户不匹配。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230819134617723.png)

1.3B的PPO-ptx model比175B的GPT-3表现得更好

- instructGPT以GPT3为基础
- chatGPT以GPT3.5为基础

## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230819134840589.png)

### supervised fine-tuning (SFT)

人工给prompt打上人类偏好的输出标签，然后用这些数据监督微调GPT-3



### reward model (RM) training

收集模型在相同prompt下的多个输出，人工给这些输出排序，然后用排序后的数据训练奖励模型RM

> RM模型也是基于GPT-3进行训练的，使用的是6B的版本，具体就是在进行SFT训练之后，把最后的embedding层去掉，改成输出一个标量



### reinforcement learning via proximal policy optimization (PPO)

使用RM的输出作为奖励分数，使用PPO算法优化模型



## Data

![=](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230819142529044.png)

- **SFT数据集**（即第一步人类根据prompt自己写理想的输出，SFT：supervised fine-tuning），包含**13K**的prompts；
- **RM数据集**（即第二步用来训练打分模型的数据），包含**33K**的prompts；
- **PPO数据集**（即第三步用来训练强化学习PPO模型的数据），包含**31K**的prompts。



## Conclusion

- 这种“调教”，会降低模型在常见NLP任务上的效果，作者称之为“对齐税”——alignment tax（实际上之前很多研究都发现了这个问题）。但是，通过改善RLHF的过程，比如在预训练过程也混合RLHF的方法，可以缓解这种损失
- 常见的公开NLP数据集，跟人类实际使用语言模型的场景，差别很大。因此单纯在公开NLP数据集进行指令微调，效果依然不够
- 虽然人类标注只有几十K，远远不能覆盖所有可能的prompts，但是实验发现InstructGPT的域外泛化能力很强，对于没有见过的prompt类型，依然有比较好的泛化能力
- instructGPT在真实性方面有改进，但在毒性方面改进不是很大



