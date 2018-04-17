---
layout: post
title:  "elasticsearch学习笔记一"
categories: elasticsearch
tag: elasticsearch学习笔记
---

Query  DSL
=======

概述
-------
<br />

Elasticsearch提供一个基于json的查询DSL，把查询DSL当做一颗查询抽象语法树（AST），查询DSL是由两部分子句组成的：

* 叶查询子句
叶查询子句在特定字段上查询特定值，例如match、term和range查询。这些查询能被自身使用。
* 混合查询子句
混合查询子句包含其他叶查询和混合查询。混合查询使用逻辑词汇来组合多个查询（如bool和dis_max查询），或者更改它们的属性（比如constant_score查询）。

<br />

查询子句的行为取决于它们的查询和过滤上下文环境。

查询和过滤上下文
--------
<br />

查询子句的行为取决于它们的查询和过滤上下文环境。

查询上下文

* 在查询上下文中使用的查询子句确定了文档对于查询子句的匹配度，但是没有确定文档是否匹配上了，查询子句计算出_score结果表示此文档相对于其他文档的匹配度。
* 当查询子句被传递一个查询参数时查询上下文生效，例如在search API中传递参数。

过滤上下文

* 在过滤上下文中使用的查询子句确定了文档是否匹配查询子句，确定结果是简单表示为yes和no,不用计算score。过滤上下文大部分使用过滤结构，例如
* timestamp在2015和2016之间
* status字段是否设置为published
被频繁使用的filter将被elasticsearch自动缓存起来加速性能

* 当查询子句被传递一个过滤参数时过滤上下文生效，例如在bool查询中传递参数filter和must_not，constant_score中的过滤参数，或者过滤聚合。

下面是查询子句在查询和过滤上下文被使用的search api的例子，这个查询使用下面的条件去匹配文档
* title字段包含字符串search
* content字段包含字符串elasticsearch
* status字段包含字符串published
* publish_date字段包含晚于2015-01-01的时间
```json
GET /_search
{
  "query": { //1
    "bool": { //2
      "must": [
        { "match": { "title":   "Search"        }}, //2
        { "match": { "content": "Elasticsearch" }}  //2
      ],
      "filter": [ //3
        { "term":  { "status": "published" }}, //4
        { "range": { "publish_date": { "gte": "2015-01-01" }}} //4
      ]
    }
  }
}
```
1. query参数指出查询上下文
2. bool和两个match子句在查询上下文中被使用，意味着它们被用来确定文档的匹配程度
3. filter参数表示过滤上下文
4. term和range子句在过滤上下文中被使用，它们将过滤掉不匹配的文档，但是不影响文档的匹配度

<br />

在查询上下文中使用影响文档匹配度的子句，其他子句应该放在过滤上下文中。

Match All Query
-------
<br />

最简单的查询是匹配所有文档，所有文档的_score都是1

```json
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

_score能被boost参数改变

```json
GET /_search
{
    "query": {
        "match_all": { "boost" : 1.2 }
    }
}
```

mathc_none和match_all相反，不匹配任何文档

```json
GET /_search
{
    "query": {
        "match_none": {}
    }
}
```

Full text queries
-------
<br />

高级全文检索一般用于在像邮件正文这样的全文字段上运行全文检索，它们理解已经被分词的字段，并且在执行之前将每个字段的分词器（或者查询分词器）应用到查询串上。

全文检索包括如下
* match查询：全文检索的标准查询模式，包括模糊查询、短语查询和近似查询
* match_phrase查询：和match查询相似，但是被用来进行短语精确匹配和单词近似匹配
* match_phrase_prefix查询：和match_phrase相似，但是被用来进行在最后的单词上进行通配符查询
* multi_match查询：在多字段上处理的match查询
* common_terms查询：一个更专有化的查询，对于出现率较低的单词进行查询表现较好
* query_string查询：支持复杂的lucene查询串语法，允许你指定AND|OR|NOT条件和多字段查询在单个查询串中。值用于专业用户。
* simple_query_string:一个更简单和强健的query_string语法，比较适用于普通用户

<br />

Match查询

<br />

match查询接受文本、数字和日期，分析这些输入，然后构造一个查询，例如：
```json
GET /_search
{
    "query": {
        "match" : {
            "message" : "this is a test"
        }
    }
}
```

注意，message是一个字段的名称，你能将它替换成任何字段的名称（包含_all）

<br />
match
match查询是一种布尔类型，这意味着被提供的文本是用来被分析，并且分析过程从被提供的文本中构建的bool查询。操作符可以是or或者and来控制bool子句，默认为or。可选的should子句最小数量能用minimum_should_match参数来设置。

analyzer参数能被设置用来控制分析器对于文本的分析过程。默认为字段精确的匹配定义，或者系统默认的分析器。

lenient参数设置为true时，将忽略配错数据类型带来的异常，例如用一个文本查询串来查询一个数字字段。一般默认为false。


<br />
Fuzziness

fuzziness参数允许基于正在被查询字段的类型进行模糊匹配，下面为被允许的配置项

在下面示例中prefix_length和max_expansions参数用来控制模糊处理过程。如果模糊想
