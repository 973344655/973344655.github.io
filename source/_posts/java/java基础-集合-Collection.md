---
title: java基础-Collection
date: 2019-07-04 11:19:34
tags: [java]
---

### 一.Collection

##### 1.为什么需要集合
- 可以提供一个容器(可变长)用于存储对象

##### 2.集合关系图
<Strong>集合主要为: Map 和 Collection<br></Strong>

Collection主要包含了Set,List和Queue<br>
![collection1](http://67.216.218.49:8000/file/blogs/java/collection/collection1.png)

##### 3.Iterator
Iterator接口方法:<br>
- hasNext()
- next()
- remove()


### 二.List
###### 概要

ArrayList底层是数组，查询快，增删慢，线程不安全.<br>

List主要: ArrayList,LinkedList,Vector<br>
原文：https://segmentfault.com/a/1190000014240704

##### 1.ArrayList
###### (1).属性
```
//初始化容量为10
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//存储空间
transient Object[] elementData;
```
可见，ArrayList底层是数组<br>
###### (2).构造方法
```
//自己设置初始化容量
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//默认的初始化容量
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

###### (3).扩容

- 新增一个元素 add(E e)

```
public boolean add(E e) {
    //确认容量是否够
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

```

- 确认容量 ensureCapacityInternal(int minCapacity)

```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        //容量不够时，进行扩容
        grow(minCapacity);
}
```
- 扩容 grow(int minCapacity)

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //右移运算，num >> 1,相当于num除以2，相当于扩容1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //第一次扩容后，还是不够，直接为minCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //将原数组拷贝到一个新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}

```

##### 2.Vector

Vector底层也是数组，它与ArrayList的区别为，它是线程安全的.<br>

Vector中方法大都为 synchronized
```
public synchronized int size() {
        return elementCount;
    }
```

##### 3.LinkedList

LinkedList 底层为双向链表<br>

###### (1).属性
```
transient int size = 0;

//头节点
transient Node<E> first;

//尾节点
transient Node<E> last;

//内部节点类
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

###### (2).构造方法
```
//空构造
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

###### (3).增 add(E e)

```
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    /将尾节点设为前节点
    final Node<E> l = last;
    //新节点，参数为(前节点(当前的尾节点)，添加的元素， 后节点)
    final Node<E> newNode = new Node<>(l, e, null);
    //将新节点变为尾节点
    last = newNode;
    //如果前面没有节点，则新节点为头节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

###### (4).删 remove(Object o)

```
public boolean remove(Object o) {
     if (o == null) {
         for (Node<E> x = first; x != null; x = x.next) {
             if (x.item == null) {
                 unlink(x);
                 return true;
             }
         }
     } else {
         for (Node<E> x = first; x != null; x = x.next) {
             if (o.equals(x.item)) {
                 unlink(x);
                 return true;
             }
         }
     }
     return false;
 }

E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    //将前节点的next 指向自己的next
    //将自己从 next链中删除
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    //将后节点的prev指向自己的prev
    //将自己从prev链中删除
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

###### (5).改

```
public E set(int index, E element) {
    //检查是否在范围内
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    //替换
    x.item = element;
    return oldVal;
}

private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private boolean isElementIndex(int index) {
     return index >= 0 && index < size;
 }

//获取指定node
Node<E> node(int index) {
   // assert isElementIndex(index);

   //从头还是尾开始遍历 二分
   if (index < (size >> 1)) {
       Node<E> x = first;
       for (int i = 0; i < index; i++)
           x = x.next;
       return x;
   } else {
       Node<E> x = last;
       for (int i = size - 1; i > index; i--)
           x = x.prev;
       return x;
   }
}
```

###### (6).查 get(int index)

```
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

```

### 三.Set

Set: 不允许有重复数据，去重<br>
Set主要:HashSet,TreeSet,LinkedHashSet<br>

HashSet:底层为哈希表加红黑树<br>
TreeSet:底层为红黑树<br>
LinkedHashSet: 底层为哈希表和一个双向链表<br>

##### 1.HashSet

```
//map<key, value>
private transient HashMap<E,Object> map;

//value
private static final Object PRESENT = new Object();

//构造函数  可以看出实际为HashMap
public HashSet() {
    map = new HashMap<>();
}

//add 看出value 为 PRESENT
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

```
