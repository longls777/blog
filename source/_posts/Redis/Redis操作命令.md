---
title: Redis操作命令总结
tags: 
categories: Redis
date: 2023-2-15 11:18:00
index_img: 
banner_img: 
math: true
---

## 键操作

![image-20230215111928680](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215111928680.png)

![image-20230215111946199](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215111946199.png)

![image-20230215111959209](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215111959209.png)

![image-20230215112012996](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215112012996.png)

![image-20230215112020623](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215112020623.png)

![image-20230215112035810](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230215112035810.png)



- SET：添加新键或更新键值
- GET：对键取值
- DEL：删除键
- RENAME：重命名键
- TYPE：查看键类型
- OBJECT IDLETIME：查看键的空转时长
- RANDOMKEY：随机返回一个键

#### 键的生存时间（Time To Live，TTL）或过期时间

- EXPIRE：设置生存时间，以秒为单位
- PEXPIRE：设置生存时间，以毫秒为单位

在经过指定的秒数或者毫秒数之后，服务器就会自动删除生存时间为0的键

- EXPIREAT：设置过期时间，以秒为单位
- PEXPIREAT：设置过期时间，以毫秒为单位

过期时间是一个UNIX时间戳，当键的过期时间来临时，服务器就会自动从数据库中删除这个键



- TTL：返回键的剩余生存时间，以秒为单位
- PTTL：返回键的剩余生存时间，以毫秒为单位

> 虽然有多种不同单位和不同形式的设置命令，但实际上EXPIRE、PEXPIRE、EXPIREAT三个命令都是使用PEXPIREAT命令来实现的

> 键的过期时间通过过期字典来保存

- PERSIST：移除一个键的过期时间（删除过期字典中的对应键值对）

  

## 数据库操作

- SELECT：选择数据库

- FLUSHDB：清空数据库
- DBSIZE：返回数据库键数量

...