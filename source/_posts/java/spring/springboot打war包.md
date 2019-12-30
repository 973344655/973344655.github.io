---
title: springboot打war包
date: 2018-10-17 15:14:40
tags: [springboot, tomcat,java]
---
#### Springboot部署war包到服务器上

1.修改打包方式为war
```
<groupId>com.xcloud</groupId>
   <artifactId>import-xcloud</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>war</packaging>
```

2.移除内置tomcat，引入外部的
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--使用外部tomcat时放开-->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--下面的两个dependency都是用来部署到外置tomcat时使用，就是去掉内置的tomcat-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>3.0-alpha-1</version>
            <scope>provided</scope>
        </dependency>
```
3.修改启动类，继承extends SpringBootServletInitializer
```
@SpringBootApplication
public class ImportXcloudApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(ImportXcloudApplication.class, args);
    }

    //返回启动类builder
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(ImportXcloudApplication.class);
    }
}
```
