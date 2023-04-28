---
title: Deque和Queue的比较
tags: 
categories: Java
date: 2023-4-27 15:07:00
index_img: 
banner_img: 
math: true
---



![Deque和Queue的区别](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230427151132003.png)

## Queue

![Queue](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230427151715632.png)

| Modifier and Type | Method       | Description                                                  |
| ----------------- | ------------ | ------------------------------------------------------------ |
| `boolean`         | `add(E e)`   | 将指定的元素插入到此队列中，如果可以立即执行此操作，而不会违反容量限制， `true`在成功后返回  `IllegalStateException`如果当前没有可用空间，则抛出IllegalStateException。 |
| `E`               | `element()`  | 检索，但不删除，这个队列的头。                               |
| `boolean`         | `offer(E e)` | 如果在不违反容量限制的情况下立即执行，则将指定的元素插入到此队列中。 |
| `E`               | `peek()`     | 检索但不删除此队列的头，如果此队列为空，则返回 `null` 。     |
| `E`               | `poll()`     | 检索并删除此队列的头，如果此队列为空，则返回 `null` 。       |
| `E`               | `remove()`   | 检索并删除此队列的头。                                       |

## Deque

![Deque](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230427151204047.png)



| Modifier and Type | Method                            | Description                                                  |
| ----------------- | --------------------------------- | ------------------------------------------------------------ |
| `boolean`         | `add(E e)`                        | 将指定的元素插入此双端队列表示的队列中（换句话说，在此双端队列的尾部），如果它是立即可行且不会违反容量限制，返回  `true`在成功时和抛出 `IllegalStateException`如果当前没有空间可用的。 |
| `void`            | `addFirst(E e)`                   | 插入此双端队列的前面，如果它是立即可行且不会违反容量限制，抛出一个指定的元素  `IllegalStateException`如果当前没有空间可用。 |
| `void`            | `addLast(E e)`                    | 在插入如果它是立即可行且不会违反容量限制，抛出此双端队列的末尾指定元素  `IllegalStateException`如果当前没有空间可用。 |
| `boolean`         | `contains(Object o)`              | 如果此deque包含指定的元素，则返回 `true` 。                  |
| `Iterator<E>`     | `descendingIterator()`            | 以相反的顺序返回此deque中的元素的迭代器。                    |
| `E`               | `element()`                       | 检索但不删除由此deque表示的队列的头部（换句话说，该deque的第一个元素）。 |
| `E`               | `getFirst()`                      | 检索，但不删除，这个deque的第一个元素。                      |
| `E`               | `getLast()`                       | 检索，但不删除，这个deque的最后一个元素。                    |
| `Iterator<E>`     | `iterator()`                      | 以正确的顺序返回此deque中的元素的迭代器。                    |
| `boolean`         | `offer(E e)`                      | 将指定的元素插入由此deque表示的队列（换句话说，在该deque的尾部），如果可以立即执行，而不违反容量限制， `true`在成功时  `false`如果当前没有可用空间，则返回false。 |
| `boolean`         | `offerFirst(E e)`                 | 在此deque的前面插入指定的元素，除非它会违反容量限制。        |
| `boolean`         | `offerLast(E e)`                  | 在此deque的末尾插入指定的元素，除非它会违反容量限制。        |
| `E`               | `peek()`                          | 检索但不删除由此deque表示的队列的头部（换句话说，此deque的第一个元素），如果此deque为空，则返回 `null` 。 |
| `E`               | `peekFirst()`                     | 检索，但不删除，此deque的第一个元素，或返回 `null`如果这个deque是空的。 |
| `E`               | `peekLast()`                      | 检索但不删除此deque的最后一个元素，如果此deque为空，则返回 `null` 。 |
| `E`               | `poll()`                          | 检索并删除由此deque（换句话说，此deque的第一个元素）表示的队列的 `null`如果此deque为空，则返回  `null` 。 |
| `E`               | `pollFirst()`                     | 检索并删除此deque的第一个元素，如果此deque为空，则返回 `null` 。 |
| `E`               | `pollLast()`                      | 检索并删除此deque的最后一个元素，如果此deque为空，则返回 `null` 。 |
| `E`               | `pop()`                           | 从这个deque表示的堆栈中弹出一个元素。                        |
| `void`            | `push(E e)`                       | 将元素推送到由此deque表示的堆栈（换句话说，在此deque的头部），如果可以立即执行此操作而不违反容量限制，则抛出  `IllegalStateException`如果当前没有可用空间）。 |
| `E`               | `remove()`                        | 检索并删除由此deque表示的队列的头（换句话说，该deque的第一个元素）。 |
| `boolean`         | `remove(Object o)`                | 从此deque中删除指定元素的第一个出现。                        |
| `E`               | `removeFirst()`                   | 检索并删除此deque的第一个元素。                              |
| `boolean`         | `removeFirstOccurrence(Object o)` | 从此deque中删除指定元素的第一个出现。                        |
| `E`               | `removeLast()`                    | 检索并删除此deque的最后一个元素。                            |
| `boolean`         | `removeLastOccurrence(Object o)`  | 从此deque中删除指定元素的最后一次出现。                      |
| `int`             | `size()`                          | 返回此deque中的元素数。                                      |