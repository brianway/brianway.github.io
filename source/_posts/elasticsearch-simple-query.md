---
layout: post
title:  Elasticsearch 5.0-简单查询
date:   2016-12-13 21:45:07 +08:00
category: 基础教程
tags: [Elasticsearch]
comments: true
---

本文是 Elasticsearch 5.0 系列博文的简单查询篇,主要介绍如何使用 Elasticsearch 进行简单查询

<!-- more -->

## 写在前面

- 本文以 Elasticsearch 5.0.1 版本为例进行讲解,不定期更新
- 该系列主要参考的 [Elasticsearch Reference: 5.0](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/index.html)，尽量避免照搬翻译，只摘录精要部分辅以简单说明
- 写这个系列博客的初衷是强迫自己梳理，同时方便一些较忙/没空耐心看英文文档的朋友快速上手，建议读者有空多读官方文档，毕竟别人写的都是二手资料
- 如需查看 ES 系列更多博文，请关注我的个人网站[@brianway](http://brianway.github.io/) 或者  [@CSDN](http://blog.csdn.net/h3243212/)

## 数据准备

随机 json 数据生成网站 [`www.json-generator.com`](www.json-generator.com/)

可以从官网下载数据 [accounts.json](https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json)，然后使用  `_bulk api` 建立索引即可


```shell
curl -O https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
```

## 查询API

两种执行 search 的方式：

- 通过 `REST request URI` 发送查询参数
- 通过 `REST request body`发送查询参数

```
# returns all documents in the bank index
GET /bank/_search?q=*&sort=account_number:asc

# using the alternative request body method
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

返回数据(部分省略)

```
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1000,
    "max_score": null,
    "hits": [
      {
        "_index": "bank",
        "_type": "account",
        "_id": "0",
        "_score": null,
        "_source": {
          "account_number": 0,
          "balance": 16623,
          "firstname": "Bradshaw",
          "lastname": "Mckenzie",
          "age": 29,
          "gender": "F",
          "address": "244 Columbus Place",
          "employer": "Euron",
          "email": "bradshawmckenzie@euron.com",
          "city": "Hobucken",
          "state": "CO"
        },
        "sort": [
          0
        ]
      },
      ...
    ]
  }
}
```

每个字段的含义根据字段名即可看出来，不赘述。其中：

- `took`的单位是毫秒
- `hits.hits`默认返回结果中的前十条记录

**需要注意的是,一旦获得返回结果，Elasticsearch 就彻底完成了请求，且不保存任何服务端资源或者状态信息，这和 SQL 里一些先得到结果子集再通过状态信息(如游标)得到剩下结果集的情况不同**


## Query DSL

介绍几个简单的查询,`size` 未指定则默认是 10,`from` 参数默认从零开始

```
# returns documents 11 through 20:
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}

# sorts the results by account balance in descending order and returns the top 10 (default size) documents
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}

# return two fields, account_number and balance
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}

# returns the account numbered 20
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

更多例子，如`must`,`should`,`bool` query 等等，请参考 [Executing Searches](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/_executing_searches.html)


## 过滤器

查询条件和文档的相关性用 `_score` 这个数值字段来表示，数值越高越相关。而有时我们不需要去计算相关性，只需要确定文档满不满足查询条件即可，这时可以用  `filter`(过滤器).

```
#  uses a bool query to return all accounts with balances between 20000 and 30000
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

在上面的例子中，返回所有 balance 在 20000～30000 的文档，所有满足条件的文档的匹配程度都是“等价”的，没有谁更相关，所以计算分数是无意义且没必要的。

## 聚合

`aggregation`(聚合)能够对数据分组和提取统计信息，大致类似 SQL 中的 `group by` 和聚合函数。

Elasticsearch 能同时分别返回查询结果和聚合结果，从而避免多次查询。

```
# 类似SQL中的 SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC

GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

更多聚合例子参考 [Executing Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/_executing_aggregations.html)
