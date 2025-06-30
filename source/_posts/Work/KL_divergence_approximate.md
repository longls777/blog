---
title: KL Divergence的三种估计方法
tags: LLM
categories: Work
date: 2025-06-18 14:41:00
index_img: 
banner_img: 
math: true
comment: true

---



> [Approximating KL Divergence](http://joschu.net/blog/kl-approx.html)

## Verl Implement

```python
def kl_penalty(logprob: torch.FloatTensor, ref_logprob: torch.FloatTensor, kl_penalty) -> torch.FloatTensor:
    """Compute KL divergence given logprob and ref_logprob.
    Copied from https://github.com/huggingface/trl/blob/main/trl/trainer/ppo_trainer.py#L1104
    See more description in http://joschu.net/blog/kl-approx.html

    Args:
        logprob:
        ref_logprob:

    Returns:

    """
    if kl_penalty in ("kl", "k1"):
        return logprob - ref_logprob

    if kl_penalty == "abs":
        return (logprob - ref_logprob).abs()

    if kl_penalty in ("mse", "k2"):
        return 0.5 * (logprob - ref_logprob).square()

    # J. Schulman. Approximating kl divergence, 2020.
    # # URL http://joschu.net/blog/kl-approx.html.
    if kl_penalty in ("low_var_kl", "k3"):
        kl = ref_logprob - logprob
        ratio = torch.exp(kl)
        kld = (ratio - kl - 1).contiguous()
        return torch.clamp(kld, min=-10, max=10)

    if kl_penalty == "full":
        # so, here logprob and ref_logprob should contain the logits for every token in vocabulary
        raise NotImplementedError

    raise NotImplementedError
```

我们要估计的真实Forward-KL应该是
$$
D_{\text{KL}}(p_\theta\|q)=\mathbb E_{a\sim p_\theta}
\bigl[-\Delta\bigr] = -(\log q-\log p) = \log p-\log q
$$

| kl_penalty              | code                                | 思路                 | 方差                      | 是否无偏      |
| ----------------------- | ----------------------------------- | -------------------- | ------------------------- | ------------- |
| `"kl"` / `"k1"`         | `logprob-ref_logprob = -Δ`          | 直接用真正的对数比   | **高**：`Δ` 在尾部可达 ±∞ | ✔             |
| `"abs"`                 | `(logprob - ref_logprob).abs() = Δ` | 绝对值，避免符号震荡 | 中                        | ✘（偏差大）   |
| `"mse"` / `"k2"`        | `0.5·Δ²`                            | 二阶泰勒展开         | 中-低                     | ✘（二阶逼近） |
| `"low_var_kl"` / `"k3"` | `r-Δ-1` , `r=e^{Δ}`                 | **低方差**近似       | **低**                    | ✘（二阶逼近） |

对于`k3`估计，具体推导如下：

设：
$$
\frac{q}{p_\theta}=e^{\Delta}
$$
定义：
$$
g(\Delta)=e^{\Delta}-\Delta-1
$$
写成 Δ 的泰勒级数：
$$
g(\Delta)=\frac{\Delta^{2}}{2}-\frac{\Delta^{3}}{6}+O(\Delta^{4})
$$
经验上，$\operatorname{Var}[g(\Delta)] \ll \operatorname{Var}[-\Delta]$，因此叫做**low-variance KL 近似**

代码中进一步把极端值裁到 [-10, 10]，再度压低方差
