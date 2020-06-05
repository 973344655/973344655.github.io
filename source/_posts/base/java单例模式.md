---
title: 单例模式
date: 2018-10-31 16:43:55
tags: [java]
---

#### 1.用途
用来维护一个全局的类，保存某些常用数据，减少资源消耗。

简单说就是，只创建一次，到处都可以使用。

#### 2.原理
维护一个类，确保只有单一的对象被创建（创建一次）。

通常有两种加载方式:

- 饿汉

程序加载时，就直接加载

- 懒汉

第一次使用时，再初次加载

显而易见，大多数情况下懒汉总是更能符合需求。

#### 3.常用实现

##### 1.饿汉式
线程安全，非懒加载

因为static 所以初始化时，就创建了实例

每次getInstance() 都获取同一实例
```
public class Singleton {
    //这句饿汉式的关键，类加载时初始化
    private static  final Singleton instance = new Singleton();
    //构造函数,初始化一次
    private Singleton(){
    };
    //获取实例
    public static final Singleton getInstance(){
        return instance;
    }
}
```
##### 2.懒汉式

懒加载，线程不安全

缺点：当有多个线程并发(同时)调用getInstance()方法时，可能会创建多个实例

```
public class Singleton{
    private static Singleton instance = null;
    private Singleton(){}
    public static Singleton getInstance(){
        if(null == instance){
            instance = new Singleton();
        }
        return instance;
    }
}
```

- 改进1 -- 加锁

线程安全

缺点: 因为synchronized比较消耗资源，所以当调用频繁时，会影响效率
```
public class Singleton{
    private static Singleton instance = null;
    private Singleton(){}
    public static synchronized Singleton getInstance(){
        if(null == instance){
            instance = new Singleton();
        }
        return instance;
    }
}
```

- 改进2 -- 双重校验锁

线程安全，懒加载

由于单例对象只需要创建一次，如果后面再次调用getInstance()只需要直接返回单例对象

因此，大部分情况下，调用getInstance()都不会执行到同步代码块，从而提高了程序性能

```
public class Singleton{
    private static Singleton instance = null;
    private Singleton(){}
    public static  Singleton getInstance(){
        if(null == instance){ //第一次校验
            synchronized(Singleton.class){
                //锁打开后，再次判断，是否再加锁期间其它地方已经创建instance了
                if(null == instance){ //第二次校验
                  instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

##### 3.静态内部类

线程安全，懒加载

利用类加载机制，当第一次调用getInstance()时， 才触发静态内部类去加载自身内部的instance属性，因为instance也是static,所以只加载一次。

即保证了线程安全，又可用懒加载。
```
public class Singleton {
    //静态内部类
    private static class SingletonHolder{
        private static  final Singleton instance = new Singleton();
    }

    //构造函数,初始化一次
    private Singleton(){

    };
    //获取实例
    public static final Singleton getInstance(){
        return SingletonHolder.instance;
    }
}

```
