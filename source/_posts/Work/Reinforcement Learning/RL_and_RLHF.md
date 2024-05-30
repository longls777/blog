---
title: Reinforcement Learning and RLHF
tags: RL
categories: Work
date: 2024-05-30 20:47:00
index_img:
banner_img:
math: true
---



> https://lilianweng.github.io/posts/2018-04-08-policy-gradient/
>
> [Proximal Policy Optimization Algorithms](https://arxiv.org/pdf/1707.06347)
>
> [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/pdf/2305.18290)
>
> [Direct Language Model Alignment from Online AI Feedback](https://arxiv.org/pdf/2402.04792)
>
> https://hrl.boyuai.com/chapter/2/%E7%AD%96%E7%95%A5%E6%A2%AF%E5%BA%A6%E7%AE%97%E6%B3%95



## PPO to DPO

> [详细推导DPO算法](https://zhuanlan.zhihu.com/p/697757566)

我们的推导目标是，从PPO的KL散度形式：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530155132756.png)

推导得到DPO：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530155317455.png)

其中$Z(x)=\sum_y{\pi_{ref}(y|x)exp(\frac{1}{\beta}r(x,y))}$​

> PPO论文里的公式如下，和DPO的有一些区别，这里后续再分析
>
> ![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530160402081.png)



在DPO的带KL散度公式中，$r(x,y)$表示reward function：

- 参数$x$，类似状态$s_t$​，对应prompt；
- 参数$y$，类似动作$a_t$，对应模型的response

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530161226849.png)

> KL散度
> $$
> D_{KL}(p||q)=\sum_{x}{p(x)log\frac{p(x)}{q(x)}}=-\sum_{x}{p(x)log\frac{q(x)}{p(x)}}
> $$
> https://hsinjhao.github.io/2019/05/22/KL-DivergenceIntroduction/
>
> https://zh.wikipedia.org/wiki/%E7%9B%B8%E5%AF%B9%E7%86%B5

- 第一步比较好理解，展开KL散度公式之后，$y$的sample范围从完整的策略$\pi$，转化为了$\pi(y|x)$，也就是说限定了$x$。（$\pi$是$\theta$的函数，$\theta$也就是我们要优化的model参数，这里的$\pi$是$\pi_{\theta}$的简写）
- 第二步除以一个$-\beta$，这里的$\beta$是控制KL散度的权重，我们的目标是在KL散度的约束下优化模型，$\beta>0$
- 第三步这里，$Z(x)=\sum_y{\pi_{ref}(y|x)exp(\frac{1}{\beta}r(x,y))}$，这里使用了一个partition function，重点是这个$Z(x)$只是$\pi_{ref}$和$x$的函数，和我们要优化的$\pi$​​​没有关系。

> 第三步这里是怎么得出来的呢？
> $$
> \begin{aligned}
> log&\frac{\pi(y|x)}{\pi_{ref}(y|x)} - \frac{1}{\beta}r(x,y)\\
> = & log\frac{\pi(y|x)}{\pi_{ref}(y|x)exp(\frac{1}{\beta}r(x,y))}\\
> = & log\frac{\pi(y|x)}{\pi_{ref}(y|x)exp(\frac{1}{\beta}r(x,y))} - log\frac{1}{Z(x)}-logZ(x)\\
> = & log\frac{\pi(y|x)}{\frac{1}{Z(x)}\pi_{ref}(y|x)exp(\frac{1}{\beta}r(x,y))}-logZ(x)\\
> \end{aligned}
> $$

然后定义：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530162721549.png)

这里作者认为$\pi^*(y|x)$是一个有效的概率分布，$\sum_y{\pi^*(y|x)}=1$，于是

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530163534496.png)

> 为什么这里能转化为KL散度呢？
> $$
> \begin{aligned}
> \mathbb{E} & _{y\sim\pi(y|x)}log\frac{\pi(y|x)}{\pi^*(y|x)}\\
> = & \sum_{x}\pi(y|x)log\frac{\pi(y|x)}{\pi^*(y|x)}\\
> = & \mathbb{D}_{KL}(\pi(y|x) ||\pi^*(y|x))
> \end{aligned}
> $$

根据Gibbs’ inequality（吉布斯不等式），当且仅当两个distribution相等时，KL散度得到最小值0

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530193848166.png)

带入得到：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530200931279.png)

## 在Bradley-Terry model下推导DPO

对于BT模型：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240530205843281.png)

