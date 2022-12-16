---
title: 伽马分布，卡方分布和指数分布
date: 2022-11-19 13:59:31
index_img: 
banner_img: 
categories: Others
math: true
---

首先需要指出的是，卡方分布和指数分布都是伽马分布的一种特殊情况

## 伽马分布（Gamma-Distribution）

伽马分布一般表示为：$X \sim \Gamma(\alpha, \beta)$，概率密度函数如下所示：

![image-20221119140432845](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221119140432845.png)

## 指数分布

指数分布一般表示为：$X \sim Exp(\lambda)$，概率密度函数如下所示：

![image-20221119140519504](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221119140519504.png)

可以看出，指数分布是伽马分布的一种特殊情况，$X \sim Exp(\lambda) = \Gamma(1, \frac{1}{\lambda})$，当伽马分布的参数$\alpha=1$且参数$\beta=\frac{1}{\lambda}$时，伽马分布就成为了指数分布

## 卡方分布

卡方分布一般表示为：$X \sim \chi^2(n)$，概率密度函数如下所示：

![image-20221119140904643](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221119140904643.png)

可以看出，卡方分布也是伽马分布的一种特殊情况，$X \sim \chi^2(n) = \Gamma(\frac{n}{2},2)$，当伽马分布的参数$\alpha=\frac{n}{2}$且参数$\beta=2$时，伽马分布就成为了卡方分布

