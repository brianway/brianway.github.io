---
layout: post
title:  Elasticsearch 5.0-安装使用
date:   2016-12-13 21:35:07 +08:00
category: 分布式系统
tags: [Elasticsearch, 安装部署]
comments: true
---

本文是 Elasticsearch 5.0  系列博文的安装使用篇,主要介绍如何安装并运行 Elasticsearch，顺带介绍 Kibana 的安装

<!-- more -->

## 写在前面

- 本文以 Elasticsearch 5.0.1 版本为例进行讲解,不定期更新
- 该系列主要参考的 [Elasticsearch Reference: 5.0](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/index.html)，尽量避免照搬翻译，只摘录精要部分辅以简单说明
- 写这个系列博客的初衷是强迫自己梳理，同时方便一些较忙/没空耐心看英文文档的朋友快速上手，建议读者有空多读官方文档，毕竟别人写的都是二手资料
- 如需查看 ES 系列更多博文，请关注我的个人网站[@brianway](http://brianway.github.io/) 或者  [@CSDN](http://blog.csdn.net/h3243212/)


## 下载安装

参考[install Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/zip-targz.html)即可，这里简单展示一下

- 前台运行

ES 默认运行在前台，日志打印到标准输出

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.1.tar.gz
sha1sum elasticsearch-5.0.1.tar.gz
tar -xzf elasticsearch-5.0.1.tar.gz
cd elasticsearch-5.0.1/
./bin/elasticsearch
```

在终端输入`curl -XGET 'localhost:9200/?pretty'`

得到响应：

```json
{
  "name": "Dp0oq_v",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "6rLSu0JMTlq_YJqyhWS_xQ",
  "version": {
    "number": "5.0.1",
    "build_hash": "080bb47",
    "build_date": "2016-11-11T22:08:49.812Z",
    "build_snapshot": false,
    "lucene_version": "6.2.1"
  },
  "tagline": "You Know, for Search"
}
```

- 后台运行

参考 [Running as a daemon](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/zip-targz.html#setup-installation-daemon)

日志信息在`$ES_HOME/logs/`文件夹

## 文件夹结构

这里我使用的是`.zip`和`.tar.gz`的包直接解压得到的，文件夹的目录结构：[Directory layout of .zip and .tar.gz archives](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/zip-targz.html#zip-targz-layout)

## 重要配置

- `path.data`和`path.logs`:使用`.zip`和`.tar.gz`的 Elasticsearch 的话，这两个路径是`$ES_HOME`的子文件夹，升级 Elasticsearch 时有被删除的风险，所以需要另外指定路径，`path.data`可指定多路径
- `cluster.name`:节点只能加进`cluster.name`相同的集群中，默认名是`elasticsearch`
- `node.name`:Elasticsearch 默认采用随机 UUID 的前 7 位字符作为节点id，且节点id一直保存，节点重启并不会改变节点名。
- `bootstrap.memory_lock`:JVM 不被交换到硬盘对于节点健康很重要，一种实现方式是将`bootstrap.memory_lock`设置成`true`
- `network.host`:Elasticsearch 默认只绑定 loopback 地址(`127.0.0.1`和`[::1]`),多节点在一个 server 上启动也是可行的，生产环境下不建议罢了。
- `discovery.zen.ping.unicast.hosts`:同一个 server 上的节点将扫描端口号 9300 到 9305 来尝试连接其他该 server 上的节点。和其他 server 节点组成集群时，需要配置该项。端口默认 9300,域名对应多IP的话会尝试所有解析出来的 IP
- `discovery.zen.minimum_master_nodes`:不设置的话可能出现`split brain`问题，造成数据丢失。为了避免这样，该项设置为`(master_eligible_nodes / 2) + 1`


## Kibana 安装

由于老在终端里使用 curl 命令很不方便，所以顺带安装了一下 [Kibana](https://www.elastic.co/guide/en/kibana/5.0/introduction.html)。简单介绍下 Kinaba，它是一个配合 Elasticsearch 工作的分析和可视化平台，一些和 Elasticsearch 通过 REST API 交互的请求可以在这里面比较方便的输入和回显。

我是在 Mac 下安装的，下载好安装好解压就行了。其他系统参考 [Installing Kibana](https://www.elastic.co/guide/en/kibana/5.0/install.html)

```
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-5.0.1-darwin-x86_64.tar.gz
tar -xzf kibana-5.0.1-darwin-x86_64.tar.gz
```

先启动 Elasticsearch，再启动 Kibana

```
cd kibana-5.0.1-darwin-x86_64.tar.gz
bin/kibana
```

默认的访问网址是`localhost:5601`，在浏览器访问即可。然后点击侧栏的 `Dev Tools`就行了。顺带提一句，在之前的版本中，这个窗口是一个叫做 `Sense` 的插件的功能，在 5.0 版本中默认和 Kibana 集成了，并改名为 `Console`。

在左边输入 `GET /`，点绿色的播放按钮发送请求，就可以看到和刚才在终端里输入`curl -XGET 'localhost:9200/?pretty'`同样的的响应了。如图所示：

![Kibana - Console界面](http://7xph6d.com1.z0.glb.clouddn.com/Kibana_Console%E7%95%8C%E9%9D%A2.png)

后面的Elasticsearch的文章演示部分都会基于 `Console - Kibana` ，请安装好。
