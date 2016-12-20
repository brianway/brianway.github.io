---
layout: post
title:  爬取知乎60万用户信息之后的简单分析
date:   2016-12-20 21:03:07 +08:00
category: project
tags: Elasticsearch Kibana webporter
comments: true
---

* content
{:toc}

使用 Java+Elasticsearch+Kibana 爬取了知乎 60 万用户数据，做了简单的可视化分析。







---

项目源码 [GitHub - webporter](https://github.com/brianway/webporter)

## 动机

在知乎上看到有个叫 @路人甲 的大神每隔一段时间就爬爬豆瓣/B站等等网站，做了很多有意思的分析，加上之前因为实验室项目接触过 Nutch，浅尝辄止了，所以一直想好好玩玩爬虫。

网上 Python 的爬虫教程很多，而自己的主语言是 Java，本着宣传 Java，以练促学的目的，我使用 Java 爬取了知乎 60 万用户信息，主要想看看知乎上妹子多不多啊/是不是都是基佬啊，标配常青藤/年薪百万是不是真的啊，等等。


## 思路

为了保证数据的质量，避免爬到一些僵尸号什么的，我选择爬取关注列表而非粉丝列表。我随机挑选了一位粉丝过千的优秀回答者作为起始，爬取他的关注列表，再对列表中的每个人爬取其关注列表，以此类推……


下载了大概 7 个小时，爬了 40 多万用户的关注列表，拿到了 10G 的数据，如图所示：

![爬取10G数据](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E4%B8%8B%E8%BD%BD%E7%9A%84%E7%9F%A5%E4%B9%8E%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E9%87%8F.png)

理论上有 800 多万用户，可惜有很多重复的，去重后将数据导入 Elasticsearch，得到 60+ 万用户数据:

![ES索引](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E5%9C%A8ES%E7%B4%A2%E5%BC%95%E7%8A%B6%E6%80%81.jpg)

## 数据验证

接下来简单看看下载下来的数据靠不靠谱，随手在知乎和我的 Kibana 分别搜了下轮子哥 @vczh

![搜索轮子哥](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E6%90%9C%E7%B4%A2%E8%BD%AE%E5%AD%90%E5%93%A5.png)

![搜索轮子哥](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-kibana%E6%90%9C%E7%B4%A2%E8%BD%AE%E5%AD%90%E5%93%A5.png)

可以看到，连同名的都搜出来是一样的，数据没啥问题。

## 关心的数据

然后使用 Elastichearch 的聚合查询配合 Kibana 对数据进行可视化展示，我主要分析了下面几个问题：

- 性别分布
- 粉丝最多的用户top10
- 员工最多的公司top10
- 校友最多的学校top10
- 人数最多的地方top10
- top10行业分布
- top10职业分布

**图中涉及性别的， 1 表示男，0 表示女，-1 表示不男不女**

### 性别分布

![性别分布](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E6%80%A7%E5%88%AB%E5%88%86%E5%B8%83.png)

可以看到知乎男性人数过半了，比女性和未知性别加起来都多。

### 粉丝最多的用户top10

![粉丝最多的用户top10](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E7%B2%89%E4%B8%9D%E6%9C%80%E5%A4%9A%E7%9A%84%E7%94%A8%E6%88%B7top10.png)

粉丝数前 10 的依次是 @张佳玮，@李开复，@黄继新，@周源，@yolfilm，@张亮，@张小北，@李淼，@葛巾，@采铜。最多的 120 万粉丝，第十也过 60 万了。不过前十里好几个都是知乎员工，有黑幕的嫌疑吧？


### 员工最多的公司top10

![员工最多的公司top10](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E5%91%98%E5%B7%A5%E6%9C%80%E5%A4%9A%E7%9A%84%E5%85%AC%E5%8F%B8top10.png)

可以看到 BAT 全部上榜了（乱入了一个学生什么鬼？），仅接着是网易,华为,谷歌,微软,美团。都是牛逼哄哄的互联网相关企业，看来国企和实体企业比较低调，不在知乎填公司信息啊。

另外华为的男女比简直不能看啊，妹子那么少，想去华为的单身狗们需要好好考虑一下了。


### 校友最多的学校top10

![校友最多的学校top10](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E6%A0%A1%E5%8F%8B%E6%9C%80%E5%A4%9A%E7%9A%84%E5%AD%A6%E6%A0%A1top10.png)

差强人意，校友人数排名前十的全特么是 985 啊，清北复交浙全部上榜，俨然中国大学排行榜。看来知乎标配不是常青藤，而是 985 嘛。另外可以看到，我科(倒数第三个)的男女比在这几个里面确实感人，难怪我现在还单身...


### 人数最多的地方top10

![人数最多的地方top10](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-%E4%BA%BA%E6%95%B0%E6%9C%80%E5%A4%9A%E7%9A%84%E5%9C%B0%E6%96%B9top10.png)

北京独领风骚，上海紧随其后。另外知乎居然把深圳和广州根据有没有“市”标记为了两个城市，简直坑爹，我也懒得二次处理了。综合来看，北上广深杭，主要集中在这五个城市，基本也是我国互联网企业分布最多的几个城市。

### top10行业分布

![top10行业分布](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-top10%E8%A1%8C%E4%B8%9A%E5%88%86%E5%B8%83.png)

可以看到，互联网和计算机软件两个加起来就占了半数以上，要是算上电子商务和电子游戏等基本是程序员的天下了，所以知乎上程序员偏多，IT 从业者占主流啊。

另外互联网的男女比大概 2:1 的样子吧，法律，信息传媒和创意艺术的男女比比较均衡，大概五五开。


### top10职业分布

![top10职业分布](http://7xph6d.com1.z0.glb.clouddn.com/webporter_%E7%9F%A5%E4%B9%8E-top10%E8%81%8C%E4%B8%9A%E5%88%86%E5%B8%83.png)

将近四分之一是产品经理，创始人和 CEO 也不少，比工程师还多，学生也占一定比例。另外除了运营和编辑的男女比差不多，其它都是男多女少啊。

## 结语

从这 60 万用户数据可以看出，知乎的主要群体是程序员和学生，平均学历 985 不是黑，是真的！虽然知乎用户远不止 60 万，这些数据分析出来的结果可能有些偏差，但应该也能说明一些问题吧。

最后按照国际惯例，附上源码，[GitHub - webporter](https://github.com/brianway/webporter)



----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
