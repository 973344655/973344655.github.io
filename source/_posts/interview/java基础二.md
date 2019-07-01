---
title: 面试--java基础二
date: 2019-06-21 15:59:21
tags: [interview]
---

### 1.抽象类和接口
###### (1).为什么要有抽象类
引入抽象方法和抽象类，是Java提供的一种语法工具，对于一些类和方法，引导使用者正确使用它们，减少被误用。<br>

使用抽象方法，而非空方法体，子类就知道他必须要实现该方法，而不可能忽略。<br>

使用抽象类，类的使用者创建对象的时候，就知道他必须要使用某个具体子类，而不可能误用不完整的父类。<br>

###### (2).有了抽象类为什么还要接口
抽象类是一个类，Java一个类可以实现多个接口，但只能继承一个类(继承的好处是复用代码)。所以需要接口。<br>
抽象类和接口是配合而非替代关系，它们经常一起使用，接口声明能力，抽象类提供默认实现，实现全部或部分方法，一个接口经常有一个对应的抽象类。抽象类实现接口部分方法，然后继承于抽象类的类就可以只实现剩下的方法。<br>

Collection接口和对应的AbstractCollection抽象类<br>
List接口和对应的AbstractList抽象类<br>
Map接口和对应的AbstractMap抽象类<br>

###### (3).接口和抽象类
抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类<br>

接口可以类似的看成一个方法全是抽象方法的抽象类。

接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。<br>
接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。<br>

所以在使用的选择时:

- 使用接口：

需要让不相关的类都实现一个方法<br>
需要使用多重继承。<br>
- 使用抽象类：

需要在几个相关的类中共享代码。(实现部分方法)<br>
需要能控制继承来的成员的访问权限，而不是都为 public。<br>
需要继承非静态和非常量字段。<br>

### 2.重写与重载
override/overload<br>

为了满足里式替换原则，重写有以下三个限制：<br>
>子类方法的访问权限必须大于等于父类方法；<br>
>子类方法的返回类型必须是父类方法返回类型或为其子类型。<br>
>子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。<br>

JVM虚拟机中，重载实际上调用 的封装类型，重写实际上调用的实际类型<br>

### 3.Object
```
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

###### (1).hashCode()
采用操作系统底层实现的哈希算法,返回对象的hash值。 同一个对象的哈希码值是唯一的。<br>
###### (2).equals()
equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。<br>
等价关系：自反性/对称性/传递性/一致性<br>
```
自反：x.equals(x); // true

对称：x.equals(y) == y.equals(x); // true

传递：if (x.equals(y) && y.equals(z))
    x.equals(z); // true;

一致：x.equals(y) == x.equals(y); // true
```
###### (3).toString()
返回 类型(class)@hash的无符号十六进制<br>
例：ToStringExample@4554617c <br>

### 4.浅拷贝和深拷贝
在 Java 中，除了基本数据类型（元类型）之外，还存在 类的实例对象 这个引用数据类型。而一般使用 = 号做赋值操作的时候。对于基本数据类型，实际上是拷贝的它的值，但是对于对象而言，其实赋值的只是这个对象的引用，将原对象的引用传递过去。<br>

而浅拷贝和深拷贝就是在这个基础之上做的区分，如果在拷贝这个对象的时候，对引用数据类型<Strong>只是进行了引用的传递</Strong>，而没有真实的创建一个新的对象，则认为是浅拷贝。反之，在对引用数据类型进行拷贝的时候，<Strong>创建了一个新的对象，并且复制其内的成员变量</Strong>，则认为是深拷贝。<br>

### 5.final与static
final可以修饰：属性，方法，类，局部变量（方法中的变量）<br>
static可以修饰：属性，方法，代码段，内部类（静态内部类或嵌套内部类）<br>

static修饰的属性强调它们只有一个，可以在不new对象的情况下调用，final强调不能修改。static final修饰的属性表示一旦给值，就不可修改，并且可以通过类名访问。<br>

初始化顺序:静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。<br>

### 6.反射

##### 为什么要用反射

反射的核心是可以在运行时才动态加载类的信息。<br>
利用这个特性，可以使代码变得灵活，如一开始不知道某个类的信息，不能new,但是可以通过配置文件或者插件等引入，使用反射在运行时加载。利用配置文件时，可以将类描述(属性信息)写在配置文件中，不用每次修改代码<br>
反射会牺牲一点性能。
