---
title: Entroy的数值稳定计算方法
tags: LLM
categories: Work
date: 2025-05-28 14:41:00
index_img: 
banner_img: 
math: true
comment: true
---

```python
def entropy_from_logits(logits: torch.Tensor):
    """Calculate entropy from logits."""
    pd = torch.nn.functional.softmax(logits, dim=-1)
    entropy = torch.logsumexp(logits, dim=-1) - torch.sum(pd * logits, dim=-1)
    return entropy
```

在查看verl代码时，发现其计算entropy的时候，使用了一种独特的方式，具体而言，这里的`entropy_from_logits`是一种数值稳定写法，等价于熵的经典定义：
$$
H(p) = -\sum_i p_i \log p_i
$$

具体而言，将第 $i$ 个类的 `logits` 记作 $\ell_i$，其概率即为：
$$
p_i = \frac{e^{\ell_i}}{\sum_j e^{\ell_j}}
$$
也就是 `softmax` 后的 `logits`，那么

$$
H(p) = -\sum_i p_i \log p_i = -\sum_i p_i (\ell_i - \log Z)
$$

其中 $Z = \sum_j e^{\ell_j}$，展开得到：

$$
H(p) = -\sum_i p_i \ell_i + (\log Z) \sum_i p_i = -\sum_i p_i \ell_i + \log Z
$$

因为 $\sum_i p_i = 1$，而
$$
\log Z = \mathrm{logsumexp}(\{\ell_i\})
$$

于是即可得到上面代码。

### 为什么数值稳定

```python
entropy = logsumexp(logits, dim=-1) - (softmax(logits)*logits).sum(dim=-1)
```

之所以称这是一种“数值稳定”的熵计算方式，主要有以下几点原因：

##### 1. 避免直接算 $\log p_i$ 的不稳定（核心原因）

最直观的香农熵写法是公式1，如果先做：

```python
p = softmax(logits)
logp = torch.log(p)
entropy = -(p * logp).sum(-1)
```

- **问题1：** 当某些 `logits` 很负时，对应的 $p_i$ 会非常接近 0，这时 $\log p_i$ 会变成一个很大的负数，`p_i * logp_i` 可能因为 “0×(−∞)” 导致数值上出现 NaN
- **问题2：** 计算 `log(p)` 也会把下溢（underflow）的微小概率映射到 $-\infty$，再做乘法和求和十分容易出错。

> IEEE 754浮点标准里，`log(0)`会被定义为`-∞`，是一个合法的无穷值，而不是NaN；所以如果此时做`0×(−∞)`，就成为一个不定式，结果就是NaN

##### 2. 用 log-sum-exp 计算 $\log Z$ 的稳定性

在等式4中，我们只需要两个量：

1. $\log Z$，即 `logsumexp(logits)`
2. $\sum_i p_i\ell_i$，即 `(softmax(logits)*logits).sum()`

其中 PyTorch 的 `logsumexp` 会自动做$\log\sum_i e^{\ell_i} = m + \log\sum_i e^{\ell_i-m},$

先减掉 $m=\max_j\ell_j$ 再 exponentiate，避免了 $e^{\ell_i}$ 可能的**上溢**（overflow）

##### 3. 避免大范围指数／对数运算

- `softmax(logits)` 本身底层也会先减 max 再做 `exp()`，保证数值不爆
- `logsumexp` 也是同样的 trick
- 于是整个式子中不会出现 “先指数化得到极大或极小值，再取对数” 这种前后相抵但中间溢出的操作
