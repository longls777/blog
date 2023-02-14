---
title: Redis的数据结构
tags: 
categories: Redis
date: 2023-2-14 13:37:00
index_img: 
banner_img: 
math: true
---

## Redis常用的五种数据类型（五种对象）

Redis是一种存储key-value的内存型数据库，它的key都是字符串类型，value支持存储5种类型的数据：

- String（字符串类型）
- List（列表类型）
- Hash（哈希表类型、即key-value类型）
- Set（无序集合类型，元素不可重复）
- Zset（有序集合类型，元素不可重复）

这五种数据类型的对象都由一个redisObject结构表示：

![image-20230214134925649](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214134925649.png)

- type：记录对象的类型，可以是下表中的任一个：

![image-20230214135105820](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214135105820.png)

可以对一个数据库键使用TYPE命令，获取此数据库键对应的值对象的类型：

![image-20230214135411221](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214135411221.png)

- encoding：记录对象使用的编码，也就是对象的底层数据结构，可以是下表中的任一个：![image-20230214135730965](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214135730965.png)

每种数据类型的对象都至少使用了两种不同的编码，下表列出了每种数据类型的对象可以使用的编码：

![image-20230214140042228](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214140042228.png)

使用OBJECT ENCODING命令可以查看一个数据库键的值对象的编码：

![image-20230214140204190](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214140204190.png)

> Redis可以根据不同的使用场景来为一个对象设置不同的编码，从而优化对象在某一场景下的效率。
> 举个例子，在列表对象包含的元素比较少时，Redis使用压缩列表作为列表对象的底层实现：因为压缩列表比双端链表更节约内存，并且在元素数量较少时，在内存中以连续块方式保存的压缩列表比起双端链表可以更快被载入到缓存中;
> 随着列表对象包含的元素越来越多，使用压缩列表来保存元素的优势逐渐消失时，对象就会将底层实现从压缩列表转向功能更强、也更适合保存大量元素的双端链表上面

### 字符串对象

字符串对象的编码可以是int、raw或embstr

- int：字符串对象保存的是整数值，并且这个整数值可以用long来表示
- raw：字符串对象保存的是字符串值，且长度大于32字节
- embstr：字符串对象保存的是字符串值，且长度小于32字节

> embstr编码是专门用于保存短字符串的一种优化编码方式，这种编码和raw编码一样，都使用redisObject结构和sdshdr结构来表示字符串对象，但raw编码会调用两次内存分配函数来分别创建redisObject结构和sdshdr结构，而embstr编码则通过调用一次内存分配函数来分配一块连续的空间，空间中依次包含redisObject和sdshdr两个结构
>
> 使用embstr编码的字符串对象来保存短字符串值有以下好处：
> embstr编码将创建字符串对象所需的内存分配次数从raw编码的两次降低为一次。释放embstr编码的字符串对象只需要调用一次内存释放函数，而释放raw编码的字符串对象需要调用两次内存释放函数。因为embstr编码的字符串对象的所有数据都保存在一块连续的内存里面，所以这种编码的字符串对象比起raw编码的字符串对象能够更好地利用缓存带来的优势。

最后要说的是，可以用long double类型表示的浮点数在Redis中也是作为字符串值来保存的。如果我们要保存一个浮点数到字符串对象里面，那么程序会先将这个浮点数转换成字符串值，然后再保存转换所得的字符串值：

![image-20230214142845803](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214142845803.png)

在有需要的时候，程序会将保存在字符串对象里面的字符串值转换回浮点数值，执行某些操作，然后再将执行操作所得的浮点数值转换回字符串值，并继续保存在字符串对象里面：

![image-20230214142945136](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214142945136.png)

![image-20230214143303902](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230214143303902.png)

> 因为Redis没有为embstr编码的字符串对象编写任何相应的修改程序（只有int编码的字符串对象和raw编码的字符串对象有这些程序)，所以embstr编码的字符串对象实际上是只读的。当我们对embstr编码的字符串对象执行任何修改命令时，程序会先将对象的编码从embstr转换成raw，然后再执行修改命令。因为这个原因，embstr编码的字符串对象在执行修改命令之后，总会变成一个raw编码的字符串对象。

### 列表对象

...