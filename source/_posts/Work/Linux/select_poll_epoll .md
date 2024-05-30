---
title: select、poll、epoll详解
tags: 
categories: Linux
date: 2023-3-21 21:28:00
index_img: 
banner_img: 
math: true
---

## IO多路复用

IO多路复用是指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程

- 在处理 IO 的时候（也就是将数据从内核复制到应用缓冲的时候），阻塞和非阻塞都是同步 IO
- 只有使用了特殊的 API 才是异步 IO

![同步IO与异步IO](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-1ad8e572f0df79ba.png)

基于select、poll、epoll的I/O多路复用模型如下：

![IO多路复用](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230321210314724.png)

> I/O多路复用有很多种实现。在linux上，2.4内核前主要是select和poll，自Linux 2.6内核正式引入epoll以来，epoll已经成为了目前实现高性能网络服务器的必备技术

## Select

当进程A调用select语句的时候，会将进程A添加到多个监听socket的等待队列中：

![select阻塞过程](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-3a36f7cc5e48276a.png)

当网卡接收到数据，然后网卡通过中断信号通知cpu有数据到达，执行中断程序，中断程序主要做了两件事：

1. 将网络数据写入到对应socket的接收缓冲区里面
2. 唤醒队列中的等待进程(A)，重新将进程A放入工作队列中

如下图，将所有等待队列的进程移除，并且添加到工作队列中：

![select中断](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-543af9ba5e6e2ba6.png)

> 上面只是一种情况，当程序调用 Select 时，内核会先遍历一遍 Socket，如果有一个以上的 Socket 接收缓冲区有数据，那么 Select 直接返回，不会阻塞

问题：

- 每次调用 Select 都需要将进程加入到所有监视 Socket 的等待队列，每次唤醒都需要从每个队列中移除。这里涉及了两次遍历，而且每次都要将整个 FDS 列表传递给内核，有一定的开销
- 进程被唤醒后，程序并不知道哪些 Socket 收到数据，还需要遍历一次

## Poll

poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大

> select和poll在内部机制方面并没有太大的差异。相比于select机制，poll只是取消了最大监控文件描述符数限制，并没有从根本上解决select存在的问题

## Epoll

epoll可以理解为event poll(基于事件的轮询)，不同于select/poll，epoll正是保存了那些收到数据的Socket到一个双向链表中，这样一来，就少了一次遍历。epoll = <font color=Orange>减少遍历</font> + <font color=Orange>保存就绪Socket</font>

- 减少遍历：将控制与阻塞分离
- 保存就绪Socket：维护一个rdlist以及rb_tree，类似于双向链表操作



epoll 在 Linux 内核中申请了一个简易的文件系统，用于存储相关的对象，每一个 epoll 对象都有一个独立的 eventpoll 结构体，这个结构体会在内核空间中创造独立的内存，用于存储使用epoll_ctl 方法向 epoll 对象中添加进来的事件。这些事件都会挂到 rbr 红黑树中，这样，重复添加的事件就可以通过红黑树而高效地识别出来

![epoll数据结构](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-32af7c797010bf43.jpg)



epoll底层实现最重要的两个数据结构：epitem和eventpoll。可以简单的认为epitem是和每个用户态监控IO的fd对应的，eventpoll是用户态创建的管理所有被监控fd的结构

#### epoll过程

调用epoll_create，内核会创建一个eventpoll对象。eventpoll对象也是文件系统中的一员，和socket一样，它也会有等待队列

![创建eventpoll对象](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-20eccf6c314f3c76.png)

通过 epoll_ctl 添加 Socket1、Socket2 和 Socket3 的监视，内核会将 eventpoll的**引用** 添加到这三个 Socket 的等待队列中

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-f5413d2fce88ff00.png)



当Socket收到数据之后，中断程序会执行将Socket的**引用**添加到eventpoll对象的rdlist就绪列表中

![添加socket到rdlist](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-129ddaf93b668a13.png)

假设计算机中正在运行进程 A 和进程 B、C，在某时刻进程 A 运行到了 epoll_wait 语句，会将进程A添加到eventpoll的等待队列中

![阻塞加入等待队列](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-5fdf94c3582f06ed.png)



当 Socket 接收到数据，中断程序一方面修改 Rdlist，另一方面唤醒 eventpoll 等待队列中的进程，进程 A 再次进入运行状态。因为Soket包含eventpoll对象的引用，因此可以直接操作eventpoll对象

![中断加入就绪队列](http://longls777.oss-cn-beijing.aliyuncs.com/img/15744422-a5ea34937d73a36a.png)

每次调用poll/select系统调用，操作系统都要把current（当前进程）挂到fd对应的所有设备的等待队列上，可以想象，fd多到上千的时候，这样“挂”法很费事；而每次调用epoll_wait则没有这么罗嗦，epoll只在epoll_ctl时把current挂一遍（这第一遍是免不了的）并给每个fd一个命令“好了就调回调函数”，如果设备有事件了，通过回调函数，会把fd放入rdllist，而每次调用epoll_wait就只是收集rdllist里的fd就可以了——epoll巧妙的利用回调函数，实现了更高效的事件驱动模型

## 对比

![select、poll、epoll之间的区别](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230321212302170.png)



**传统select/poll的另一个致命弱点就是当你拥有一个很大的socket集合，由于网络延时，使得任一时间只有部分的socket是"活跃" 的，而select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降**

**但是epoll不存在这个问题，它只会对"活跃"的socket进 行操作，这是因为在内核实现中epoll是根据每个fd上面的callback函数实现的。于是，只有"活跃"的socket才会主动去调用 callback 函数，其他状态的socket则不会**



## 触发方式

- **Level_triggered(水平触发) **：当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll_wait()时，它还会通知你在上没读写完的文件描述符上继续读写，当然如果你一直不去读写，它会一直通知你，如果系统中有大量你不需要读写的就绪文件描述符，而它们每次都会返回，这样会大大降低处理程序检索自己关心的就绪文件描述符的效率
- **Edge_triggered(边缘触发) **：当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据全部读写完(如读写缓冲区太小)，那么下次调用 epoll_wait()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你，这种模式比水平触发效率高，系统不会充斥大量你不关心的就绪文件描述符

**select，poll模型都是水平触发模式，信号驱动IO是边缘触发模式，epoll模型既支持水平触发，也支持边缘触发，默认是水平触发**



> https://www.jianshu.com/p/722819425dbd/
>
> https://mp.weixin.qq.com/s/U2xJgL7rPqOJ8in-V-Ck6A