---
layout: post
title:  Elasticsearch 5.0-配置检查
date:   2016-12-13 21:48:07 +08:00
category: 入门系列笔记
tags: [Elasticsearch]
comments: true
---

本文是 Elasticsearch 5.0 系列博文的配置检查篇,主要介绍 Elasticsearch 的 Bootstrap Checks

<!-- more -->

## 写在前面

- 本文以 Elasticsearch 5.0.1 版本为例进行讲解,不定期更新
- 该系列主要参考的 [Elasticsearch Reference: 5.0](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/index.html)，尽量避免照搬翻译，只摘录精要部分辅以简单说明
- 写这个系列博客的初衷是强迫自己梳理，同时方便一些较忙/没空耐心看英文文档的朋友快速上手，建议读者有空多读官方文档，毕竟别人写的都是二手资料
- 如需查看 ES 系列更多博文，请关注我的个人网站[@brianway](http://brianway.github.io/) 或者  [@CSDN](http://blog.csdn.net/h3243212/)



## 为什么有 Bootstrap Checks

`Bootstrap Checks` 是 Elasticsearch 5.0 新加入的，在之前的 2.x 版本是没有的。之前的版本中，错误的配置会被当成 warning 记录到日志中，但这些信息往往被用户忽视。为了保证一些重要的配置得到应有的重视，Elasticsearch 会在启动时进行 `Bootstrap Checks` .

`Bootstrap Checks` 会检查很多 Elasticsearch 和系统的配置。在**开发模式**下，所有没通过的检查都会报 warnings 并写进日志文件，即使检查没通过，依然可以启动节点运行 Elasticsearch；而在**生产模式**下，任何没通过的 `Bootstrap Checks` 都会报异常并阻止 Elasticsearch 启动。

## 开发模式 vs. 生产模式

Elasticsearch 的 HTTP 默认绑定到`localhost`，并且 transport 使用内部通信，适用于日常开发；而组成集群时，由于每个 ES 实例要可达，故 transport 必须绑定到外部接口。

一般 Elasticsearch 默认你是在开发模式下工作；一旦配置了诸如`network.host`的网络配置项，Elasticsearch会认为你处于生产环境。这是避免服务器因不良配置造成数据丢失的重要安全措施。

另外，HTTP 和 transport 可以分别通过 `http.host` 和 `transport.host`进行配置，所以配置单点实例可达时，可以用 HTTP 进行测试而无需触发生产模式。

## Bootstrap Checks

有很多检查项，以 `Heap size check`为例子，由于 Elasticsearch 是使用 Java 写的，程序在 JVM 上运行，而 JVM 的堆大小是可以配置的。如果 JVM 的起始堆大小不等于最大堆大小，那么在堆 resize 的时候很容易造成系统停滞，为了避免这种`resize pauses`,一开始就应将两者设置成相等。

类似的检查还有很多，大部分是针对 JVM 配置项的检查，有些检查项只在 Linux 系统上会检查，有些在所有平台都会检查。这里只列举出检查项，不作进一步说明了，具体每一项说明可参考 [Bootstrap Checks](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/bootstrap-checks.html)

- Heap size check
- File descriptor check
- Memory lock check
- Maximum number of threads check
- Maximum size virtual memory check
- Maximum map count check
- Client JVM check
- Use serial collector check
- OnError and OnOutOfMemoryError checks


## 重要的系统配置

从上节可知，很多`Bootstrap Checks`涉及到系统配置，我们需要对系统进行一些配置来使 Elasticsearch 可以获取更多的资源。

一般必须配置以下几条设置：

- Set JVM heap size
- Disable swapping
- Increase file descriptors
- Ensure sufficient virtual memory
- Ensure sufficient threads


在哪里配置系统设置取决于你使用的安装包以及你使用的操作系统，具体的配置方法见 [Configuring system settings](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/setting-system-settings.html)

JVM 参数建议通过 `jvm.options` 配置文件进行配置，当然，也可以通过 `ES_JAVA_OPTS` 环境变量来配置。
