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
设置header与发送接收类型
```
try {
           String url = "http://xxxxx";
           //设置ContentType
           HttpHeaders httpHeaders = new HttpHeaders();
           MediaType type = MediaType.parseMediaType( "application/x-www-form-urlencoded;charset=utf-8");
           httpHeaders.setContentType(type);
           //form表单
           MultiValueMap<String, String> paramMap = new LinkedMultiValueMap<String, String>();
           paramMap.add("number", deliver.getUserNumber().substring(2));
           paramMap.add("port", deliver.getSPNumber());
           paramMap.add("content",new String(deliver.getMessageByte(), "UnicodeBigUnmarked"));
           paramMap.add("time", df.format(new Date()));
           HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<MultiValueMap<String, String>>(paramMap, httpHeaders);
           //调用回执接口
           String re = restTemplate.postForObject(url, requestEntity, String.class);
       }catch (Exception e){
         e.printStackTrace();
       }
```
