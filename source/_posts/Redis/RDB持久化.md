---
title: RDB持久化
tags: 
categories: Redis
date: 2023-2-16 16:55:00
index_img: 
banner_img: 
math: true
---

## 什么是RDB持久化

Redis的RDB持久化功能可以将Redis在内存中的数据库状态保存到磁盘里面，避免数据意外丢失

RDB持久化既可以手动执行，也可以根据服务器配置选项定期执行，该功能可以将某个时间点上的数据库状态保存到一个RDB文件中

RDB持久化功能所生成的RDB文件是一个经过压缩的二进制文件，通过该文件可以还原生成RDB文件时的数据库状态

## RDB文件的创建与载入

有两个Redis命令可以用于生成RDB文件：

- SAVE：会阻塞Redis服务器进程，直到RDB文件创建完毕为止，在服务器进程阻塞期间，服务器不能处理任何命令请求
- BGSAVE：会派生出一个子进程，然后由子进程负责创建RDB文件，服务器进程（父进程）继续处理命令请求

RDB文件的载入工作是在服务器启动时自动执行的，所以 Redis并没有专门用于载入RDB文件的命令，只要Redis服务器在启动时检测到RDB文件存在，它就会自动载入RDB文件

> 因为AOF文件的更新频率通常比RDB文件的更新频率高，所以：
>
> - 如果服务器开启了AOF持久化功能，那么服务器会优先使用AOF文件来还原数据库状态
> - 只有在AOF持久化功能处于关闭状态时，服务器才会使用RDB文件来还原数据库状态

在BGSAVE命令执行期间，服务器处理**SAVE、BGSAVE、BGREWRITEAOF**三个命令的方式会和平时有所不同：

- SAVE：客户端发送的SAVE命令会被服务器拒绝，服务器禁止SAVE命令和BGSAVE命令同时执行是**为了避免父进程（服务器进程）和子进程同时执行两个rdbSave调用，防止产生竞争条件**

- BGSAVE：客户端发送的BGSAVE命令会被服务器拒绝，因为**同时执行两个BGSAVE命令也会产生竞争条件**

- BGREWRITEAOF：BGREWRITEAOF和BGSAVE两个命令不能同时执行

  - 如果BGSAVE命令正在执行，那么客户端发送的BGREWRITEAOF命令会被延迟到BGSAVE命令执行完毕之后执行
  - 如果BGREWRITEAOF命令正在执行，那么客户端发送的BGSAVE命令会被服务器拒绝

  > BGREWRITEAOF和BGSAVE两个命令的实际工作都由**子进程**执行，所以这两个命令在操作方面并没有什么冲突的地方，不能同时执行它们只是一个性能方面的考虑——**并发出两个子进程，并且这两个子进程都同时执行大量的磁盘写入操作**，这怎么想都不会是一个好主意



服务器在载入RDB文件期间，会一直处于阻塞状态，直到载入工作完成为止

## 自动间隔性保存

因为BGSAVE命令可以在不阻塞服务器进程的情况下执行，所以Redis允许用户通过设置服务器配置的save选项，让服务器每隔一段时间自动执行一次BGSAVE命令

用户可以通过save选项设置多个保存条件，但只要其中任意一个条件被满足，服务器就会执行BGSAVE命令

例如配置如下：

```python
save 900 1 # 服务器在900秒之内，对数据库进行了至少1次修改
save 300 10 # 服务器在300秒之内，对数据库进行了至少10次修改
save 60 10000 # 服务器在60秒之内，对数据库进行了至少10000次修改
```

Redis的服务器周期性操作函数serverCron默认每隔100毫秒就会执行一次，该函数用于对正在运行的服务器进行维护，它的其中一项工作就是检查save选项所设置的保存条件是否已经满足，如果满足的话，就执行BGSAVE命令

## RDB文件结构

![image-20230217100034471](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230217100034471.png)

- REDIS：常量，五个字符，长度为5字节，通过这五个字符，程序可以在载入文件时，快速检查所载入的文件是否是RDB文件
- db_version：一个字符串表示的整数，长度为4字节，记录了RDB文件的版本号
- databases：包含任意个数据库，以及各个数据库中的键值对数据
- EOF：常量，长度为1字节，标志着RDB文件正文内容的结束
- check_sum：8字节长的无符号整数，保存一个校验和，这个校验和是程序通过对REDIS、db_version、databases、EOF四个部分的内容进行计算得出的。服务器在载入RDB文件时，会将载入数据所计算出的校验和与check_sum所记录的校验和进行对比，以此来检查RDB文件是否有出错或者损坏的情况出现

![image-20230217100955846](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230217100955846.png)

每个非空数据库在RDB文件中都可以保存为SELECTDB、db_number、key_value_pairs三个部分：

- SELECTDB：常量，长度为1字节，表示接下来将读入一个数据库号
- db_number：数据库号
- key_value_pairs：保存了数据库中的所有键值对和过期时间

![image-20230217101541373](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230217101541373.png)

![image-20230217101551487](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230217101551487.png)

> 关于value的编码方式，这里不再介绍，有兴趣可以查阅《redis设计与实现》10.3.3节