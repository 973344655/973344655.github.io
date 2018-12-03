---
title: xss
date: 2018-11-06 14:43:30
tags:
---

1.印象笔记6.14客户端版本
存储型xss,上传的图片保存时为用户输入名字，(某个属性，src属性或者value属性?)未过滤编码等。
src属性或者value属性?
<img .. src="图片路径 + 图片名">
<img .. src="图片路径 + " onclick="xss">.jpg ">
