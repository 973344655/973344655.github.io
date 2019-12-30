---
title: spring中事务
date: 2019-6-10 15:53:43
tags: [spring]
---

-------------------修改于2019/12/17--------------

# 一.事务

## 1.概念

事务（Transaction）是并发控制的单位，是用户定义的一个操作序列。这些操作要么都做且操作成功，要么都不做。

## 2.四个特点

- 原子性 (Atomicity)

事务的原子性是指事务中包含的所有操作要么全做，要么全不做

- 一致性 (Consistency)

在事务开始以前，数据库处于一致性的状态，事务结束后，数据库也必须处于一致性状态。

- 隔离性 (Isolation)

事务不受其他并发执行的事务的影响

- 持续性 (Durability)

一个事务一旦成功完成，它对数据库的改变必须是永久的



# 二.Spring中事务


## 1.几个属性

- Propagation 传播属性

https://blog.csdn.net/yanyan19880509/article/details/53041564

- Isolation 隔离属性

https://www.jianshu.com/p/6ffbd0a8b0b7

- Timeout

一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务

- Read-only

只读事务，事务中只读取，不能修改数据。


## 2.例子

### 1.pom.xml
```
<!-- mybatis配置 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<!-- end mybatis -->
<!-- mysql驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- end mysql -->
```
### 2.@EnableTransactionManagement
启动类打开@EnableTransactionManagement注解

### 3.使用事务
```
@Transactional(readOnly = false, rollbackFor = { Exception.class })
public void function1(){
   sql1;
   sql2;
}

@Transactional(readOnly = false, rollbackFor = { Exception.class })
 public void function2(){
     try{
        sql1;
        sql2;  
     }catch (Exception e){
       //因为已经catch了 异常，所以事务捕捉不到
       //手动调用回滚
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
     }
 }
```



mysql -uroot -pboncQ!W@E#admin





CREATE TABLE `rep_source_data_industry_region`  (
  `id` bigint(20) NULL DEFAULT NULL,
  `rep_project_name` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
  `hangye` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
  `city` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
  `district` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;


alter table rep_source_data_industry_region change district district text character set utf8 COLLATE utf8_general_ci NULL;
