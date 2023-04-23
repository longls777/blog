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





# 四、HashMap





# 五、ConcurrentHashMap





# 六、HashTable





# 七、fail-fast机制



