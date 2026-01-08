---
title: A Unified Perspective on RL for LLMs
tags: LLM RLHF
categories: Work
date: 2024-10-29 15:49:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20241030133028665.png
banner_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/pexels-sergey-pesterev-69811391-14578422.jpg
math: true
comment: true
hide: true
---

> **Change Log:**
>
> * **2025-12-2x:** 
> * **2024-10-29:** Initial release of the article.

# Notations

| **Symbol**         | **Definition**                                           |
| ------------------ | -------------------------------------------------------- |
| $t$                | Current timestep                                         |
| $S$                | State space                                              |
| $A$                | Action space                                             |
| $s_t$              | State at timestep $t$                                    |
| $a_t$              | Action taken at timestep $t$                             |
| $\tau$             | Trajectory, usually $\tau = (s_0, a_0, r_0, s_1, ...)$   |
| $r_t$              | Immediate reward received at timestep $t$                |
| $R$                | Cumulative return                                        |
| $\mathbf{r}$       | Reward function                                          |
| $G$                | Reward-to-Go                                             |
| $J(\theta)$        | Expected reward/return obtained by $\pi_\theta$          |
| $\pi_\theta$       | Policy distribution parameterized by $\theta$            |
| $\pi_{\text{old}}$ | Policy distribution before the update                    |
| $\pi_{\text{ref}}$ | Reference policy distribution                            |
| $\eta$             | Learning Rate                                            |
| $\hat{A}_t$        | Estimator of the Advantage function                      |
| $\hat{g}$          | Estimator of the policy gradient                         |
| $V(s)$             | Value function estimating expected return from state $s$ |
|                    |                                                          |
| $\hat{A}_t^{GAE}$  | Generalized Advantage Estimator                          |
| $\gamma$           | Discount factor                                          |
| $\lambda$          | GAE parameter for bias-variance trade-off                |
| $\Phi(s)$          | Potential function for reward shaping                    |
| $\mathbf{x}$       | The prompt input                                         |
| $\mathbf{y}$       | The generated response                                   |
| $\mathcal{D}$      | Dataset                                                  |
| $\text{KL}[\cdot]$ | Kullback-Leibler divergence                              |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |
|                    |                                                          |

### ~~Potential-Based Reward Shaping (PBRS)~~

> ~~**Definition**~~
>
> ~~A shaping reward function $F : S \times A \times S \to \mathbb{R}$ is potential-based if there exists $\Phi : S \to \mathbb{R}$ such that:~~
> $$
> F(s, a, s') = \gamma \Phi(s') - \Phi(s)
> $$
> ~~for all $s \neq s_0, a, s'$.~~

~~PBRS cleverly introduces the concept of potential, guiding the agent to search more quickly in the state space without altering the optimal policy of the original problem.~~

>~~[Policy Invariance Under Reward Transformations: Theory and Application to Reward Shaping](https://people.eecs.berkeley.edu/~pabbeel/cs287-fa09/readings/NgHaradaRussell-shaping-ICML1999.pdf)~~

# Policy Gradient Algorithm

In standard reinforcement learning, policy gradient algorithm aimed to maximize the expected cumulative reward over trajectories. A trajectory $\tau = (s_0, a_0, r_0, s_1, ...)$ is a sequence of states and actions generated by the policy $\pi_\theta$. The objective function is:
$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} [R(\tau)]
$$
where $R(\tau)$ is the cumulative reward. We optimize this using gradient ascent:
$$
\theta \leftarrow \theta + \eta \nabla_\theta J(\theta)
$$
However, directly computing the gradient of an expectation is intractable, because it's non-differentiable. To solve this, we use the "**Log-Derivative Trick**" (Policy Gradient Theorem). This theorem allows us to shift the gradient computation from the reward function to the policy's probability distribution, which is differentiable. 

> [!NOTE]
>
> **How do we turn the gradient of an expectation into an expectation of a gradient?**
>
> Expand the expectation:
> $$
> \nabla_\theta J(\theta) = \nabla_\theta \int P(\tau|\theta) R(\tau) d\tau
> $$
> Move the gradient inside the integral:
>
> $$
> = \int \nabla_\theta P(\tau|\theta) R(\tau) d\tau
> $$
> Apply the Identity Trick: 
>
> $$
> = \int P(\tau|\theta) \frac{\nabla_\theta P(\tau|\theta)}{P(\tau|\theta)} R(\tau) d\tau
> $$
> Use the Log-Derivative Rule: From calculus, we know that $\frac{\nabla x}{x} = \nabla \log x$.
>
> $$
> = \int P(\tau|\theta) \nabla_\theta \log P(\tau|\theta) R(\tau) d\tau
> $$
> Convert back to Expectation: The equation is now in the form $\int P(x) [\dots] dx$, which is the definition of expectation.
>
> $$
> = \mathbb{E}_{\tau \sim \pi_\theta} [\nabla_\theta \log P(\tau|\theta) R(\tau)]
> $$
> This trick is Log-Derivative Trick, which transforms the gradient into an Expectation, allowing us to approximate the gradient simply by sampling trajectories.

Using the Log-Derivative trick, we theoretically obtain:
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau} \left[ \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) R(\tau) \right]
$$
However, because of the causality, the action $a_t$ taken at time $t$ can only influence rewards received after time $t$. Therefore, we can replacing the total reward $R(\tau) = \sum_{k=0}^{T} r_k$ with the return-to-go $G_t = \sum_{k=t}^{T} r_k $:
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau} \left[ \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) G_t \right]
$$
Besides, to further reduce variance and improve training stability, we subtract a baseline $b(s_t)$—typically the state-value function $V(s_t)$—from the return. Crucially, subtracting a baseline that depends only on the state (and not the action) does not introduce bias. So that is $(G_t - V(s_t))$, which effectively estimates the Advantage Function $A(s_t, a_t) = Q(s_t, a_t) - V(s_t)$. Therefore,  the final gradient becomes: 
$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[ \sum_{t} \nabla_\theta \log \pi_\theta(a_t \mid s_t) \hat{A}_t \right]
$$
> [!NOTE]
>
> **Why is the Baseline Unbiased?**
> We stated that subtracting a baseline $b(s_t)$ does not introduce bias. This is because the expected value of the gradient of the baseline term is zero: 
> $$
> \mathbb{E}_{a_t \sim \pi} \left[ \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot b(s_t) \right] = 0
> $$
> Proof:
>
> Write out the definition of Expectation:
>
> $$
> \sum_{a_t} \pi_\theta(a_t|s_t) \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot b(s_t)
> $$
> Expand the log-derivative ($\nabla \log x = \frac{\nabla x}{x}$):
>
> $$
> \sum_{a_t} \pi_\theta(a_t|s_t) \frac{\nabla_\theta \pi_\theta(a_t|s_t)}{\pi_\theta(a_t|s_t)} \cdot b(s_t)
> $$
> Cancel the probability terms $\pi_\theta$:
>
> $$
> \sum_{a_t} \nabla_\theta \pi_\theta(a_t|s_t) \cdot b(s_t)
> $$
> Extract $b(s_t)$: Since the baseline $b(s_t)$ depends only on the state and is independent of the action $a_t$, we can move it outside the summation:
>
> $$
> b(s_t) \cdot \sum_{a_t} \nabla_\theta \pi_\theta(a_t|s_t)
> $$
> Swap Summation and Gradient:
>
> $$
> b(s_t) \cdot \nabla_\theta \left( \sum_{a_t} \pi_\theta(a_t|s_t) \right)
> $$
> The Axiom of Probability: The sum of probabilities of all possible actions must equal 1.
>
> $$
> b(s_t) \cdot \nabla_\theta (1)
> $$
> Result: The derivative of a constant (1) is 0.
>
> $$
> b(s_t) \cdot 0 = 0
> $$
> Subtracting a baseline term effectively adds 0 to the expected gradient, so the estimator remains unbiased.

In practice, we approximate this expectation using sampled data, leading to the following estimator:
$$
\hat{g} = \hat{\mathbb{E}}_t \left[ \nabla_\theta \log \pi_\theta(a_t \mid s_t) \hat{A}_t \right]
$$
Mathematically, this estimator $\hat{g}$ is equivalent to differentiating the following surrogate objective:
$$
L^{PG}(\theta) = \hat{\mathbb{E}}_t \left[ \log \pi_\theta (a_t \mid s_t)\hat{A}_t \right].
$$
However, directly using $L^{PG}$ as a loss for optimization may lead to excessively large policy updates, potentially pushing the policy away from well-performing regions, and it often performs poorly in practice. Therefore, various methods such as PPO are typically used, as they incorporate mechanisms to control update sizes and ensure more stable and efficient learning.

# REINFORCE

In the previous section, we derived the gradient estimator using the cumulative return $G_t$:
$$
\nabla_\theta J(\theta) = \mathbb{E} \left[ \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t \mid s_t) G_t \right]
$$
This formulation is essentially the REINFORCE algorithm (also known as Monte Carlo Policy Gradient). It directly uses the actual return $G_t$ obtained from a complete trajectory as the weight for the gradient.

However, simply using the cumulative return $G_t$ is problematic.

While REINFORCE is mathematically unbiased, it suffers from extremely high variance. Because $G_t$ is the sum of rewards over a long sequence of random actions and state transitions, the value of $G_t$ fluctuates wildly between different trajectories.

而









### Generalized Advantage Estimator(GAE)

GAE is a widely applicable advantage estimation method that can effectively reduce the variance of gradient estimation in policy gradient methods. The formula of GAE is:
$$
\hat{A}_t^{GAE(\gamma, \lambda)} = \sum_{l=0}^{\infty} (\gamma \lambda)^l \delta_{t+l}^V = \sum_{l=0}^{\infty} (\gamma \lambda)^l (r_{t+l} + \gamma V(s_{t+l+1}) - V(s_{t+l}))
$$

> [High-Dimensional Continuous Control Using Generalized Advantage Estimation](https://arxiv.org/abs/1506.02438)

### TRPO(Trust Region policy optimization) 

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

## Methods

### PPO(Proximal Policy Optimization)

#### PPO - CLIP

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

#### PPO - Penalty

PPO-penalty employs several epochs of minibatch SGD to optimize the KL-penalized objective given by:

$$
L^{KL PEN}(\theta) = \hat{\mathbb{E}}_t \left[ \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)} \hat{A}_t - \beta \text{KL}[\pi_{\theta_{\text{old}}}(\cdot \mid s_t), \pi_\theta(\cdot \mid s_t)] \right]
$$
then computes $d = \hat{\mathbb{E}}_t[\text{KL}[\pi_{\theta_{\text{old}}}(\cdot \mid s_t),\pi_\theta(\cdot \mid s_t)]]$

- If $d < d_{\text{targ}}/1.5$, $\beta \leftarrow \beta/2$
- If $d > d_{\text{targ}} \times 1.5$, $\beta \leftarrow \beta \times 2$

where $d_{targ}$ is a pre-set hyperparameter used to limit the difference between the current policy and the policy from the previous iteration. The updated $\beta$ is used for the next policy update.

Apparently, PPO-penalty imposes less strict constraints on the magnitude of policy updates compared to PPO-clip, and its performance in experiments is also inferior to the latter according to the PPO paper.

#### PPO's Advantage Estimator

PPO adopts a truncated generalized advantage estimator(GAE), when $\lambda = 1$:
$$
\hat{A}_t = \delta_t + (\gamma \lambda) \delta_{t+1} + \cdots + (\gamma \lambda)^{T-t+1} \delta_{T-1},
$$
where $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$. In RLHF, $r_t$ is provided by the reward model, generally using the following formula:
$$
r(s_t, \mathbf{a}_t) = 
\begin{cases} 
\beta \log \pi_{\text{ref}}(\mathbf{a}_t|s_t), & \text{if } s_{t+1} \text{ is not terminal} \\
r(\mathbf{x}, \mathbf{y}) + \beta \log \pi_{\text{ref}}(\mathbf{a}_t|s_t), & \text{if } s_{t+1} = \mathbf{y} \text{ is terminal}
\end{cases}
$$
$V(s_t)$ is given by the critic model, which fits the expected return by calculating an MSE loss with the actual return $R_t=r_t + r_{t+1} + ... + r_T$.

### DPO

> [High-Dimensional Continuous Control Using Generalized Advantage Estimation](https://arxiv.org/pdf/1506.02438)
>
> [Proximal Policy Optimization Algorithms](https://arxiv.org/pdf/1707.06347)
>
> [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/pdf/2305.18290)
>
> https://lilianweng.github.io/posts/2018-04-08-policy-gradient/
>
> [关于LLM+RL(HF)的片面脉络梳理](https://zhuanlan.zhihu.com/p/1686790674)
>
> https://zhuanlan.zhihu.com/p/1962307894105048283