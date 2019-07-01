---
title: spring基础一 bean
date: 2019-06-23 00:58:44
tags: [java,spring]
---


### 1.Bean的属性
bean的 .xml 文件<br>
```
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">

<!-- 自己的bean -->
<bean></bean>

</beans>
```
将bean交给spring:
```
@ImportResource(locations = {"classpath:xxxx.xml"})
```
###### (1).自身属性


- id

id为bean的唯一标识名，也就是beanName.
```
<bean id="userEntity" class="com.bonc.apimanage.test.UserEntity">
      <!-- 通过setter注入属性值 -->
       <property name="id" value="123"></property>
</bean>

<bean id="userService" class="com.bonc.apimanage.test.UserService">
    <!-- 注入 userEntity 属性对象 -->
    <property name="userService" ref="userEntity"></property>
</bean>
```

- name

bean的别名。

- class

bean的类。子类bean不用定义该属性。

- parent

子类Bean定义它所引用它的父类Bean。这时class属性失效。子类Bean会继承父类Bean的所有属性，子类Bean也可以覆盖父类Bean的属性。注意：子类Bean和父类Bean是同一个Java类

- abstract

默认为”false”，用来定义Bean是否为抽象Bean。它表示这个Bean将不会被实例化，一般用于父类Bean，因为父类Bean主要是供子类Bean继承使用。

- scope

默认为singleton（单例）<br>

prototype（原型）状态，BeanFactory将为每次Bean请求创建一个新的Bean实例。<br>
session：创建对象把创建的对象放到session域里面去<br>
request：创建对象把创建的对象放到request域里面去<br>
globalsession：创建对象把创建的对象放到globalsession域里面去<br>

- lazy-init

用来定义这个Bean是否实现懒初始化<br>
默认为 default  <br>
如果为 true ，它将在BeanFactory启动时初始化所有的SingletonBean<br>
如果为 false ,它只在Bean请求时才开始创建SingletonBean。

- autowire

它定义了Bean的自动装载方式<br>
no :不使用自动装配功能<br>
byName :通过Bean的属性名实现自动装配<br>
byType :通过Bean的类型实现自动装配<br>
constructor :类似于byType，但它是用于构造函数的参数的自动组装<br>
default：默认值，自动装配


- depends-on

Bean在初始化时依赖的对象，这个对象会在这个Bean初始化之前创建<br>
可以用来控制bean初始化顺序

- init-method

用来定义Bean的初始化方法，它会在Bean组装之后调用。它必须是一个无参数的方法

- destory-method

用来定义Bean的销毁方法，它在 BeanFactory 关闭时调用。同样，它也必须是一个无参数的方法。它只能应用于 singletonBean。

- factory-method

定义创建该Bean对象的工厂方法。它用于下面的 factory-bean ，表示这个Bean是通过工厂方法创建。此时， class 属性失效

- factory-bean

定义创建该 Bean 对象的工厂类。如果使用了 factory-bean 则 class 属性失效

###### (2).子属性

- meta

元数据。当需要使用里面的信息时可以通过key获取<br>
meta 所声明的 key 并不会在 Bean 中体现，只是一个额外的声明，当我们需要使用里面的信息时，通过 BeanDefinition 的 getAttribute() 获取<br>

- lookup-method

获取器注入，是把一个方法声明为返回某种类型的 bean 但实际要返回的 bean 是在配置文件里面配置的。该方法可以用于设计一些可插拔的功能上，解除程序依赖

- replaced-method

可以在运行时调用新的方法替换现有的方法，还能动态的更新原有方法的逻辑

- constructor-arg

通过构造函数注入。<br
index：指定注入属性的顺序索引(在构造函数中的顺序)，从0开始<br>
type：指该属性所对应的类型<br>
ref：引用的依赖对象<br>
value：当注入的不是依赖对象，而是基本数据类型时，就用value<br>
```
<bean id="userEntity" class="com.bonc.apimanage.test.UserEntity">
      <!-- 通过构造器注入属性值 有参的需要有参的构造函数 -->
      <constructor-arg index="0" value="1"></constructor-arg>  
      <constructor-arg index="1" value="xxl"></constructor-arg>
      <!-- 或这样  -->
      <constructor-arg name="id" value="1"></constructor-arg>  
      <constructor-arg nam="name" value="xxl"></constructor-arg>   
</bean>

<bean id="userService" class="com.bonc.apimanage.test.UserService">
      <constructor-arg  ref="userEntity"></constructor-arg>
</bean>

```

- property

通过setter对应的方法注入。<br>

- qualifier

明确指定bean的名称进行注入

### 2.Bean的生命周期
Spring 只帮我们管理单例模式 Bean 的完整生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。<br>


![bean生命周期](https://pic3.zhimg.com/80/754a34e03cfaa40008de8e2b9c1b815c_hd.jpg)

- 实例化

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。<br>
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。<br>
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。


- 填充属性

Spring将值和Bean的引用注入进Bean对应的属性中(依赖注入).

- Aware相关



Spring中 特定的Aware接口，提供了对IOC容器的操作.<br>

- BeanPostProcessor前置处理(postProcessorBeforeInitializion)

- InitiallizionBean 中 afterProjectSet方法

- 自定义的init-method

- BeanPostProcessor的后置处理(postProcessorAfterInitializion)

- Bean就绪

- DisposableBean的destory方法

- 自定义的destory-method

### 3.Spring的七个模块

- 1.Spring Core

Core模块是Spring的核心类库，Spring的所有功能都依赖于该类库，Core主要实现IOC功能，Sprign的所有功能都是借助IOC实现的

- 2.Spring Aop

AOP模块是Spring的AOP库，提供了AOP（拦截器）机制，并提供常用的拦截器，供用户自定义和配置

- 3.Spring ORM

- 4.Spring DAO

- 5.Spring Web

- 6.Spring context

Spring上下文是spring的配置文件，向Spring框架提供上下文信息,可以用来管理bean.

- 7.Spring WebMVC

WEB MVC模块为Spring提供了一套轻量级的MVC实现，在Spring的开发中，我们既可以用Struts也可以用Spring自己的MVC框架，相对于Struts，Spring自己的MVC框架更加简洁和方便  
