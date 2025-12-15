---
title: <Paper Reading> LMM-R1: Empowering 3B LMMs with Strong Reasoning Abilities Through
Two-Stage Rule-Based RL
tags: LLM RLHF
categories: PaperReading
date: 2025-08-11 14:26:00
index_img:
banner_img:
math: true
comment: true
hide: true
---

> https://arxiv.org/pdf/2402.03300

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811171759429.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811174317848.png)

![GRPO](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811175229026.png)

最近面试被人问到一个问题：为什么PPO + verified reward不能训练出reasoning model，此为回顾



![PPO](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811175751090.png)

![GRPO](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811175816289.png)

对于PPO，我们都知道：

- PPO采用critic model + GAE来估计token优势，critic model一般是一个和policy model大小相当的模型，因此在显存和计算上消耗比较大
- 在LLM场景下，PPO中reward只会赋予最后一个token，其余token的优势通过critic model来估计，而这个很难估计准，这就导致**奖励稀疏，信用分配困难**
- Token Level KL Reward限制了模型探索

于是为了解决这两个问题，deepseek提出了GRPO，用组内相对优势代替critic基线，除此之外

- 增加过程监督
- 

