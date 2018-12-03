---
title: tomcat-redis-session
date: 2018-09-28 15:47:53
tags: [tomcat, redis, session,java]
---
#### 1.需求
不同Ip主机上的tomcat,保持session同步。<br>
#### 2.查资料
一般做法为，利用redis保持session,不同tomcat从redis存取，外层nginx做tomcat的负载均衡。<br>
开源项目https://github.com/jcoleman/tomcat-redis-session-manager， 提供了tomcat连接redis的jar包，但是，我试了多次，总是不行，版本不对，或者class找不到等。<br>
解决方案参照：https://my.oschina.net/Listening/blog/674759, 直接用gradle生成jar包。

#### 3.实现
tomcat7.0.61/nginx1.10.3/ubuntu虚拟机
###### 1.tomcat
- 解压两个tomcat到两个虚拟机
- conf/server.xml更改端口，使之不重复
- 添加index.jsp到 webapps/Root目录下
- 分别启动tomcat,看看是否正常
```
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <base href="<%=basePath%>">  
    <title>My JSP 'index.jsp' starting page</title>  
    <meta http-equiv="pragma" content="no-cache">  
    <meta http-equiv="cache-control" content="no-cache">  
    <meta http-equiv="expires" content="0">      
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
    <meta http-equiv="description" content="This is my page">  
    <!--
    <link rel="stylesheet" type="text/css" href="styles.css">
    -->  
  </head>  
  <body>  
        SessionID:<%=session.getId()%>  
        <BR>  
        SessionIP:<%=request.getServerName()%>  
        <BR>  
        SessionPort:<%=request.getServerPort()%>  
        <%  
        out.println("This is Tomcat Server 22222");  
        %>  
  </body>  
</html>
```

###### 2.nginx
编辑/etc/nginx/nginx.conf 添加tomcat配置
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
	# multi_accept on;
}

http {
 .......
	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;

upstream tomcat{
	#ip_hash;
	server   192.168.93.132:8081	weight=1;
	server   192.168.93.133:8080	weight=1;
	}
server{
	listen 80;
	server_name localhost 127.0.0.1;
            location / {
                root    html;
                index   index.jsp;
                proxy_pass      http://tomcat;
                proxy_set_header        X-Real-IP       $remote_addr;
                client_max_body_size    100m;
            }
	}
}
........
}
```

###### 3.redis
这一步很坑，参照：https://my.oschina.net/Listening/blog/674759, 进行配置<br>

将生成的jar包全部放入tomcat/lib/目录下<br>
编辑tomcat/conf/content.xml文件,添加
```
<Context>
    ........
    <!--
    <Valve className="org.apache.catalina.valves.CometConnectionManagerValve" />
    -->
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
    host="127.0.0.1" //注意这里，应该是redis地址
    port="6379"
    database="0"
    maxInactiveInterval="60"
/>

</Context>
```
在这里可能会出现redis连接不上问题，原因是/etc/redis/redis.conf 文件里默认配置为 bind 127.0.0.1, 改为 0.0.0.0<br>
除此之外也有可能防火墙原因.<br>
重启 redis-server /etc/redis/redis.conf <br>
查看是否监听0.0.0.0:6379端口 ps -ef | grep redis
![redisport](../images/redis.png)


#### 4. 重启tomcat 查看效果
![redisport](../images/tomcat1.png)
![redisport](../images/tomcat2.png)

可以看到现在session已经保持不变
