---
title: googlehacking
date: 2018-10-10 05:24:40
tags: [security]
---

#### 1.GooleHacking
利用搜索引擎的强大搜索能力，针对性的全网搜集所需资料。<br>


#### 2.关键字
- intext/allintext: 在正文中搜索
- intitle/allintitle: 网页标题中， 例:intitle:后台管理，有很多
- cache: 缓存中？？
- filetype: 文件类型,   例:信息安全 filetype:doc
- info: 摘要信息
- inurl/allinurl: 在网址中
- site: 域名，返回域名中的所有url地址，用于检查网站拓扑结构。 例: site: baidu.com
- related: 搜索相关站点。  例:related:pku.edu.cn
- inanchor: 在链接文本中搜索。 例：inanchor:pku.edu.cn
- author: 新闻组贴子的作者。例:在google scholar搜索作者. author:"John"
- datarange: 某个日期范围内的网页.格式为天文学儒掠日。例：”Geri Halliwell” “Spice Girls” daterange:2450958-2450968
- weather: 天气. 例: weather:beijing
- group:
- stocks: 股票信息。 例:stocks:alibaba
- define:显示某术语定义。例:define:网络安全
- phonebook: 电话列表？？。 rphonebook:住宅电话 ; bphonebook:商业电话
- link：搜索与当前网页存在链接的网页（不能与其他操作符或搜索关键字混合使用. 例：link:www.vvv518.com
- msgid: 通过消息id搜索

#### 3.语法
- "" 精确操作
- 关键字之间不需要间隔，空格代表逻辑与操作
- \- 忽略某个关键词. 例:a-b 搜索有a没b的网页
- ～ 同义词
- \+ 加入被忽略的关键字。google对com等频率高的英文单词做了忽略处理，加上 + ，使之强制搜索。
- 布尔and/or/not 例：aorb
- 通配符，出现通配符的关键字需要用引号。\*多个， ？和 . 单个

#### 4.注意
  1. google只能32个单词查询。<br>
  2. 操作符、冒号、关键字之间是没有空格的。
  3. 高级操作符能够和单独的查询混合使用

#### 5.例子
