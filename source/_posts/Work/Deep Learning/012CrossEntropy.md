---
title: CrossEntropyLoss
tags: 
categories: Deep Learning
date: 2023-9-12 21:27:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-c63bf0c8bbb078faff7bee6cba28b4ee_720w.webp
banner_img: 
math: true
---



> pytorch中交叉熵损失nn.CrossEntropyLoss()的真正计算过程 - 月光下的小趴菜的文章 - 知乎 https://zhuanlan.zhihu.com/p/415829154

## CrossEntropyLoss公式

![img](https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-c63bf0c8bbb078faff7bee6cba28b4ee_720w.webp)

代码如下：

```python
import torch
import torch.nn as nn
import math
import numpy as np

entroy=nn.CrossEntropyLoss()
input=torch.Tensor([[0.1234, 0.5555,0.3211],[0.1234, 0.5555,0.3211],[0.1234, 0.5555,0.3211],])
target = torch.tensor([0,1,2])
output = entroy(input, target)
print(output)
#输出 tensor(1.1142)
# ----------------------------------------------------------------------------
input=np.array(input)
target = np.array(target)
def cross_entorpy(input, target):
    output = 0
    length = len(target)
    for i in range(length):
        hou = 0
        for j in input[i]:
            hou += np.exp(j)
        output += -input[i][target[i]] + np.log(hou)
    return np.around(output / length, 4)
print(cross_entorpy(input, target))
#输出 1.1142
```

