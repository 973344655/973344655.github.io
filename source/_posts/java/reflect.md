---
title: Java反射
date: 2018-11-07 10:30:33
tags: [java]
---

详见:https://www.sczyh30.com/posts/Java/java-reflection-1/<br>
https://blog.csdn.net/ljphhj/article/details/12858767
#### 1.什么是反射
>Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control

反射机制允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。<br>
反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。<br>
Java 反射主要提供以下功能：
- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
- 在运行时调用任意一个对象的方法

<strong>在运行时，而不是编译时.</strong>
