---
title: java8新特性
date: 2019-08-012 19:07:33
tags: [java]
---

java8 方便了函数式编程<br>

# 一.lambda表达式

## 1.意义

lambda函数比较轻便，即用即仍，很适合需要完成一项功能，但是此功能只在此一处使用，连名字都很随意的情况下<br>

<strong> lambda表达式可以方便的和以下一些特性结合使用。 </strong><br>

## 2.形式

```
// 1. 不需要参数,返回值为 5  
() -> 5  

// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  

// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  

// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  

// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

# 二. :: 方法引用

## 1.目的

简化代码

## 2.例子

```
List<String> list = Arrays.asList("1","2","3");

//forEach
for(String s:list){
   System.out.println(s);
}

//lambda
list.forEach(e -> System.out.println(e));

//::
list.forEach(System.out::println);
```

# 三.接口默认方法

## 1.意义

接口与其实现类之间的 耦合度 太高了（tightly coupled），当需要为一个接口添加方法时，所有的实现类都必须随之修改。默认方法解决了这个问题，它可以为接口添加新的方法，而不会破坏已有的接口的实现。<br>


## 2.样例

```
public class MyTest {

    public static void main(String[] args){
        new MyTest().new A().testA();
    }

    interface InterfaceA{
        default void test(){
            System.out.println("a");
        }
    }

    class A implements InterfaceA{
        public void testA(){
            new A().test();
        }
    }
}
```

# 四.Stream

https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html

## 1.意义

函数式编程？？

对集合功能的增强，提供便利的聚合操作



## 2.使用

简单说，对 Stream 的使用就是实现一个 filter-map-reduce 过程，产生一个最终结果，或者导致一个副作用（side effect）

### 1.流

```
//串行流
Collection.Stream()

//并行流
Collection.parallelStream()
```
常用构造方式
```
// 1. Individual values
Stream stream = Stream.of("a", "b", "c");
// 2. Arrays
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);
// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();
```

### 2.操作类型


- Intermediate  中间操作

map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered<br>

一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。

- Terminal   终结操作

forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator<br>

一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

- short-circuiting 短路

anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit<br>

当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

### 3.操作例子

- filter

对stream进行过滤, 满足过滤条件的留下

```
List<String> list = Arrays.asList("1","2","2","3");
list.stream().filter(a -> a.equals("2")).forEach(System.out::println);
// result: 2 2
```

- map

map 对元素进行某种转换处理，得到结果，一一映射。<br>

```
List<String> list = Arrays.asList("1","2","2","3");
list.stream().map(String::hashCode).forEach(System.out::println);
// result: 49 50 50 51
```

mapToInt 将元素转成int类型
```
List<String> list = Arrays.asList("1","2","2","3");
list.stream().mapToInt(data -> Integer.parseInt(data)).forEach(System.out::println);
```

flatMap 是将Stream中的每个元素都转换成Stream类型，然后将每个Stream的元素提出取来，聚合成一个最外围的Stream<br>

例子：将两个Stream组合成一个
```
List<String> list = Arrays.asList("1","2","2","3");
List<String> list2 = Arrays.asList("4","5","6");
Stream.of(list, list2).flatMap(List::stream).forEach(System.out::println);
//result: 1 2 2 3 4 5 6
```
两个list去重合并：
```

List<String> list1 = Arrays.asList("1","2","3","4");
List<String> list2 = Arrays.asList("3","4","5","6");

List list = Stream.of(list1, list2).flatMap(List::stream).distinct().collect(Collectors.toList());

list.stream().forEach( System.out::println);
```

- reduce

终结操作，把 Stream 元素组合起来

```
List<String> list = Arrays.asList("1","2","2","3");
List<String> list2 = Arrays.asList("4","5","6");
Optional<String> s = Stream.of(list, list2).flatMap(List::stream).reduce(String::concat);
s.ifPresent(System.out::println);
//result: 1223456
```

- forEach

终结操作，遍历stream中的元素<br>

可用 <strong>peek</strong> 来遍历，中间操作<br>
```
List<String> list = Arrays.asList("1","2","2","3");
list.stream().peek(System.out::println).map(e -> e + "map").peek(System.out::println).count();
//result:
1
1map
2
2map
2
2map
3
3map
```
- limit/skip

limit 得到前n个元素，skip 抛弃前n个元素
```
List<String> list = Arrays.asList("1","2","2","3");
list.stream().limit(3).skip(1).forEach(System.out::println);
//result: 2 2
```

- Match

allMatch：Stream 中全部元素符合传入的 predicate，返回 true<br>
anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true<br>
noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true<br>

```
boolean flag = list.stream().limit(3).anyMatch(e -> e.equals("2"));
```

# 五.optional


https://www.zhihu.com/question/20125256

## 1.意义

Optional<T>配合Lambda可以使Java对于null的处理变的异常优雅<br>

## 2.例子

```
Optional<String> s = Optional.ofNullable(null);

//1.存在则操作
s.ifPresent(System.out::println);

//2.存在则返回，否则返回其它
s.orElse("0");

//3.简化if-else ,多个if-else时效果明显
// if s 存在 则 转大写并+ a  else null
s.map(e -> e.toUpperCase()).map(e2 -> e2.concat("a")).orElse("null");
```
