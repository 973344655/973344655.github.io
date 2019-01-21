---
title: interface
date: 2019-01-21 14:26:34
tags: [java]
---

[接口与抽象类区别](https://blog.csdn.net/u010466329/article/details/78133282)

### 一.例子
```
package base.interfacedemo;

//定义一个接口
interface TestInterface {

    void test();

    //接口静态内部类实现 默认 public static final
    //方便扩展
    TestInterface Demo1 = new TestInterface() {
        @Override
        public void test() {
            System.out.println("test1");
        }
    };

    //default方法 java8新特性
    default void Demo3(){
       System.out.println("test3");
    }
}

//正常的接口实现
class Demo2 implements TestInterface{
    @Override
    public void test(){
        System.out.println("test2");
    }
}

public class DemoInterface{

    public static void main(String[] args){
        TestInterface.Demo1.test();
        new Demo2().test();
        Demo2 demo3 = new Demo2();
        demo3.Demo3();
    }
}
```
