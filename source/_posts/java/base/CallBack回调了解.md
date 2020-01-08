---
title: java CallBack
date: 2019-12-25 15:00:00
tags: [java]
---

https://www.cnblogs.com/xrq730/p/6424471.html

# 一.调用

## 1.同步调用

同步调用是，类A的方法a()调用类B的方法b()，一直等待b()方法执行完毕，a()方法继续往下走。

## 2.异步调用

类A的方法方法a()通过新起线程的方式调用类B的方法b()，代码接着直接往下执行，这样无论方法b()执行时间多久，都不会阻塞住方法a()的执行。但是这种方式，由于方法a()不等待方法b()的执行完成，在方法a()需要方法b()执行结果的情况下（视具体业务而定，有些业务比如启异步线程发个微信通知、刷新一个缓存这种就没必要），必须通过一定的方式对方法b()的执行结果进行监听。在Java中，可以使用Future+Callable的方式做到这一点。

## 3.回调

回调也分为同步回调和异步回调(待续。。。。。)

网上理解: 类A的a()方法调用类B的b()方法，类B的b()方法执行完毕主动调用类A的callback()方法

我的理解: 回调就是，A的某个方法中，有一些操作需要在B中完成，B中操作完成后，需要将结果通知给A(不一定是通知，反正就是一个CallBack)，这个通知操作就是回调。

# 二.回调怎么实现

```
/**
 * Java 回调机制 Callback
 */
// 回调接口,泛型参数
interface CallBack<T,E>{
    T doSomeThing1();
    void doSomeThing2(E e);
}


// class A 实现回调接口
class A<T,E> implements CallBack<String,E>{
    @Override
    public String doSomeThing1() {
        return "接收到回调";
    }
    @Override
    public void doSomeThing2(E e) {
        System.out.println("接收到回调: " + e);
    }

    //调用B中的方法，将自己作为回调告诉B,B完成后应该向我返回结果
    public void myFunc(B b){
        b.excute(this);
    }
}


//class B
class B<E>{
    private E e;
    //接收一个Callback,知道完成后，向谁汇报完成结果
    public void excute(CallBack callBack){
        try {
            //一些其它操作
            Thread.sleep(3000);
            //结果
            e = (E)"result1";
        }catch (Exception e1){
            e1.printStackTrace();
        }

        //进行回调操作,将结果告诉调用方
        callBack.doSomeThing2(e);

        //接下来的操作

    }
}


public class CallBackDemo {

    public static void main(String [] args){
        B b = new B();
        A a = new A();
        // 方式1，回调
        a.myFunc(b);

        //方式2，使用匿名函数
        b.excute(new CallBack() {
            @Override
            public Object doSomeThing1() {
                return null;
            }

            @Override
            public void doSomeThing2(Object o) {
                System.out.println("匿名函数实现回调");
            }
        });
    }
}

```
