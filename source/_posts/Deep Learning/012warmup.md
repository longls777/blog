---
title: warmup
tags: 
categories: Deep Learning
date: 2023-8-4 21:07:00
index_img: 
banner_img: 
math: true
---

> 神经网络中 warmup 策略为什么有效；有什么理论解释么？ - 香侬科技的回答 - 知乎 https://www.zhihu.com/question/338066667/answer/771252708



- 有助于减缓模型在初始阶段对mini-batch的提前过拟合现象，保持分布的平稳
- 有助于保持模型深层的稳定性

如果来总结一下现在大batch size和大learning rate下的学习率变化和规律的话，可以认为，学习率是呈现“上升——平稳——下降”的规律，这尤其在深层模型和具有多层MLP层的模型中比较显著。如果从模型学习的角度来说，学习率的这种变化对应了模型“平稳——探索——平稳”的学习阶段。