---
title: Energy Based Models(EBMs)
tags: RL
categories: Work
date: 2024-06-20 10:44:00
index_img:
banner_img:
math: true

---

> https://arxiv.org/pdf/2101.03288

## 概述

具有易处理的likelihood的概率模型是一把双刃剑：

- 不同模型之间的对比是非常容易的，能够通过数据的log-likelihood直接优化参数
- 但是这些模型都需要具体的形式，例如auto regressive模型的分布是 概率分布的乘积

这些模型都基于这样一个假设：**可以通过一个特定的、易处理的程序从模型中精确合成伪数据**

但是这些假设并不总是自然成立的



Energy based models(EMBs)则限制少的多，区别于一个normalized probability，EMBs使用unnormalized 负log- probability

，也就是energy function（能量函数）

由于能量函数不需要积分为 1，因此可以用任何非线性回归函数进行参数化





> Since the energy function does not need to integrate to one, it can be parameterized with any nonlinear regression function.



