---
layout: post
title:  Elasticsearch 5.0-基本操作
date:   2016-12-13 21:42:07 +08:00
category: 分布式系统
tags: [Elasticsearch]
comments: true
---

本文是 Elasticsearch 5.0 系列博文的基本操作篇,主要介绍 Elasticsearch 的 CRUD 操作

<!-- more -->

## 写在前面

- 本文以 Elasticsearch 5.0.1 版本为例进行讲解,不定期更新
- 该系列主要参考的 [Elasticsearch Reference: 5.0](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/index.html)，尽量避免照搬翻译，只摘录精要部分辅以简单说明
- 写这个系列博客的初衷是强迫自己梳理，同时方便一些较忙/没空耐心看英文文档的朋友快速上手，建议读者有空多读官方文档，毕竟别人写的都是二手资料
- 如需查看 ES 系列更多博文，请关注我的个人网站[@brianway](http://brianway.github.io/) 或者  [@CSDN](http://blog.csdn.net/h3243212/)


## 基础指令

下面是在 `Console` 中输入的一些命令，可以依次运行看看结果。

```
# Cluster Health
GET _cat/health?v
#  list of nodes
GET /_cat/nodes?v

#List All Indices
GET /_cat/indices?v

# Create an Index
PUT /customer?pretty
GET /_cat/indices?v
# Index and Query a Document
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
GET /customer/external/1?pretty
DELETE /customer/external/1

# Delete an Index
DELETE /customer?pretty
```

可以看出Elasticsearch 的 REST API基本格式：

`<REST Verb> /<Index>/<Type>/<ID>`



## 索引/替换文档

```
# Indexing/Replacing Documents
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}

# using the POST verb instead of PUT since we didn’t specify an ID
POST /customer/external?pretty
{
  "name": "Jane Doe"
}
```

- 使用 `PUT` 方法需要明确指定 ID,两次 PUT 的 id 相同则是替换之前的文档，第二次 id 不同则是创建新的文档
- 没明确指定 ID 则使用 `POST` 方法

## 更新文档

```
# Updating Documents
POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}

POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}

# uses a script to increment the age by 5
POST /customer/external/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

更新文档其实就是先删除再建一个新的文档

> Whenever we do an update, Elasticsearch deletes the old document and then indexes a new document with the update applied to it in one shot


## 删除文档

```
# Deleting Documents
DELETE /customer/external/2?pretty
```

直接删除整个 index 要比删除 index 里的所有文档要更有效率

> It is worth noting that it is much more efficient to delete a whole index instead of deleting all documents with the Delete By Query API.


## 批处理

```
# Batch Processing
POST /customer/external/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }


POST /customer/external/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

- 删除操作后面没有相应的文档数据，只提供要删除的 ID 即可
- 批处理对每个操作(action)按顺序依次执行(sequentially and in order)，如果单个操作出错，也会继续执行剩下的操作
- 批处理放回结果时，按照请求顺序为每个操作提供一个状态以便用户检查
