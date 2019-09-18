---
title: java参数传递
date: 2019-06-17 11:26:34
tags:
---

https://blog.csdn.net/jiangnan2014/article/details/22944075

https://juejin.im/post/5bce68226fb9a05ce46a0476

## 1.参数传递

参数传递分为引用传递和值传递.

- 值传递

方法接收的是调用着提供的值

- 引用传递

方法接收的是调用者提供的变量地址


这里我们需要注意的是一个方法可以修改 引用传递 所对应的变量值，而不能修改 值传递 所对应的变量值，这是按值调用与引用调用的根本区别。

## 2.Java中的参数传递

### 1.例子:
```
public class MyTest {

    class User{
        private String userName;
        private String userId;

        public void setUserName(String userName){
            this.userName = userName;
        }
        public void setUserId(String userId){
            this.userId = userId;
        }
        public String getUser(){
           return "name: " + this.userName + "  id: " + this.userId;
        }
    }

    public static void transTest(User user){
        user.setUserName("传递");
    }

    public static void transTest2(int value){
        value = value * 2;
    }

    public static void transTest3(String value){
        value = value  + "-changed";
    }

    public static void transTest4(StringBuffer value){
        value.append("-changed");
    }

    public static void transTest5(char ch){
        ch = '2';
    }

    public static void main(String[] args){

        //测试1
        User user = new MyTest().new User();
        user.setUserId("01");
        System.out.println("传递前的值: " + user.getUser());
        transTest(user);
        System.out.println("传递后的值: " + user.getUser());

        //测试2
        System.out.println();
        int num = 2;
        System.out.println("传递前的值: " + num);
        transTest2(num);
        System.out.println("传递后的值: " + num);

        //测试3
        System.out.println();
        String s = "123";
        System.out.println("传递前的值: " + s);
        transTest3(s);
        System.out.println("传递后的值: " + s);

        //测试4
        System.out.println();
        StringBuffer sb = new StringBuffer("123");
        System.out.println("传递前的值: " + sb.toString());
        transTest4(sb);
        System.out.println("传递后的值: " + sb);

        //测试5
        System.out.println();
        char ch = '1';
        System.out.println("传递前的值: " + ch);
        transTest5(ch);
        System.out.println("传递后的值: " + ch);
    }
}

```

结果:
```
传递前的值: name: null  id: 01
传递后的值: name: 传递  id: 01

传递前的值: 2
传递后的值: 2

传递前的值: 123
传递后的值: 123

传递前的值: 123
传递后的值: 123-changed

传递前的值: 1
传递后的值: 1

```

### 2.结论

先来结论:<br>

1.java的参数传递都是传值，没有传引用.<br>

2.对引用类型的传参，传的是引用类型的值(地址的值).<br>


### 3.分析

从结果来看:<br>

- 测试2和测试5

参数基本类型参数，传的是实参的值的一个拷贝，所以传递过后，方法内部对拷贝的值进行操作，实参本身值不变。

- 测试1和测试4

参数是引用类型，传的是变量的拷贝(地址)，所以在方法内进行修改，会将对应地址的信息进行修改，在外层打印时，打印的是地址中的信息(已被修改).

- 测试3

参数是引用类型，但是String不可变，对String的修改会新建一个对象.


### 4.疑惑: 传地址为什么是传值，而不是传引用？

https://www.zhihu.com/question/31203609
知乎高赞讲解，一看就懂。<br>

#### (1)类型区别

```
int num = 10; //基本类型
String str = "hello"; //引用类型
```

![transfer1](http://67.216.218.49:8000/file/blogs/java/base/paramsTransfer1.jpg)
如图：num是基本类型，值就直接保存在变量中。而str是引用类型，变量中保存的只是实际对象的地址。一般称这种变量为"引用"，引用指向实际对象，实际对象中保存着内容。

#### (2)= 赋值的作用
```
num = 20;
str = "java";
```
![transfer2](http://67.216.218.49:8000/file/blogs/java/base/paramsTransfer2.jpg)
对于基本类型 num ，赋值运算符会直接改变变量的值，原来的值被覆盖掉。
对于引用类型 str，赋值运算符会改变引用中所保存的地址，原来的地址被覆盖掉。但是原来的对象不会被改变（重要）。

#### (3).调用方法时的参数传递

```
StringBuilder sb = new StringBuilder("iphone");
void foo(StringBuilder builder) {
    builder.append("4");
}
foo(sb); // sb 被改变了，变成了"iphone4"


StringBuilder sb = new StringBuilder("iphone");
void foo(StringBuilder builder) {
    builder = new StringBuilder("ipad");
}
foo(sb); // sb 没有被改变，还是 "iphone"。

```
![transfer3](http://67.216.218.49:8000/file/blogs/java/base/paramsTransfer3.jpg)
引用类型时，传递了变量的一份拷贝，(存储着地址的值)，然后进行修改操作后，修改了地址中的信息.

![transfer4](http://67.216.218.49:8000/file/blogs/java/base/paramsTransfer4.jpg)
new StringBuilder()时，创建了一个新的地址，原地址信息还是没有改变.
