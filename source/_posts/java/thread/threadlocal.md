---
title: ThreadLocal了解
date: 2019-07-27 16:43:55
tags: [java]
---
http://www.jasongj.com/java/threadlocal/

------------------
2019/12/30

一个简单易懂的blog:
https://benjaminwhx.com/2018/04/28/%E3%80%90%E7%BB%86%E8%B0%88Java%E5%B9%B6%E5%8F%91%E3%80%91%E8%B0%88%E8%B0%88ThreadLocal/

# 一.ThreadLocal作用

## 1.作用

一半都说是实现了变量在线程间隔离,在类或者方法间共享。<br>
但是看到这句话一脸迷茫，怎么实现的，为什么要实现??




## 2.为什么要用

通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。而使用ThreadLocal创建的变量只能被当前线程访问，其他线程则无法访问和修改(线程间隔离),同时在改线程的类或方法中，都能获取该变量的同一实例(方法间共享)。

## 3.例子

```
public class ThreadLocalTest {

    public static void main(String[] args) throws InterruptedException {
        InnerClass innerClass = new InnerClass();
        for(int i = 1; i <= 2; i++) {
            new Thread(() -> {
                for(int j = 0; j < 2; j++) {
                    innerClass.add(String.valueOf(j));
                    innerClass.print();
                }
                innerClass.set("hello world");
            }, "thread - " + i).start();
        }

    }

    private static class InnerClass {
        public void add(String newStr) {
            StringBuilder str = Counter.counter.get();
            Counter.counter.set(str.append(newStr));
        }

        public void print() {
            System.out.printf("Thread name:%s , ThreadLocal hashcode:%s, Instance hashcode:%s, Value:%s\n",
                    Thread.currentThread().getName(),
                    Counter.counter.hashCode(),
                    Counter.counter.get().hashCode(),
                    Counter.counter.get().toString());
        }

        public void set(String words) {
            Counter.counter.set(new StringBuilder(words));
            System.out.printf("Set, Thread name:%s , ThreadLocal hashcode:%s,  Instance hashcode:%s, Value:%s\n",
                    Thread.currentThread().getName(),
                    Counter.counter.hashCode(),
                    Counter.counter.get().hashCode(),
                    Counter.counter.get().toString());
        }
    }

    private static class Counter {
        private static ThreadLocal<StringBuilder> counter = new ThreadLocal<StringBuilder>() {
            //重写 initialValue 为ThreadLocal get()提供初始值
            @Override
            protected StringBuilder initialValue() {
                return new StringBuilder();
            }
        };

    }
}

```

结果

```
1 Thread name:thread - 1 , ThreadLocal hashcode:1101022441, Instance hashcode:1332191917, Value:0
2 Thread name:thread - 1 , ThreadLocal hashcode:1101022441, Instance hashcode:1332191917, Value:01
3 Set, Thread name:thread - 1 , ThreadLocal hashcode:1101022441,  Instance hashcode:1821539951, Value:hello world
4 Thread name:thread - 2 , ThreadLocal hashcode:1101022441, Instance hashcode:831382771, Value:0
5 Thread name:thread - 2 , ThreadLocal hashcode:1101022441, Instance hashcode:831382771, Value:01
6 Set, Thread name:thread - 2 , ThreadLocal hashcode:1101022441,  Instance hashcode:1061821618, Value:hello world
```

结果分析：<br>
在这个例子里面 StringBuilder 为 ThreadLocal中的变量.<br>

- 从结果 1-6 ThreadLocal hashcode可以看到，所有threadlocal都是同一个实例

- 从结果 1,2和4,5  Instance hashcode 和 value可以看到, 在同一个线程中,变量是共享的，在不同线程中，是不同的变量

- 从结果 12和3，45和6中的 Instance hashCode 可以看出，ThreadLocal 的set() 方法，可以改变ThreadLocal中的变量实例

- 从代码中可见， 虽然都是 Counter.counter.get() 得到value再拼接，但是，在不同线程中，有自己的副本，不影响其它现在，这里set()没有改变变量实例，是因为 用的就是当前实例，没有new StringBuilder(words)；

# 二.原理

实现

## 1.构造方法

```
public ThreadLocal() {
}

//如果想在初始化时设置get()的初始值，需要重写initialValue方法
private static ThreadLocal<StringBuilder> counter = new ThreadLocal<StringBuilder>() {
    @Override
    protected StringBuilder initialValue() {
        return new StringBuilder();
    }
};
```

## 2.get()

```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

可以看出，在ThreadLocal中，使用了ThreadLocalMap 来维护变量与线程之间的关系，其中 Thread为key.<br>
如果不存在，则调用 setInitialValue()初始化，setInitialValue()又调用了initialValue(), 所以，初始化时，需要重写initialValue方法,可以在get()时得到我们相要的值.

## 3.ThreadLocalMap

ThreadLocalMap 为 ThreadLocal 的静态内部类。<br>

```
static class ThreadLocalMap {

    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    .....
}
```

在ThreadLocalMap中 Entry是弱引用<br>
使用ThreadLocal的弱引用作为key<br>

- 为什么要用弱引用

弱引用实例不会影响到被应用对象的GC回收行为，什么意思呢？<br>
如果一个对象通过一串强引用链接可到达(Strongly reachable)，它是不会被回收的。而弱引用不能阻挡垃圾回收器对其回收。<br>

但是由于每个线程访问某 ThreadLocal 变量后，都会在自己的 Map 内维护该 ThreadLocal 变量与具体实例的映射(ThreadLocalMap)，如果不删除这些引用（映射），则这些 ThreadLocal 不能被回收，可能会造成内存泄漏<br>

所以,使用弱引用的原因在于，当没有强引用指向 ThreadLocal 变量时，它可被回收，从而避免上文所述 ThreadLocal 不能被回收而造成的内存泄漏的问题。<br>

但是，因为key为弱引用，当垃圾回收后，可能会形成key为null，value还存在的内存泄露问题。所以，ThreadLocal 会在 get或set时，擦除这种value.


--------------
2019/12/30

 弱引用只是 key,只有key定义为弱引用，所以GC回收时，回收的是key。

 那问题来了，value怎么办？  剩下一些key为null,value为entry的对象？？

```
private Entry getEntry(ThreadLocal<?> key) {
     int i = key.threadLocalHashCode & (table.length - 1);
     Entry e = table[i];
     if (e != null && e.get() == key)
         return e;
     else
         return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}

//擦除
private int expungeStaleEntry(int staleSlot) {
       Entry[] tab = table;
       int len = tab.length;

       // expunge entry at staleSlot
       tab[staleSlot].value = null;
       tab[staleSlot] = null;
       size--;
 ......
}
```

## 4.set()

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
