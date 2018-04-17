---
layout: post
title:  "elasticsearch学习笔记二"
categories: elasticsearch
tag: elasticsearch学习笔记
---

term和match的使用区别
=======
<br />

term
-------

* 在过滤上下文中使用（filter），用来过滤文档
* terms，多值,terms:[]


<br />

match
-------

* 在查询上下文中使用(query)，用来查询文档
* 实质上为通过analyzer分词，将分词后的词通过bool和term进行组合查询
* boost：提高查询权重，加大评分权重

bool
=======

语法
"bool" : {
      "must" :     [], 
      "should" :   [],
      "must_not" : [],
   }

must：必须匹配
should：应该匹配，至少一个，理解为or
must_not：不能匹配

minimum_should_match:最少匹配should个数，默认为1

range
=======

语法

range{
    "price" : {
        "gte" : 20,
        "lte" : 40
    }
}

gte  >=
lte  <=
lt   <
gt   >


分析器
=======

索引时：字段映射配置的分析器>索引设置中名为 default 的分析器>默认分析器standard 标准分析器
查询时：查询自己定义的 analyzer>字段映射配置的分析器>索引设置中名为 default 的分析器>默认分析器standard 标准分析器

设置字段map时有一个可选的 search_analyzer 映射，它仅会应用于搜索时，有一个等价的 default_search 映射，用以指定索引层的默认查询analyzer配置

如果考虑到这些额外参数，一个搜索时的 完整 顺序会是下面这样：

* 查询自己定义的 analyzer ，否则
* 字段映射里定义的 search_analyzer ，否则
* 字段映射里定义的 analyzer ，否则
* 索引设置中名为 default_search 的分析器，默认为
* 索引设置中名为 default 的分析器，默认为
* standard 标准分析器

索引模板
=======

通过设置索引模板，当新的索引创建时，通过模板来设置索引的一些配置，如字段mapping等。

相关度
=======

相关度用来评断文档匹配度

当数据量太少时会出现TF-IDF算法的分片IDF不准确的情况，会导致_score不对的情况，但现实世界一般数据量都够大，能迅速均化IDF。