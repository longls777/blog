---
title: A unified perspective of RLHF
tags: LLM RLHF
categories: Work
date: 2024-10-29 15:49:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20241030133028665.png
banner_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/pexels-sergey-pesterev-69811391-14578422.jpg
math: true
comment: true
---

# Currently popular RLHF Method

![rlhf pipline](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20241030133028665-20241101110743521.png)

To this day，the post-training diagram for LLMs is still CPT, SFT and RLHF. There are no signs that this diagram will change currently. Focusing on RLHF, I will attempt to provide a comprehensive review, summary and future outlook in the following, in order to clarify its development path and hopefully gain some interesting insights.

## Policy Gradient Algorithm

As a reinforcement learning algorithm that directly optimizes the policy itself, policy gradient methods compute a estimator of the gradient and optimize it using the gradient ascent algorithm. Specifically, this estimator is generally:

$$
\hat{g} = \hat{\mathbb{E}}_t \left[ \nabla_\theta \log \pi_\theta(a_t \mid s_t) \hat{A}_t \right]
$$


where $\pi_\theta$ is the stochastic policy and $\hat{A}_t$ is the estimator of the advantage function at timestep $t$. 

So the estimator $\hat{g}$ is actually obtained by differentiating the objective:
$$
L^{PG}(\theta) = \hat{\mathbb{E}}_t \left[ \log \pi_\theta (a_t \mid s_t)\hat{A}_t \right].
$$
However, directly using $L^{PG}$ as a loss for optimization may lead to excessively large policy updates, potentially pushing the policy away from well-performing regions, and it often performs poorly in practice. Therefore, various methods such as PPO are typically used, as they incorporate mechanisms to control update sizes and ensure more stable and efficient learning.

## TRPO(Trust Region policy optimization) 

TRPO maximizes an objective function while subjecting it to a constraint on the size of the policy update. Specifically:

$$
\text{maximize}_{\theta} \, \hat{\mathbb{E}}_t \left[ \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)} \hat{A}_t \right]
$$
subject to $\hat{\mathbb{E}}_t[\text{KL}[\pi_{\theta_{\text{old}}}(\cdot \mid s_t), \pi_\theta(\cdot \mid s_t)]] \le \delta$, where, $\theta_{\text{old}}$ is the policy parameters before the update. 



By incorporating a penalty term in the objective function to control the size of policy updates, the problem becomes an unconstrained optimization problem:
$$
\text{maximize}_{\theta} \hat{\mathbb{E}}_t \left[ \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)} \hat{A}_t - \beta \text{KL}[\pi_{\theta_{\text{old}}}(\cdot \mid s_t), \pi_\theta(\cdot \mid s_t)] \right]
$$
where $\beta$ is the coefficient. However, experiments show that using a fixed $\beta$ and optimizing the upper objective with SGD is insufficient, which leads to the development of PPO.

## PPO(Proximal Policy Optimization)

### PPO - CLIP

We define that $r_t(\theta) = \frac{\pi_{\theta}(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$, then the main objective proposed by PPO  is:
$$
L^{\text{CLIP}}(\theta) = \hat{\mathbb{E}}_t \left[ \min \left( r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1 - \epsilon, 1 + \epsilon) \hat{A}_t \right) \right]
$$
where $\epsilon$ is the hyperparameter. This formulation ensures that $r_t(\theta)$ remains within a certain range, preventing  $\pi_{\theta}(a_t|s_t)$ from deviating too much from $\pi_{\theta_{old}}(a_t|s_t)$.

![$L^{CLIP}$](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20241103180514694.png)

The choice between $1 + \epsilon$ and $1 - \epsilon$ depends on the sign of the advantage function $\hat{A}_t$. 

**When $\hat{A}_t > 0$ (i.e., the current action is better than the baseline):** 

Increasing $r_t(\theta)$ (i.e., the new policy is more inclined to choose this action) will enhance the objective function because it amplifies the positive advantage. However, to prevent excessively large policy updates that could destabilize training, if $r_t(\theta) > 1 + \epsilon$, the clipped term $(1 + \epsilon) \hat{A}_t$ will be smaller than the unclipped term $r_t(\theta) \hat{A}_t$. Since the objective function takes the minimum of the two, in this case, the clipped term $(1 + \epsilon) \hat{A}_t$ will be selected, thereby ignoring further increases in $r_t(\theta)$ that would otherwise beneficially change the objective.

**When $\hat{A}_t < 0$ (i.e., the current action is worse than the baseline):**

Decreasing $r_t(\theta)$ (i.e., the new policy is less inclined to choose this action) will enhance the objective function because it reduces the impact of the negative advantage. If $r_t(\theta) < 1 - \epsilon$, the clipped term $(1 - \epsilon) \hat{A}_t$ will be smaller than the unclipped term $r_t(\theta) \hat{A}_t$ (since $\hat{A}_t$ is negative).  In this case, the clipped term $(1 - \epsilon) \hat{A}_t$ will be selected, thereby preserving $r_t(\theta)$ from decreasing too much.

In fact, the role of $L^{\text{CLIP}}$  is to constrain the updates of $r_t(\theta)$ within a certain range.

### PPO - Penalty





## DPO

## ORPO

## KTO

## IPO

## SimPO



# Relate Work

## UNA: Unifying Alignments of RLHF/PPO, DPO and KTO by a Generalized Implicit Reward Function

**paper**: https://arxiv.org/pdf/2408.15339

**source**: Salesforce























> [Proximal Policy Optimization Algorithms](https://arxiv.org/pdf/1707.06347)
>
> [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/pdf/2305.18290)
>
> https://lilianweng.github.io/posts/2018-04-08-policy-gradient/
>
> [关于LLM+RL(HF)的片面脉络梳理](https://zhuanlan.zhihu.com/p/1686790674)