---
title: java 几种队列，初步有个概念
date: 2019-12-27 10:31:08
tags: [java]
---



参考:https://benjaminwhx.com/2018/05/05/%E8%AF%B4%E8%AF%B4%E9%98%9F%E5%88%97Queue/

![queue](http://67.216.218.49:8000/file/blogs/java/collection/queue.png)

## 1.Queue 和 Deque

- Queue

队列 先入先出 (FIFO)

操作| 说明| |
 --|--|--
add   |     增加一个元索     |                如果队列已满，则抛出一个IIIegaISlabEepeplian异常
remove   |移除并返回队列头部的元素  |  如果队列为空，则抛出一个NoSuchElementException异常
element | 返回队列头部的元素       |      如果队列为空，则抛出一个NoSuchElementException异常
offer   |    添加一个元素并返回true   |    如果队列已满，则返回false
poll    |     移除并返问队列头部的元素  |  如果队列为空，则返回null
peek    |   返回队列头部的元素           |  如果队列为空，则返回null
put       |  添加一个元素        |              如果队列满，则阻塞
take   |     移除并返回队列头部的元素   |  如果队列为空，则阻塞



- Deque

双端队列 双端队列是指该队列两端的元素既能入队(offer)也能出队(poll),如果将Deque限制为只能从一端入队和出队，则可实现栈的数据结构

Queue方法| 对应Deque | 分割线| 堆栈 | 对应Deque
--|--|--|--|--
add(e) |addLast(e) | |push(e)| addFirst(e)
offer(e) |offerLast(e)| |pop() |removeFirst()
remove |	removeFirst()| |peek() |peekFirst()
poll | 	pollFirst()|| |
elment |getFirst() || |
peek |peekFirst() || |


## 2.BlockingQueue 和 Queue

BlockingQueue 取出时会判断队列是否空，添加时会判断队列是否满了。会造成阻塞。

## 3.Linked 和 Array


链表和数组区别：https://blog.csdn.net/SNOW_wu/article/details/53172721

主要考虑 增删改查的效率问题(时间)，和 存储(空间)问题。

## 4.SynchronousQueue

因为SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合使用同步队列。

## 5.DelayQueue

一个无界阻塞队列，只有在延迟期满时才能从中提取元素.

使用例子:https://segmentfault.com/a/1190000011266078

## 6.PriorityQueue

PriorityQueue是不同于先进先出队列的另一种队列，特点是存入其中的每项数据都另外附有一个数值，表示这个像的优先级, 每次从队列中取出的是具有最高优先权的元素。优先队列常用堆来实现.

## 7.ConcurrentLinkedQueue

使用CPU原语Compare-And-Swap(CAS，汇编指令CMPXCHG) 操作来保证线程安全。一种轻量级锁。

补充:在Java发展初期，java语言是不能够利用硬件提供的这些便利来提升系统的性能的。但随着java不断的发展，Java本地方法(JNI)的出现，使得java程序能够越过JVM直接调用本地方法。CAS也成为java.util.concurrent的基石，CAS实现了区别于synchronouse同步锁的一种乐观锁。

## 8.TransferQueue

TransferQueue对比与BlockingQueue更强大的一点是，生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费（不仅仅是添加到队列里就完事）.

TransferQueue 效率高。
