---
title: java基础-Map
date: 2019-07-08 16:50:34
tags: [java]
---

### 一.Map和Collection

Java 集合类总共两大接口: Map 和 Collection<br>

其中: Map 中元素是成对出现的(key, value), Collection 中元素是单独出现的.  如:<br>
```
Map<String,String> //true ,成对

List<String> //true, 单独
List<Map<String,String>>// true, 单独， 一个Map对象

List<String, String> //false
```

### 二.主要实现类

- HashMap
- ConcurrentHashMapp
- HashTable
- TreeMap
- LinkendHashMap

### 三.具体实现

#### 1.HashMap
在JDK1.8中 HashMap由位桶+链表+红黑树实现.<br>

##### 1.1 散列表

###### 1.1.1 概念
原文：https://www.cnblogs.com/absfree/p/5508570.html<br>

符号表是一种用于存储键值对（key-value pair）的数据结构，我们平常经常使用的数组也可以看做是一个特殊的符号表，数组中的“键”即为数组索引，值为相应的数组元素。也就是说，当符号表中所有的键都是较小的整数时，我们可以使用数组来实现符号表，将数组的索引作为键，而索引处的数组元素即为键对应的值，但是这一表示仅限于所有的键都是比较小的整数时，否则可能会使用一个非常大的数组。<strong>散列表是对以上策略的一种“升级”，但是它可以支持任意的键而并没有对它们做过多的限定。</strong> <br>
对于基于散列表实现的符号表，若我们要在其中查找一个键，需要进行以下步骤：<br>

- 首先我们使用散列函数将给定键转化为一个“数组的索引”，理想情况下，不同的key会被转为不同的索引，但在实际应用中我们会遇到不同的键转为相同的索引的情况，这种情况叫做碰撞。

- 得到了索引后，我们就可以像访问数组一样，通过这个索引访问到相应的键值对。


以上就是散列表的核心思想，散列表是 <strong>时空权衡</strong> 的经典例子。当我们的空间无限大时，我们可以直接使用一个很大的数组来保存键值对，并用key作为数组索引，因为空间不受限，所以我们的键的取值可以无穷大，因此查找任何键都只需进行一次普通的数组访问。反过来，若对查找操作没有任何时间限制，我们就可以直接使用链表来保存所有键值对，这样把空间的使用降到了最低，但查找时只能顺序查找。在实际的应用中，我们的时间和空间都是有限的，所以我们必须在两者之间做出权衡，散列表就在时间和空间的使用上找到了一个很好的平衡点。散列表的一个优势在于我们只需调整散列算法的相应参数而无需对其他部分的代码做任何修改就能够在时间和空间的权衡上做出策略调整。<br>

###### 1.1.2 散列函数

在散列表内部，我们使用 <strong>桶（bucket）</strong>来保存键值对，我们前面所说的数组索引即为桶号，决定了给定的键存于散列表的哪个桶中。散列表所拥有的桶数被称为散列表的<strong> 容量（capacity）</strong><br>

现在假设我们的散列表中有M个桶，桶号为0到M-1。我们的散列函数的功能就是把任意给定的key转为[0, M-1]上的整数。我们对散列函数有两个基本要求：一是计算时间要短，二是尽可能把键分布在不同的桶中。对于不同类型的键，我们需要使用不同的散列函数，这样才能保证有比较好的散列效果。<br>

我们使用的散列函数应该尽可能满足均匀散列假设,使用的散列函数能够均匀并独立地将所有的键散布于0到M – 1之间。这样一来，满足均匀性与独立性能够保证键值对在散列表的分布尽可能的均匀，不会出现“许多键值对被散列到同一个桶，而同时许多桶为空”的情况。<br>

Java中的常用类，基本都重写了 <Strong>hashCode()</strong> 方法，用于获取其散列值.<br>

###### 1.1.3 获取桶号

前面我们介绍了计算对象hashCode的一些方法，那么我们获取了hashCode之后，如何进一步得到桶号呢？一个直接的办法就是直接拿得到的hashCode除以capacity（桶的数量），然后用所得的余数作为桶号。不过在Java中，hashCode是int型的，而Java中的int型均为有符号，所以我们要是直接使用返回的hashCode的话可能会得到一个负数，显然桶号是不能为负的。所以我们先将返回的hashCode转变为一个非负整数，再用它除以capacity取余数，作为key的对应桶号，具体代码如下：<br>
```
private int hash(K key) {
    return (key.hashCode() & 0x7fffffff) % M;
}
```

###### 1.1.4 处理碰撞

- 拉链法

以这种方式实现的散列表，每个桶里都存放了一个链表。初始时所有链表均为空，当一个键被散列到一个桶时，这个键就成为相应桶中链表的首结点，之后若再有一个键被散列到这个桶（即发生碰撞），第二个键就会成为链表的第二个结点，以此类推。这样一来，当桶数为M，散列表中存储的键值对数目为N时，平均每个桶中的链表包含的结点数为N / M。因此，当我们查找一个键时，首先通过散列函数确定它所在的桶，这一步所需时间为O(1)；然后我们依次比较桶中结点的键与给定键，若相等则找到了指定键值对，这一步所需时间为O(N / M)。所以查找操作所需的时间为O(N / M)，而通常我们都能够保证N是M的常数倍，所以散列表的查找操作的时间复杂度为O(1)，同理我们也可以得到插入操作的复杂度也为O(1)。<br>

在上面的实现中，我们固定了散列表的桶数，当我们明确知道我们要插入的键值对数目最多只能到达桶数的常数倍时，固定桶数是完全可行的。但是若键值对数目会增长到远远大于桶数，我们就需要动态调整桶数的能力。实际上，散列表中的键值对数与桶数的比值叫做负载因子（load factor）。通常负载因子越小，我们进行查找所需时间就越短，而空间的使用就越大；若负载因子较大，则查找时间会变长，但是空间使用会减小。比如，Java标准库中的HashMap就是基于拉链法实现的散列表，它的默认负载因子为0.75。HashMap实现动态调整桶数的方式是基于公式loadFactor = maxSize / capacity，其中maxSize为支持存储的最大键值对数，而loadFactor和capacity（桶数）都会在初始化时由用户指定或是由系统赋予默认值。当HashMap中的键值对的数目达到了maxSize时，就会增大散列表中的桶数。<br>

- 线性探测法

线性探测法是另一种散列表的实现策略的具体方法，这种策略叫做开放定址法。开放定址法的主要思想是：用大小为M的数组保存N个键值对，其中M > N，数组中的空位用于解决碰撞问题。线性探测法的主要思想是：当发生碰撞时（一个键被散列到一个已经有键值对的数组位置），我们会检查数组的下一个位置，这个过程被称作线性探测。<br>

线性探测可能会产生三种结果:<br>
命中：该位置的键与要查找的键相同；<br>
未命中：该位置为空；<br>
该位置的键和被查找的键不同。<br>

当我们查找某个键时，首先通过散列函数得到一个数组索引后，之后我们就开始检查相应位置的键是否与给定键相同，若不同则继续查找（若到数组末尾也没找到就折回数组开头），直到找到该键或遇到一个空位置。由线性探测的过程我们可以知道，若数组已满的时候我们再向其中插入新键，会陷入无限循环之中。<br>

有必要实现动态增长数组来保持查找操作的常数时间复杂度。当键值对总数很小时，若空间比较紧张，可以动态缩小数组，这取决于实际情况。<br>

##### 1.2 红黑树

###### 1.2.1 二叉查找树

二叉查找树（Binary Search Tree），也称有序二叉树（ordered binary tree）,排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：<br>

- 若任意结点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若任意结点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 任意结点的左、右子树也分别为二叉查找树。
- 没有键值相等的结点（no duplicate nodes）。

二叉查找树一般查找时间为O(lgn),当极端情况出现时，二叉树退化为一颗具有n个节点的线性链后，复杂度退化为线性O(n).

###### 1.2.2 红黑树

红黑树本质上是一颗二叉查找树,红黑树增加了一些性质，保证在最坏的情况下，复杂的也是O(logn).<br>

- 每个结点要么是红的，要么是黑的。  
- 根结点是黑的。  
- 每个叶结点（叶结点即指树尾端NIL指针或NULL结点）是黑的。  
- 如果一个结点是红的，那么它的俩个儿子都是黑的。  
- 对于任一结点而言，其到叶结点树尾端NIL指针的每一条路径都包含相同数目的黑结点。  

图片来源(https://github.com/julycoding/The-Art-Of-Programming-By-July/blob/master/ebook/zh/03.01.md)<br>
![shu](http://67.216.218.49:8000/file/blogs/java/collection/hongheishu.png)

###### 1.2.3 红黑树的操作

当我们在对红黑树进行插入和删除等操作时，对树做了修改，那么可能会违背红黑树的性质。为了继续保持红黑树的性质，我们可以通过对结点进行重新着色，以及对树进行相关的旋转操作，即修改树中某些结点的颜色及指针结构，来达到对红黑树进行插入或删除结点等操作后，继续保持它的性质或平衡。<br>
不在这里详述，单独写一篇文章<br>

##### 1.3 HashMap

###### 1.3.1 属性

```
//初始容量  10000(2进制)
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//当节点中的元素个数大于该值时，桶中链表转换为树
static final int TREEIFY_THRESHOLD = 8;

//同上，不过是将树转化为链表表
static final int UNTREEIFY_THRESHOLD = 6;

//桶可能被转为树形结构时的最小容量
static final int MIN_TREEIFY_CAPACITY = 64;

```

###### 1.3.2 构造方法

```
public HashMap(int initialCapacity, float loadFactor) {
   if (initialCapacity < 0)
       throw new IllegalArgumentException("Illegal initial capacity: " +
                                          initialCapacity);
   if (initialCapacity > MAXIMUM_CAPACITY)
       initialCapacity = MAXIMUM_CAPACITY;
   if (loadFactor <= 0 || Float.isNaN(loadFactor))
       throw new IllegalArgumentException("Illegal load factor: " +
                                          loadFactor);
   this.loadFactor = loadFactor;
   this.threshold = tableSizeFor(initialCapacity);
}


public HashMap(int initialCapacity) {
   this(initialCapacity, DEFAULT_LOAD_FACTOR);
}


public HashMap() {
   this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(Map<? extends K, ? extends V> m) {
   this.loadFactor = DEFAULT_LOAD_FACTOR;
   putMapEntries(m, false);
}

```

其中，初始化threshold时:

```
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

此算法原理为: <br>
原文:https://www.cnblogs.com/loading4/p/6239441.html<br>
通过位运算，找到大于入参的最近一个2的整数次幂<br>

- 先来假设n的二进制为01xxx...xxx。接着

- 对n右移1位：001xx...xxx，再位或：011xx...xxx

- 对n右移2为：00011...xxx，再位或：01111...xxx

- 此时前面已经有四个1了，再右移4位且位或可得8个1

- 同理，有8个1，右移8位肯定会让后八位也为1。
- 综上可得，该算法让最高位的1后面的位全变为1。
- 最后再让结果n+1，即得到了2的整数次幂的值了

<strong>threshold这个成员变量是阈值，决定了是否要将散列表再散列。它的值应该是：capacity * load_factor才对的。里仅仅是一个初始化，当创建哈希表的时候，它会重新赋值<br></strong>

###### 1.3.3 增 put()

HashMap的内部链表结构:<br>
```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
  .....
}
```
添加元素的put()方法:<br>
```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
  //链表结构存储
   Node<K,V>[] tab; Node<K,V> p; int n, i;
   //散列表为null时  初始化
   if ((tab = table) == null || (n = tab.length) == 0)
       n = (tab = resize()).length;
  //没有发生碰撞，直接存储
   if ((p = tab[i = (n - 1) & hash]) == null)
       tab[i] = newNode(hash, key, value, null);
  //发生碰撞
   else {
       Node<K,V> e; K k;
      //hash和key都相等，记录下来，下一步处理
       if (p.hash == hash &&
           ((k = p.key) == key || (key != null && key.equals(k))))
           e = p;
      //树形结构，调用树的方法
       else if (p instanceof TreeNode)
           e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
      //链表结构
       else {
           for (int binCount = 0; ; ++binCount) {
               //没有找到映射的节点，在链表尾部插入
               if ((e = p.next) == null) {
                   p.next = newNode(hash, key, value, null);
                   //节点数大于等于TREEIFY_THRESHOLD 转为树形存储
                   if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                       treeifyBin(tab, hash);
                   break;
               }
               //找到hash值映射的节点，插入
               if (e.hash == hash &&
                   ((k = e.key) == key || (key != null && key.equals(k))))
                   break;
               p = e;
           }
       }
       //上一步的e,因为hash和key都存在，用新值覆盖旧值，返回旧值
       if (e != null) { // existing mapping for key
           V oldValue = e.value;
           if (!onlyIfAbsent || oldValue == null)
               e.value = value;
           afterNodeAccess(e);
           return oldValue;
       }
   }
   ++modCount;
   //检查阈值，是否再次散列
   if (++size > threshold)
       resize();
   afterNodeInsertion(evict);
   return null;
}

```

###### 1.3.4 删 remove()

```
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //hash值指向的桶是否存在
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        //在首位进行查找，记录下找到的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        //没找到，继续找
        else if ((e = p.next) != null) {
           //树里面找
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
              //链表里面找
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //将找到的node删除
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

###### 1.3.5 查 get()

```
public V get(Object key) {
   Node<K,V> e;
   return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

```

###### 1.3.6 resize()

resize()对散列表进行扩容。<br>
```
final Node<K,V>[] resize() {
   Node<K,V>[] oldTab = table;
   //容量
   int oldCap = (oldTab == null) ? 0 : oldTab.length;
   //负载
   int oldThr = threshold;
   int newCap, newThr = 0;

   if (oldCap > 0) {
       //大于等于最大容量，不能继续散列扩大
       if (oldCap >= MAXIMUM_CAPACITY) {
           threshold = Integer.MAX_VALUE;
           return oldTab;
       }
       else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
          //扩大两倍
           newThr = oldThr << 1; // double threshold
   }
   else if (oldThr > 0) // initial capacity was placed in threshold
       newCap = oldThr;
   //第一次初始化散列表
   else {               // zero initial threshold signifies using defaults
       newCap = DEFAULT_INITIAL_CAPACITY;
       newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
   }
   if (newThr == 0) {
       float ft = (float)newCap * loadFactor;
       newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                 (int)ft : Integer.MAX_VALUE);
   }
   threshold = newThr;

   //将旧的散列表复制到新的散列表
   @SuppressWarnings({"rawtypes","unchecked"})
       Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
   table = newTab;
   if (oldTab != null) {
       for (int j = 0; j < oldCap; ++j) {
           Node<K,V> e;
           if ((e = oldTab[j]) != null) {
               oldTab[j] = null;
               if (e.next == null)
                   newTab[e.hash & (newCap - 1)] = e;
              //树     
               else if (e instanceof TreeNode)
                   ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
              //链表
               else { // preserve order
                   Node<K,V> loHead = null, loTail = null;
                   Node<K,V> hiHead = null, hiTail = null;
                   Node<K,V> next;
                   do {
                       next = e.next;
                       if ((e.hash & oldCap) == 0) {
                           if (loTail == null)
                               loHead = e;
                           else
                               loTail.next = e;
                           loTail = e;
                       }
                       else {
                           if (hiTail == null)
                               hiHead = e;
                           else
                               hiTail.next = e;
                           hiTail = e;
                       }
                   } while ((e = next) != null);
                   if (loTail != null) {
                       loTail.next = null;
                       newTab[j] = loHead;
                   }
                   if (hiTail != null) {
                       hiTail.next = null;
                       newTab[j + oldCap] = hiHead;
                   }
               }
           }
       }
   }
   return newTab;
}

```

###### 1.3.7 HashMap总结注意

在散列表中有装载因子这么一个属性，当装载因子*初始容量小于散列表元素时，该散列表会再散列，扩容2倍！<br>

装载因子的默认值是0.75，无论是初始大了还是初始小了对我们HashMap的性能都不好<br>

- 装载因子初始值大了，可以减少散列表再散列(扩容的次数)，但同时会导致散列冲突的可能性变大(散列冲突也是耗性能的一个操作，要得操作链表(红黑树)！
- 装载因子初始值小了，可以减小散列冲突的可能性，但同时扩容的次数可能就会变多！


初始容量的默认值是16，它也一样，无论初始大了还是小了，对我们的HashMap都是有影响的：<br>

- 初始容量过大，那么遍历时我们的速度就会受影响~
- 初始容量过小，散列表再散列(扩容的次数)可能就变得多，扩容也是一件非常耗费性能的一件事~

从源码上我们可以发现：HashMap并不是直接拿key的哈希值来用的，它会将key的哈希值的高16位进行异或操作，使得我们将元素放入哈希表的时候增加了一定的随机性。<br>

还要值得注意的是：并不是桶子上有8位元素的时候它就能变成红黑树，它得同时满足我们的散列表容量大于64才行的<br>

###### 1.3.8 HashTable

从存储结构和实现来讲基本上都是相同的。它和HashMap的最大的不同是它是线程安全的，另外它不允许key和value为null。Hashtable是个过时的集合类，不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换<br>


#### 2.LinkendHashMap

LinkendHashMap 底层为 HashMap 和一个双向链表，保证了存储数据的有序.<br>


##### 2.1 内部节点类

```
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```
可以看出，是在HashMap的节点上，添加了before/after的一个双向链表.<br>


#### 2.2 accessOrder

```
/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
final boolean accessOrder;

//带accessOrder的构造方法， 不带该参数时，默认为false
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

LinkedHashMap 是有序的， accessOrder 可以控制其排序方式.<br>
- true

基于访问排序（使用 LRU 最近最少使用算法)

- false

基于插入排序

#### 2.3 重写HashMap方法

因为在HashMap的基础上，添加了双向链表，所以得重写HashMap 的部分方法<br>

```
//1.初始化散列表时，也初始化双向链表
void reinitialize() {
    super.reinitialize();
    head = tail = null;
}

//2.创建entry时，将entry加入双向链表的末尾
//可以看出，新节点不再是 Node 而是 Entry
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
   LinkedHashMap.Entry<K,V> last = tail;
   tail = p;
   if (last == null)
       head = p;
   else {
       p.before = last;
       last.after = p;
   }
}
```

这里只是举了两个例子，其它还有很多方式也类似的重写，添上对双向链表的操作。<br>

#### 2.4 构造方法

```
public LinkedHashMap(int initialCapacity, float loadFactor) {
   super(initialCapacity, loadFactor);
   accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
   super(initialCapacity);
   accessOrder = false;
}

public LinkedHashMap() {
   super();
   accessOrder = false;
}

public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

public LinkedHashMap(int initialCapacity, float loadFactor,   boolean accessOrder) {
   super(initialCapacity, loadFactor);
   this.accessOrder = accessOrder;
}

```

#### 2.5  get()

```
public V get(Object key) {
   Node<K,V> e;
   if ((e = getNode(hash(key), key)) == null)
       return null;
   if (accessOrder)
       afterNodeAccess(e);
   return e.value;
}

void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```
在 accessOder的情况下， 每次 get 后  会将该元素放到双向链表最后
