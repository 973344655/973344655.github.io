---
title: 面试--java基础一
date: 2019-06-17 11:26:34
tags: [interview]
---

### 1.java的8种基本数据类型
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

### 2.java面向对象
<strong>java面向对象的三大特征：封装， 继承， 多态 <br></strong>
##### (1)封装
合并特征和行为，进行抽象。将数据和具体细节等隐藏，提供一个统一的外部访问入口。
##### (2)继承
在已有类的基础上，创建拥有父类属性和方法的子类。子类能进行重写和拓展。允许将对象视为自己本身或基类进行处理。
##### (3)多态
同一个接口，使用不同的实例会进行不同的操作。父类引用子类对象，调用方法时，会调用子类的实现，而不是父类的实现。

### 3.String不可变
###### 为什么String不可变
String 类为 public <front color=#CB4335 >final</front> class String{}<br>
可见，String类被定义成了final，所以String不可变.<br>
###### 为什么要定义String类不可变
- 1.可以缓存hash值<br>

因为String的hash值(用来标明不同对象)经常被用到，不可变的特性使hash值也可保持不变，不用每次都计算hash值.<br>

- 2.String pool需要<br>

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。<br>
JVM （heap中）维护着一个String pool.<br>
字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。<br>

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

- 3.方便其它Object使用<br>

如一个Set中存着String类型的值，要保证Set中的值不可变。如下，按照原则，Set中的值应该不可变，如果a的值改变了，明显违反了规则。
```
HashSet<String> set = new HashSet<String>();
set.add(new String("a"));
set.add(new String("b"));
set.add(new String("c"));

for(String a: set)
	a.value = "a";
```

- 4.线程安全<br>

因为String不可变，天生具备线程安全，所以可以在多个线程中安全的使用。<br>
StringBuffer的线程安全 : public synchronized StringBuffer append(String str) {},通过synchronized实现

### 4.参数传递
java中参数传递，是以值的方式传递到方法中，而不是引用传递.<br>
以下代码中 Dog dog 的 dog 是一个指针，存储的是对象的地址。在将一个参数传入一个方法时，<Strong>本质上是将对象的地址以值的方式传递到形参中</Strong>。因此在方法中使指针引用其它对象，那么这两个指针此时指向的是完全不同的对象，在一方改变其所指向对象的内容时对另一方没有影响.
```
public class Dog {
    String name;
    Dog(String name) {
        this.name = name;
    }
    String getName() {
        return this.name;
    }
    void setName(String name) {
        this.name = name;
    }
    String getObjectAddress() {
        return super.toString();
    }
}
```
```
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);//传递的引用的拷贝的值，即地址的值
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());          // A
    }

    private static void func(Dog dog) {
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());          // B
    }
}
```
如果在方法中改变对象的字段值会改变原对象该字段值，因为改变的是同一个地址指向的内容。<br>
```
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());          // B
    }

    private static void func(Dog dog) {
        dog.setName("B");
    }
}
```

### 5.向下转型
java不能 <Strong>隐式的</Strong> 向下转型，会使精度降低。<br>

1.1 字面量属于 double 类型，不能直接将 1.1 直接赋值给 float 变量，因为这是向下转型。<br>
```
// float f = 1.1; false
float f = 1.1f; true ,将1.1转为float 1.1f
```
字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。<br>
```
short s1 = 1;
// s1 = s1 + 1;
```
但是使用 += 或者 ++ 运算符可以执行隐式类型转换<br>
```
s1 += 1;
// s1++;
```
上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：
```
s1 = (short) (s1 + 1);
```

### 6.访问权限
private,default,protected,public<br>
protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。<br>

|    |    同一个类    |    同一个包  | 不同包的子类 | 不同包的非子类 |
| :--:   |   :--:    | :--:     |   :--:     | :--:     |
|private | √       |  |   |   |
|default  | √  | √ |     |   |
|protected | √  | √ |  √   |   |
|public  | √  | √ |  √   | √  |

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。

字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。例如下面的例子中，AccessExample 拥有 id 公有字段，如果在某个时刻，我们想要使用 int 存储 id 字段，那么就需要修改所有的客户端代码。
```
public class Example {
    //public String id;  false
    private String id;   true
}
```
