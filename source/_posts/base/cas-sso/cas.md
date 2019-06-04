---
title: Cas 单点登录原理
date: 2019-5-30 16:30:07
tags: [base]
---
### 单点登录
单点登录（Single Sign-On ,简称SSO),只用户只需在一个地方登录，就能保持登录状态访问所有相互信任的子系统。

### CAS
CAS 为 Central Authentication Service。<br>
CAS 分为 CAS Server 和 CAS Client.<br>
- CAS Server 负责用户的认证和授权
- CAS Client 与受保护的客户端应用部署在一起，负责认证用户是否登录,同时负责处理对客户端受保护资源的访问请求，需要登录时，重定向到CAS Server，以Filter方式保护受保护的资源。

#### 一.授权与登录操作
###### 1.未登录时
![cas01](http://67.216.218.49:8000/file/blogs/base/cas/cas01.png)
如图:<br>
- (1)客户端(即浏览器)请求资源(需要登录的受限资源)，CAS Client发现无权限(未登录，通过cookies中JSESSIONID判断是否登录)，尝试获取ticket(未获取到)。
- (2)CAS Client 返回重定向(CAS Server 登录地址)并在请求地址中带上原本要请求的资源地址，客户端请求该地址(重定向自动请求)获取ticket，此时ticket不存在(未登录)，返回登录页面。
- (3)客户端进行登录(带上认证信息，原本的请求资源地址，sessionid,sessionid为CAS Client生成)，CAS Server 验证通过，生成与sessionid进行关联的TGT与ST,将请求的cookie中jsessionid设置为传过来的sessionid，并返回重定向(要请求的资源地址，带上ticket(ST))
![cas_session01](http://67.216.218.49:8000/file/blogs/base/cas/cas_sessionid01.png)
将会话的jsessionid设置为图中1.
- (4)此时，CAS Client发现请求无权限(未登录)，尝试获取ticket(获取到),CAS Client拿着ticket去CAS Server 进行认证
- (5)此时，CAS Server 发现该ticket认证通过，返回登录成功的信息，CAS Client(可以理解为服务端) 利用前面设置的JSESSIONID为该用户创建session会话，维护登录状态(JSESSIONID)，并返回原本受限的请求资源

###### 2.已登录时
当已登录的用户第一次访问其它互相信任的系统时：<br>
- (1)客户端请求资源(需要登录的受限资源)，CAS Client发现无权限(此时该系统还未登录过，session会话未创建)，尝试获取ticket(未获取到)。
- (2)CAS Client 返回重定向(CAS Server 登录地址)并在请求地址中带上原本要请求的资源地址，客户端请求该地址(重定向自动请求)根据session获取ticket，此时ticket已存在,重定向请求。
- (3)此时，CAS Client发现请求无权限(未登录)，尝试获取ticket(获取到),CAS Client拿着ticket去CAS Server 进行认证，认证成功后设置该域的session.
- (4)此时，CAS Server 发现该ticket认证通过，返回登录成功的信息，CAS Client 为该用户创建session会话，保存在域下的登录状态，返回原本受限的请求资源。

#### 二.登录状态判断
已登录的用户在再次请求受限资源时，不能每次都去认证中心(CAS Server)判断是否登录(效率低)。<br>
![cas01](http://67.216.218.49:8000/file/blogs/base/cas/cas_cookie01.png)
当一个已经登录过的用户，再次请求该系统资源时，CAS Client会去session中查找用户信息(通过请求头的cookie中JSESSIONID字段的值)，判断是否登录。不用再去CAS Server认证。<br>
不同的域，各自维护自己的session.<br>

#### 三.退出
- (1)客户端向当前web应用发起退出请求。
- (2)应用取消本地会话session，同时通知CAS Server，用户已登出，清除TGT。
- (3)应用返回客户端登出请求。
- (4)CAS Server 通知所有用户登录访问的应用，用户已登出。

#### 四.CAS中的ticket
- TGT：TGT是CAS为用户签发的登录票据，拥有了TGT，用户就可以证明自己在CAS成功登录过。TGT封装了Cookie值以及此Cookie值对应的用户信息。用户在CAS 认证成功后，CAS生成cookie，写入浏览器，同时生成一个TGT对象，放入自己的缓存，TGT对象的ID就是cookie的值。当HTTP再次请求到来时，如果传过来的有CAS生成的cookie，则CAS以此cookie值为key查询缓存中有无TGT ，如果有的话，则说明用户之前登录过，如果没有，则用户需要重新登录。
- ST：服务票据，ST是CAS为用户签发的访问某一service的票据。用户访问service时，service发现用户没有ST，则要求用户去CAS获取ST。用户向CAS发出获取ST的请求，如果用户的请求中包含cookie，则CAS会以此cookie值为key查询缓存中有无TGT，如果存在TGT，则用此TGT签发一个ST，返回给用户。用户凭借ST去访问service，service拿ST去CAS验证，验证通过后，允许用户访问资源。
- LT: 定制login页面时候，提交以后还是要提交到cas进行验证，那么参数中需要带lt，这个lt必须在前面获取到，防止重复提交之类的作用。
- TGC:在系统A登录成功后，用户和认证中心之间建立起了全局会话，这个全局会话就是TGT(Ticket Granting Ticket)，TGT位于CAS服务器端，TGT并没有放在Session中，也就是说，CAS全局会话的实现并没有直接使用Session机制，而是利用了Cookie自己实现的，这个Cookie叫做TGC(Ticket Granting Cookie)，它存放了TGT的id(jsessionid),保存在用户浏览器上
