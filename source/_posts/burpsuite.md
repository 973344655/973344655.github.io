---
title: burpsuite
date: 2018-12-12 10:48:18
tags: [burpsuite]
---

#### 1.配置
先配置浏览器代理，127.0.0.1：8080，使所有流量都要经过这<br>
  ![proxy_1](../images/security/burpsuite/chrome_proxy.png)

  ![proxy_1_1](../images/security/burpsuite/chrome_proxy2.png)

再配置burpsuite监听这个端口的流量<br>
  ![proxy_2](../images/security/burpsuite/burpsuite_proxy.png)

此外 ，因为要访问https,配置完这些后，打开浏览器访问http://burp，下载证书，安装到受信任颁发机构<br>
![proxy_3](../images/security/burpsuite/burpsuite_ca_certificate.png)
![proxy_3_1](../images/security/burpsuite/ca_certificate_install.png)

#### 实战
##### 1.第一个抓包小例子
1.先随意搜索后台登录页面,inurl:login.html,在这里我随机找了一个http://usr.005.tv/User/login.html
![login_page_1](../images/security/burpsuite/usr005tv_login_page1.png)
先点击注册
![register_page_1](../images/security/burpsuite/usr005tv_register1.png)

<br>
<strong>其中我们邮箱和手机号都是填的假号码。
然后开始拦截抓包，滑动滑块

![package1](../images/security/burpsuite/usr005tv_package_1.png)
修改号码为自己手机号，收到验证码
![package1](../images/security/burpsuite/usr005tv_package_2.png)
输入验证码
![package1](../images/security/burpsuite/usr005tv_package_3.png)
在这里发现，后端验证时，同时验证了发送验证码的手机号和验证码，所以验证失败，如果在这也修改为自己手机号，那注册时的假号码就无意义了。
#### 问题
1.使用burpsuite抓包后，vpn没用了，不能访问外网<br>

解决，虚拟机里使用burpsuite,nat模式连接，物理机还是挂了vpn
