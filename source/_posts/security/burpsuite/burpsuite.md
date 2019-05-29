---
title: burpsuite
date: 2018-12-12 10:48:18
tags: [security]
---

#### 1.配置
先配置浏览器代理，127.0.0.1：8080，使所有流量都要经过这<br>
  ![proxy_1](./image/chrome_proxy.png)

  ![proxy_1_1](./image/chrome_proxy2.png)

再配置burpsuite监听这个端口的流量<br>
  ![proxy_2](./image/burpsuite_proxy.png)

此外 ，因为要访问https,配置完这些后，打开浏览器访问http://burp，下载证书，安装到受信任颁发机构<br>
![proxy_3](./image/burpsuite_ca_certificate.png)
![proxy_3_1](./image/ca_certificate_install.png)

#### 实战
##### 1.第一个抓包小例子
1.先随意搜索后台登录页面,inurl:login.html,在这里我随机找了一个http://usr.005.tv/User/login.html
![login_page_1](./image/usr005tv_login_page1.png)
先点击注册
![register_page_1](./image/usr005tv_register1.png)

<br>
<strong>其中我们邮箱和手机号都是填的假号码。
然后开始拦截抓包，滑动滑块

![package1](./image/usr005tv_package_1.png)
修改号码为自己手机号，收到验证码
![package1](./image/usr005tv_package_2.png)
输入验证码
![package1](./image/usr005tv_package_3.png)
在这里发现，后端验证时，同时验证了发送验证码的手机号和验证码，所以验证失败，如果在这也修改为自己手机号，那注册时的假号码就无意义了。

##### 2.一个爆破小例子
来dvwn的登录（admin/password）
![intruder_01](./image/dvwn_login_1.png)
随便输入，抓取数据包，send to intruder
![intruder_02](./image/burpsuite_intruder_01.png)
选择Cluster模式，可以使用同的字典，进行穷举（笛卡尔积）<br>
然后，clear清空变量，鼠标选中后，add添加自己需要的变量
![intruder_03](./image/burpsuite_intruder_02.png)
分别添加变量对应的字典，进行爆破
![intruder_04](./image/burpsuite_intruder_03.png)
爆破完成后，点击length，查看length长度(正确的和错误的length长度会不样)
![intruder_05](./image/burpsuite_intruder_04.png)

#### 问题
1.使用burpsuite抓包后，vpn没用了，不能访问外网<br>

解决1，虚拟机里使用burpsuite,nat模式连接，物理机还是挂了vpn<br>

2018/12/20 添加:<br>
解决2，使用代理链 https://www.anquanke.com/post/id/85925<br>
同理，也可以使用代理链来用tor进行代理(防封ip)
![proxy_chain_01](./image/proxy_chain_01.png)
不稳定，容易出错：SOCKS server general failure<br>
原因待查。。。

#### 3.返回包修改与抓手机包
###### 3.1修改返回包
![modify_response](./image/modify_response.png)
点击拦截后, forward会将返回结果拦截，然后自己进行修改。<br>
再次点击forward将修改完的数据返回给原请求。
###### 3.2抓取手机http流量
https需要配置证书，其它一样。<br>
手机和burp在同一局域网下<br>
![intercept_mobile](./image/intercept_mobile.png)
配置burp监听本机ip,端口随意<br>
到手机设置里面，对连接的wifi进行修改。选择连接的wifi->显示高级选项->选择代理(手动)->
配置ip和端口(上一步burp中设置的)->ip(DHCP)。<br>
现在可以到burp中进行抓包了<br>
