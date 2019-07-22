---
title: location模块
date: 2019-07-19 14:54:32
tags: [tools]
---

测试所用版本:
```
root@localhost:~# nginx -v
nginx version: nginx/1.10.3 (Ubuntu)
```
# 一. Location表达式

 <strong> location [ = | \~ | \~\* | ^~ | @ ] /uri/ { … } </strong>
- /{}

  无前缀，表示最大前缀匹配.

- @

  几乎见不到，不用管

- =

 完全匹配<br>
 url部分为啥就精确匹配啥.


 ```
# 1
location = /t1 {
      return 601;
}
# 2
location = /t2 {
        return 602;
}
# 3
location = /t1/ {
        return 603;
}
 ```
 例子:<br>
http://67.216.218.49:8080/t1 601

http://67.216.218.49:8080/t2 602

http://67.216.218.49:8080/t 404

加上条件3前<br>
http://67.216.218.49:8080/t1/ 404<br>
加上条件3后<br>
http://67.216.218.49:8080/t1/ 603

- \~

正则,大小写敏感
```
location ~ /T1 {
       return 601;
}
location ~ /t1 {
       return 603;
}
```
http://67.216.218.49:8080/t1 603<br>
自动转为了 小写?<br>
http://67.216.218.49:8080/T1 603

```
location ~ /T1/TT {
       return 601;
}
location ~ /t1/tt {
       return 603;
}
```
http://67.216.218.49:8080/t1/tt 603<br>
http://67.216.218.49:8080/T1/TT 601

- \~\*

正则,大小写不敏感
```
location ~* /t1/tt {
        return 601;
}
location ~* /T1/TT {
        return 603;
}
```

忽略大小写，按正则出现顺序开始匹配<br>
http://67.216.218.49:8080/T1/TT 601<br>
http://67.216.218.49:8080/t1/tt 601<br>
http://67.216.218.49:8080/t1/TT 601<br>
http://67.216.218.49:8080/t1/TT/tt/tt 601<br>

- ^~

表示匹配时，忽略正则
```
location ^~ /t1 {
        return 601;
}
location  /t1tt {
        return 603;
}
```
http://67.216.218.49:8080/t1 601<br>
http://67.216.218.49:8080/t1t 6011<br>
http://67.216.218.49:8080/t1tt 6031<br>
http://67.216.218.49:8080/t1ttt 6031<br>

# 二.location 匹配顺序


顺序: 先普通匹配，再正则匹配,中间注意的下面讲.<br>

以 ~ 开头的为正则匹配，其余为普通匹配.<br>

## 1.实例

- 普通匹配中，先精确匹配

```
location ^~ /t1 {
        return 601;
}
location = /t1 {
        return 602;
```
http://67.216.218.49:8080/t1  602

- 正则按顺序来

```
location ~* /t1{
        return 601;
}
location ~* /t1tt {
        return 603;
}
```

http://67.216.218.49:8080/t1tt 601<br>
```
location ~* /t1ttt{
        return 601;
}
location ~* /t1tt {
        return 603;
}
```
http://67.216.218.49:8080/t1ttt 601

- 正则普通(最大前缀),先匹配普通，匹配到了，先记录下来，再匹配正则，匹配到了使用正则

```
location ~* /t1tt {
       return 601;
}
location  /t1tt {
       return 603;
}
```
http://67.216.218.49:8080/t1tt 601<br>
http://67.216.218.49:8080/t1ttt 601<br>


- 正则和普通(^~),不使用正则，直接普通匹配

```
location ^~ /t1tt {
        return 601;
}
location ~* /t1tt {
        return 603;
}
```

http://67.216.218.49:8080/t1tt 601

- 正则和普通(精确匹配)，精确匹配到了，直接停止匹配

```
location ~* /t1tt {
       return 601;
}
location  = /t1tt {
       return 603;
}
```
http://67.216.218.49:8080/t1tt 603<br>

- 普通匹配中 ^~ 和 / 不能有重复的

```
location ^~ /t1t {
       return 601;
}
#  false
#location  /t1t {
#       return 603;
#}

location  /t1 {
       return 603;
}

```
http://67.216.218.49:8080/t1t 601<br>
http://67.216.218.49:8080/t1r  603


## 2.顺序总结

总结

先精确匹配，匹配到了就使用<br>
普通匹配，普通匹配时，选择匹配最长的(记录下来)，再进行正则匹配<br>
正则匹配时，如果匹配成功且在普通匹配中不存在相应的^~ 使用正则匹配的结果，否则直接使用^~的结果（当普通匹配的最长前缀匹配有符号“^~”的时候，就不会在匹配正则），如果正则匹配不成功，使用普通匹配(最大前缀匹配)的结果



## 3.注意问题

- 1.url中/的使用注意,会进行匹配

```
location /t1tt/ {
        return 601;
}

location ^~ /t1 {
        return 603;
}
```

http://67.216.218.49:8080/t1tt  603

- 2.普通按照最长匹配

```
location /t1tt {
        return 601;
}

location ^~ /t1 {
        return 603;
}
```
http://67.216.218.49:8080/t1tt  601

- 3.正则使用时小心被覆盖

/t1tt 被正则覆盖，导致访问不到, 实际遇到的问题之一<br>
```
location ~* /t1 {
        return 601;
}
location  /t1tt {
        return 603;
}
```
http://67.216.218.49:8080/t1tt  601

# 三.代理属性


## 1. 请求头信息 proxy_set_header

可设置请求头-并将头信息传递到服务器端。该值可以包含文本、变量和它们的组合<br>

proxy_set_header field value 本质上就是把 header里面的 field字段 设置为想要的value.<br>

- proxy_set_header Host $proxy_host;

$proxy_host(默认值): 该代理nginx的ip
- proxy_set_header Host $http_host;

$http_host: 使用请求原host
- proxy_set_header Host $host;

$host: 如果客户端请求头中没有携带这个头部，那么传递到后端服务器的请求也不含这个头部。 这种情况下，更好的方式是使用$host变量——它的值在请求包含“Host”请求头时为“Host”字段的值，在请求未携带“Host”请求头时为虚拟主机的主域名<br>

- proxy_set_header Host $host:$proxy_port;

同理,proxy_set_header Host $host:$server_port<br>
服务器名可以和后端服务器的端口一起传送<br>

- proxy_set_header Host 127.0.0.1:8080;

还可以不使用变量，直接写文本，或者变量加文本<br>

- proxy_set_header X-Real-IP $remote_addr;

其中这个X-real-ip是一个自定义的变量名，名字可以随意取，这样做完之后，用户的真实ip就被放在X-real-ip这个变量里了，然后，在web端可以这样获取：<br>
request.getAttribute("X-real-ip")<br>

- proxy_set_header X-Forwarded-For $remote_addr;

一层代理时,可以直接存入地址<br>
- proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

多层代理时，可以存入代理链<br>
每次经过proxy转发都会有记录,格式就是client1, proxy1, proxy2,以逗号隔开各个地址<br>


## 2.代理 proxy_pass

项目名: test, 路径 test<br>
<strong>注意 / 在匹配中的作用 </strong><br>
```
# 1
location  /test {
      proxy_pass http://127.0.0.1:8002;
}
# 2
location  /test/ {
      proxy_pass http://127.0.0.1:8002;
}
# 3
location  /test {
      proxy_pass http://127.0.0.1:8002/;
}
# 4
location  /test {
      proxy_pass http://127.0.0.1:8002/test;
}

```

使用 1 时<br>
http://67.216.218.49:8080/test/test 200<br>
使用 2 时<br>
http://67.216.218.49:8080/test/test 200<br>
使用 3 时<br>
http://67.216.218.49:8080/test 200(tomcat页面)<br>
http://67.216.218.49:8080/test/test  404<br>
http://67.216.218.49:8080/test/test/test 200(接口返回)<br>
使用 4 时<br>
和 3 一样， 跳转时不会自动加上 location 中 url, 而是使用 proxy_pass 中 url<br>

总结: 在匹配时，location中url后面加不加 / 没用影响，但是 proxy_pass 中 url 不加 / 会默认加上 location中url跳转， 加上 / 则不会带上 location中 url 跳转.<br>

## 3.重定向 proxy_redirect

当服务返回重定向时（http code为301或302），浏览器会根据response中的location字段进行跳转(浏览器直接跳转到服务地址).<br>
此时，为了能让跳转经过nginx,可以使用 proxy_redirect 修改location的重定向信息.<br>

```
# 1 重定向到 其它location
location /portal{
proxy_set_header Host $host:$server_port;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header msec $msec;
proxy_pass http://portal;
index index.html index.htm;
proxy_redirect  http://10.191.21.105:8090/cas /cas;
proxy_redirect  http://10.191.21.105:8090/portal  /portal;
}

# 2 重定向到具体地址
location /cas{
proxy_set_header Host $host:$server_port;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header msec $msec;
proxy_pass http://cas;
index index.html index.htm;
proxy_redirect  http://10.191.21.105:8090/portal  http://ip/port/path;
...
}
```
