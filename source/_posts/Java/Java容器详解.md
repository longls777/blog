---
title: Java集合详解
tags: 八股
categories: Java
date: 2023-4-23 16:21:00
index_img: 
banner_img: 
math: true
---

1

# 一、概览

![Java集合框架图](http://longls777.oss-cn-beijing.aliyuncs.com/img/Java集合框架图.png)



容器主要包括 Collection 和 Map 两种，Collection 是存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表

**Collection**

![Collection](http://longls777.oss-cn-beijing.aliyuncs.com/img/Collection.png)

1. Set

- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)
- HashSet：基于HashMap实现，不能存入重复元素，不能get，只能遍历，且使用 Iterator 遍历 HashSet 得到的结果不确定，也就是说失去了元素的插入顺序信息。Hash冲突参照HashMap的处理方法(拉链法，红黑树)
- LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序

2. List

- ArrayList：基于动态数组实现，支持随机访问
- Vector：和 ArrayList 类似，但它是线程安全的
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列

3. Queue

- LinkedList：可以用它来实现双向队列
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列



**Map**

![Map](http://longls777.oss-cn-beijing.aliyuncs.com/img/Map.png)

- TreeMap：基于红黑树实现
- HashMap：基于哈希表实现
- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁
- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序



# 二、ArrayList

**1. 说说ArrayList是如何实现的？**

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20191208232221265.png)

ArrayList 的底层是数组队列，相当于**动态数组**。与 Java 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用ensureCapacity操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量

ArrayList继承于 AbstractList，实现了 List，RandomAccess，Cloneable，java.io.Serializable 这些接口

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
  }
```

- RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是**支持快速随机访问**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问
- ArrayList 实现了 Cloneable 接口 ，即重写了函数clone()，能被克隆
- ArrayList 实现了 java.io.Serializable 接口，这意味着ArrayList支持序列化，能通过序列化去传输

**2. 说说ArrayList的扩容机制？**

**重点流程：**

1. 调用add()方法时，调用ensureCapacityInternal（size+1），判断需要的最小容量（如果采用默认无参构造创建ArrayList，则在添加第一个元素时，将数组容量扩充为10）

2. 如果最小容量大于数组长度，则进入grow()方法

3. 在grow()方法中，默认新容量为旧容量的1.5倍左右(oldCapacity为偶数就是1.5倍)

4. 1. 如果新容量仍小于所需最小容量，则扩容为所需最小容量

   2. 如果新容量大于MAX_ARRAY_SIZE（Integer.MAX_VALUE-8）：

   3. 1. 如果所需最小容量大于MAX_ARRAY_SIZE，扩容为Integer.MAX_VALUE
      2. 否则，扩容为MAX_ARRAY_SIZE

5. 调用Arrays.copy()方法，复制并扩容数组

6. 可以在大量add()操作之前调用ensureCapacity（）方法，以提前扩容，避免多次扩容

**详细分析：**

**先来看add()方法**

```java
/**
    * 将指定的元素追加到此列表的末尾。
*/
public boolean add(E e) {
    //添加元素之前，先调用ensureCapacityInternal方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //这里看到ArrayList添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```

**再来看 ensureCapacityInternal() 方法**

```java
//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
   }

   ensureExplicitCapacity(minCapacity);
}
```

当add 进第 1 个元素时，minCapacity 为 1，在 Math.max()方法比较后，minCapacity 为 10，就是说：

以无参数构造方式创建ArrayList时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10

**ensureExplicitCapacity() 方法**

```java
//判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
   modCount++;//操作数+1
    
    //考虑int溢出
  // overflow-conscious code
  if (minCapacity - elementData.length > 0)
     //调用grow方法进行扩容，调用此方法代表已经开始扩容了
     grow(minCapacity);
}
```

**grow() 方法**

```java
/**
     * 要分配的最大数组大小
*/
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/xmmm.png)

**hugeCapacity() 方法**

```java
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



最好在 add 大量元素之前用 ensureCapacity 方法，以减少增量重新分配的次数

**ensureCapacity()方法**

```java
 /**
    如有必要，增加此 ArrayList 实例的容量，以确保它至少可以容纳由minimum capacity参数指定的元素数。
     *
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```



# 三、LinkedList

**1. 说说LinkedList？**

![LinkedList](http://longls777.oss-cn-beijing.aliyuncs.com/img/LinkedList.png)

- LinkedList是一个实现了List接口和Deque接口的双端链表
- LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性
- 使用队列用Queue；使用堆栈用Deque

**2. 说说Arraylist 与 LinkedList 的区别?**

1. **是否保证线程安全**： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
2. **底层数据结构**： Arraylist 底层使用的是 Object 数组；LinkedList 底层使用的是 双向链表 （JDK1.6 之前为循环链表）
3. **插入和删除是否受元素位置的影响**： 
   1.  ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e)方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element)）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作
   2.  LinkedList 采用链表存储，所以对于add(E e)方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置i插入和删除元素的话（(add(int index, E element)） 时间复杂度近似为O(n)因为需要先移动到指定位置再插入
4. **是否支持快速随机访问**： LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)
5. **内存空间占用**： ArrayList 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）



# 四、HashMap

**1. 说说HashMap的原理？**

![HashMap](http://longls777.oss-cn-beijing.aliyuncs.com/img/HashMap.png)

JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）

JDK1.8 开始 HashMap 的组成多了红黑树，在满足下面两个条件之后，会执行链表转红黑树操作，以此来加快搜索速度

- 链表长度大于阈值（默认为 8）
- HashMap 的Node数组（也就是上图的table）长度超过 64

**2. HashMap的键值可以为null吗**

HashMap最多只允许一条Node的key为Null，但允许多条Node的value为Null

**3. HashMap储存的是Java对象实例吗？**

虽然容器号称存储的是 Java 对象，但实际上并不会真正将 Java 对象放入容器中，只是在容器中保留这些对象的引用。也就是说，Java 容器实际上包含的是引用变量，而这些引用变量指向了我们要实际保存的 Java 对象

**4. 说说HashMap的构造函数？**

HashMap 一共提供了四个构造函数，其中 默认无参的构造函数 和 参数为Map的构造函数 为 Java Collection Framework 规范的推荐实现，其余两个构造函数则是 HashMap 专门提供的

```java
//无参构造函数，仅仅将负载因子初始化为默认值，而不会初始化Node数组，
///也就是说，当使用无参构造函数创建HashMap时，HashMap的容量（Node数组长度）和大小（size）都为0
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
    // all other fields defaulted
}
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

**HashMap(int initialCapacity, float loadFactor)**构造函数意在构造一个指定初始容量和指定负载因子的空HashMap，其源码如下：

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //容量最大为2的30次方
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    //这里调用函数计算触发扩容的阈值，threshold/loadFactor就是容量
    this.threshold = tableSizeFor(initialCapacity);
}
```

**tableSizeFor()**方法源码如下，从注释就可以看出来，其**目的是要获得大于等于cap的最小的2的幂**。比如cap=10，则返回16。这里给threshold赋初值而并没有初始化Node数组的容量，那么这个initialCapacity是如何起作用的呢？

答案在resize函数中，**扩容时如果Node数组没有初始化而threshold > 0，那么就会将Node数组初始化为threshold的大小，之后再将threshold更新为length*loadFactor。这样就可以达到，"传入一个初始容量，HashMap中的数组大小初始化为大于等于初始容量的最小二次幂"**

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;//减1是为了让结果>=cap,如cap=8=1000b,不减1而直接操作，最后会得到16=10000b
    n |= n >>> 1;//假设n=001..xxxx 逻辑右移一位并按位或得 0011..xxxx
    n |= n >>> 2;//同理 得到 001111..xxxx
    n |= n >>> 4;// 0011111111..xxxx
    n |= n >>> 8;//接下来同理，可以看到1+2+4+8+16=31，而cap为32位，就是说会把最高位的1之后全置为1
    n |= n >>> 16;//最后再让n+1，就可以得到>=cap的2的最小整数次幂了
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

**5. 说说HashMap的数据结构？**

![HashMap结构](http://longls777.oss-cn-beijing.aliyuncs.com/img/HashMap结构.png)

在JDK1.6和JDK1.7中，HashMap采用**数组+链表**实现，即使用链表处理冲突，同一hash值的key-value键值对都存储在一个链表里。但是当数组中一个位置上的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低 **O(n)**

而在JDK1.8中，HashMap采用**数组+链表+红黑树**实现，当链表长度超过阈值8时，并且数组总容量超过64时，将链表转换为红黑树，这样大大减少了查找时间 **O(logn)** 

而当发生扩容或remove键值对导致原有的红黑树内节点数量小于6时，则又将红黑树转换成链表

**6. 说说HashMap的put方法？**

**JDK1.8**

HashMap 通过 key 的 hashCode 经过**扰动函数处理**过后得到 hash 值，然后通过 **(n - 1) & hash** 判断当前元素存放的位置，这里的 n 指的是数组的长度，相当于**对hash值除n取余**

为什么hash & （n-1）相当于对hash除n取余？原因是Node数组的默认容量是16，随后每次扩容都乘2，也就是说n一直都是2的幂，例如对8取余，不就是&0111吗？ 早期CPU都是使用这样的方式计算余数 ：X & (2^N - 1)，这种方式效率比较高

1. 如果定位到的数组位置没有元素 就直接插入
2. 如果定位到的数组位置有元素就和要插入的 key 使用equals比较，如果 key 相同就直接覆盖并返回旧Value，如果 key 不相同，就判断 p 是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素插入。如果不是就遍历链表插入（**插入的是链表尾部**）
3. 如果在链表插入过程中发现链表长度达到阈值（默认是8），会进入treeifyBin方法判断是否需要转化为红黑树，如果数组长度小于64就先扩容（resize）
4. 在put方法中，最后如果发现实际大小大于阈值则扩容

**JDK1.7**

使用拉链法时采用的是头插法

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值(默认为 8 )，执行 treeifyBin 方法
                    // 这个方法会根据 HashMap 数组来决定是否转换为红黑树。
                    // 只有当数组长度大于或者等于 64 的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是对数组扩容。
                    // 具体可看treeifyBin源码
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) {
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

**3. 说说resize方法？**

随着HashMap中元素的数量越来越多，发生碰撞的概率将越来越大，所产生的子链长度就会越来越长，这样势必会影响HashMap的存取速度。为了保证HashMap的效率，系统必须要在某个临界点进行扩容处理，该临界点就是HashMap中元素的数量在数值上等于threshold（**table数组长度 * 负载因子**）

- threshold：表示最多能容纳Node的阈值，一般threshold=length * loadFactor
- loadFactor：负载因子，默认是0.75，这个默认的值是基于时间和空间考虑而得到的一个比较平衡的点

扩容过程：

- 如果原数组长度超过最大容量（1<<30），不进行扩容，只是把阈值提高到Integer.MAX_VALUE；

- 否则**将数组长度扩容为原来的两倍，阈值也扩大到原来的两倍**

- 随后进行数组的复制：

- - 如果只有一个元素，就直接赋值
  - 如果是红黑树，就拆分红黑树
  - 如果是多个元素的链表，需要进行重新映射，重新映射后，元素的索引要么不变，要么变为index+oldCap，具体原因如下图所示：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20170805175826627)



图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit（红色），因此新的index就会发生这样的变化：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20170805175936719)

因此，我们在扩充HashMap的时候，**不需要像JDK1.7的实现那样重新计算hash**，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“**原索引+oldCap**”

这个设计确实非常的巧妙，既**省去了重新hash值的时间**，而且同时，**由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了**，这一块就是JDK1.8新增的优化点。

以下是一个扩容示例：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20170805180104619)

**4. HashMap中hash函数是怎么实现的?**

**JDK1.8**

高16bit和低16bit做异或获得hash值，然后（n-1）& hash获得下标

使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法，换句话说使用扰动函数之后可以减少碰撞

可以看到key==null时，直接返回0，所以**HashMap中键为null的键值对，一定在第一个桶中**

绝大多数情况下length一般都小于2^16即小于65536。所以hash & (length-1)；结果始终是hash的低16位与（length-1）进行&运算。要是能让hash的高16位也参与运算，会让得到的下标更加散列

如何让高16位也参与运算呢？方法就是让hashCode()和自己的高16位进行^运算。由于与运算和或运算都会使得结果偏向0或者1，并不是均匀的概念，所以用异或

```java
    static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

**JDK1.7**

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

**5. 如何解决hash冲突？**

- HashMap使用的是链地址法（拉链法），将所有hash地址相同的元素都连接在同一链表中

- 除此之外，还有：

- - 开放地址法：外加增量d（线性探测再散列，二次探测再散列）
  - 再哈希：对冲突的hash值再次进行hash处理，直到没有冲突
  - 建立公共溢出区：建立公共溢出区存储所有哈希冲突的数据

**6. 说说HashMap的并发安全性问题？**

- HashMap的线程不安全体现在会**造成死循环、数据丢失、数据覆盖**这些问题。其中死循环和数据丢失是在JDK1.7中出现的问题，在JDK1.8中已经得到解决，然而1.8中仍会有数据覆盖的问题，即在并发执行HashMap的put操作时会发生数据覆盖的情况
- **头插法会将链表的顺序翻转，这也是在多线程环境下会形成死循环的关键点**
- 在JDK1.8的putVal源码中，if((p = tab[i = (n - 1) & hash]) == null)是判断是否出现hash碰撞，假设两个线程A、B都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完这行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所以此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全

**7. HashMap为什么引入红黑树而不是AVL树？**

上面这个问题也可以理解为：有了二叉查找树、平衡树（AVL）为什么还需要红黑树？

二叉查找树，也称有序二叉树（ordered binary tree），或已排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

- 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 任意节点的左、右子树也分别为二叉查找树;
- 没有键值相等的节点（no duplicate nodes）

因为一棵由N个结点随机构造的二叉查找树的高度为logN，所以顺理成章，二叉查找树的一般操作的执行时间为O(logN)。但二叉查找树若退化成了一棵具有N个结点的线性链后，则这些操作最坏情况运行时间为O(N)。

可想而知，我们不能让这种情况发生，为了解决这个问题，于是我们引申出了平衡二叉树，即AVL树，它对二叉查找树做了改进，在我们每插入一个节点的时候，必须保证每个节点对应的左子树和右子树的树高度差不超过1。如果超过了就对其进行左旋或右旋，使之平衡。

虽然平衡树解决了二叉查找树退化为近似链表的缺点，能够把查找时间控制在 O(logN)，不过却不是最佳的，因为平衡树要求每个节点的左子树和右子树的高度差至多等于1，这个要求实在是太严了，导致每次进行插入/删除节点的时候，几乎都会破坏平衡树的规则，进而我们都需要通过左旋和右旋来进行调整，使之再次成为一颗符合要求的平衡树。

显然，如果在那种插入、删除很频繁的场景中，平衡树需要频繁着进行调整，这会使平衡树的性能大打折扣，为了解决这个问题，于是有了红黑树。**红黑树是不符合AVL树的平衡条件的，即每个节点的左子树和右子树的高度最多差1的二叉查找树，但是提出了为节点增加颜色，红黑树是用非严格的平衡来换取增删节点时候旋转次数的降低，任何不平衡都会在三次旋转之内解决，相较于AVL树为了维持平衡的开销要小得多。**

AVL树是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多，**所以红黑树的插入效率相对于AVL树更高。单单在查找方面比较效率的话，由于AVL高度平衡，因此AVL树的查找效率比红黑树更高。**

对主要的几种平衡树作个比较，发现红黑树有着良好的稳定性和完整的功能，性能表现也很不错，综合实力强，在诸如STL的场景中需要稳定表现。实际应用中，若搜索的频率远远大于插入和删除，那么选择AVL，如果发生搜索和插入删除操作的频率差不多，应该选择红黑树

> https://blog.csdn.net/login_sonata/article/details/76598675非常不错

# 五、ConcurrentHashMap

**说说ConcurrentHashMap？**

Java1.7 中 ConcruuentHashMap 使用的**分段锁**，也就是每一个 Segment 上同时只有一个线程可以操作，每一个 Segment 都是一个类似 HashMap 数组的结构，它可以扩容，它的冲突会转化为链表。但是 Segment 的个数一但初始化就不能改变

Java1.8 中的 ConcruuentHashMap 使用的 **Synchronized 锁加 CAS 的机制**。结构也由 Java1.7 中的 Segment 数组 + HashEntry 数组 + 链表 进化成了 Node 数组 + 链表 / 红黑树，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表

![ConcurrentHashMap1.7储存结构](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap储存结构.png)

![ConcurrentHashMap1.8储存结构](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap结构.png)



ConcurrentHashMap 同样也分为 1.7 、1.8 版，两者在实现上略有不同。

#### **Base 1.7**

先来看看 1.7 的实现，下面是他的结构图：

![ConcurrentHashMap1.7](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap1.7.png)



如图所示，是由 Segment 数组、HashEntry 组成，和 HashMap 一样，仍然是数组加链表。

它的核心成员变量：

```java
    /**
     * Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
     */
    final Segment<K,V>[] segments;

    transient Set<K> keySet;
    transient Set<Map.Entry<K,V>> entrySet;
```

Segment 是 ConcurrentHashMap 的一个内部类，主要的组成如下：

```java
 static final class Segment<K,V> extends ReentrantLock implements Serializable {
 
         private static final long serialVersionUID = 2249069246763182397L;
 
         // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
         transient volatile HashEntry<K,V>[] table;
 
         transient int count;
 
        transient int modCount;

        transient int threshold;

        final float loadFactor;
}
```

看看其中 HashEntry 的组成：

```java
 static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile value;
        volatile HashEntry<K,V> next;

        HashEntry(int hash, K key, V value, HashEntry<K,V> next){
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
```

和 HashMap 非常类似，**唯一的区别就是其中的核心数据如 value ，以及链表都是 volatile 修饰的，保证了获取时的可见性**

原理上来说：ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel（Segment 数组数量）的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

下面也来看看核心的 put() 和 get() 方法：

**put() 方法**

```java
   public V put(K key, V value) {
         Segment<K,V> s;
         if (value == null)
             throw new NullPointerException();
         int hash = hash(key);
         int j = (hash >>> segmentShift) & segmentMask;
         if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
              (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
             s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }
```

首先是通过 key 定位到 Segment，之后在对应的 Segment 中进行具体的 put：

```java
   final V put(K key, int hash, V value, boolean onlyIfAbsent) {
             HashEntry<K,V> node = tryLock() ? null :
                 scanAndLockForPut(key, hash, value);
             V oldValue;
             try {
                 HashEntry<K,V>[] tab = table;
                 int index = (tab.length - 1) & hash;
                 HashEntry<K,V> first = entryAt(tab, index);
                 for (HashEntry<K,V> e = first;;) {
                    if (e != null) {
                        K k;
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        e = e.next;
                    }
                    else {
                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        int c = count + 1;
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
                unlock();
            }
            return oldValue;
        }
```

虽然 HashEntry 中的 value 是用 volatile 关键词修饰的，但是并不能保证并发的原子性，所以 put 操作时仍然需要加锁处理

**首先第一步的时候会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 scanAndLockForPut() 自旋获取锁**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap1.png)

1. 尝试自旋获取锁
2. 如果重试的次数达到了 MAX_SCAN_RETRIES 则改为阻塞锁获取，保证能获取成功

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap2.png)

再结合图看看 put 的流程

1. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry
2. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value
3. 为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容
4. 最后会解除在 1 中所获取当前 Segment 的锁

**get() 方法**

```java
 public V get(Object key) {
         Segment<K,V> s; // manually integrate access methods to reduce overhead
         HashEntry<K,V>[] tab;
         int h = hash(key);
         long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
         if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
             (tab = s.table) != null) {
             for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                      (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
                 e != null; e = e.next) {
                K k;
                if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                    return e.value;
            }
        }
        return null;
    }
```

get 逻辑比较简单：

只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上

由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值

ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**

#### **Base 1.8**

1.7 已经解决了并发问题，并且能支持 N 个 Segment 这么多次数的并发，但依然存在 HashMap 在 1.7 版本中的问题，那就是查询遍历链表效率太低

因此 1.8 做了一些数据结构上的调整，首先来看下底层的组成结构：

![ConcurrentHashMap1.8](http://longls777.oss-cn-beijing.aliyuncs.com/img/ConcurrentHashMap1.8_.png)

看起来是不是和 1.8 HashMap 结构类似？

其中抛弃了原有的 Segment 分段锁，而采用了 **CAS + synchronized** 来保证并发安全性

```java
 static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```

将 1.7 中存放数据的 HashEntry 改为 Node，但作用都是相同的，其中的 val next 都用了 volatile 修饰，保证了可见性

**put() 方法**

重点来看看 put() 函数：

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
 		//判断key,value是否为null,如果为null抛出空指针异常
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //判断是否需要初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
             //f 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //如果当前位置的 hashcode == MOVED == -1,则需要进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                //如果都不满足，则利用 synchronized 锁写入数据
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                	//如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

- 根据 key 计算出 hashcode 
- 判断是否需要进行初始化
- f 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，**利用 CAS 尝试写入，失败则自旋保证成功**
- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容
- **如果都不满足，则利用 synchronized 锁写入数据**
- 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树

**get() 方法**

```java
 public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            //根据计算出来的 hashcode 寻值，如果就在桶上那么直接返回值
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            //如果是红黑树那就按照树的方式获取值
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            //就不满足那就按照链表的方式遍历获取值。
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值
- 如果是红黑树那就按照树的方式获取值
- 就不满足那就按照链表的方式遍历获取值

1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的



# 六、HashTable

**Hashtable和HashMap的不同之处**

**1、继承的父类不同**

Hashtable继承自Dictionary类

HashMap继承自AbstractMap类，但二者都实现了Map接口

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {}
```

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {}
```

**2、线程安全性不同**

HashMap是线程不安全的

Hashtable是线程安全的，方法上加上了 Synchronized 

```java
public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * Clears this hashtable so that it contains no keys.
     */
    public synchronized void clear() {
        Entry<?,?> tab[] = table;
        modCount++;
        for (int index = tab.length; --index >= 0; )
            tab[index] = null;
        count = 0;
    }
```

**3、是否提供contains方法**

HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解

Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同

```java
public boolean containsValue(Object value) {
        return contains(value);
    }
```

```java
 public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry<?,?> tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }
```

**4、key和value是否允许null值**

Hashtable中，key和value都不允许出现null值。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的

HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断

**5、内部实现使用的数组初始化和扩容方式不同**

HashTable在不指定容量的情况下的默认容量为11，而HashMap为16，Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂

Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍

> **在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计**，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数，具体证明可以参考http://blog.csdn.net/liuqiyao_01/article/details/14475159，Hashtable初始化桶大小为11，就是桶大小设计为素数的应用（Hashtable扩容后不能保证还是素数）。HashMap采用这种非常规设计，**主要是为了在取模和扩容时做优化，同时减少冲突**，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。

**6、计算hash值方式不同**

为了得到元素的位置，首先需要根据元素的 key计算出一个hash值，然后再用这个hash值来计算得到最终的位置。

HashMap有个hash方法重新计算了key的hash值，避免过高的hash冲突：

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

注意这里计算hash值，先调用hashCode方法计算出来一个hash值，再将hash与右移16位后相异或，从而得到新的hash值。

HashMap在求hash值对应的位置索引时：

```java
int index = (length - 1) & hash
```

将哈希表的大小固定为了2的幂，因为是取模得到索引值，故这样取模时，不需要做除法，只需要做位运算。位运算比除法的效率要高很多

**Hashtable通过计算key的hashCode()来得到hash值就为最终hash值**

HashTable在求hash值位置索引时计算index的方法：（从put方法中可以找到计算索引的方式）

```java
int index = (hash & 0x7FFFFFFF) % tab.length;
```

& 0x7FFFFFFF 的目的是为了将负的hash值转化为正值，因为hash值有可能为负数，而&0x7FFFFFFF后，只有符号位改变，而后面的位都不变

**7 、解决hash冲突方式不同（地址冲突）**

Java8，HashMap中，当出现冲突时可以：

1. 如果冲突数量小于8，则是以链表方式解决冲突
2. 而当冲突大于等于8，且Entry数组大于64时，就会将冲突的Entry转换为红黑树进行存储
3. 而又当数量小于6时，则又转化为链表存储。

而在HashTable中， 都是**以链表方式**存储

**8、迭代器不同**

HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 的 Enumerator 不是 fail-fast 的

快速失败（fail—fast）是java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出ConcurrentModificationException

> 摘自 https://blog.csdn.net/hello_cmy/article/details/105138993

# 七、fail-fast机制

**引入**

在前面介绍 ArrayList的扩容问题时对于modCount的操作没有详细说明，该变量的操作在add，remove等操作中都会发生改变。那么该变量到底有什么作用呢？

**简介**

fail-fast 机制，即快速失败机制，是java集合(Collection)中的一种错误检测机制。**当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生fail-fast，即抛出 ConcurrentModificationException异常。fail-fast机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测bug。**

**fail-fast的出现场景**

在我们常见的java集合中就可能出现fail-fast机制，比如ArrayList，HashMap。在多线程和单线程环境下都有可能出现快速失败

**1、单线程环境下的fail-fast：**

ArrayList发生fail-fast例子：

```java
public static void main(String[] args) {
     List<String> list = new ArrayList<>();
     for (int i = 0 ; i < 10 ; i++ ) {
         list.add(i + "");
     }
     Iterator<String> iterator = list.iterator();
     int i = 0 ;
     while(iterator.hasNext()) {
         if (i == 3) {
             list.remove(3);
         }
         System.out.println(iterator.next());
         i++;
     }
}
```

该段代码定义了一个Arraylist集合，并使用迭代器遍历，在遍历过程中，刻意在某一步迭代中remove一个元素，这个时候，就会发生fail-fast

**HashMap发生fail-fast：**

```java
public static void main(String[] args) {
     Map<String, String> map = new HashMap<>();
     for (int i = 0 ; i < 10 ; i ++ ) {
         map.put(i+"", i+"");
     }
     Iterator<Entry<String, String>> it = map.entrySet().iterator();
     int i = 0;
     while (it.hasNext()) {
         if (i == 3) {
             map.remove(3+"");
         }
         Entry<String, String> entry = it.next();
         System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
         i++;
     }
}
```

**2、多线程环境下：**

```java
public class FailFastTest {
     public static List<String> list = new ArrayList<>();
 
     private static class MyThread1 extends Thread {
           @Override
           public void run() {
                Iterator<String> iterator = list.iterator();
                while(iterator.hasNext()) {
                     String s = iterator.next();
                     System.out.println(this.getName() + ":" + s);
                     try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                }
                super.run();
           }
     }
 
     private static class MyThread2 extends Thread {
           int i = 0;
           @Override
           public void run() {
                while (i < 10) {
                     System.out.println("thread2:" + i);
                     if (i == 2) {
                           list.remove(i);
                     }
                     try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                     i ++;
                }
           }
     }
 
     public static void main(String[] args) {
           for(int i = 0 ; i < 10;i++){
            list.add(i+"");
        }
           MyThread1 thread1 = new MyThread1();
           MyThread2 thread2 = new MyThread2();
           thread1.setName("thread1");
           thread2.setName("thread2");
           thread1.start();
           thread2.start();
     }
}
```

启动两个线程，分别对其中一个对list进行迭代，另一个在线程1的迭代过程中去remove一个元素，结果也是抛出了java.util.ConcurrentModificationException

#### **fail-fast的原理**

fail-fast是如何抛出ConcurrentModificationException异常的，又是在什么情况下才会抛出?

我们知道，对于集合如list，map类，我们都可以通过迭代器来遍历，而Iterator其实只是一个接口，具体的实现还是要看具体的集合类中的内部类去实现Iterator并实现相关方法。这里我们就以ArrayList类为例。在ArrayList中，当调用list.iterator()时，其源码是：

```java
public Iterator<E> iterator() {
	return new Itr();
}
```

即它会返回一个新的Itr类，而Itr类是ArrayList的内部类，实现了Iterator接口，下面是该类的源码：

```java
 /**
     * An optimized version of AbstractList.Itr
     */
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
 
        public boolean hasNext() {
            return cursor != size;
        }
 
        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
 
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();
 
            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
 
        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
 
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

其中，有三个属性：

```java
int cursor;       // index of next element to return
int lastRet = -1; // index of last element returned; -1 if no such
int expectedModCount = modCount;
```

cursor是指集合遍历过程中的即将遍历的元素的索引，lastRet是cursor -1，默认为-1，即不存在上一个时，为-1，它主要用于记录刚刚遍历过的元素的索引

expectedModCount这个就是fail-fast判断的关键变量了，它初始值就为ArrayList中的modCount。（modCount是抽象类AbstractList中的变量，默认为0，而ArrayList 继承了AbstractList ，所以也有这个变量，modCount用于记录集合操作过程中作的修改次数，与size还是有区别的，并不一定等于size）

我们一步一步来看：

```java
public boolean hasNext() {
  	return cursor != size;
}
```

迭代器迭代结束的标志就是hasNext()返回false，而该方法就是用cursor游标和size(集合中的元素数目)进行对比，当cursor等于size时，表示已经遍历完成。

接下来看看最关心的next()方法，看看为什么在迭代过程中，如果有线程对集合结构做出改变，就会发生fail-fast：

```java
@SuppressWarnings("unchecked")
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
    	throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
    	throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```

从源码知道，每次调用next()方法，在实际访问元素前，都会调用**checkForComodification**方法，该方法源码如下：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
    	throw new ConcurrentModificationException();	
}
```

可以看出，该方法才是判断是否抛出ConcurrentModificationException异常的关键。在该段代码中，当modCount != expectedModCount时，就会抛出该异常。但是在一开始的时候，expectedModCount初始值默认等于modCount，为什么会出现modCount != expectedModCount，很明显expectedModCount在整个迭代过程除了一开始赋予初始值modCount外，并没有再发生改变，所以可能发生改变的就只有modCount，在前面关于ArrayList扩容机制的分析中，可以知道在ArrayList进行add，remove，clear等涉及到修改集合中的元素个数的操作时，modCount就会发生改变(modCount ++)

**所以当另一个线程(并发修改)或者同一个线程遍历过程中，调用相关方法使集合的个数发生改变，就会使modCount发生变化，这样在checkForComodification方法中就会抛出ConcurrentModificationException异常**

类似的，hashMap中发生的原理也是一样的

#### **避免fail-fast**

了解了fail-fast机制的产生原理，接下来就看看如何解决fail-fast

**方法1**

在单线程的遍历过程中，如果要进行remove操作，可以调用**迭代器的remove方法**而不是集合类的remove方法。看看ArrayList中迭代器的**remove方法**的源码：

```java
 public void remove() {
     if (lastRet < 0)
     	throw new IllegalStateException();
     checkForComodification();

    try {
    	ArrayList.this.remove(lastRet);
    	cursor = lastRet;
    	lastRet = -1;
    	expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
    	throw new ConcurrentModificationException();
    }
}
```

可以看到，该remove方法并不会修改modCount的值，并且不会对后面的遍历造成影响，因为该方法remove不能指定元素，只能remove当前遍历过的那个元素，所以调用该方法并不会发生fail-fast现象。该方法有局限性

例子：

```java
 public static void main(String[] args) {
           List<String> list = new ArrayList<>();
           for (int i = 0 ; i < 10 ; i++ ) {
                list.add(i + "");
           }
           Iterator<String> iterator = list.iterator();
           int i = 0 ;
           while(iterator.hasNext()) {
                if (i == 3) {
                     iterator.remove(); //迭代器的remove()方法
                }
                System.out.println(iterator.next());
                i ++;
           }
     }
```

**方法2**

使用java并发包(java.util.concurrent)中的类来代替 ArrayList 和HashMap

比如使用 **CopyOnWriterArrayList**代替 ArrayList， CopyOnWriterArrayList在是使用上跟 ArrayList几乎一样， CopyOnWriter是写时复制的容器(COW)，在读写时是线程安全的。该容器在对add和remove等操作时，并不是在原数组上进行修改，而是将原数组拷贝一份，在新数组上进行修改，待完成后，才将指向旧数组的引用指向新数组，所以对于 CopyOnWriterArrayList在迭代过程并不会发生fail-fast现象。但 CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性

对于HashMap，可以使用**ConcurrentHashMap**， ConcurrentHashMap采用了锁机制，是线程安全的。在迭代方面，ConcurrentHashMap使用了一种不同的迭代方式。在这种迭代方式中，当iterator被创建后集合再发生改变就不再是抛出ConcurrentModificationException，取而代之的是在改变时new新的数据从而不影响原有的数据 ，iterator完成后再将头指针替换为新的数据 ，这样iterator线程可以使用原来老的数据，而写线程也可以并发的完成改变。**即迭代不会发生fail-fast，但不保证获取的是最新的数据**

> 摘自https://blog.csdn.net/zymx14/article/details/78394464