---
layout: post
title:  Elasticsearch 5.0-基础概念
date:   2016-12-13 21:40:07 +08:00
category: 分布式系统
tags: Elasticsearch
comments: true
---

* content
{:toc}

本文是 Elasticsearch 5.0 系列博文的基础概念篇，主要介绍集群，节点，索引，类型，文档，分片，副本等基础概念






## 写在前面

- 本文以 Elasticsearch 5.0.1 版本为例进行讲解,不定期更新
- 该系列主要参考的 [Elasticsearch Reference: 5.0](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/index.html)，尽量避免照搬翻译，只摘录精要部分辅以简单说明
- 写这个系列博客的初衷是强迫自己梳理，同时方便一些较忙/没空耐心看英文文档的朋友快速上手，建议读者有空多读官方文档，毕竟别人写的都是二手资料
- 如需查看 ES 系列更多博文，请关注我的个人网站[@brianway](http://brianway.github.io/) 或者  [@CSDN](http://blog.csdn.net/h3243212/)

------

## 基本概念

有关概念在[Basic Concepts](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html)中已经介绍的很详细了，这里简单说一下。

- 集群(cluster)：集群由一个或者多个节点组成，由名称唯一标识
- 节点(node)：一个单独的 Elasticsearch 实例
- 索引(index)：文档的集合
- 类型(type)：文档的逻辑分类/分区
- 文档(document)：能够被索引的信息基础单元
- 分片(shard)：索引的物理分区，是一个最小的 Lucene 索引单元。分为 primary shard(主分片) 和 replica shard(简称replicas)。
- 副本/备份(replicas)：主分片的备份


下面就这几个概念进一步说明

## 类比关系型数据库

其中索引,类型,文档的概念可以类比关系型数据库

|Elasticsearch|关系型数据库|
|:---|:----|
|索引(index)|数据库(database)|
|类型(type)|表(table)|
|文档(document)|行记录(row)|
|字段(field)|列(column)|


## 为什么有shard和replica

为什么有 shard?

- 可以水平切分和扩展内容容量
- 在shards 间分发和并行执行操作，从而提供性能和吞吐量

为什么有replica?

- 当 shard 失效时提供高可用性。因为这个原因，一个primary shard的replica不会分配到和该shard所处的同一节点
- 扩展查询的容量/吞吐量，因为查询操作是一个读操作，可以在所有replica上并行执行



## 其他补充


Elasticsearch 默认为每个 index 创建 5 个主分片，且备份数为 1。也就是说，每个索引由 5 个主分片组成，并且每个分片拥有一个备份。需要注意的是，主分片的数量一旦确定，之后是不能更改的（除非重新建立索引），而 replicas 的数量可以在之后随时更改。


所以在上一篇文章中，我们启动 Kibana 后在 `Consonle` 查询索引状态`GET /_cat/indices?v`，会发现 `health` 是 `yellow` 而不是 `green`，就是因为我们只开启了一个节点，而且 Kibana 启动后在 Elasticsearch 中建立了一个默认索引 `.kibana`，该索引只有 1 个主分片和一个副本，故 shard 都在该节点上，而 shard 的副本不能和该 shard 分配在同一节点，故未生效，从而导致状态是黄色。


另外，每个索引被分配到多个分片，但 `number_of_shards` 的值只适用于索引，而不是整个集群。这个值指定了每个索引的分片数，而非整个集群中的全部主分片数。（摘自[Optimizing Elasticsearch: How Many Shards per Index?](https://qbox.io/blog/optimizing-elasticsearch-how-many-shards-per-index)）




----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
