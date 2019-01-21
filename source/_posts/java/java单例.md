---
title: java单例
date: 2018-10-31 16:43:55
tags: [java]
---

#### 1.用途
用来维护一个全局的类，保存某些常用数据，减少资源消耗。
#### 2.原理
维护一个类，确保只有单一的对象被创建（创建一次），提供一个唯一的访问接口。私有的构造函数确保外部不能进行访问。

#### 3.两种常用实现
1.饿汉式
```
public class Singleton {
    //这句饿汉式的关键，类加载时初始化
    private static  final Singleton instance = new Singleton();
    //维护的数据
    private Map<String,Map> data;
    //构造函数,初始化一次
    private Singleton(){
        this.data = new HashMap<String, Map>();
    };
    //获取实例
    public static final Singleton getInstance(){
        return instance;
    }

    //全局获取维护的数据
    public  Map<String, Map> getData(){
        return this.data;
    }
    //修改数据
    public void setData(Map<String, Map> data) {
        this.data = data;
    }
}
```

2.静态内部类
可用调用时再初始化加载(lazyLoad)
```
public class Singleton {
    //静态内部类
    private static class SingletonHolder{
        private static  final Singleton instance = new Singleton();
    }
    //维护的数据
    private Map<String,Map> data;
    //构造函数,初始化一次
    private Singleton(){
        this.data = new HashMap<String, Map>();
    };
    //获取实例
    public static final Singleton getInstance(){
        return SingletonHolder.instance;
    }

    //全局获取维护的数据
    public  Map<String, Map> getData(){
        return this.data;
    }
    //修改数据
    public void setData(Map<String, Map> data) {
        this.data = data;
    }
}

```
