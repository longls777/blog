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



> Bradley-Terry Model, define $p_i = e^{\beta_i}$
> $$
> P(i>j)=\frac{p_i}{p_i+p_j}=\frac{e^{\beta_i}}{e^{\beta_i}+e^{\beta_j}}=\frac{1}{1+e^{\beta_j-\beta_i}}=\sigma(\beta_i-\beta_j)
> $$
> https://en.wikipedia.org/wiki/Bradley%E2%80%93Terry_model

根据上一节得到的公式，我们稍作变换就能得到：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531102918306.png)

这里的$r^*(x,y)$指的是ground-truth reward model，$\pi^*(y|x)$是optimal model，那么把这个带入上面的BT model 公式就可以实现消掉reward model：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531103829599.png)

所以最后得到DPO的公式：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531104447834.png)

其中$y_w$是preferred response，$y_l$是dispreferred response



## DPO在优化什么？

> https://blog.csdn.net/v_JULY_v/article/details/134242910

![DPO Loss的梯度](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531105102570.png)

其中$\hat{r}_{\theta}(x,y)=\beta log\frac{\pi_{\theta}(y|x)}{\pi_{ref}(y|x)}$

对于上面这个梯度公式，只看最后一项的话，DPO Loss目的是增大$y_w$的likelihood，减少$y_l$的likelihood，这符合我们的需求，即让模型倾向于输出preferred response

这里更重要的是，最后这一项加了一个权重$\sigma(\hat{r}_{\theta}(x,y_l)-\hat{r}_{\theta}(x,y_w))$，这里表示当非偏好答案$y_l$的奖励大于偏好答案$y_w$的奖励时，梯度越大



## DPO code from paper

```python
import torch.nn.functional as F

def dpo_loss(pi_logps, ref_logps, yw_idxs, yl_idxs, beta):
  """
  pi_logps: policy logprobs, shape (B,)
  ref_logps: reference model logprobs, shape (B,)
  yw_idxs: preferred completion indices in [0, B-1], shape (T,)
  yl_idxs: dispreferred completion indices in [0, B-1], shape (T,)
  beta: temperature controlling strength of KL penalty
  Each pair of (yw_idxs[i], yl_idxs[i]) represents the
  indices of a single preference pair.
  """
  pi_yw_logps, pi_yl_logps = pi_logps[yw_idxs], pi_logps[yl_idxs]
  ref_yw_logps, ref_yl_logps = ref_logps[yw_idxs], ref_logps[yl_idxs]
  pi_logratios = pi_yw_logps - pi_yl_logps
  ref_logratios = ref_yw_logps - ref_yl_logps
  losses = -F.logsigmoid(beta * (pi_logratios - ref_logratios))
  rewards = beta * (pi_logps - ref_logps).detach()
  return losses, rewards
```



## SimPO

> [SimPO: Simple Preference Optimization with a Reference-Free Reward](https://arxiv.org/pdf/2405.14734)

> The effectiveness of SimPO is attributed to a key design: 
>
> **using the average log probability of a sequence as the implicit reward**

![SimPO抛弃了reference model，变得更轻量](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531142809489.png)

SimPO认为DPO的两个缺点：

- $\pi_{ref}$ reference model在训练过程中会消耗额外的memory和计算能力
- DPO训练过程中的reward和推理时的metric存在discrepancy，generation时，$\pi_{\theta}$的目标是拟合如下的average log likelihood:

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531143521009.png)

对于DPO而言，$r(x,y_w)>r(x,y_l)$和$p_{\theta}(y_w|x)>p_{\theta}(y_l|x)$并不等价，根据实验，其实是一半一半的关系：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531144800398.png)

SimPO使用下式替换DPO中的reward formulation，从而和generation阶段对齐

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531144903078.png)

除此之外，SimPO还引入了一个target reward margin term $\gamma(\gamma>0)$，来确保reward存在一个下限：

![SimPO BT model](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531145417352.png)

最终得到SimPO的公式：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531145632835.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240531154718476.png)

Browse得到了一些观点

- SimPO在下游任务上并没有什么优势 （SimPO paper's Table 9）
- 实际上SimPO就是BT model加了一个margin，以及根据response length做了一个normalization（偏好数据集可能存在“标注人员偏好long response”这样的bias，从而使model根据response length区分reward）
- 有人在之前讨论过BT model加一个margin就能有比较好的效果 https://arxiv.org/pdf/2404.04932
- SimPO去掉$\pi_{ref}$这里，paper缺失理论证明
- maybe **DPO + BT margin + length normalization**可以获得更好的效果，that is：

$$
DPO_{update} = -log\sigma(\frac{\beta}{|y_w|}log\frac{\pi_{\theta}(y_w|x)}{\pi_{ref}(y_w|x)}-\frac{\beta}{|y_l|}log\frac{\pi_{\theta}(y_l|x)}{\pi_{ref}(y_l|x)}-\gamma)
$$

## online-DPO

> [Direct Language Model Alignment from Online AI Feedback](https://arxiv.org/pdf/2402.04792)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240601135421788.png)

DPO paper中采用的偏好数据集是从一开始就从SFT model采样好的，也就是offline-DPO，然而模型在训练过程中是会不断变化的，也就是$\pi_{\theta}$会不断更新，所以从更新后的$\pi_{\theta}$采样偏好对(pair)无疑更加make sence

![Online-DPO process](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240601151827984.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240601151948288.png)

offline DPO训练超过3000 steps后就会迅速collapse，相反online DPO可以稳定优化

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20240601152245038.png)
