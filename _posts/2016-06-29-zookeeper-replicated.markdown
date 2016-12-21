---
layout: post
title:  zookeeper单机伪集群配置
date:   2016-06-29 20:36:11 +08:00
category: 分布式系统
tags: ZooKeeper 安装部署
comments: true
---

* content
{:toc}


本对zookeeper做简单介绍，分享查阅时搜集的一些好的链接，并以最新的稳定版zookeeper-3.4.8为例，对单机模式和伪分布式的部署步骤做记录和说明。






## zookeeper简介

Zookeeper 分布式服务框架是曾Apache Hadoop的一个子项目，现在是一个独立的顶级项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。

通读下面几个链接，可以对zookeeper有个大致了解

- 简单介绍：[Apache ZooKeeper - 维基百科](https://zh.wikipedia.org/wiki/Apache_ZooKeeper)
- 项目主页：[Apache ZooKeeper - Home](http://zookeeper.apache.org/)
- IBM developerWorks:[分布式服务框架 Zookeeper -- 管理分布式环境中的数据](https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/)
- 文档outline:[ZooKeeper 3.4 Documentation](https://zookeeper.apache.org/doc/trunk/s)



## 单机单server运行

基本上照着[ZooKeeper Getting Started Guide](https://zookeeper.apache.org/doc/trunk/zookeeperStarted.html)走一遍就OK了，这里简单描述下步骤

0. 环境要求:各种系统都行，java １.6＋
1. 下载压缩包,解压
2. 把解压目录下`conf/zoo_sample.cfg`复制一份在同目录下，重命名为`zoo.cfg`,`dataDir`属性可设置成别的
3. 执行解压目录下的`bin/zkServer.sh start`开启zookeeper
4. 执行解压目录下的`bin/zkCli.sh -server 127.0.0.1:2181`连接zookeeper

这时可以看到连接成功和一些欢迎信息，如果日志选项没改的话，默认是`INFO`级别，所以会在控制台看到一些日志输出，至此，已经运行成功，可以输入`help`查看帮助命令，试着玩一玩`ls,create,get,set,delete`等命令体验下


## 伪分布式

官网对分布式讲的不是很详细，这里简单记录一下

大体流程就是把压缩包解压三份，每份单独配置`conf/zoo.cfg`,并在`dataDir`对应的目录下添加一个只含数字的文本文件`myid`表明自己是哪台服务器。

- 部署规模为3的单机伪机群

以我的电脑为例，我新建了一个根目录`zookeeper`,并在该目录下分别新建了三个文件夹：`server0`,`server1`,`server2`,然后在每个文件夹解压zookeeper的压缩包，并另外新建`data`，`logs`文件夹来分别存放数据和日志,目录结构如下：

```
.
├── server0
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.8
├── server1
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.8
└── server2
├── data
├── logs
└── zookeeper-3.4.8
```

然后在每个data目录下创建一个myid的文件(另外两个文件是运行后自动生成的，开始没有)，里面写入一个数字，这个数字和配置文件里的一致

```
mi@mi-OptiPlex-9020:~/MyPrograms/zookeeper/server0/data$ cat
myid                  version-2/            zookeeper_server.pid
mi@mi-OptiPlex-9020:~/MyPrograms/zookeeper/server0/data$ cat myid
0
```

配置`conf/zoo.cfg`,比如我的`server0`目录下的配置文件，其他几个类似：


```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/home/mi/MyPrograms/zookeeper/server0/data
dataLogDir=/home/mi/MyPrograms/zookeeper/server0/logs
# the port at which the clients will connect
clientPort=2180

## 单机集群
server.0=127.0.0.1:2880:3880
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
```


`server.A=B：C：D`:其中 A 是一个数字，就是myid里的那个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址,C和D是两个端口。

两个端口的作用，官网描述如下：

> Finally, note the two port numbers after each server name: " 2888" and "3888". Peers use the former port to connect to other peers. Such a connection is necessary so that peers can communicate, for example, to agree upon the order of updates. More specifically, a ZooKeeper server uses this port to connect followers to the leader. When a new leader arises, a follower opens a TCP connection to the leader using this port. Because the default leader election also uses TCP, we currently require another port for leader election. This is the second port in the server entry.

简单来说，第一个端口用来集群成员的信息交换以及与集群中的Leader 服务器交换信息，第二个端口是在leader挂掉时专门用来进行选举leader所用。

因为是伪分布式，所以`dataDir`,`clientPort`也不一样，同时C,D两个端口也不能相同。

- 启动ZooKeeper伪机群的所有服务器

分别进入三个服务器文件夹的解压目录的/bin目录下，启动服务：

```
bin/zkServer.sh start
```


- 接入客户端

进入解压目录的/bin目录下(3个server中任意一个)，连接任意一个服务器,比如我就是进入了server2的目录下，连接的server0

```
bin/zkCli.sh  -server 127.0.0.1:2180
```

运行截图

![zookeeper_单机集群运行截图](http://7xph6d.com1.z0.glb.clouddn.com/zookeeper_%E5%8D%95%E6%9C%BA%E9%9B%86%E7%BE%A4%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE-1.png)


## 参考链接

>* [Apache ZooKeeper - 维基百科](https://zh.wikipedia.org/wiki/Apache_ZooKeeper)
>* [Apache ZooKeeper - Home](http://zookeeper.apache.org/)
>* [分布式服务框架 Zookeeper -- 管理分布式环境中的数据](https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/)
>* [ZooKeeper 3.4 Documentation](https://zookeeper.apache.org/doc/trunk/s)
>* [zookeeper 单机伪集群配置](http://blog.csdn.net/xymyeah/article/details/6320668)
>* [zookeeper入门（1）在单机上实现ZooKeeper伪机群/伪集群部署](http://blog.csdn.net/poechant/article/details/6633923)




----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
