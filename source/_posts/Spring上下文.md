---
title: Spring上下文
date: 2018-08-30 16:34:21
tags: [springboot, java]
---

非spring管理的类（普通类），获得一个Bean。
在一个普通类中，@Autowired和new都不能导入一个Bean对象
```
public class AcceptDeal extends Thread {
  @Autowired
  LogService logservice;//但此时mapper中信息又为null
  //或者new 一个对象
  //LogService logservice = new LogService();//此时mapper又不能导入
}
```
解决方案：使用Spring上下文，获取Bean

###### 1.新建一个工具类SpringUtil
可以根据自己的需要添加上下文相关的管理方法
```
package com.bonc.sms.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * 普通类调用Spring bean对象：
 */
@Component
public class SpringUtil implements ApplicationContextAware{
	private static ApplicationContext applicationContext = null;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
	   if(SpringUtil.applicationContext == null){
		   SpringUtil.applicationContext  = applicationContext;
	   }
	}

	//获取applicationContext
	public static ApplicationContext getApplicationContext() {
	   return applicationContext;
	}

	//通过name获取 Bean.
	public static Object getBean(String name){
	   return getApplicationContext().getBean(name);
	}

	//通过class获取Bean.
	public static <T> T getBean(Class<T> clazz){
	   return getApplicationContext().getBean(clazz);
	}

	//通过name,以及Clazz返回指定的Bean
	public static <T> T getBean(String name,Class<T> clazz){
	   return getApplicationContext().getBean(name, clazz);
	}
}
```

###### 2.在Springboot启动类中，向上下文工具类SpringContextUtil中注入applicationContext
不确定是否必须加这一步,网上教程说加，我没加
```
@SpringBootApplication
public class App{
	public static void main( String[] args ){
		System.out.println( "Hello World!" );
		ApplicationContext app = SpringApplication.run(App.class, args);
		SpringContextUtil.setApplicationContext(app);
	}
}
```

###### 3.在普通类中通过getBean()获取
```
//导入Logservice
LogService logService = (LogService) SpringUtil.getBean(LogService.class);
```
