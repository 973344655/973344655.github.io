---
title: java基本类型
date: 2018-09-21 16:23:45
tags: [java]
---
#### 1.基本类型
- short：16位，最大数据存储量是65536，数据范围是-32768~32767之间。
- int：32位，最大数据存储容量是2的32次方减1，数据范围是负的2的31次方到正的2的31次方减1。
- long：64位，最大数据存储容量是2的64次方减1，数据范围为负的2的63次方到正的2的63次方减1。
- float：32位，数据范围在3.4e-45~1.4e38，直接赋值时必须在数字后加上f或F。
- double：64位，数据范围在4.9e-324~1.8e308，赋值时可以加d或D也可以不加。
- boolean：只有true和false两个取值。
- char：16位，存储Unicode码，用单引号赋值。

基本类型并不是类对象。<br>
为了让这8中基本类型也能面向对象，Java为其提供了包装器类型.
对应上面顺序为：Byte, Short, Integer, Long, Float, Double, Boolean, Character
现在，这些包装器类型都是面向对象的类了，具有对象方法可以进行很多高级的操作，而不是简单的基本类型，都是引用类型了.
java提供了自动拆箱/装箱来实现基本类型数据和包装器类型数据之间相互转换、赋值。

#### 2.引用类型
- 强引用<br>
最常用的引用类型，如Object obj = new Object(); 。只要强引用存在则GC时则必定不被回收。<br>
其它三个暂时不太理解，以后接触再添加
- 软引用
- 弱引用
- 虚引用

#### 3.基本类型与引用类型区别
基本类型就是一个值<br>
引用类型：它的值是指向内存空间的引用，就是地址，所指向的内存中保存着变量<br>
<!-- more -->
#### 4.引用与实例，内部类，枚举类，访问控制符的基本概念及常见用法
- 引用(reference) :
指向一个对象<br>
- 实例(instance):
按照通俗的说法，每个对象都是某个类（class）的一个实例（instance），这里，‘类’就是‘类型’的同义词

- 内部类<br>
将一个类定义在另一个类的内部。
内部类可以很好的解决多重继承问题。（继承接口或类，接口本来可以多重继承，类不能多重继承，内部类可以解决类不能多重继承问题）。<br>
使用内部类最吸引人的原因是：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。<br>
原文连接：https://blog.csdn.net/qq_38242407/article/details/78159494

父类：
```
public class Father {
	public int strong(){
		return 9;
	}
}
public class Mother {
	public int kind(){
		return 8;
	}
}
```

子类，内部类多重继承
```
public class Son {
  //内部类继承Father类
	class Father_1 extends Father{  //继承一
		public int strong(){
			return super.strong() + 1;
		}
	}
	class Mother_1 extends  Mother{  //继承二
		public int kind(){
			return super.kind() - 2;
		}
	}
   //获取父类1方法
	public int getStrong(){
		return new Father_1().strong();
	}
   //获取父类2方法
	public int getKind(){
		return new Mother_1().kind();
	}
}
```

- 枚举类<br>
在某些情况下，一个类的对象时有限且固定的，如季节类，它只有春夏秋冬4个对象这种实例有限且固定的类，在 Java 中被称为枚举类。<br>
枚举类最基本的用法是实现一个类型安全的枚举（final)。
枚举常量用逗号分隔,每个枚举常量都是一个对象。
每一个枚举都是枚举类的实例，支持初始化/构造函数/匿名类/覆盖基类等。

- 访问控制符<br>
public	共有的，对所有类可见。<br>
protected	受保护的，对同一包内的类和所有子类可见。<br>
private	私有的，在同一类内可见。<br>
default (即不加修饰控制符情况下)默认的	在同一包内可见。默认不使用任何修饰符。<br>
