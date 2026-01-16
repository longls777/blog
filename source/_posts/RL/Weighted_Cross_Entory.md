---
title: 加权交叉熵在概率单纯形约束下的解
tags: LLM RL
categories: Work
date: 2026-01-16 19:14:00
index_img: 
banner_img: 
math: true
comment: true
---



问题来自论文：Q-SFT: Q-Learning for Language Models via Supervised Fine-Tuning

LLM的objective定义为 $\mathcal{L}_{CE}(\phi)=\mathbb{E}_{(s,a)\sim\mathcal{D}}[\log~\pi_{\phi}(a|s)]$

设数据集$\mathcal{D}$的分布为$\pi_{\beta}(a|s)$，那么上述objective将使策略$\pi_{\phi}(a|s)$趋近于$\pi_{\beta}(a|s)$，也就是实现：
$$
\pi_{\phi}(a|s) \approx \pi_{\beta}(a|s)
$$
这称之为behavioral cloning / BC



Q-SFT希望实现将Q-Learning融入到LLM的BC中，为此，Q-SFT提出了一个加权的CE objective：
$$
\mathcal{L}_{WCE}(\theta) = \mathbb{E}_{(s, a)\sim\mathcal{D}} [ w(s, a) \log p_{\theta}(a|s) + (1-w(s, a)) \log p_{\theta}(a_d|s)]
$$
其中$a_{d}$是dummy action，$  0<w(s, a) < 1$，论文称这个objective将实现：$\hat{p}_{\theta}(a|s)\approx w(s,a)\pi_{\beta}(a|s)$  for all $a\ne a_{d}$ 

也就是说，最终将实现一个带权的原始数据集概率分布

这里其实是一个**交叉熵在概率单纯形（simplex）上的最优化问题**：

**最大化 $\sum_i c_i \log p_i$  $(c_i \ge 0,p_i\ge0,\sum p_i=1)$的最优解是$p_i = \frac{c_i}{\sum_j c_j}$**

直觉上来说就是**加权最大似然**： $\log p_i$ 喜欢把质量分给系数大的项；归一化约束迫使它按比例分配

------

#### 在 Q-SFT 里，$c_i$对应什么？

- 真实动作空间：$\mathcal A$（词表 token）

- 行为策略：$\pi_\beta(\cdot|s)$ 只定义在$\mathcal A$ 上，$\sum_{a\in\mathcal A}\pi_\beta(a|s)=1$

- dummy：$a_d\notin\mathcal A$

固定一个状态 $s$，那么：
$$
\mathbb E_{a\sim\pi_\beta(\cdot|s)}\big[w(s,a)\log p_{\theta}(a|s)+ (1-w(s, a)) \log p_{\theta}(a_d|s)\big]
= \sum_{a \in \mathcal A}\pi_\beta(a|s)[w(s,a)\log p(a|s)+ (1-w(s, a)) \log p_{\theta}(a_d|s)]
$$

- 对于每个真实动作 $ a\neq a_d$，它在目标里出现的系数就是
  $$
  c_a = \pi_\beta(a|s)w(s,a)
  $$

- dummy 动作 $a_d$ 的总权重是
  $$
  c_d = \sum_{a \in \mathcal A} \pi_\beta(a|s)(1-w(s,a))
  $$

- 所以最优解应该是
  $$
  p^*(a|s)=\frac{\pi_\beta(a|s)w(s,a)}{c_d+\sum_{a' \in \mathcal A}\pi_\beta(a'|s)w(s,a')}
  $$

而注意分母：

$$
c_d+\sum_{a' \in \mathcal A}\pi_\beta(a'|s)w(s,a')
= \sum_{a \in \mathcal A} \pi_\beta(a|s)(1-w)+\sum_{a \in \mathcal A} \pi_\beta(a|s)w
= \sum_{a \in \mathcal A} \pi_\beta(a|s)=1
$$
所以分母刚好是 1，得到：
$$
\boxed{p^*(a|s)=\pi_\beta(a|s)w(s,a)\quad(a \in \mathcal A)}
$$
而对于dummy action：
$$
p^*(a_d|s)=1-\sum_{a \in \mathcal A} p^*(a|s)=\sum_{a \in \mathcal A}\pi_\beta(a|s)-\sum_{a \in \mathcal A}\pi_\beta(a|s)w(s,a)=\sum_{a \in \mathcal A} \pi_\beta(a|s)(1-w(s,a))=c_d
$$
所以dummy action其实是类似一个垃圾桶：

- 真实 token 只拿到一部分质量：$\sum_{a\in\mathcal A} p(a|s) \le 1$
- 剩下的质量由 dummy 吃掉：$p(a_d|s)=1-\sum_{a\in\mathcal A} p(a|s)$

直觉上：dummy = 以上这些 token 都不太值得选/不太可信的那部分概率质量



