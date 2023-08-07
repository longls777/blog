---
title: 多GPU训练
tags: 
categories: Practice
date: 2023-08-07 15:27:00
index_img: 
banner_img: 
---



> https://github.com/jia-zhuang/pytorch-multi-gpu-training  有一些点没放过来，可以进去看一下
>
> 上手Distributed Data Parallel的详尽教程 - ZachZhang的文章 - 知乎 https://zhuanlan.zhihu.com/p/486346821



## 方案一：nn.DataParallel



核心在于使用`nn.DataParallel`将模型wrap一下，代码其他地方不需要做任何更改:

```python
model = nn.DataParallel(model)
```

为方便说明，我们假设模型输入为(32, input_dim)，这里的 32 表示batch_size，模型输出为(32, output_dim)，使用 4 个GPU训练。`nn.DataParallel`起到的作用是将这 32 个样本拆成 4 份，发送给 4 个GPU 分别做 forward，然后生成 4 个大小为(8, output_dim)的输出，然后再将这 4 个输出都收集到`cuda:0`上并合并成(32, output_dim)。

可以看出，`nn.DataParallel`没有改变模型的输入输出，因此其他部分的代码不需要做任何更改，非常方便。但弊端是，后续的loss计算只会在`cuda:0`上进行，没法并行，因此会导致负载不均衡的问题。

如果把`loss`放在模型里计算的话，则可以缓解上述负载不均衡的问题，示意代码如下：

```python
class Net:
    def __init__(self,...):
        # code
    
    def forward(self, inputs, labels=None)
        # outputs = fct(inputs)
        # loss_fct = ...
        if labels is not None:
            loss = loss_fct(outputs, labels)  # 在训练模型时直接将labels传入模型，在forward过程中计算loss
            return loss
        else:
            return outputs
```

按照我们上面提到的模型并行逻辑，在每个GPU上会计算出一个loss，这些loss会被收集到`cuda:0`上并合并成长度为 4 的张量。这个时候在做backward的之前，必须对将这个loss张量合并成一个标量，一般直接取mean就可以。这在Pytorch官方文档nn.DataParallel函数中有提到：

> When `module` returns a scalar (i.e., 0-dimensional tensor) in forward(), this wrapper will return a vector of length equal to number of devices used in data parallelism, containing the result from each device.

**例子**

```python
import os
os.environ["CUDA_VISIBLE_DEVICES"]="2,3"  # 必须在`import torch`语句之前设置才能生效
import torch
import torch.optim as optim
import torch.nn as nn
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

from model import Net
from data import train_dataset

device = torch.device('cuda')
batch_size = 64

train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)

model = Net()
model = model.to(device)
optimizer = optim.SGD(model.parameters(), lr=0.1)
model = nn.DataParallel(model)  # 就在这里wrap一下，模型就会使用所有的GPU

# training!
tb_writer = SummaryWriter(comment='data-parallel-training')
for i, (inputs, labels) in enumerate(train_loader):
    # forward
    inputs = inputs.to(device)
    labels = labels.to(device)
    outputs = model(inputs, labels=labels)
    loss = outputs[0]  # 对应模型定义中，模型返回始终是tuple
    loss = loss.mean()  # 将多个GPU返回的loss取平均
    # backward
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    # log
    if i % 10 == 0:
        tb_writer.add_scalar('loss', loss.item(), i)
tb_writer.close()
```



## 方案二：Distributed Data Parallel

分布式数据并行（distributed data parallel），是通过多进程实现的，相比与方案一要复杂很多。可以从以下几个方面理解：

1. 从一开始就会启动多个进程（进程数等于GPU数），每个进程独享一个GPU，每个进程都会独立地执行代码。这意味着每个进程都独立地初始化模型、训练，当然，在每次迭代过程中会通过进程间通信共享梯度，整合梯度，然后独立地更新参数。

2. 每个进程都会初始化一份训练数据集，当然它们会使用数据集中的不同记录做训练，这相当于同样的模型喂进去不同的数据做训练，也就是所谓的数据并行。这是通过`torch.utils.data.distributed.DistributedSampler`函数实现的，不过逻辑上也不难想到，只要做一下数据partition，不同进程拿到不同的parition就可以了，官方有一个简单的demo，感兴趣的可以看一下代码实现：[Distributed Training](https://pytorch.org/tutorials/intermediate/dist_tuto.html#distributed-training)

3. 进程通过`local_rank`变量来标识自己，`local_rank`为0的为master，其他是slave。这个变量是`torch.distributed`包帮我们创建的，使用方法如下：

   ```python
   import argparse  # 必须引入 argparse 包
   parser = argparse.ArgumentParser()
   parser.add_argument("--local_rank", type=int, default=-1)
   args = parser.parse_args()
   ```

   必须以如下方式运行代码：

   ```sh
   python -m torch.distributed.launch --nproc_per_node=2 --nnodes=1 train.py
   ```

   这样的话，`torch.distributed.launch`就以命令行参数的方式将`args.local_rank`变量注入到每个进程中，每个进程得到的变量值都不相同。比如使用 4 个GPU的话，则 4 个进程获得的`args.local_rank`值分别为0、1、2、3。

   上述命令行参数`nproc_per_node`表示每个节点需要创建多少个进程(使用几个GPU就创建几个)；`nnodes`表示使用几个节点，因为我们是做单机多核训练，所以设为1。

4. 因为每个进程都会初始化一份模型，为保证模型初始化过程中生成的随机权重相同，需要设置随机种子。方法如下：

   ```python
   def set_seed(seed):
       random.seed(seed)
       np.random.seed(seed)
       torch.manual_seed(seed)
       torch.cuda.manual_seed_all(seed)
   ```



**DDP完整示例代码：**

```python
import os
os.environ["CUDA_VISIBLE_DEVICES"]="0,1,2,3"
import argparse
import random
import numpy as np
import torch
import torch.optim as optim
import torch.nn as nn
from torch.utils.data.distributed import DistributedSampler
from torch.utils.tensorboard import SummaryWriter

from model import Net
from data import train_dataset, test_dataset

parser = argparse.ArgumentParser()
parser.add_argument("--local_rank", type=int, default=-1)
args = parser.parse_args()

torch.cuda.set_device(args.local_rank)
device = torch.device('cuda', args.local_rank)
torch.distributed.init_process_group(backend='nccl')

# 固定随机种子
seed = 42
random.seed(seed)
np.random.seed(seed)
torch.manual_seed(seed)
torch.cuda.manual_seed_all(seed)

batch_size = 64

model = Net()
model.to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.1)

# training!
if args.local_rank == 0:
    tb_writer = SummaryWriter(comment='ddp-3')

train_sampler = DistributedSampler(train_dataset)
train_loader = torch.utils.data.DataLoader(train_dataset, sampler=train_sampler, batch_size=batch_size)

model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[args.local_rank], output_device=args.local_rank, find_unused_parameters=True)

for i, (inputs, labels) in enumerate(train_loader):
    # forward
    inputs = inputs.to(device)
    labels = labels.to(device)
    outputs = model(inputs)
    loss = criterion(outputs, labels)
    # backward
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    # log
    if args.local_rank == 0 and i % 5 == 0:
        tb_writer.add_scalar('loss', loss.item(), i)

if args.local_rank == 0:
    tb_writer.close()
```



## BUG



发现一个BUG，单机单GPU训练时没问题，但是转换成多GPU训练，以下的pq_end_pos就传不进model里面

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230807153304750.png)

经过研究后发现，单机单GPU训练的时候，model的type为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230807153057827.png)

但是多GPU训练时，model的type变为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230807152623767.png)

原因是model经过torch.nn.parallel.DistributedDataParallel后转变了类型

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230807153214829.png)

