---
title: Mysql索引了解
date: 2019-08-09 09:32:54
tags: [database]
---

本文主要是为了对Mysql的索引有一个感性的理解.<br>

优秀文章:https://tech.meituan.com/2014/06/30/mysql-index.html<br>

# 一.索引简介

## 1.什么是索引

索引(Index)是帮助MySQL高效获取数据的数据结构. 实际上，索引也是一张表，利用某种数据结构(B+树或Hash索引，可以进行快速查找)，存储了索引与实体表记录直接的映射，可以通过查找索引，快速的查找到要找数据在实体表中的位置，减少IO操作。

## 2.为什么要用或不用索引


在查询操作时，索引可以极高的提升查询速度。<br>

在更新(insert/update/delete)操作,mysql不仅需要保持数据，还需要更新保存一下索引文件(维护的数据与其所在位置关系).<br>

注意，索引更新保存不等于索引重建<br>

所以，什么时候要用或者不用索引，就有有了个大概方向了。在查询操作数量较多，或时间要求较高时，可以建索引，反之，就不用索引。<br>

## 3.什么时候索引重建

重建索引在常规的数据库维护操作中经常使用。在数据库运行了较长时间后，索引都有损坏的可能，这时就需要重建。对数据重建索引可以起到提高检索效率。<br>


# 二.索引建立

## 1.创建索引

1．ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引<br>
```
ALTER TABLE table_name ADD INDEX index_name (column_list)
ALTER TABLE table_name ADD UNIQUE (column_list)
ALTER TABLE table_name ADD PRIMARY KEY (column_list)
```
2.CREATE INDEX可对表增加普通索引或UNIQUE索引<br>
```
CREATE INDEX index_name ON table_name (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)
```
3.联合索引
```
create index 索引名 on 表名（字段名1，字段名2）
```


## 2.最左前缀匹配

在mysql建立 <strong>联合索引时</strong> 会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配.<br>

```
Index顺序 (col,col3,col2)
SELECT * FROM test WHERE col1=“1” AND col2=“2” AND col3=“3”
```
意思就是，如上，该查询会先进行col1=1匹配，再进行col3=3,最好进行col2=2<br>

## 3.explain

可用explain来分析sql执行情况.
```
EXPLAIN
SELECT org_seq, api_code
    FROM api_log a
INNER JOIN
    cost_org_pre_stock_change_detail b ON a.org_code=b.orgcode
WHERE a.sequence='vGGztA1b'
```
![index1](http://67.216.218.49:8000/file/blogs/database/mysql/mysql_index_1.png)

一般来说，rows 越小，说明查找的行数越少，速度越快.<br>
