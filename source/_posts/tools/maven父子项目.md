---
title: 创建一个maven聚合项目
date: 2020-01-06 16:00:54
tags: [tools]
---

目的: 一个系列的项目，创建多个项目，管理起来不方便。同时也是消除重复配置



# 一. pom.xml

## 1.一个pom.xml 大致结构
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <packaging>pom</packaging>

    <groupId>com.demo.example</groupId>
    <artifactId>mutiplemodulesdemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent></parent>

    <properties></properties>

    <modules></modules>

    <dependencies></dependencies>

    <dependencyManagement></dependencyManagement>

</project>
```

## 2.大概介绍

### 2.1 packaging
```
<packaging>pom</packaging>
```
表示这只是一个引用其它项目的pom, 而不是常用的jar或者war

### 2.2 parent
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.17.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```
parent 表示继承，作用就是复用 里面的内容

### 2.3modules

modules 用来管理各个子模块
```
<modules>
    <module>springcloud-demo</module>
    <module>springboot-demo</module>
</modules>
```
### 2.4 dependencies

dependencies 用来放一些公有的依赖，在这里引入后，子模块就不需要再引入了
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
### 2.5 properties

通过<properties>元素用户可以自定义一个或多个Maven属性，然后在POM的其他地方使用${属性名}的方式引用该属性，这种做法的最大意义在于消除重复和统一管理。

Maven总共有6类属性，内置属性、POM属性、自定义属性、Settings属性、java系统属性和环境变量属性.

- 内置属性

两个常用内置属性 ${basedir} 表示项目跟目录，即包含pom.xml文件的目录；${version} 表示项目版本

- pom属性

用户可以使用该类属性引用POM文件中对应元素的值。如${project.artifactId}就对应了<project> <artifactId>元素的值。

常用的POM属性包括：
>${project.build.sourceDirectory}:项目的主源码目录，默认为src/main/java/
>${project.build.testSourceDirectory}:项目的测试源码目录，默认为src/test/java/
>${project.build.directory} ： 项目构建输出目录，默认为target/
>${project.outputDirectory} : 项目主代码编译输出目录，默认为target/classes/
>${project.testOutputDirectory}：项目测试主代码输出目录，默认为target/testclasses/
>${project.groupId}：项目的groupId
>${project.artifactId}：项目的artifactId
>${project.version}：项目的version,与${version} 等价
>${project.build.finalName}：项目打包输出文件的名称，默认为${project.artifactId}->${project.version}

- 自定义属性

```
<properties>
   <custom.property>true</custom.property>
</properties>
```

这样一个自定义属性，可以在其它部分，或者子模块直接引用.引用方式为 : ${custom.property}


- ettings属性

与POM属性同理，用户使用以settings. 开头的属性引用settings.xml文件中的XML元素的值

- ava系统属性
所有java系统属性都可以用Maven属性引用，如${user.home}指向了用户目录。

- 环境变量属性
所有环境变量属性都可以使用以env. 开头的Maven属性引用，如${env.JAVA_HOME}指代了JAVA_HOME环境变量的的值

### 2.6 dependencyManagement


所有在dependencies里的依赖都会自动引入，并默认被所有的子项目继承。

dependencyManagement里只是声明依赖，并不自动实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

所以dependecyManagement用来灵活配置。 比如某个模块，在多数子模块中存在，少数子模块中不存在，则可以使用它。


```
<dependencyManagement>
   <dependencies> <!-- 配置共有依赖 -->
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-core</artifactId>
       <version>4.0.2.RELEASE</version>
   </dependency>
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-beans</artifactId>
       <version>4.0.2.RELEASE</version>
   </dependency>
   </dependencies>
</dependencyManagement>
```

# 二.一个具体例子


## 1.父级

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <packaging>pom</packaging>

    <groupId>com.demo.example</groupId>
    <artifactId>mutiplemodulesdemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.17.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>
    <modules>
        <module>springcloud-demo</module>
        <module>springcloud-service-demo</module>
        <module>springcloud-service-demo2</module>
        <module>springboot-demo</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>


    </dependencies>

</project>
```

## 2.子模块

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.demo.example</groupId>
        <artifactId>mutiplemodulesdemo</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>springboot-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>springboot-demo</name>
    <description>Demo project for Spring Boot</description>


    <dependencies>

        <!-- mybaties -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
        </dependency>

        <!-- mybaties end -->

        <!-- 分页集成 -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.8</version>
        </dependency>
        <!-- 分页 end -->

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
