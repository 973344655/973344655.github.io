---
title: springboot 整合 es 的RestHighLevelClient
date: 2019-07-16 18:55:31
tags: [tools]
---

官方文档: https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html

###  一.基本查询方式
- match

在查询前会用分析器去分析字符串,会进行分词匹配搜索. <br>
如果在一个精确值的字段上使用它， 例如数字、日期、布尔或者一个 not_analyzed 字符串字段，那么它将会精确匹配给定的值<br>

- match_all

匹配所有文档.<br>

- multi_match

可以在多个fileds上执行匹配<br>

- range

在区间内匹配<br>

- term

精确查询，完全匹配，不会进行分词.<br>

- terms

可以匹配多个值<br>

- exists/missing

exists 和 missing 被用于查找那些指定字段中有值 (exists) 或无值 (missing) 的文档.

### 二.组合查询

#### 1.bool


接收的参数:
```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }}
        }
    }
}
```


- must

必须匹配

- must_not

必须不匹配

- should

如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。但，如果存在至少一条 must 语句，则对 should 语句的匹配没有要求<br>
有 must 时，如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。<br>
用于增加评分<br>

#### 2.constant_score

<strong> 不常用</strong><br>
constant_score 它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。
```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" }
        }
    }
}
```
可以使用它来取代只有 filter 语句的 bool 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。

- filter

根据过滤标准来排除或包含文档。
