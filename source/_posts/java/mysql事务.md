---
title: spring中mysql事务
date: 2019-6-10 15:53:43
tags: [mysql]
---

### 1.springboot中使用mysql事务
##### (1).pom.xml
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
##### (2)@EnableTransactionManagement
启动类打开@EnableTransactionManagement注解

##### (3)使用事务
```
@Transactional(rollbackFor = Exception.class)
   public void addData(){
       testMapper.insertData1(1,"111");
       testMapper.insertData2(1,"012345678910");
   }
```
