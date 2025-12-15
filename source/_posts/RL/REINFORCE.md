---
title: REINFORCE / RLOO / REINFORCE++ / GRPO
tags: LLM
categories: Work
date: 2025-07-01 14:39:00
index_img: 
banner_img: 
math: true
comment: true
hide: true
---

## Policy Gradient

策略梯度的目标是，找到一个最优策略$\pi^*_{\theta}$，让agent在与环境交互时，获得的累积回报期望$J(\theta)$最大

最直接的优化方法就是梯度上升，即沿着梯度的方向更新参数：
$$
θ_{new}=θ_{old}+\alpha\nabla_{\theta}J(\theta)
$$
这里的 $\alpha$是学习率，$\nabla_{\theta}J(\theta)$ 就是目标函数 $J(\theta)$对参数$\theta$的梯度，也就是**策略梯度**。

直接求$J(\theta)$的梯度很困难，因为它依赖于与环境交互产生的整个轨迹的概率分布，而这个分布本身又依赖于 $\theta$。

所以策略梯度采用一种这样的估计方式：
$$
\nabla_{\theta} J(\theta) = \sum_{s} \mu(s) \sum_{a} q_{\pi}(s, a) \nabla_{\theta} \pi(a|s, \theta)
$$

- $\pi(a|s, \theta)$：我们的策略，即在状态$s$下选择动作 $a$的概率
- $\nabla_{\theta} \pi(a|s, \theta)$：策略函数对参数 $\theta$的梯度。这个梯度向量指向了这样一个方向：**如果我们沿着这个方向更新$\theta$，那么在状态 $s$下选择动作 $a$的概率 $\pi(a|s, \theta)$会增加得最快**
- $q_{\pi}(s, a)$：**动作价值函数（Action-Value Function / $Q$*-Function*）**，指在状态 $s$下，执行动作$a$，然后继续遵循当前策略 $\pi$，所能获得的**期望总回报**
- $\mu(s)$：在当前策略 $\pi$下，状态$s$的**稳态分布（discounted state-visitation distribution）**

分析：

1. **内层求和 ($\sum_{a}$)**：对于一个特定的状态 $s$，我们遍历所有可能的动作 $a$
2. **$q_{\pi}(s, a) \cdot \nabla_{\theta} \pi(a|s, \theta)$**：这是一个加权的梯度。$\nabla_{\theta} \pi(a|s, \theta)$ 告诉我们增加动作 $a$概率的方向，而 $q_{\pi}(s, a)$ 则为这个方向赋予权重
3. **外层求和 $\sum_{s}$**：我们将所有状态 $s$下的梯度贡献加权平均起来就得到了最终的策略梯度





但是上面的估计是无法直接计算的，因为我们既不知道真实的状态分布 $\mu(s)$，也不知道真实的动作价值$q_{\pi}(s, a)$。所以我们需要用采样的思想来近似它。
$$
\theta_{t+1} = \theta_t + \alpha \sum_{a} \hat{q}(S_t, a, w) \nabla_{\theta} \pi(a|S_t, \theta)
$$
这里的变化是：

1. 不再对所有状态 $s$求和，而是在某一个时间步$t$，使用当前状态$S_t$。这相当于从状态分布$\mu(s)$ 中采样了一个状态
2. 用一个**近似的价值函数$\hat{q}(S_t, a, w)$** 来代替真实的 $q_{\pi}(S_t, a)$。这个$\hat{q}$也就是**Critic**，通过学习来逼近真实的 q 函数

这个更新方式被称为**All-Actions**方法，因为它在更新时，理论上需要对当前状态 $S_t$下的所有动作$a$ 都计算其$\hat{q}$值和梯度



(2)可以简化为
$$
\nabla_{\theta}J(\theta)
= \mathbb{E}_{s\sim\mu,\;a\sim\pi}\!\bigl[q_{\pi}(s,a)\,\nabla_{\theta}\log\pi(a|s,\theta)\bigr].
$$

> 使用log-trick:
>
> 对概率分布 $p(x,\theta)$ 有恒等式
> $$
> \nabla_{\theta}p(x,\theta)\;=\;p(x,\theta)\,\nabla_{\theta}\log p(x,\theta).
> $$
> 把它应用到 $\pi(a|s,\theta)$：
> $$
> \nabla_{\theta}\pi(a|s,\theta)
> 
> = \pi(a|s,\theta)\,\nabla_{\theta}\log\pi(a|s,\theta).
> $$
> 上述转化基于**对数的导数**：$\dfrac{\mathrm d}{\mathrm d\theta}\log f =\dfrac{1}{f}\dfrac{\mathrm d f}{\mathrm d\theta}$
>
> 然后
> $$
> \begin{aligned}
> 
> \nabla_{\theta}J(\theta)
> 
> &= \sum_{s}\mu(s)\sum_{a} q_{\pi}(s,a)\;
> 
> ​      \pi(a|s,\theta)\,\nabla_{\theta}\log\pi(a|s,\theta) \\[6pt]
> 
> &= \sum_{s}\mu(s)\;
> 
> ​      \underbrace{\sum_{a}\pi(a|s,\theta)\,q_{\pi}(s,a)\,
> 
> ​        \nabla_{\theta}\log\pi(a|s,\theta)}_{\displaystyle
> 
> ​        =\;\mathbb{E}_{a\sim\pi(\cdot|s,\theta)}
> 
> ​        \!\!\bigl[q_{\pi}(s,a)\,\nabla_{\theta}\log\pi(a|s,\theta)\bigr]}.
> 
> \end{aligned}
> $$

是另一种常见写法，意味着我们可以用单条轨迹对梯度做无偏的 Monte‑Carlo 估计，也就是**Sample‑Action**

同时，由于策略梯度可以 减去任意状态函数 $b(s)$而不改变期望值：
$$
\mathbb{E}\bigl[(q_{\pi}(s,a)-b(s))\nabla_{\theta}\log\pi(a|s,\theta)\bigr].
$$
所以可以选择 $b(s)=V_{\pi}(s)$ 最小化方差，这也正是 Actor‑Critic 的思想来源

> 这是因为
> $$
> \mathbb{E}_{a\sim\pi}\!\bigl[\nabla_{\theta}\log\pi(a\mid s,\theta)\bigr]
> =\sum_{a}\pi(a\mid s,\theta)\,\nabla_{\theta}\log\pi(a\mid s,\theta)
> =\nabla_{\theta}\sum_{a}\pi(a\mid s,\theta)
> =\nabla_{\theta}1
> =0.
> $$
> 所以 $b(s)$那一项可以提出来（因为不依赖动作$a$，而$q_{\pi}(s,a)$就不可以），而后面的基准项为0，所以可以消掉

当 $b(s)=V_\pi(s)$ 时，$(q_\pi(s,a)-b(s))$ 就等于 **优势函数**
$$
A_\pi(s,a)=Q_\pi(s,a)-V_\pi(s).
$$


## REINFORCE

REINFORCE算法是一种更简单的蒙特卡洛方法，它对公式3做了进一步的简化和近似：

1. **不用Critic**：REINFORCE不学习一个 $\hat{q}$ 函数
2. **不用“All-Actions”**：它只关心在 $S_t$时刻实际采取的那个动作$a_t$
3. **用 $G_t$ 代替 $q_{\pi}(S_t, a_t)$**：它使用从$t$ 时刻开始，直到本次交互结束的**完整回报 $G_t$**（一个具体的、带随机性的样本值）来作为$q_{\pi}(S_t, a_t)$ 的无偏估计

所以REINFORCE的更新公式就变成了：
$$
\theta_{t+1} = \theta_t + \alpha G_t \nabla_{\theta} \log \pi(a_t|S_t, \theta)
$$

> 注：这里用$\nabla \log \pi$是数学上的一个等价变换，称为log-derivative trick，其效果和$\nabla \pi$的目标一致，但在计算上更稳定、更方便
>
> 其中 $G_t=\sum_{k=t}^{T}\gamma^{\,k-t}R_k$（一般而言 $\gamma=1$）

可以看到，REINFORCE只针对**实际执行的动作**进行更新，用**整条轨迹的未来回报 $G_t$** 作为其好坏的评判标准，这就是它被称为蒙特卡洛策略梯度的原因。

## REINFORCE Leave-One-Out (RLOO)

------

在标准 `REINFORCE` 中，我们为 **每条采样轨迹** 计算一次回报 $G_t$，直接作为优势（`advantage`）。虽然无偏，但**方差很大**。

`RLOO `的核心思想是：一次采样 **同一状态（或同一 prompt）下的 $K$ 条轨迹**，并用其余 $K-1$ 条的平均回报作为当前轨迹的 baseline，从而构造 **Leave-One-Out (L-O-O) 优势**，显著降低方差，同时保持无偏性。



设 $q$ 为` prompt`，$a_k$ 为第 $k$ 条采样答案，$R(q,a_k)$ 为`reward`，则
$$
\begin{aligned} b(q,a_k) &= \frac1{K-1}\sum_{i=1,i\neq k}^{K} R(q,a_i),\\[4pt] A(q,a_k) &= R(q,a_k)-b(q,a_k) \\         &=\frac{K}{K-1}\!\Bigl(R(q,a_k) - \tfrac1K\sum_{i=1}^{K} R(q,a_i)\Bigr). \end{aligned}
$$
策略更新仍沿 $\nabla_\theta\log\pi_\theta(a_k|q),A(q,a_k)$ 方向，但由于 baseline 依赖于同批样本，方差大幅下降。

## REINFORCE ++

后续工作发现 `RLOO/GRPO `等  单prompt多轨迹 方法在 RLHF 中易**过拟合简单 prompt**，并可能出现reward hack；同时降低了批次多样性

于是REINFORCE++诞生，其改进是：

- **全局批归一化基线**： 不再为每个 prompt 计算 baseline，而是用 **整个 mini-batch 回报的均值** $\overline{R}_{\text{batch}}$：

$$
\begin{aligned} A_{q,o_t} & = r(o_{1:t},q)\;-\;\beta\sum_{i=t}^{T}\mathrm{KL}(i), \\[4pt] \mathrm{KL}(t) & =\log\frac{\pi_{\text{RL},\theta_{\text{old}}}(o_t \mid q, o_{<t})}{\pi_{\text{SFT}}(o_t \mid q, o_{<t})}, \\[6pt] \widetilde{A}_{q,o_t} & = \frac{A_{q,o_t}-\mu_{\text{batch}}}{\sigma_{\text{batch}}}\end{aligned}
$$

其中 $(\mu_{\text{batch}},\sigma_{\text{batch}})$ 是 **整个batch** `reward`的均值 / 标准差，实现 **批归一化优势**，无偏且显著降方差

- **PPO 裁剪，但** **无 Critic**： 直接把上式 $\widetilde{A}$ 塞进 PPO 的 `surrogate loss`，并保留概率比率裁剪，既保持 **步长安全**，又避免使用`critic model`

$$
L(\theta)=\mathbb{E}_{q,o}\!\Bigl[\tfrac1{|o|}\sum_{t} \min\bigl(r_t(\theta)\,\widetilde{A}_{q,o_t},\; \text{clip}(r_t(\theta),1-\epsilon,1+\epsilon)\,\widetilde{A}_{q,o_t}\bigr)\Bigr],
$$

$$
r_t(\theta)=\frac{\pi_{\theta}(o_t\!\mid q,o_{<t})}{\pi_{\theta_{\text{old}}}(o_t\!\mid q,o_{<t})}.
$$

- **一次一答**：每个 prompt 只采样 1 条输出，再依靠批归一化生成统一基线——这样既避免了 `RLOO / GRPO `的 prompt-级过拟合，也把 GPU batch 容量留给更多不同 prompt，进一步提升多样性

### verl practice

在`verl`中，对于`REINFORCE++`，提供了两种优势计算方式：

```python
def compute_reinforce_plus_plus_outcome_advantage(token_level_rewards: torch.Tensor, response_mask: torch.Tensor, gamma: torch.Tensor):
    """
    Compute advantage for REINFORCE++.
    This implementation is based on the paper: https://arxiv.org/abs/2501.03262

    Args:
        token_level_rewards: `(torch.Tensor)`
            shape: (bs, response_length)
        response_mask: `(torch.Tensor)`
            shape: (bs, response_length)

    Returns:
        advantages: `(torch.Tensor)`
            shape: (bs, response_length)
        Returns: `(torch.Tensor)`
            shape: (bs, response_length)
    """

    with torch.no_grad():
        returns = torch.zeros_like(token_level_rewards)
        running_return = 0

        for t in reversed(range(token_level_rewards.shape[1])):
            running_return = token_level_rewards[:, t] + gamma * running_return
            returns[:, t] = running_return
            # Reset after EOS
            running_return = running_return * response_mask[:, t]

        advantages = verl_F.masked_whiten(returns, response_mask)
        advantages = advantages * response_mask

    return advantages, returns
  
  

def compute_reinforce_plus_plus_baseline_outcome_advantage(token_level_rewards: torch.Tensor, response_mask: torch.Tensor, index: torch.Tensor, epsilon: float = 1e-6):
    """
    Compute advantage for RF++-baseline (https://arxiv.org/abs/2501.03262), operating only on Outcome reward
    (with only one scalar reward for each response).

    Args:
        token_level_rewards: `(torch.Tensor)`
            shape: (bs, response_length)
        response_mask: `(torch.Tensor)`
            shape: (bs, response_length)

    Returns:
        advantages: `(torch.Tensor)`
            shape: (bs, response_length)
        Returns: `(torch.Tensor)`
            shape: (bs, response_length)
    """
    response_length = token_level_rewards.shape[-1]
    scores = token_level_rewards.sum(dim=-1)

    id2score = defaultdict(list)
    id2mean = {}

    with torch.no_grad():
        bsz = scores.shape[0]
        for i in range(bsz):
            id2score[index[i]].append(scores[i])
        for idx in id2score:
            if len(id2score[idx]) == 1:
                id2mean[idx] = torch.tensor(0.0)
            elif len(id2score[idx]) > 1:
                id2mean[idx] = torch.mean(torch.tensor(id2score[idx]))
            else:
                raise ValueError(f"no score in prompt index: {idx}")
        for i in range(bsz):
            scores[i] = scores[i] - id2mean[index[i]]

        scores = scores.unsqueeze(-1).tile([1, response_length]) * response_mask
        scores = verl_F.masked_whiten(scores, response_mask) * response_mask

    return scores, scores
```

| reinforce_plus_plus_outcome_advantage                        | reinforce_plus_plus_baseline_outcome_advantage               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 逐token计算return(monte-carol return)<br />$G_t = r_t + \gamma,G_{t+1}$ | 只用outcome奖励，将一条response的所有reward累加，后续所有token在这一个response内的reward相同 |
| 全局批归一化(batch z-score)                                  | 先减去组内均值，然后再全局批归一化(batch z-score)            |

`reinforce_plus_plus_baseline_outcome_advantage`通过先**prompt组内归一化**再**全局batch z-score归一化**，实现两级降方差，方差更低



对比之下，`GRPO`：

```python
# NOTE(sgm): this implementation only consider outcome supervision, where the reward is a scalar.
def compute_grpo_outcome_advantage(
    token_level_rewards: torch.Tensor,
    response_mask: torch.Tensor,
    index: np.ndarray,
    epsilon: float = 1e-6,
    norm_adv_by_std_in_grpo: str = True,
):
    """
    Compute advantage for GRPO, operating only on Outcome reward
    (with only one scalar reward for each response).

    Args:
        token_level_rewards: `(torch.Tensor)`
            shape is (bs, response_length)
        response_mask: `(torch.Tensor)`
            shape is (bs, response_length)
        norm_adv_by_std_in_grpo: (bool)
            whether to scale the GRPO advantage.
            If True, the advantage is scaled by the std, as in the original GRPO.
            If False, the advantage is not scaled, as in Dr.GRPO (https://arxiv.org/abs/2503.20783).

    Returns:
        advantages: `(torch.Tensor)`
            shape is (bs, response_length)
        Returns: `(torch.Tensor)`
            shape is (bs, response_length)
    """
    scores = token_level_rewards.sum(dim=-1)

    id2score = defaultdict(list)
    id2mean = {}
    id2std = {}

    with torch.no_grad():
        bsz = scores.shape[0]
        for i in range(bsz):
            id2score[index[i]].append(scores[i])
        for idx in id2score:
            if len(id2score[idx]) == 1:
                id2mean[idx] = torch.tensor(0.0)
                id2std[idx] = torch.tensor(1.0)
            elif len(id2score[idx]) > 1:
                id2mean[idx] = torch.mean(torch.tensor(id2score[idx]))
                id2std[idx] = torch.std(torch.tensor([id2score[idx]]))
            else:
                raise ValueError(f"no score in prompt index: {idx}")
        for i in range(bsz):
            if norm_adv_by_std_in_grpo:
                scores[i] = (scores[i] - id2mean[index[i]]) / (id2std[index[i]] + epsilon)
            else:
                scores[i] = scores[i] - id2mean[index[i]]
        scores = scores.unsqueeze(-1) * response_mask

    return scores, scores
```

和`compute_reinforce_plus_plus_baseline_outcome_advantage`一样，都采用outcome-only先算一个整体reward，然后计算**同一prompt的组内均值和标准差**

`norm_adv_by_std_in_grpo`控制是否除以标准差(+ epsilon)，Dr. GRPO提出除以标准差会引入**长度偏置**，但是Dr. GRPO方差稍高

总体上，REINFORCE 方差最大、RLOO 次之，GRPO 和 REINFORCE++ 由于批或组内归一化，方差进一步下降但保持无偏。



# Summary

总结一下，我们要优化的目标是：
$$
θ_{new}=θ_{old}+\alpha\nabla_{\theta}J(\theta)
$$
其中$\nabla_{\theta}J(\theta)$是下式这样的无偏估计：
$$
\nabla_{\theta}J(\theta)
= \mathbb{E}_{s\sim\mu,\;a\sim\pi}\!\bigl[q_{\pi}(s,a)\,\nabla_{\theta}\log\pi(a|s,\theta)\bigr].
$$
根据**基线不变性**以及**重要性采样**：

-  **基线不变性**：对任意 $b(s)$，令 $A_\pi(s,a)=q_\pi(s,a)-b(s)$，则

  $\mathbb{E}[\nabla\log\pi\,b(s)]=0\Rightarrow$ 可把 $q_\pi$ 换成优势 $A_\pi$ 而不改梯度期望（无偏）

-  **重要性采样（Importance Sampling, IS）**：若从 $\pi_{\text{old}}$ 采样，  $\mathbb{E}_{\pi_\theta}[f]=\mathbb{E}_{\pi_{\text{old}}}\!\big[r(s,a)f\big]$,

  其中 $r(s,a)=\frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)}$

我们可以总结常见算法的公式如下

### REINFORCE (baseline)

**采样**：通常 on-policy，即 $a\sim\pi_\theta(\cdot|s)$。

**等效权重**：$q_\pi(s,a)$ 的无偏蒙特卡洛估计 $\hat q(s,a)$，常写成优势 $\hat A(s,a)=\hat q(s,a)-b(s)$。

**梯度**：
$$

\nabla_\theta J

=\mathbb{E}_{\pi_\theta}\!\big[\hat A(s,a)\,\nabla_\theta\log\pi_\theta(a|s)\big].
$$

### PPO (clipped surrogate)

**采样**：从 $\pi_{\text{old}}$（收集到的轨迹）采样

**关键**：用 IS 比率 $r(s,a)=\frac{\pi_\theta}{\pi_{\text{old}}}$，再用**裁剪**抑制更新过大；优势 $\hat A$ 通常由 $\pi_{\text{old}}$ 的回放评估而来（e.g. GAE, TD-Error)

**目标**：$\mathcal L(\theta)=\mathbb{E}_{\pi_{\text{old}}}\big[\min\{r\hat A,\;\mathrm{clip}(r,1\!-\!\epsilon,1\!+\!\epsilon)\hat A\}\big]$

**梯度**：
$$
\nabla_\theta J
=\mathbb{E}_{\pi_{\text{old}}}\!\Big[\underbrace{\tilde q_{\text{PPO}}(s,a;\theta)}_{\text{等效“优势”权重}}\;\nabla_\theta\log\pi_\theta(a|s)\Big],
$$
其中
$$
\tilde q_{\text{PPO}}(s,a;\theta)=

\begin{cases}

r(s,a)\,\hat A(s,a), & (\hat A\ge 0 \land r\le 1+\varepsilon)\;\;\vee\;\;(\hat A\le 0 \land r\ge 1-\varepsilon),\\

0, &  (\hat A>0 \land r>1+\varepsilon)\;\;\vee\;\;(\hat A<0 \land r<1-\varepsilon).

\end{cases}
$$
也就是PPO把 REINFORCE 的“优势权重”换成了“**被信赖域掩码过的 IS 加权优势**”。



### RLOO

**采样**：on-policy；对同一 状态$s$ 一次采样 $K$ 个动作 $\{a_i\}$，得奖励 $\{r_i\}$。

**留一基线**（降方差）：$A_i=r_i-\frac{1}{K-1}\sum_{j\neq i} r_j$。

**梯度**：
$$
\nabla_\theta J

=\mathbb{E}_{\pi_\theta}\!\Big[\frac{1}{K}\sum_{i=1}^K

A_i\,\nabla_\theta\log\pi_\theta(a_i|s)\Big].
$$

### GRPO

**采样**：通常像 RLOO 一样，每个状态 $s$ 采样一组动作（序列）。

**组相对优势**：以组均值（可配合组标准差）做基线与归一化：
$$
\tilde A_i=\frac{r_i-\frac{1}{G}\sum_j r_j}{\mathrm{Std}(r_{1:G})+\varepsilon}.
$$


**PPO 化**：对**序列级**概率比率 $r(s,a_i)=\pi_\theta(a_i|s)/\pi_{\text{old}}(a_i|s)$ 做裁剪；

再加到参考策略的 **KL 正则**：$-\beta\,D_{\text{KL}}(\pi_\theta\|\pi_{\text{ref}})$。

**梯度**（分两部分）：
$$
\underbrace{\mathbb{E}_{\pi_{\text{old}}}\!\big[\tilde q_{\text{GRPO}}(s,a;\theta)\,\nabla\log\pi_\theta(a|s)\big]}_{\text{PPO 裁剪后的组优势项}}

\;+\;

\underbrace{\mathbb{E}_{\pi_\theta}\!\big[q_{\text{KL}}(s,a)\,\nabla\log\pi_\theta(a|s)\big]}_{\text{KL 正则项}},
$$
其中
$$
\tilde q_{\text{GRPO}}(s,a;\theta)=

\begin{cases}

r(s,a)\,\hat A(s,a), & (\hat A\ge 0 \land r\le 1+\varepsilon)\;\;\vee\;\;(\hat A\le 0 \land r\ge 1-\varepsilon),\\

0, &  (\hat A>0 \land r>1+\varepsilon)\;\;\vee\;\;(\hat A<0 \land r<1-\varepsilon).

\end{cases}
$$


> $\nabla_\theta D_{\text{KL}}(\pi_\theta\|\pi_{\text{ref}})=\mathbb{E}_{\pi_\theta}[(\log\pi_\theta-\log\pi_{\text{ref}})\,\nabla\log\pi_\theta]$
>
>$q_{\text{KL}}(s,a)= -\beta\big(\log\pi_\theta(a|s)-\log\pi_{\text{ref}}(a|s)\big).$



实践里既有把 KL **作为额外正则项**（不被裁剪）的做法，也有把 KL **折算进奖励**（会被白化/裁剪）的做法；前者归为**PPO/GRPO 风格**，后者归为**REINFORCE++ 风格**

### REINFORCE++

**采样**：多样本/组采样（on-policy 或近似 on-policy）。

**优势**：不学价值函数，直接用回报并做**跨批/跨组的全局标准化**得到 $\hat A_{\text{global}}$

**KL reward shaping**：把**逐 token** 的 $-\lambda\,\mathrm{KL}(\pi_\theta\|\pi_{\text{ref}})$ 当作额外即时负奖励，等价于把
$$
q_\pi(s,a)\quad\mapsto\quad q_\pi(s,a)\;-\;\lambda\sum_{t\le \tau(s,a)}\!\big(\log\pi_\theta(a_t|s_t)-\log\pi_{\text{ref}}(a_t|s_t)\big).
$$
**PPO 化**：同 GRPO，对（序列或 token 级）比率做裁剪
