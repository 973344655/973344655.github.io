---
title: java数据类型
date: 2019-06-17 11:26:34
tags: [java]
---

## 一.8种基本数据类型

### 1.大小

- 位 bit(b)  
- 字节 Byte(B) = 8b
- 字  = 18b

|  类型  |    大小    |    范围  |
| :--:   |   :--:    | :--:     |
|boolean |1位        | ture/false(0/1)|
|byte    | 8位/1字节  | -128--127 |
|short   |16位/2字节  |-32768--132767|
|char    |16位/2字节  |\u0000--\uffff(一个16unicode字符)|
|int     |32位/4字节  |-2^31--2^31-1|
|float   |32位/4字节  | |
|long    |64位/8字节  |-2^63--2^63|
|double  |64位/8字节  | |

### 2.转换

- 转换顺序从低级到高级

byte,short,char—> int —> long—> float —> double

- 将大容量强转为小容量，会损失精度

```
int i = 128;
(byte) i; // -128
```
- 浮点数转为整数，舍弃小数部分

```
float f = 12.55f;
(int)f; // 12
```

### 3.包装类型

基本数据类型：byte，int， short， long, boolean，char, float，double等<br>

包装类型： Byte，Integer，Short，Long，Boolean，Character，Float,Double等<br>

- 为什么要有包装类 ?

Java语言是一个面向对象的语言,但Java中的基本数据类型却是不面向对象的。例如:集合类中只能存放对象List\<Integer\>, 不能存放基本类型数据List\<int\>。

- 怎么用 ?


基本类型与其包装类之间转换:
```
/** 将int类型转换为Integer类型*/
int intNum = 10;
Integer integer = new Integer(intNum);
/** 将Integer类型转换为int类型*/
int intValue = integer.intValue();
```
实际上Jdk有自动装包/拆包机制，不需要我们手动进行转换.
```
/** int类型会自动转换为Integer类型*/
int n = 12;
Integer m = n;
```

## 二.引用数据类型

- 引用数据类型包括: 类/接口和数组.<br>

如 String s = "123"; 类型为 java.lang.String 类<br>

- 引用类型的默认值为null

## 三.其它

### 1.关于String不可变

- 理解String中的变与不变

```
String s = "123";
System.out.println(System.identityHashCode(s)); //1650967483
s += "456";
System.out.println(System.identityHashCode(s)); //87285178
```
String为不可变，所以创建后 s = "123",就不再变化。<br>
那我们为什么能重新给它赋值呢? 实际上是，新建了一个String，s指向了新的地址引用<br>.

- 为什么不可变 ?

String 类为 public <front color=#CB4335 >final</front> class String{}<br>
可见，String类被定义成了final，所以String不可变.<br>

- 为什么要不可变 ?

1.用于Hash<br>

因为String的hash值(用来标明不同对象)经常被用到,例如Map的key，不可变的特性使hash值也可保持不变，不用每次都计算hash值.<br>

2.用于线程安全<br>

因为String不可变，天生具备线程安全,无论多少个线程操作，都不会改变其值。<br>

3.用于String pool<br>

JVM （heap中）维护着一个String pool(字符串常量池),保存着所有字符串字面量.<br>
如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。<br>

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。<br>

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。<br>
```
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true;  
```
如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。<br>
```
String s5 = "bbb";//不存在，创建
String s6 = "bbb";//已存在，获得引用
System.out.println(s5 == s6);  // true
```

4.方便其它对象使用<br>

如一个Set中存着String类型的值，要保证Set中的值不可变。如下，按照原则，Set中的值应该不可变，如果a的值改变了，明显违反了规则。
```
HashSet<String> set = new HashSet<String>();
set.add(new String("a"));
set.add(new String("b"));
set.add(new String("c"));

for(String a: set)
	a.value = "a";
```
