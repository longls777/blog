---
title: Java并发知识点详解
tags: 八股
categories: Java
date: 2023-4-25 21:49:00
index_img: 
banner_img: 
math: true
---

# 一、Java内存模型（JMM）

**1. 说说Java内存模型（JMM）？**

![Java内存模型](http://longls777.oss-cn-beijing.aliyuncs.com/img/Java内存模型.png)

- JMM规定线程之间的所有共享变量（包括实例字段，静态字段和数组元素）存储在主内存中，每个线程有自己私有的工作内存，工作内存中存储的是主存中共享变量的副本
- 线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的数据
- **一个线程不能直接访问其他线程的工作内存**
- 主内存和工作内存只是JMM的一个抽象概念，主内存主要对应于Java堆中的对象实例数据部分，而工作内存对应于虚拟机栈的部分区域；从更底层来看，主内存对应于物理硬件的内存，工作内存优先存储于寄存器和高速缓存中，因为程序运行时主要访问的是工作内存
- JMM的目的是屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果

**2. 那么Java线程间是如何通信的？（联系到操作系统中进程间通信方式）**

- 共享内存：通过对主内存中的共享变量进行读-写来进行隐式通信
- 消息传递：线程之间通过发送消息进行显式通信
- Java并发采用的是共享内存，Java线程间的通信总是隐式进行的

**3. 说说Java内存模型中的原子性、可见性和有序性？**

**原子性**

- 指一个操作或者多个操作，要么都执行，并且不会被任何因素打断，要么都不执行
- JMM保证对基本数据类型的访问和读写都是具备原子性的
- synchronized关键字可以保证代码块的原子性

**可见性**

- 指当一个线程修改了共享变量的值，其他线程可以立即得知这个修改，就是一个线程修改的状态对另一个线程是可见的
- JMM通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种方式来实现可见性
- volatile关键字可以保证变量的可见性，volatile的特殊规则保证新值能够立即同步回主存，以及每次使用前立即从主存刷新
- synchronized关键字可以保证可见性，它规定对一个变量执行 unlock 操作之前，必须把变量值同步回主内存（unlock作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才能被其他线程锁定）

**有序性**

- 指在本线程内观察，所有操作都是有序的；在一个线程观察另一个线程，所有操作都是无序的
- 前半句是指“线程内表现为串行的语义（as-if-serial）”，后半句指的是“指令重排序”和“工作内存与主内存同步延迟”，**也就是说，在单线程中，指令重排序不会有危害，相反还会提高程序执行的效率，而在多线程中，指令重排序可能会造成意想不到的错误。**
- volatile关键字可以禁止指令重排序
- synchronized关键字通过“一个变量在同一时刻只允许一条线程对其进行lock操作”来保证有序性（lock作用于主内存的变量，它把一个变量标识为一条线程独占的状态）

**4. 什么是happens-before原则？**

由于存在线程本地内存和主内存的原因，再加上重排序，会导致多线程环境下存在可见性的问题。那么我们正确使用同步、锁的情况下，线程A修改了变量a何时对线程B可见？

我们无法就所有场景来规定某个线程修改的变量何时对其他线程可见，但是我们可以指定某些规则，这规则就是happens-before，从JDK 5 开始，JMM就使用happens-before的概念来阐述多线程之间的内存可见性。

**在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。**

happens-before原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要依据，依靠这个原则，我们解决在并发环境下两操作之间是否可能存在冲突的所有问题。

JSR-133（Java内存模型规范）使用happens-before的概念来指定两个操作之间的执行顺序。由于这两个操作可以在一个线程之内，也可以是在不同线程之间。因此，JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）

- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作（用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性）
- 锁定规则：一个 unLock 操作先行发生于后面对同一个锁的 lock 操作（无论在单线程中还是多线程中，同一个锁如果处于被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行 lock 操作）
- volatile 变量规则：对一个volatile变量的写操作先行发生于后面对这个变量的读操作
- 传递规则：如果操作 A 先行发生于操作 B，而操作 B 又先行发生于操作 C，则可以得出操作 A 先行发生于操作C( happens-before 原则具备传递性)
- 线程启动规则：Thread 对象的 start() 方法先行发生于此线程的每个一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 Thread.join() 方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

# 二、volatile和synchronized

**1. 说说volatile和synchronized？**

**volatile**

volatile关键字有两个语义：

- 一是保证修饰的变量对所有其他线程的可见性。可见性指的是当一个线程修改了共享变量的值，其他线程可以立即得知这个修改，volatile保证新值能够立即同步回主存，以及每次使用前立即从主存刷新
- 二是禁止指令重排序。Java线程中如果操作之间不存在数据依赖关系，这些操作就能被编译器和处理器重排序，重排序时要保证as-if-serial，也就是不管怎么重排序，程序的执行结果不能被改变，而volatile修饰的变量会添加一个内存屏障，禁止将后面的指令重排序到内存屏障之前的位置

> 内存屏障：
>
> 在写volatile变量v之后，插入一个sfence（StoreStore Barriers）
>
> 在读volatile变量v之前，插入一个lfence（LoadLoad Barriers）
>
> https://blog.csdn.net/xindoo/article/details/90236236

**synchronized**

- synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行

- synchronized是可重入锁，一个线程可以多次对自己已经加锁的资源加锁，这种就是重入锁，通俗一点讲就是说一个线程拥有了锁仍然还可以重复申请锁

- synchronized锁的资源只有两类：对象和类（.class），当对非static方法加锁时，锁的资源就是实例对象；对static方法加锁时，资源是该类（.class）

- synchronized修饰方法和修饰代码块的底层实现不同：

- - 修饰代码块：通过monitorenter进入，通过monitorexit释放锁
  - 修饰方法：在flags中设置ACC_SYNCHRONIZED标志，来告诉JVM这是一个同步方法。当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

> synchronized同步代码块：
>
> synchronized关键字经过编译之后，会在同步代码块前后分别形成monitorenter和monitorexit字节码指令，在执行monitorenter指令的时候，首先尝试获取对象的锁，如果这个锁没有被锁定或者当前线程已经拥有了那个对象的锁，锁的计数器就加1，在执行monitorexit指令时会将锁的计数器减1，当减为0的时候就释放锁。如果获取对象锁一直失败，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。
>
> 同步方法：
>
> 方法级的同步是隐式的，无须通过字节码指令来控制，JVM可以从方法常量池的方法表结构中的ACC_SYNCHRONIZED访问标志得知一个方法是否声明为同步方法。当方法调用的时，调用指令会检查方法的ACC_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先持有monitor对象，然后才能执行方法，最后当方法执行完（无论是正常完成还是非正常完成）时释放monitor对象。在方法执行期间，执行线程持有了管程，其他线程都无法再次获取同一个管程。

**2. 说说volatile和synchronized的区别？**

- volatile 关键字只能用于变量而 synchronized 关键字可以修饰方法以及代码块
- volatile 关键字能保证数据的可见性，但不能保证操作的原子性。synchronized 关键字两者都能保证
- volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized 关键字解决的是多个线程之间访问资源的同步性

**3.volatile为什么不能保证原子性？**

volatile方式的i++，总共是四个步骤：

i++实际为load、Increment、store、Memory Barriers 四个操作。

内存屏障是线程安全的，但是内存屏障之前的指令并不是。在某一时刻线程1将 i 的值load取出来，放置到cpu缓存中，然后再将此值放置到寄存器A中，然后A中的值自增1（寄存器A中保存的是中间值，没有直接修改 i，因此其他线程并不会获取到这个自增1的值）。如果在此时线程2也执行同样的操作，获取值 i == 10，自增1变为11，然后马上刷入主内存。此时由于线程2修改了 i的值，实时的线程1中的 i == 10的值缓存失效，重新从主内存中读取，变为11。接下来线程1恢复，将自增过后的A寄存器值11赋值给cpu缓存 i 。这样就出现了线程安全问题。

> https://www.zhihu.com/question/329746124/answer/1205806238 写的很好

**4. synchronized为什么要优化？**

在JDK1.5之前，synchronized是重量级锁，1.6以后对其进行了优化，有了一个 无锁-->偏向锁-->自旋锁-->重量级锁 的锁升级的过程，而不是一上来就是重量级锁了。为什么呢？

因为重量级锁获取锁和释放锁需要经过操作系统，是一个重量级的操作。对于重量锁来说，一旦线程获取失败，就要陷入阻塞状态，并且是操作系统层面的阻塞，这个过程涉及用户态到核心态的切换，是一个开销非常大的操作。而研究表明，线程持有锁的时间是比较短暂的，也就是说，当前线程即使现在获取锁失败，但可能很快地将来就能够获取到锁，这种情况下将线程挂起是很不划算的行为。所以要对"synchronized总是启用重量级锁"这个机制进行优化。

在JVM中synchronized重量级锁的底层原理monitorenter和monitorexit字节码依赖于底层的操作系统的Mutex Lock来实现的，但是由于使用Mutex Lock需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的。

**5. 说说synchronized锁升级的过程？**

synchronized用的锁是存在Java对象头里的，在Java中，对象分为三部分：**对象头、实例数据、对齐填充**

![对象实例存储内容](http://longls777.oss-cn-beijing.aliyuncs.com/img/对象实例存储内容.png)

如果对象是数组类型，对象头有三个字宽；如果是非数组类型，对象头是两个字宽

| 长度     | 内容                   | 说明                           |
| -------- | ---------------------- | ------------------------------ |
| 32/64bit | MarkWord               | 存储对象的hashCode或锁信息等   |
| 32/64bit | Class Metadada Address | 存储对象类型数据的指针         |
| 32/64bit | Array Length           | 数组的长度(如果当前对象是数组) |

![32位JVM MarkWord](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230428172100266.png)

从JDK1.6开始，对synchronized锁进行了优化，分为以下四种：

- 无锁状态
- 偏向锁状态
- 轻量级锁状态
- 重量级锁状态

**锁只能升级不能降级**

不同的锁对应的MarkWord标志位如下：

- 偏向锁：01
- 轻量级锁：00
- 重量级锁：10
- GC标记：11

![32位JVM不同的锁状态](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230428172242426.png)

**无锁->偏向锁**

- 在对象头和栈帧中的锁记录里储存锁偏向的线程ID，以后该线程进入和退出同步块时**不需要进行CAS操作来加锁和解锁, 只需要简单的测试一下锁对象的对象头的MarkWord里是否存储着指向当前线程的偏向锁**

- **当有两个线程来竞争该锁的话，那么偏向锁就失效了， 进而升级成轻量级锁**

**轻量级锁->重量级锁**

- 在锁竞争时，没有抢到锁的线程将**自旋**，不断地循环判断是否能够成功获取锁
- 自旋非常消耗资源，**当某个线程自旋次数达到最大时，轻量级锁就会升级为重量级锁**
- 重量级锁时，后续尝试获取锁的线程会**直接挂起**，等待被唤醒

**锁的比较：**

| 锁       | 优点                                                         | 缺点                                           | 适用场景                           |
| -------- | ------------------------------------------------------------ | ---------------------------------------------- | ---------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步代码方法的性能相差无几 | 如果线程间存在锁竞争, 会带来额外的锁撤销的消耗 | 适用于只有一个线程访问的同步场景   |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度                     | 如果始终得不到锁竞争的线程，使用自旋会消耗CPU  | 追求响应时间，同步块执行速度非常快 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU                              | 线程阻塞，响应时间缓慢                         | 追求吞吐量，同步块执行时间速度较长 |

**6. synchronized作用于静态方法？**

当synchronized作用于静态方法时，其**锁就是当前类的class对象锁**。由于静态成员不专属于任何一个实例对象，是类成员，因此通过class对象锁可以控制静态成员的并发操作。

需要注意的是如果一个线程A调用一个实例对象的非static synchronized方法，而线程B需要调用这个实例对象所属类的静态 synchronized方法，是**允许**的，不会发生互斥现象，因为访问**静态 synchronized 方法占用的锁是当前类的class对象，而访问非静态 synchronized 方法占用的锁是当前实例对象锁**

**7. 说说Monitor锁对象？**

重量级锁也就是通常说synchronized的对象锁，锁标识位为10，其中指针指向的是**monitor对象（也称为管程或监视器锁）的起始地址**。

每个对象都存在着一个 monitor 与之关联，对象与其 monitor 之间的关系有存在多种实现方式，如monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个 monitor 被某个线程持有后，它便处于锁定状态。

在HotSpot虚拟机中，monitor是由**ObjectMonitor**实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）：

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;  //_owner指向持有ObjectMonitor对象的线程
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

ObjectMonitor中有两个队列，**_WaitSet 和 _EntryList**，用来保存ObjectWaiter对象列表( 每个等待锁的线程都会被封装成ObjectWaiter对象)；

**整个monitor运行的机制过程如下：**

_owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合，当线程获取到对象的monitor 后进入 _Owner 区域并把monitor中的owner变量设置为当前线程，同时monitor中的计数器count加1，若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSet 集合中等待被唤醒。若当前线程执行完毕，也将释放monitor并复位count的值，以便其他线程进入获取monitor

具体见下图：

![monitor获取释放流程](http://longls777.oss-cn-beijing.aliyuncs.com/img/monitor获取释放流程.png)

**因此，monitor对象存在于每个Java对象的对象头中（存储的指针的指向），synchronized锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因，同时也是notify/notifyAll/wait等方法存在于顶级对象Object中的原因**

> **notify/notifyAll和wait**方法：在使用这3个方法时，必须处于synchronized代码块或者synchronized方法中，否则就会抛出IllegalMonitorStateException异常，这是因为调用这几个方法前必须拿到当前对象的监视器monitor对象，也就是说notify/notifyAll和wait方法依赖于monitor对象。
>
> 我们知道**monitor 存在于对象头的MarkWord**中（存储monitor引用指针），而**synchronized关键字可以获取 monitor**，这也就是为什么notify/notifyAll和wait方法必须在synchronized代码块或者synchronized方法中调用的原因。
>
> 与**sleep方法**不同的是wait方法调用完成后，线程将被暂停，但wait方法将会释放当前持有的监视器锁（monitor），直到有线程调用notify/notifyAll方法后方能继续执行，而sleep方法**只让线程休眠并不释放锁**。
>
> 同时notify/notifyAll方法调用后，并不会马上释放监视器锁，而是**在相应的synchronized(){}/synchronized方法执行结束后**才自动释放锁。
>
> 摘自https://blog.csdn.net/mulinsen77/article/details/88635558

**8.什么是自旋锁和自适应自旋？**

如果持有锁的线程能在很短时间内释放锁资源，就可以让线程执行一个忙循环（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。但是线程自旋需要消耗cpu的资源，如果一直得不到锁就会浪费cpu资源。因此在jdk1.6引入了自适应自旋锁，自旋等待的时候不固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

**9.什么是锁消除？**

锁消除是指虚拟机即时编译器在运行时，对于一些代码上要求同步但是被检测不可能存在共享数据竞争的锁进行消除。例如String类型的连接操作，String是一个不可变对象，字符串的连接操作总是通过生成新的String对象来进行的，Javac编译器会对String连接做自动优化，在JDK1.5的版本中使用的是StringBuffer对象的append操作，StringBuffer的append方法是同步方法，这段代码在经过即时**编译器**编译之后就会忽略掉所有的同步直接执行。在JDK1.5之后是使用的StringBuilder对象的append操作来优化字符串连接的。

**10.什么是锁粗化？**

将多次连接在一起的加锁、解锁操作合并为一次，将多个连续的锁扩展成一个范围更大的锁。例如每次调用StringBuffer.append方法都需要加锁，如果虚拟机检测到有一系列的连续操作都是对同一个对象反复加锁和解锁，就会将其合并成一个更大范围的加锁和解锁操作

> synchronized锁详解 https://blog.csdn.net/jinjiniao1/article/details/91546512

**11.synchronized在获锁的过程中的阻塞态能不能被中断？**

**synchronized在获锁的过程中的阻塞态是不能被中断的**， 原因是中断（thread.interrupt()）仅是会**设置该线程的中断状态位为true**，至于中断的结果线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身（看你代码里怎么写，检测到中断状态为true后怎么做）

当如果一个线程处于了阻塞状态（如线程调用了thread.sleep、thread.join、thread.wait、condition.await、以及可中断的通道上的 I/O 操作等方法后可进入阻塞状态），则在线程在检查中断标示时如果发现中断标示为true，则会在这些**阻塞方法调用处抛出InterruptedException异常，并且在抛出异常后立即将线程的中断标示位清除，即重新设置为false**，抛出异常是为了线程从阻塞状态醒过来，并在结束线程前让程序员有足够的时间来处理中断请求。

也就是说，线程A在调用thread.interrupt()设置线程B中断状态为true后，如果线程B进入了阻塞状态，则线程中断，抛出异常。

> 而对于synchronized，如果一个线程在等待调用synchronized方法或进入synchronized代码块，这个线程此时并不是处于阻塞状态，而是处于**RUNNABLE**状态，也就不会因为线程中断状态被设置为true而中断。
>
> **这里对不对呢？有问题，以后研究**
>
> synchronized确实是等待获取锁的时候不可中断，拿到锁之后可中断，没获取到锁的情况下，中断操作一直不会生效

也就是说对于synchronized，如果一个线程在等待synchronized锁，那么结果只有两种，要么它获得这把锁继续执行，要么它就保存等待，**即使调用中断线程的方法，也不会生效**



> 一些解释，但是并没有给出原因
>
> https://blog.csdn.net/lmlzww/article/details/126336817
>
> https://juejin.cn/post/7020231777721516040
>
> https://cloud.tencent.com/developer/article/1622813

# 三、线程状态

**1. 说说线程的状态？**

![java线程状态](http://longls777.oss-cn-beijing.aliyuncs.com/img/java线程状态.png)

- 新建（NEW） 创建后尚未启动
- 可运行（RUNNABLE）正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度
- 阻塞（BLOCKED）请求获取 monitor lock 从而**进入 synchronized 函数或者代码块**，但是其它线程已经占用了该 monitor lock，所以处于阻塞状态。要结束该状态进入从而 RUNNABLE 需要其他线程释放 monitor lock
- 等待（WAITING）等待其它线程显式地唤醒。阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入
- 限期等待（TIMED_WAITING）无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒
- 死亡（TERMINATED）可以是线程结束任务之后自己结束，或者产生了异常而结束

**联系操作系统中线程的状态**

- JVM中的RUNNABLE状态对应于操作系统的RUNNING和READY状态，所以说RUNNABLE状态下的线程只是可运行，也有可能没在运行
- 操作系统中的WAITING状态包括了JVM的WAITING，TIMED_WAITING和BLOCKED

**2. join() wait() sleep()  yield()的区别？**

**join：**

一个线程运行中调用另外线程的join方法，则当前线程停止执行，一直等到新join进来的线程执行完毕，才会继续执行。

**Wait：**

- 必须在同步代码块中使用，wait()方法会释放对象的“锁标志”。
- 当一个线程调用wait的时候，会释放同步锁，然后该线程进入等待状态。其他挂起的线程会竞争这个锁，得到锁的继续执行。

**sleep:**

sleep()方法需要指定等待的时间，它可以让当前正在执行的线程在指定的时间内暂停执行，进入阻塞状态，但是sleep()方法不会释放“锁标志”，也就是说如果有synchronized同步块，其他线程仍然不能访问共享数据。

**yield:**

- yield()方法和sleep()方法类似，也不会释放“锁标志
- 线程让步，建议线程调度器让出自己的时间片，给优先级更高的线程执行，但不一定会被线程调度器采纳

# 四、创建线程的方式

**说说Java创建线程有哪几种方式？**

有三种使用线程的方法：

- 实现 Runnable 接口
- 实现 Callable 接口
- 继承 Thread 类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。

- 实现 Runnable 接口

需要实现接口中的 run() 方法

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}
```

使用 Runnable 实例再创建一个 Thread 实例，然后调用 Thread 实例的 start() 方法来启动线程

```java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

- 实现 Callable 接口	

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

- 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

**实现接口 VS 继承 Thread**

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口
- 类可能只要求可执行就行，继承整个 Thread 类开销过大

# 五、乐观锁 悲观锁 读写锁

**1. 说说乐观锁和悲观锁？**

乐观锁和悲观锁是并发状况下的两种不同的策略

- 乐观锁

- - 认为冲突不会发生，所以不会上锁，在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则返回给用户错误的信息，让用户决定如何去做
  - 适合读操作多，写操作少的情况，这样可以省去锁的开销
  - 在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式**CAS**实现的

- 悲观锁

- - 认为冲突一定会发生，所以每次都会上锁，这样线程就会被阻塞，直到悲观锁释放
  - 适合写操作多，冲突多的情况
  - Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现

**2. 说说读写锁？**

- 读写锁是一组锁，包括读锁和写锁
- 读锁可以在没有写锁的时候被多个线程同时持有，写锁是独占的。
- JDK提供ReadWriteLock读写锁接口，唯一一个实现类ReentrantReadWriteLock

# 六、ReentrantLock

**1. 对比一下synchronized 和 ReentrantLock？**

- 锁的实现

- - synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK （AQS）实现的

- 性能

- - 新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 的效率大致相同

- 等待可中断

- - 当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情
  - ReentrantLock 可中断，而 synchronized 不行

- 公平锁

- - 公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁
  - synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的

- 锁绑定多个条件

- - 一个 ReentrantLock 可以同时绑定多个 Condition 对象

- 两者都是可重入锁

- - “可重入锁” 指的是自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁

**ReentrantLock 比 synchronized 增加了一些高级功能**

主要有三点：

- 等待可中断 ：ReentrantLock提供了一种能够中断等待锁的线程的机制，通过 lock.lockInterruptibly() 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- 可实现公平锁 ：ReentrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。ReentrantLock默认情况是非公平的，可以通过 ReentrantLock类的ReentrantLock(boolean fair)构造方法来制定是否是公平的。
- 可实现选择性通知（锁可以绑定多个条件）：synchronized关键字与wait()和notify()/notifyAll()方法相结合可以实现等待/通知机制。ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition()方法。Condition是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用notify()/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知” ，这个功能非常重要，而且是 Condition 接口默认提供的。而synchronized关键字就相当于整个 Lock 对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。

**2. ReentrantLock如何实现公平锁？**

ReentrantLock实现公平锁的方式是通过在AQS中维护一个等待队列，当线程请求锁时，会将线程放到等待队列的队尾，等待前面的线程释放锁后依次获取锁。同时，在释放锁时，也会按照队列顺序来通知下一个等待线程获取锁。

# 七、AQS

**1.说说AQS？**

AQS（AbstractQueuedSynchronizer） 类如其名，抽象的队列式的同步器，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch。

核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 **CLH 队列锁**实现的，即**将暂时获取不到锁的线程加入到队列中**。

CLH（Craig,Landin,and Hagersten）队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

![CLH队列锁](http://longls777.oss-cn-beijing.aliyuncs.com/img/AQS.png)

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过 protected 类型的 getState，setState，compareAndSetState 进行操作

```java
//返回同步状态的当前值
protected final int getState() {
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) {
        state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**AQS 定义两种资源共享方式**

- Exclusive（独占）：只有一个线程能执行，如 ReentrantLock。又可分为公平锁和非公平锁：

- - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- Share（共享）：多个线程可同时执行，如 CountDownLatch、Semaphore、 CyclicBarrier、ReadWriteLock

**AQS 底层使用了模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承 AbstractQueuedSynchronizer 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

AQS 使用了模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的模板方法：

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 UnsupportedOperationException。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS 类中的其他方法都是 final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

# 八、CAS

**说说CAS?**

- CAS就是compare and swap，比较和交换，CAS是一种基于锁的操作，而且是乐观锁

- JVM中的CAS操作利用处理器提供的CMPXCHG指令实现原子操作

- CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B

- CAS存在三大问题：

- - ABA问题：如果一个值原来是A，变成了B，又变成了A，那么CAS会认为它的值没有发生变化，但是实际上却变化了，解决方法是使用版本号，在变量前追加版本号
  - 循环时间长开销大：自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销，如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升
  - 只能保证一个共享变量的原子操作：当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作

# 九、双重校验锁实现单例模式

**说说双重校验锁实现单例模式（DCL）？**

```java
/**
 * @author Li Shilong
 * @data 2021/5/4 12:54
 * @Description 双重检验锁实现单例模式
 */
public class Singleton {
    
    private volatile static Singleton  singleton;
    
    private Singleton(){};//注意私有构造器
    
    public Singleton getSingleton(){
        if(singleton==null){
            synchronized (Singleton.class){
                if(singleton==null){
                    singleton = new Singleton();
                    return singleton;
                }
            }
        }
        return singleton;
    }
}
```

**为什么是双重校验锁实现单例模式呢？**

- 第一次校验：也就是第一个if（singleton==null），这个是为了**提高代码执行效率**，由于单例模式只要一次创建实例即可，所以当创建了一个实例之后，再次调用getInstance方法就不必要进入同步代码块，不用竞争锁。直接返回前面创建的实例即可。
- 第二次校验：也就是第二个if（singleton == null），这个校验是**防止二次创建实例**，假如有一种情况，当singleton还未被创建时，线程t1调用getInstance方法，由于第一次判断singleton==null，此时线程t1准备继续执行，但是由于资源被线程t2抢占了，此时t2页调用getInstance方法，同样的，由于singleton并没有实例化，t2同样可以通过第一个if，然后继续往下执行，同步代码块，第二个if也通过，然后t2线程创建了一个实例singleton。此时t2线程完成任务，资源又回到t1线程，t1此时也进入同步代码块，如果没有这个第二个if，那么，t1就也会创建一个singleton实例，那么，就会出现创建多个实例的情况，但是加上第二个if，就可以完全避免这个多线程导致多次创建实例的问题。

**所以说：两次校验都必不可少**

还有，这里的private static volatile Singleton singleton;中的volatile也必不可少，volatile关键字可以防止JVM指令重排优化，因为 singleton = new Singleton() 这句话可以分为三步：

1. 为 singleton 分配内存空间

2. 初始化 singleton

3. 将 singleton 指向分配的内存空间

   但是由于JVM具有指令重排的特性，执行顺序有可能变成 1-3-2。 指令重排在单线程下不会出现问题，但是在多线程下会导致一个线程获得一个未初始化的实例。例如：线程T1执行了1和3，此时T2调用 getInstance() 后发现 singleton 不为空，因此返回 singleton，但是此时的 singleton 还没有被初始化。

   使用 volatile 会禁止JVM指令重排，从而保证在多线程下也能正常执行

# 十、线程池

**说说Java的线程池？**

**1. 为什么要使用线程池？**

- 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

**2.  JUC中的线程池体系**

![线程池体系](http://longls777.oss-cn-beijing.aliyuncs.com/img/线程池体系.png)

**3. ThreadPoolExecutor的创建参数**

```java
public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
    int maximumPoolSize,//线程池的最大线程数
    long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
    TimeUnit unit,//时间单位
    BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
    ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
    RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
    )
```

- corePoolSize：**核心线程数**定义了最小可以同时运行的线程数量。
- maximumPoolSize：当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为**最大线程数**。
- workQueue：当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。默认为LinkedBlockingQueue

- keepAliveTime：默认为0，当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁
- unit：keepAliveTime 参数的时间单位
- threadFactory：是构造Thread的方法，可以使用默认的default实现，也可以自己去包装和传递，需要事项newThread方法
- handler：饱和策略，当线程池中线程数量达到maximumPollSize时采取的抛弃策略，默认是AbortPolicy

**3. 线程池增长策略**

![线程池增长策略](http://longls777.oss-cn-beijing.aliyuncs.com/img/线程池增长策略.png)

- 如果此时线程池中的数量小于 corePoolSize ，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
- 如果此时线程池中的数量等于 corePoolSize ，但是缓冲队列 workQueue 未满，那么任务被放入缓冲队列。
- 如果此时线程池中的数量大于 corePoolSize ，缓冲队列 workQueue 满，并且线程池中的数量小于maximumPoolSize ，建新的线程来处理被添加的任务。
- 如果此时线程池中的数量大于 corePoolSize ，缓冲队列 workQueue 满，并且线程池中的数量等于maximumPoolSize ，那么通过 handler 所指定的策略来处理此任务。
- 处理任务的优先级为：核心线程 corePoolSize 、任务队列 workQueue 、最大线程 maximumPoolSize ，如果三者都满了，使用handler 处理被拒绝的任务。
- 当线程池中的线程数量大于 corePoolSize 时，如果某线程空闲时间超过keepAliveTime ，线程将被终止。这样，线程池可以动态的调整池中的线程数。

![线程池流程](http://longls777.oss-cn-beijing.aliyuncs.com/img/线程池流程.png)

**4. 饱和策略**

如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，ThreadPoolTaskExecutor 定义一些策略:

- ThreadPoolExecutor.AbortPolicy：抛出 RejectedExecutionException来拒绝新任务的处理。
- ThreadPoolExecutor.CallerRunsPolicy：调用执行自己的线程运行任务，也就是直接在调用execute方法的线程中运行(run)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- ThreadPoolExecutor.DiscardPolicy： 不处理新任务，直接丢弃掉。
- ThreadPoolExecutor.DiscardOldestPolicy： 此策略将丢弃最早的未处理的任务请求，也就是工作队列头部的任务，然后重试执行程序

**5. 线程池参数如何设置（线程池的应用场景）？**

- 如果任务为IO密集型，比如读取数据库，文件读写以及网络通信等，这样的任务不会占据很多CPU但是会比较耗时，这时候线程数量设置为2倍CPU数以上，充分利用CPU资源
- 如果任务为CPU密集型，比如大量计算，压缩，解压等操作，这时候一般设置线程数为CPU数+1，+1有一种说法是备份线程
- 如果任务既有IO密集型，又有CPU密集型，那么最好分开处理，IO密集型用IO密集型线程池，CPU密集型用CPU密集型线程池

**6. Executors工厂类实现的线程池**

**7. 线程池中使用的BlockingQueue**

- ArrayBlockingQueue：使用数组实现的有界阻塞队列，特性先进先出
- LinkedBlockingQueue：使用链表实现的阻塞队列，特性先进先出，可以设置其容量，默认为Interger.MAX_VALUE，特性先进先出
- PriorityBlockingQueue：使用平衡二叉树堆，实现的具有优先级的无界阻塞队列
- DelayQueue：无界阻塞延迟队列，队列中每个元素均有过期时间，当从队列获取元素时，只有过期元素才会出队列。队列头元素是最快要过期的元素。
- SynchronousQueue：一个不存储元素的阻塞队列，每个插入操作，必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态

**8. 说说ArrayBlockingQueue和LinkedBlockingQueue的区别？**

- 队列大小不同： ArrayBlockingQueue是有界的，必须指定队列的大小，而 LinkedBlockingQueue默认是无界的（Integer.MAX_VALUE）也可以是有界的，若用默认大小且当生产速度大于消费速度时候，有可能会内存溢出
- 数据储存容器不同： ArrayBlockingQueue使用的是数组，而 LinkedBlockingQueue使用的是以Node为节点的链表
- 队列中锁的实现不同： ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；LinkedBlockingQueue实现的队列中的锁是分离的，即添加用的是putLock，移除是takeLock，这样意味着在高并发的情况下，生产者和消费者可以并行地操作队列中的数据，以此来提高队列的并发性能

# 十一、ThreadLocal

多线程访问同一个共享变量的时候容易出现并发问题，特别是多个线程对一个变量进行写入的时候，为了保证线程安全，一般使用者在访问共享变量的时候需要进行额外的同步措施才能保证线程安全性。ThreadLocal是除了加锁这种同步方式之外的一种保证一种规避多线程访问出现线程不安全的方法，当我们在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量这样就不会存在线程不安全问题。

　　ThreadLocal是JDK包提供的，它提供线程本地变量，如果创建一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个副本，在实际多线程操作的时候，操作的是自己本地内存中的变量，从而规避了线程安全问题，如下图所示

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/1368768-20190613220434628-1803630402.png)



![ThreadLocal的类图结构](http://longls777.oss-cn-beijing.aliyuncs.com/img/1368768-20190614000329689-872917045.png)



上面是ThreadLocal的类图结构，从图中可知：Thread类中有两个变量threadLocals和inheritableThreadLocals，二者都是ThreadLocal内部类ThreadLocalMap类型的变量，我们通过查看内部内ThreadLocalMap可以发现实际上它类似于一个HashMap。在默认情况下，每个线程中的这两个变量都为null

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/1368768-20190614002358591-1764103391.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/1368768-20190614002426565-1963789099.png)

只有当线程第一次调用ThreadLocal的set或者get方法的时候才会创建他们（后面我们会查看这两个方法的源码）。除此之外，**每个线程的本地变量不是存放在ThreadLocal实例中，而是放在调用线程的ThreadLocals变量里面**（前面也说过，该变量是Thread类的变量）。也就是说，**ThreadLocal类型的本地变量是存放在具体的线程空间上**，其本身相当于一个装载本地变量的工具壳，通过set方法将value添加到调用线程的threadLocals中，当调用线程调用get方法时候能够从它的threadLocals中取出变量。**如果调用线程一直不终止，那么这个本地变量将会一直存放在他的threadLocals中，所以不使用本地变量的时候需要调用remove方法将threadLocals中删除不用的本地变量**

> https://www.cnblogs.com/fsmly/p/11020641.html

# 十二、unsafe

Unsafe类是在sun.misc包下，不属于Java标准。但是很多Java的基础类库，包括一些被广泛使用的高性能开发库都是基于Unsafe类开发的，比如Netty、Cassandra、Hadoop、Kafka等。Unsafe类在提升Java运行效率，增强Java语言底层操作能力方面起了很大的作用。

Unsafe类使Java拥有了像C语言的指针一样操作内存空间的能力，同时也带来了指针的问题。过度的使用Unsafe类会使得出错的几率变大，因此Java官方并不建议使用的，官方文档也几乎没有。Oracle正在计划从Java 9中去掉Unsafe类，如果真是如此影响就太大了。

**一、内存管理。**包括分配内存、释放内存等。

该部分包括了allocateMemory（分配内存）、reallocateMemory（重新分配内存）、copyMemory（拷贝内存）、freeMemory（释放内存 ）、getAddress（获取内存地址）、addressSize、pageSize、getInt（获取内存地址指向的整数）、getIntVolatile（获取内存地址指向的整数，并支持volatile语义）、putInt（将整数写入指定内存地址）、putIntVolatile（将整数写入指定内存地址，并支持volatile语义）、putOrderedInt（将整数写入指定内存地址、有序或者有延迟的方法）等方法。getXXX和putXXX包含了各种基本类型的操作。

利用copyMemory方法，我们可以实现一个通用的对象拷贝方法，无需再对每一个对象都实现clone方法，当然这通用的方法只能做到对象浅拷贝。

**二、非常规的对象实例化。**

allocateInstance()方法提供了另一种创建实例的途径。通常我们可以用new或者反射来实例化对象，使用allocateInstance()方法可以直接生成对象实例，且无需调用构造方法和其它初始化方法。

这在对象反序列化的时候会很有用，能够重建和设置final字段，而不需要调用构造方法。

**三、操作类、对象、变量。**

这部分包括了staticFieldOffset（静态域偏移）、defineClass（定义类）、defineAnonymousClass（定义匿名类）、ensureClassInitialized（确保类初始化）、objectFieldOffset（对象域偏移）等方法。

通过这些方法我们可以获取对象的指针，通过对指针进行偏移，我们不仅可以直接修改指针指向的数据（即使它们是私有的），甚至可以找到JVM已经认定为垃圾、可以进行回收的对象。

**四、数组操作。**

这部分包括了arrayBaseOffset（获取数组第一个元素的偏移地址）、arrayIndexScale（获取数组中元素的增量地址）等方法。arrayBaseOffset与arrayIndexScale配合起来使用，就可以定位数组中每个元素在内存中的位置。

由于Java的数组最大值为Integer.MAX_VALUE，使用Unsafe类的内存分配方法可以实现超大数组。实际上这样的数据就可以认为是C数组，因此需要注意在合适的时间释放内存。

**五、多线程同步。**包括锁机制、CAS操作等。

这部分包括了monitorEnter、tryMonitorEnter、monitorExit、compareAndSwapInt、compareAndSwap等方法。

其中monitorEnter、tryMonitorEnter、monitorExit已经被标记为deprecated，不建议使用。

Unsafe类的CAS操作可能是用的最多的，它为Java的锁机制提供了一种新的解决办法，比如AtomicInteger等类都是通过该方法来实现的。compareAndSwap方法是原子的，可以避免繁重的锁机制，提高代码效率。这是一种乐观锁，通常认为在大部分情况下不出现竞态条件，如果操作失败，会不断重试直到成功。

**六、挂起与恢复。**

这部分包括了park、unpark等方法。

将一个线程进行挂起是通过park方法实现的，调用 park后，线程将一直阻塞直到超时或者中断等条件出现。unpark可以终止一个挂起的线程，使其恢复正常。整个并发框架中对线程的挂起操作被封装在 LockSupport类中，LockSupport类中有各种版本pack方法，但最终都调用了Unsafe.park()方法。

**七、内存屏障。**

这部分包括了loadFence、storeFence、fullFence等方法。这是在Java 8新引入的，用于定义内存屏障，避免代码重排序。

loadFence() 表示该方法之前的所有load操作在内存屏障之前完成。同理storeFence()表示该方法之前的所有store操作在内存屏障之前完成。fullFence()表示该方法之前的所有load、store操作在内存屏障之前完成。