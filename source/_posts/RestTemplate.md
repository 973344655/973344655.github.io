---
title: RestTemplate
date: 2018-08-30 16:51:32
tags: springboot, java, RestTemplate
---

因为在代码中需要调用外部的接口，所以使用RestTemplate

###### 1.导入
```
<!-- 包含有restTemplate -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

###### 2.完成配置类
```
@Configuration
public class RestTemplateConfig {
  //因为在普通类中使用它，所以给Bean去个名字，方便getBean()导入
	@Bean(name = "RestTemplate")
	public RestTemplate restTemplate(ClientHttpRequestFactory factory){
		return new RestTemplate(factory);
	}

	@Bean
	public ClientHttpRequestFactory simpleClientHttpRequestFactory(){
		SimpleClientHttpRequestFactory factory=new SimpleClientHttpRequestFactory();
		factory.setConnectTimeout(15000);
		factory.setReadTimeout(5000);
		return factory;
	}
}
```

###### 3.使用
```
public class AcceptDeal extends Thread {

  //通过名字导入
	RestTemplate restTemplate = (RestTemplate) SpringUtil.getBean("RestTemplate");

	//回执接口
	String url = "url" + "/" + report.getState();
	restTemplate.getForEntity("url", String.class);

}

```
