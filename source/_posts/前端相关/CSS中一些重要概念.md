---
title: CSS 中一些重要知识点
date: 2019-11-06 15:23:25
tags: [前端]
---

为了能马马虎虎的写个页面，学习一下CSS.

不是专业的，所以写的没逻辑，也没主次，用来自己看看。

参考：https://geekplux.com/2014/04/25/several_core_concepts_of_css/

# 一.盒模型

## 1.盒

![box](http://67.216.218.49:8000/file/blogs/others/css-box-model.png)

如图: 盒模型由 content -> padding -> border -> margin 四部分组成。

块 和 行内元素 都是包含在盒中的。

## 2.块元素

每个块级元素默认占一行高度，一行内添加一个块级元素后无法一般无法添加其他元素（float浮动后除外）。两个块级元素连续编辑时，会在页面自动换行显示。块级元素一般可嵌套块级元素或行内元素；

## 3.行内元素

和其他元素都在一行上。

对行内元素，需要注意如下：<br>

设置宽度width 无效。<br>
设置高度height 无效，可以通过line-height来设置。<br>
设置margin 只有左右margin有效，上下无效。<br>
设置padding只有左右padding有效，上下则无效。注意元素范围是增大了，但是对元素周围的内容是没影响的。

## 4.位置问题

 position属性：

值	| 描述
--- | ---
absolute |	生成绝对定位的元素，相对于static定位以外的第一个父元素进行定位。元素的位置通过left, right, top, bottom进行规定
fixed	 | 生成绝对定位的元素，相对于浏览器窗口进行定位。元素的位置通过left, right, top, bottom进行规定
relative | 	生成相对定位的元素，相对于其正常位置定位。元素的位置通过left, right, top, bottom进行规定
static	| 默认值，忽略 top, bottom, left, right和z-index
inherit |	从父元素继承该属性的值

## 5.float

自己还一知半解。

参考人家的：https://www.jianshu.com/p/07eb19957991


# 二.选择器

太多了记不住，选择几个感觉够用的。来自w3school.

找时间了解一下伪元素。

选择器 | 例子 |  描述
--- | --- | ----
.class |	.intro	| 选择 class="intro" 的所有元素。
\#id	| \#firstname	| 选择 id="firstname" 的所有元素。
\*	| \*	| 选择所有元素。
element	| p	| 选择所有 \<p\> 元素。
element,element	| div,p	| 选择所有 \<div\> 元素和所有 \<p\> 元素。
element element	| div p	| 选择 \<div\> 元素内部的所有 \<p\> 元素。
element>element	| div>p	| 选择父元素为 \<div\> 元素的所有 \<p\> 元素。
element+element	| div+p	| 选择紧接在 \<div\> 元素之后的所有 \<p\> 元素。
[attribute]	| [target]	| 选择带有 target 属性所有元素。
[attribute=value]	| [target=\_blank] |	选择 target="\_blank" 的所有元素。
[attribute~=value]	| [title~=flower]	| 选择 title 属性包含单词 "flower" 的所有元素。
[attribute\|=value] |	[lang\|=en]	| 选择 lang 属性值以 "en" 开头的所有元素。
:link	 | a:link	| 选择所有未被访问的链接。
:visited	| a:visited	| 选择所有已被访问的链接。
:active	| a:active	| 选择活动链接。
:hover	| a:hover	| 选择鼠标指针位于其上的链接。
:focus	| input:focus	| 选择获得焦点的 input 元素。
:before	| p:before	| 在每个 <\p\> 元素的内容之前插入内容。
:after	| p:after	| 在每个 <\p\> 元素的内容之后插入内容.

# 三.重要属性

原文:https://www.html.cn/archives/558



# 四.伪元素和伪类

https://segmentfault.com/a/1190000000484493
