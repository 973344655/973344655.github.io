---
title: springboot Controller 接收参数
date: 2019-02-22 10:34:34
tags: [java]
---
#### 一.原因
一直对springboot前后端交互的传参接参浑浑噩噩，准备仔细梳理一下。

#### 二.常见注解
###### 1.@PathVariable
用于Get请求<br>
获取请求url中的值，一般用来接收url/{param}类型的参数<br>
一般返回类型为json或页面跳转等
```
//请求url: http://127.0.0.1:8080/get1/123
@ResponseBody
@RequestMapping(value = "/get1/{param}",method = RequestMethod.GET)
public String  get1(@PathVariable("param") String param){
    System.out.println(param);
    return param;
}
```
###### 2.@RequestParam
用于Get/Post都行<br>
在get请求时  获取请求参数中的值<br>
参数一般为url后面的param=param(&xx=xx)或者form-data
```
//请求url: http://127.0.0.1:8080/get2?param1=param1&param2=param2
@ResponseBody
    @RequestMapping(value = "/get2", method = RequestMethod.GET)
    public String get2(@RequestParam("param1") String param1,
                       @RequestParam("param2") String param2){
        return param1 + param2;
    }
```
###### 3.@RequestBody
用于Get/Post都行<br>
该注解和@RequestParam殊途同归，我们使用该注解将所有参数转换，在代码部分在一个个取出来，也是目前我使用到最多的注解来获取参数<br>
@RequestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型
```
//请求url: http://127.0.0.1:8080/get3
//参数为json: {"param1":"11","param2": "22"}
@ResponseBody
@RequestMapping(value = "/get3", method = RequestMethod.GET)
public String get3(@RequestBody Map<String,String> param){
    return param.get("param1") + param.get("param2");
}
```
###### 4.HttpServletRequest
get或post<br>
适用于param=param(&xx=xx)或者form-data<br>
form-data会解析为param=param&xx=xx形式
```
 //请求参数为: http://127.0.0.1:8080/get4?param1=11&param2=22
 //或参数为form-data
 @ResponseBody
 @RequestMapping(value = "/get4", method = RequestMethod.GET)
 public String get4(HttpServletRequest request) {
     return request.getParameter("param1")+request.getParameter("param2");
 }
```

###### 5.直接获取
get请求<br>
适用于param=param(&xx=xx)或者form-data<br>
```
@ResponseBody
@RequestMapping(value = "/get5", method = RequestMethod.GET)
public String get4(String param1, String param2) {
    return param1+param2;
}
```

######
