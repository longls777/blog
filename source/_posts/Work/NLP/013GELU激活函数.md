---
title: GELU激活函数
tags: Activation Function
categories: NLP
date: 2023-7-22 20:51:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-45d6312c444b7548558132cea30e3808_720w.webp
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-45d6312c444b7548558132cea30e3808_720w.webp
math: true
---

> BERT中的激活函数GELU：高斯误差线性单元 - 清欢鱼的文章 - 知乎 https://zhuanlan.zhihu.com/p/349492378
>
> 为什么GELU激活函数越来越常见？ - 利先生的文章 - 知乎 https://zhuanlan.zhihu.com/p/609897428
>
> 深度学习中激活函数的导数在不连续可导时的处理 - puppy的文章 - 知乎 https://zhuanlan.zhihu.com/p/101321997

## 定义和引入

GELU（Gaussian Error Linear Unit，高斯误差线性单元）于2016年由Hendrycks和Gimpel在论文《Gaussian Error Linear Units (GELUs)》中提出。

与其他常用的激活函数（如ReLU和sigmoid）相比，GELU具有更平滑的非线性特征，这有助于提高模型的性能。GELU可以看作是一种sigmoid和ReLU的混合体。



ReLU和Dropout都会返回一个神经元的输出，其中，ReLU会确定性的将输入乘上一个0或者1，Dropout则是随机乘上0。而GELU也是通过将输入乘上0或1来实现这个功能，但是输入是乘以0还是1，**是在同时取决于输入自身分布的情况下随机选择的。** 换句话说，是0还是1取决于当前的输入有多大的概率大于其余的输入。
$$
m \sim \operatorname{Bernoulli}(\Phi(x)), \text { where } \Phi(x)=P(X<=x)
$$
GELU将神经元的输入$x$乘上一个服从伯努利分布的 $m$ ，而该伯努利分布又是依赖于$x$的

因为$X \sim N(0,1)$，所以$\Phi(x)$就是标准正态分布的累积分布函数，这么做的原因是因为神经元的输入$x$往往遵循正态分布，尤其是深度网络中普遍存在Batch Normalization的情况下

![正态分布的累积分布函数](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-53dabc61fef4c916739a83893837af37_720w.webp)

当$x$减小时，$\Phi(x)$的值也会减小，此时$x$被“丢弃”的可能性更高，所以说这是随机依赖于输入的方式



## GELU标准形式

$$
GELU(x) = xP(X<=x)=x\Phi(x)=x \cdot \frac{1}{2}\left[1+\operatorname{erf}\left(\frac{x}{\sqrt{2}}\right)\right]
$$

其中
$$
\Phi(x)=\int_{-\infty}^x \frac{e^{-t^2 / 2}}{\sqrt{2 \pi}} d t=\frac{1}{2}\left[1+\operatorname{erf}\left(\frac{x}{\sqrt{2}}\right)\right]
$$

> 高斯误差函数：$\operatorname{erf}(x)=\frac{2}{\sqrt{\pi}} \int_0^x e^{-t^2} d t$
>
> 互补误差函数：$\operatorname{erfc}(x)=1-\operatorname{erf}(x)=\frac{2}{\sqrt{\pi}} \int_x^{\infty} e^{-t^2} d t$
>
> ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/1015018-20210910114414069-356400172.png)

![GELU](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-45d6312c444b7548558132cea30e3808_720w.webp)

因为erf无解析表达式（实际上已经有精确的计算函数了），因此原论文给出了两种近似表达：

**Sigmoid近似：**
$$
x \Phi(x) \approx x \sigma(1.702 x)
$$
![sigmoid近似GELU](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-9df2f1afc405f0d4fc4c025f4dd348a8_720w.webp)

**tanh近似：**
$$
x \Phi(x) \approx \frac{1}{2} x\left[1+\tanh \left(\sqrt{\frac{2}{\pi}}\left(x+0.044715 x^3\right)\right)\right]
$$
![tanh近似GELU](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-8e88381d6426a69149c342de1eebbb3e_720w.webp)



**GPT中的计算方式**：

```python
class NewGELU(nn.Module):
    """
    Implementation of the GELU activation function currently in Google BERT repo (identical to OpenAI GPT).
    Reference: Gaussian Error Linear Units (GELU) paper: https://arxiv.org/abs/1606.08415
    """
    def forward(self, x):
        return 0.5 * x * (1.0 + torch.tanh(math.sqrt(2.0 / math.pi) * (x + 0.044715 * torch.pow(x, 3.0))))

```

可以看到是采用tanh近似的方式计算



**BERT中的计算方式：**

在pretrained-BERT-pytorch/modeling.py中，已经有了精确的计算方式：

```python
def gelu(x):
    """Implementation of the gelu activation function.
        For information: OpenAI GPT's gelu is slightly different (and gives slightly different results):
        0.5 * x * (1 + torch.tanh(math.sqrt(2 / math.pi) * (x + 0.044715 * torch.pow(x, 3))))
        Also see https://arxiv.org/abs/1606.08415
    """
    return x * 0.5 * (1.0 + torch.erf(x / math.sqrt(2.0)))
```



## 优点

- 在处理负数时不会像ReLU一样将输入裁剪到0，这可能导致梯度消失的问题
- 在大多数情况下比sigmoid和tanh表现更好，因为它在处理较大的输入时，会比sigmoid和tanh产生更强的非线性响应
- 在处理较小的输入时，又比ReLU表现更好，因为它有一个非零的梯度
- GELU函数的导数是连续的，这使得在训练深度神经网络时可以更容易地传播梯度，避免了ReLU函数在 $x=0$ 处的导数不连续的问题，从而减少了训练过程中出现的梯度消失问题



## 延伸问题：激活函数的导数在不可导处怎么处理？

Q: 深度学习中激活函数在不连续可导时的导数怎么处理呢？

**A: 激活函数不要求处处连续可导，在不连续可导处定义好该处的导数即可。**

sigmoid函数是处处连续可导的。其他如ReLU，在0处不连续可导。

**以caffe中的ReLU为例：**

在caffe中，给定输入x，ReLU层可以表述为：

$f(x) = x$, if $x>0$;

$f(x) = negative\_slope * x$, if $x <=0$.

当$negative\_slope>0$时，ReLU是leaky ReLU

$negative\_slope$默认为0， 即标准ReLU

输入$x=0$时，导数为$negative\_slope$（标准ReLU在$x=0$处的导数即为0）