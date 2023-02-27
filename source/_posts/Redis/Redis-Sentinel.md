---
title: Sentinel
tags: 
categories: Redis
date: 2023-2-27 9:47:00
index_img: 
banner_img: 
math: true
---

## 什么是Sentinel

Sentinel（哨岗、哨兵）是Redis的高可用性（high availability）解决方案：由一个或多个Sentinel实例（instance）组成的Sentinel系统（system）可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求

![image-20230227094959045](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227094959045.png)

![image-20230227095019825](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227095019825.png)

![image-20230227095031511](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227095031511.png)

![image-20230227095037937](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227095037937.png)

## Sentinel初始化

初始化Sentinel的最后一步是创建连向被监视主服务器的网络连接，Sentinel将成为主服务器的客户端，它可以向主服务器发送命令，并从命令回复中获取相关的信息

对于每个被Sentinel监视的主服务器来说，Sentinel会创建两个连向主服务器的异步网络连接：

一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复。另一个是订阅连接，这个连接专门用于订阅主服务器的`__sentinel__:hello`频道

> **为什么有两个连接？**
>
> 在Redis目前的发布与订阅功能中，被发送的信息都不会保存在Redis服务器里面，如果在信息发送时，想要接收信息的客户端不在线或者断线，那么这个客户端就会丢失这条信息。因此，为了不丢失`__sentinel__:hello`频道的任何信息，Sentinel必须专门用一个订阅连接来接收该频道的信息
>
> 另一方面，除了订阅频道之外，Sentinel还必须向主服务器发送命令，以此来与主服务器进行通信，所以Sentinel还必须向主服务器创建命令连接
>
> 因为Sentinel需要与多个实例创建多个网络连接，所以Sentinel使用的是异步连接

## 获取主服务器信息

Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息

![image-20230227100635899](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227100635899.png)

通过分析主服务器返回的INFO命令回复，Sentinel可以获取以下两方面的信息：

- 一方面是关于主服务器本身的信息，包括run_id域记录的服务器运行ID，以及role域记录的服务器角色
- 另一方面是关于主服务器属下所有从服务器的信息，每个从服务器都由一个"slave"字符串开头的行记录，每行的ip=域记录了从服务器的P地址，而port=域则记录了从服务器的端口号。根据这些IP地址和端口号，Sentinel无须用户提供从服务器的地址信息，就可以自动发现从服务器

## 获取从服务器信息

当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个新的从服务器创建相应的实例结构之外，Sentinel会创建连接到从服务器的命令连接和订阅连接

在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送INFO命令

![image-20230227101951620](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227101951620.png)

## 向主服务器和从服务器发送信息

在默认情况下，Sentinel会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送命令，命令向服务器的`__sentinel__:hello`频道发送了一条信息，信息的内容由多个参数组成：

![image-20230227102822674](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227102822674.png)

![image-20230227102830312](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227102830312.png)

## 接收来自主服务器和从服务器的频道信息

当Sentinel与一个主服务器或者从服务器建立起订阅连接之后，Sentinel就会通过订阅连接，向服务器发送以下命令：

`SUBSCRIBE __sentinel__:hello`

Sentinel对`__sentinel__:hello`频道的订阅会一直持续到Sentinel与服务器的连接断开为止

这也就是说，对于每个与Sentinel连接的服务器，Sentinel既通过命令连接向服务器的`__sentinel__:hello`频道发送信息，又通过订阅连接从服务器的`__sentinel__:hello`频道接收信息

对于监视同一个服务器的多个Sentinel来说，一个Sentinel发送的信息会被其他Sentinel接收到，这些信息会被用于更新其他Sentinel对发送信息Sentinel的认知，也会被用于更新其他Sentinel对被监视服务器的认知

![image-20230227103414016](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227103414016.png)

![image-20230227103420653](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227103420653.png)

## 创建连向其他Sentinel的命令连接

当Sentinel通过频道信息发现一个新的Sentinel时，它不仅会为新Sentinel在sentinels字典中创建相应的实例结构，还会创建一个连向新Sentinel的命令连接，而新Sentinel也同样会创建连向这个Sentinel的命令连接，最终监视同一主服务器的多个Sentinel将形成相互连接的网络

使用命令连接相连的各个Sentinel可以通过向其他Sentinel发送命令请求来进行信息交换，Sentinel实现主观下线检测和客观下线检测都会使用Sentinel之间的命令连接来进行通信

> **Sentinel之间不会创建订阅连接**
>
> Sentinel在连接主服务器或者从服务器时，会同时创建命令连接和订阅连接，但是在连接其他Sentinel时，却只会创建命令连接，而不创建订阅连接。这是因为Sentinel需要通过接收主服务器或者从服务器发来的频道信息来发现未知的新Sentinel，所以才需要建立订阅连接，而相互已知的Sentinel只要使用命令连接来进行通信就足够了

## 检测主观下线状态

在默认情况下，Sentinel会以每秒一次的频率向所有与它创建了命令连接的实例（包括主服务器、从服务器、其他Sentinel在内）发送PING命令，并通过实例返回的PING命令回复来判断实例是否在线

Sentinel配置文件中的down-after-milliseconds选项指定了Sentinel判断实例进入主观下线所需的时间长度：如果一-个实例在down-after-milliseconds毫秒内，连续向Sentinel返回无效回复，那么Sentinel会修改这个实例所对应的实例结构，在结构的flags属性中打开SRI_S_DOWN标识，以此来表示这个实例已经进入主观下线状态

> **主观下线时长选项的作用范围**
>
> 用户设置的down-after-milliseconds 选项的值，不仅会被Sentinel用来判断主服务器的主观下线状态，还会被用于判断主服务器属下的所有从服务器，以及所有同样监视这个主服务器的其他Sentinel的主观下线状态

> **多个Sentinel设置的主观下线时长可能不同**
>
> down-after-milliseconds选项一个需要注意的地方是，对于监视同一个主服务器的多个Sentinel来说，这些Sentinel所设置的down-after-milliseconds 选项的值也可能不同，因此，当一个Sentinel将主服务器判断为主观下线时，其他Sentinel可能仍然会认为主服务器处于在线状态

## 检查客观下线状态

当Sentinel将一个主服务器判断为主观下线之后，为了确认这个主服务器是否真的下线了，它会向同样监视这一主服务器的其他Sentinel进行询问，看它们是否也认为主服务器已经进入了下线状态（可以是主观下线或者客观下线）。当Sentinel从其他Sentinel那里接收到足够数量的已下线判断之后，Sentinel就会将从服务器判定为客观下线，并对主服务器执行故障转移操作

## 选举领头Sentinel

当一个主服务器被判断为客观下线时，监视这个下线主服务器的各个Sentinel 会进行协商，选举出一个领头Sentinel，并由领头Sentinel对下线主服务器执行故障转移操作

以下是Redis选举领头Sentinel的规则和方法：

- 所有在线的Sentinel都有被选为领头Sentinel的资格，换句话说，监视同一个主服务器的多个在线Sentinel中的任意一个都有可能成为领头Sentinel
- 每次进行领头Sentinel选举之后，不论选举是否成功，所有Sentinel的配置纪元（configuration epoch）的值都会自增一次。配置纪元实际上就是一个计数器，并没有什么特别的
- 在一个配置纪元里面，所有Sentinel都有一次将某个Sentinel设置为局部领头Sentinel的机会，并且局部领头一旦设置，在这个配置纪元里面就不能再更改
- 每个发现主服务器进入客观下线的Sentinel都会要求其他Sentinel将自己设置为局部领头Sentinel
- Sentinel设置局部领头Sentinel的规则是先到先得：最先向目标Sentinel发送设置要求的源Sentinel将成为目标Sentinel的局部领头Sentinel，而之后接收到的所有设置要求都会被目标Sentinel拒绝
- 如果有某个Sentinel被半数以上的Sentinel设置成了局部领头 Sentinel，那么这个Sentinel成为领头Sentinel。举个例子，在一个由10个Sentinel组成的Sentinel系统里面，只要有大于等于10/2+1=6个Sentinel将某个Sentine1设置为局部领头Sentinel，那么被设置的那个Sentinel就会成为领头Sentinel
- 因为领头Sentinel的产生需要半数以上Sentinel的支持，并且每个Sentinel在每个配置纪元里面只能设置一次局部领头Sentinel，所以在一个配置纪元里面，只会出现一个领头Sentinel。
- 如果在给定时限内，没有一个Sentinel被选举为领头Sentinel，那么各个Sentinel将在一段时间之后再次进行选举，直到选出领头Sentinel为止

## 故障转移

在选举产生出领头 Sentinel之后，领头Sentinel将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤：

1. 在已下线主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转换为主服务器
2. 让已下线主服务器属下的所有从服务器改为复制新的主服务器
3. 将已下线主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器

>  **新的主服务器是怎样挑选出来的**
>
> 领头Sentinel 会将已下线主服务器的所有从服务器保存到一个列表里面，然后按照以下规则，一项一项地对列表进行过滤：
>
> 1. 删除列表中所有处于下线或者断线状态的从服务器，这可以保证列表中剩余的从服务器都是正常在线的
> 2. 删除列表中所有最近五秒内没有回复过领头Sentinel的INFO命令的从服务器，这可以保证列表中剩余的从服务器都是最近成功进行过通信的
> 3. 删除所有与已下线主服务器连接断开超过down-after-milliseconds * 10毫秒的从服务器: down-after-milliseconds 选项指定了判断主服务器下线所需的时间，而删除断开时长超过down-after-milliseconds * 10毫秒的从服务器，则可以保证列表中剩余的从服务器都没有过早地与主服务器断开连接，换句话说，列表中剩余的从服务器保存的数据都是比较新的
>
> 之后，领头Sentinel将根据从服务器的优先级，对列表中剩余的从服务器进行排序，并选出其中优先级最高的从服务器。
> 如果有多个具有相同最高优先级的从服务器，那么领头Sentinel将按照从服务器的复制偏移量，对具有相同最高优先级的所有从服务器进行排序，并选出其中偏移量最大的从服务器（复制偏移量最大的从服务器就是保存着最新数据的从服务器）。最后，如果有多个优先级最高、复制偏移量最大的从服务器，那么领头Sentinel将按照运行ID对这些从服务器进行排序，并选出其中运行ID最小的从服务器

