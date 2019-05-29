---
title: awk用法
date: 2018-12-31 11:55:31
tags: [linux]
---
#### 1.awk
用于处理文本
#### 2.用法
参考[菜鸟教程](http://www.runoob.com/linux/linux-comm-awk.html)
```
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```
###### 1.选项参数
- -F fs or --field-separator fs<br>
指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F,。<br>

```
awk -F, '{print $1,$2}' xxx.txt
以逗号为分隔符，打印第一第二列
```
- -v var=value or --asign var=value
赋值一个用户定义变量。

```
awk -va=1 '{print $1,$1+a}' xxx.txt
设置a为变量，打印第一和a+1列
```
###### 2.运算符
```
= += -= *= /= %= ^= **=	赋值
?:	C条件表达式
||	逻辑或
&&	逻辑与
~ ~!	匹配正则表达式和不匹配正则表达式
< <= > >= != ==	关系运算符
空格	连接
+ -	加，减
* / %	乘，除与求余
+ - !	一元加，减和逻辑非
^ ***	求幂
++ --	增加或减少，作为前缀或后缀
$	字段引用
in	数组成员
```
```
awk '$1>2' xxx.txt
输出第一列大于2的行
awk -F, '$1>2' xxx.txt
输出第一列大于2的行(字符串也会进行比较 例:value1-2 > 2)
value1-1 + value1-2 = 0?
awk -F, '$1=2' xxx.txt
将第一列赋值为2
awk '$1>2 && $2=="Are" {print $1,$2,$3}' xxx.txt
第一列大于2并且第二列等于'Are'的行,输出第一二三列
```

###### 3.内建变量
```
$n	当前记录的第n个字段，字段间由FS分隔
$0	完整的输入记录
ARGC	命令行参数的数目
ARGIND	命令行中当前文件的位置(从0开始算)
ARGV	包含命令行参数的数组
CONVFMT	数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组
ERRNO	最后一个系统错误的描述
FIELDWIDTHS	字段宽度列表(用空格键分隔)
FILENAME	当前文件名
FNR	各文件分别计数的行号
FS	字段分隔符(默认是任何空格)
IGNORECASE	如果为真，则进行忽略大小写的匹配
NF	一条记录的字段的数目
NR	已经读出的记录数，就是行号，从1开始
OFMT	数字的输出格式(默认值是%.6g)
OFS	输出记录分隔符（输出换行符），输出时用指定的符号代替换行符
ORS	输出记录分隔符(默认值是一个换行符)
RLENGTH	由match函数所匹配的字符串的长度
RS	记录分隔符(默认是一个换行符)
RSTART	由match函数所匹配的字符串的第一个位置
SUBSEP	数组下标分隔符(默认值是/034)
```
