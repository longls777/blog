---
title: The Wisdom of Hindsight Makes Language Models Better Instruction Followers
tags: 
categories: Paper Reading Notes
date: 2023-08-23 13:14:00
index_img: 
banner_img: 
math: true
---

**标题：**《The Wisdom of Hindsight Makes Language Models Better Instruction Followers》

**论文来源：** ICML 2023

**原文链接：** https://arxiv.org/abs/2302.05206

**源码：** https://github.com/tianjunz/HIR



## 概述

强化学习如RLHF帮助GPT等LLM在下游任务上获得良好的表现，但是RLHF过程过于复杂，需要构建奖励模型，本文提出另一种方式：将反馈转化为指导，通过重新标记原始反馈并以监督方式训练模型以实现更好的对齐

> converting feedback to instruction by relabeling the original one and training the model for better alignment in a supervised manner. 

这种方式不需要额外的训练参数，并且可以reuse预训练pipeline。为了实现这一目标，本文将语言模型的指令对齐问题表述为决策制定中的目标达成问题，提出了HIR算法（Hindsight Instruction Relabeling）

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823114946253.png)



## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823115153238.png)

### Instruction Following as Goal-conditioned RL

将大模型的对齐看作是给定目标的RL问题

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823120125490.png)

Prompt作为目标Goal， Query作为状态空间S，输出token作为动作空间，转移概率也就是模型的预测概率，奖励R可以通过人类反馈得到，但是HIR不需要这一步

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823120142886.png)

G， S和A都是词向量空间，所以也可以将其看作给定目标的RL问题



### Algorithm Overview

HIR包括两阶段：

- online sampling 在线采样
-  offline relabeling 离线重标记



#### Online Sampling

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823121229161.png)

这个$P_i$，好像在代码里就是一句话：

> Solve the question with correct answer.

这个采样怎么感觉就是加了个prompt



#### Offline Relabeling

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823121440395.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823121507921.png)

重标记看起来是替换了$P$，新的prompt 通过奖励函数R和提示生成函数生成（可以学习也可以固定）

这个生成函数看起来是固定的，类似FARL

HIR和FARL的区别是是否使用**事后的经验**（hindsight）：

- FARL过滤了正确的instruction-output对
- HIR允许从错误case中学习

![整个HIR算法的流程](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823122325727.png)



## 总结

强化学习，PPO的改进，但是感觉数据集有点少，而且大多是MCQA的形式

（RL看不懂一点）
