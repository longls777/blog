---
title: The relationship between prompt and acc expected values
tags: LLM RL
categories: Work
date: 2025-09-26 14:54:00
index_img: 
banner_img: 
math: true
comment: true
---

## prompt的PPL

设`prompt`为$x_{1:T}$，给定LLM $\pi_{\theta}$（也就是条件分布$p_{\theta}(\cdot|\cdot)$） 那么其在teacher forcing下对于整个prompt的对数似然为：
$$
logp_{\theta}(x_{1:T}) = \sum_{t=1}^T{logp_{\theta}(x_t|x_{t<t})}.
$$
平均交叉熵：
$$
H_\theta(x_{1:T}) \;=\; -\frac{1}{T}\sum_{t=1}^{T}\log p_\theta(x_t\mid x_{<t}).
$$
Perplexity：
$$
\mathrm{PPL}_\theta(x_{1:T}) \;=\; \exp\!\Big(H_\theta(x_{1:T})\Big)

\;=\; \exp\!\Big(-\frac{1}{T}\sum_{t=1}^{T}\log p_\theta(x_t\mid x_{<t})\Big).
$$

## 把prompt的生成看作MDP过程

