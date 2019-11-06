---
title: 是时候彻底分清Cookie/Session和Token了
date: 2019-10-31 16:10:25
tags: [base]
---

# 一.Cookie/Session

Cookie: 因为http是无状态协议，为了保存用户状态，所以Cookie由此产生。

Session: 存储在服务器端，里面存着用户信息，过期时间以及其它想要的数据。每个session有其唯一的sessionId与之对应。

Cookie与Session通常都是一起配合使用的。

cookie存在浏览器端，浏览器第一次请求服务器时，服务器会返回个set-cookie字段，然后以这个cookie作为浏览器和服务器之间的连接凭证，这个cookie多半为sessionId。
当下次客户端请求服务器时，浏览器会自动带上cookie,服务端通过该cookie中的信息，去查找判断对应的用户信息。


# 二.Token

有了Cookie和Session, 已经能保存用户状态了，为什么还会出现token呢？

答案， 为了防止csrf.
一个有csrf漏洞的网页，攻击者可以获取用户的cookie去进行危险操作(因为发请求的时候，浏览器自动带上了cookie)，
但是，token不同，token是特别设计的令牌，需要手动加到header中，浏览器不会自己往header中增加token字段，攻击者也无法获得token,
服务端验证不通过，导致csrf攻击不成功。


# 三.一点补充

session是有状态的，因为需要存放在服务器端(内存或者数据库)，所以当有多个服务器时，会有session共享问题，

token是一个令牌，无状态，数据都加密保存在token中，服务器拿到后，直接解密就能用，所以没有共享问题。
