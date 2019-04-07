---
layout: post
title:  hadoop完全分布式模式的安装和配置步骤
date:   2015-12-17 22:35:11 +08:00
category: 分布式系统
tags: [Hadoop, 安装部署]
comments: true
---

本文是入门教程，以**hadoop-1.2.1**为例，介绍hadoop完全分布式的部署和配置步骤

<!-- more -->

实验条件：

>* 三台阿里云云服务器(已经配置好Java环境)
>* 一台PC机用于远程登录服务器

前排提示：

>* JAVA环境需每个服务器单独配置(注意路径一致)
>* hadoop相关配置只需配置一个master即可，其他的机子直接scp复制

----

## 配置步骤

### 0.下载解压

- 下载解压Hadoop安装包

>* 国内镜像[hadoop-1.2.1.tar.gz](http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz)
>* 官网[hadoop-1.2.1.tar.gz](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz)

下载：
`wget https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz`

解压：`tar xzvf hadoop-1.2.1.tar.gz`



### 1.配置hosts文件和hadoop-env.sh文件
- 修改/etc/host，使彼此能解析主机名

```
root@RfidLabMaster:/etc# cat hosts
127.0.0.1 localhost
127.0.1.1       localhost.localdomain   localhost
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
#10.116.155.242 iZ945z9p7yxZ
#10.116.155.242 RfidLabMaster
120.25.162.238 RfidLabMaster
120.27.138.14  RfidLabSlave1
120.27.137.211 RfidLabSlave2
```

- 进入hadoop的解压目录，编辑conf/hadoop-env.sh(版本不同，配置文件位置有所变化)

查看本机的JAVA_HOME：`env |grep JAVA_HOME`

显示：`JAVA_HOME=/usr/lib/jvm/jdk1.8.0_60`

编辑hadoop-env.sh：`vim hadoop-env.sh`

找到`export JAVA_HOME`，去掉`#`注释改为本机的
`export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_60`

### 2.ssh配置
2.1.以brian用户登录，在brian主目录下进行操作
进入root目录：
`cd /root`
生成密钥:

`ssh-keygen -t rsa`
`cd .ssh`
`cp id_rsa.pub authorized_keys`

2.2分发ssh公钥
把各个节点的authorized_keys的内容相互拷贝到对方的此文件中，即可免密码彼此ssh连入
把所有节点的authorized_keys的内容拷贝到一起形成一个大文件，再用这个新的大authorized_keys覆盖所有节点的原来的该文件。

### 3.编辑conf目录下core-site.xml,hdfs-site.xml,mapred-site.xml三个核心配置文件

- 修改core-site.xml文件
在`<configuration></configuration>`标签中添加：

```xml
<property>
 <name>fs.default.name</name>
  <value>hdfs://RfidLabMaster:9000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
<value>/home/brian/hadoopdir/tmp</value>
</property>
```

- 修改hdfs-site.xml文件
在`<configuration></configuration>`标签中添加：

```xml
<property>
<name>dfs.name.dir</name>
<value>/home/brian/hadoopdir/name</value> #hadoop的name目录路径
<description>  </description>
</property>
<property>
<name>dfs.data.dir</name>
<value>/home/brian/hadoopdir/data</value>
<description> </description>
</property>
<property>
  <name>dfs.replication</name>
  <!-- 我们的集群有两个结点，所以rep两份 -->
  <value>2</value>
</property>
```

- 修改mapred-site.xml文件
在`<configuration></configuration>`标签中添加：

```xml
<property>
  <name>mapred.job.tracker</name>
  <value>hdfs://RfidLabMaster:9001</value>
</property>
<property>
  <name>mapred.local.dir</name>
 <value>/home/brian/hadoopdir/local</value>
</property>
```

### 4.修改masters和slaves文件
conf/masters

```
RfidLabMaster
```

conf/slaves

```
RfidLabSlave1
RfidLabSlave2
```

### 5.向各个节点复制hadoop

`scp -r ./hadoop-1.2.1 RfidLabSlave1:/home/brian`

### 6.格式化分布式文件系统
在hadoop目录下输入
`bin/hadoop namenode -format`

### 7.启动守护进程
在hadoop目录下输入
`bin/start-all.sh`

----

## 结果
主节点

```
brian@RfidLabMaster:~/hadoop-1.2.1/logs$ jps
26721 JobTracker
26449 NameNode
26889 Jps
26633 SecondaryNameNode
```

从节点

```
brian@RfidLabSlave1:~$ jps
20402 Jps
20204 DataNode
20302 TaskTracker
```

查看日志文件，均无ERROR和异常


----


## 遇到的问题

1.[mater日志异常]:hadoop/hdfs/name is in an inconsistent state: storage directory(hadoop/hdfs/data/) does not exist or is not accessible

>* [stack overflow question 1.1](http://stackoverflow.com/questions/27271970/hadoop-hdfs-name-is-in-an-inconsistent-state-storage-directoryhadoop-hdfs-data)

2.[slave日志异常]:Hadoop : java.io.IOException: Call to  failed on local exception: java.io.EOFException

>* [stack overflow question 2.1](http://stackoverflow.com/questions/25130799/hadoop-java-io-ioexception-call-to-localhost-127-0-0-154310-failed-on-local)

**问题1和2:好像是忘记先格式化分布式文件系统了，[问题2]好像是[问题1]的连带问题，参看[6.格式化分布式文件系统]**

3.[slave日志异常]：ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: java.io.IOException: Incompatible namespaceIDs in

>* [stack overflow question 3.1](http://stackoverflow.com/questions/3425688/why-does-the-hadoop-incompatible-namespaceids-issue-happen)
>* [stack overflow question 3.2](http://www.hadoopinrealworld.com/fixing-java-io-ioexception-incompatible-namespaceids/)

**问题3：好像是重复格式化后ID冲突的问题，上面的两个链接有各种解决办法，什么在版本文件里改ID之类的，最简单的好像是直接删掉在文件[3.编辑conf目录下core-site.xml,hdfs-site.xml,mapred-site.xml三个核心配置文件]中涉及到的文件夹**

4.[master日志]：ERROR org.apache.hadoop.security.UserGroupInformation: PriviledgedActionException as:brian cause:java.io.IOException: File/home/brian/hadoop_dir/tmp/mapred/system/jobtracker.info could only be replicated to 0 nodes, instead of 1

>* [stack overflow question 4.1](http://stackoverflow.com/questions/15585630/file-jobtracker-info-could-only-be-replicated-to-0-nodes-instead-of-1)
>* [stack overflow question 4.2](http://stackoverflow.com/questions/5293446/hdfs-error-could-only-be-replicated-to-0-nodes-instead-of-1)

**问题4：可能我之前的配置有问题，重新配置了一遍，然后按照[问题3]里删文件解决的**

> 补充参考
[hadoop 可能遇到的错误](http://wwangcg.iteye.com/blog/1152481)
