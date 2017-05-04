---
layout: post
title:  docker入门概览
date:   2016-07-29 21:55:11 +08:00
category: 分布式系统
tags: Docker 安装部署
comments: true
---

* content
{:toc}


本文对docker进行大致介绍，包括概述,安装,简单使用,架构,基本原理等方面







## 写在前面

- 本文是自己学习docker的一个记录和整理,啃英文文档挺吃力的,懒得翻译，所以写这篇类似“索引”的文章,希望能帮助和我一样的新手快速入门
- 本文主要参考[官方文档(Docker Document)](https://docs.docker.com/)和相关技术博客
- 如果有理解有误的地方还望不吝指正


## 概述

### 什么是Docker?

可以参考下面三篇文章。从我使用的感受来看，我觉得Docker就是一个应用打包工具，把写好的应用用docker打包发布，然后别人就可以直接部署使用了，特别方便。

- [Docker是什么](http://www.docker.org.cn/book/docker/what-is-docker-16.html)
- [理解Docker](http://www.jianshu.com/p/a75ddf6915e0)
- [What is Docker？(译文)](http://dockone.io/article/1534)



### 什么是Docker Engine?

Docker Engine is a **client-server application** with these major components:

- A **server** which is a type of long-running program called a daemon process.
- A **REST API** which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.
- A command line interface **(CLI) client**.

我觉得官网的解释很言简意赅，附上图(摘自官网)

![Docker Engine](http://7xph6d.com1.z0.glb.clouddn.com/docker_docker-engine.png)

### Docker的用处

- Faster delivery of your applications
- Deploying and scaling more easily
- Achieving higher density and running more workloads


## 安装

安装参考[Install Docker Engine](https://docs.docker.com/engine/installation/#installation)

### Ubuntu

以ubuntu 14.04 为例，参考[Installation on Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntulinux/)安装Docker engine

这里列出重要的步骤：

1. 更新apt源，包括添加证书，密钥等
2. 用sudo apt-get安装
3. 进一步配置，主要是创建docker用户组

注 ：如果输入`docker info`出问题，多半是权限问题，以sudo运行试试

### Mac OS X

Mac下安装参考[Installation on Mac OS X](https://docs.docker.com/engine/installation/mac/)

Mac下有两种安装方式供选

- Docker for Mac : Mac的原生应用，没有使用虚拟机(VirtualBox)，而是使用的HyperKit
- Docker Toolbox : 会安装虚拟机，使用docker-machine来运行boot2docker 和Docker Engine

两者的区别请参考 [Docker for Mac vs. Docker Toolbox](https://docs.docker.com/docker-for-mac/docker-toolbox/)

## 演示

先不多说，跑起来体验下。具体的步骤和指令在[Docker简明教程](http://blog.saymagic.cn/2015/06/01/learning-docker.html#bqlkp)这篇文章已经写得很清楚了，在此不再赘述


## 架构和原理

![dokcer architecture](http://7xph6d.com1.z0.glb.clouddn.com/dokcer_architecture.png)

由上图可知，docker是一个client-server架构

- The Docker daemon : 运行在主机上
- The Docker client : 用户和dokcer daemon交互的接口

docker的内部主要有三种资源/组件

- docker images : **build** component,只可读
- docker registries : **distribution** component,images共享库
- docker containers : **run** component


这里重点说说images and containers

Docker使用[union file systems](https://en.wikipedia.org/wiki/UnionFS) 把不同的层(layer)做整合成单一的image. Union File System的中一种是AUFS,可以参考[这篇博文](http://coolshell.cn/articles/17061.html)

[官网文档](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)对image的layers是这么描述的

> Each Docker image references a list of **read-only** layers that represent filesystem differences. Layers are stacked on top of each other to form a base for a container’s root filesystem

新版docker(version>=1.10)的存储模型有变化

> Previously, image and layer data was referenced and stored using a randomly generated UUID. In the new model this is replaced by a secure content hash.

![container-based-on-ubuntu15.04](http://7xph6d.com1.z0.glb.clouddn.com/docker_container-based-on-ubuntu.png)

而container和image的主要区别就在于**top writable layer**，所有对image的更改都保存在这一层。换句话说，多个container可以共享同一个image，这就大大节省了空间。实现image和container的管理有两个关键的技术：stackable image layers 和 copy-on-write (CoW).

![multiple containers](http://7xph6d.com1.z0.glb.clouddn.com/docker_multiple-containers.png)

从图中可以看出，copy-on-write (CoW)是一个很好的策略，既节省了空间，又避免了因数据共享带来的写冲突问题，从而提高效率。


## 结语

本文主要对docker做简单介绍，对于一些更详细的知识，如docker file,volume,network,docker compose等等，会另写文章进行介绍。至于很具体的操作指令，比如怎么安装，怎么build image和run container来跑一个简单的docker hello world，请参考官方文档Docker Engine部分的“get started with docker”或者"learn by example",也可参考文末的一些参考资料


## 参考资料

>* [Docker Documentation](https://docs.docker.com/)(官方文档)
>* [Docker入门教程](http://dockone.io/article/111)(系列)
>* [Docker简明教程](http://blog.saymagic.cn/2015/06/01/learning-docker.html#bqlkp)（使用演示）
>* [docker中文](http://www.docker.org.cn/book/docker/what-is-docker-16.html)（系列)
>* [docker资源](http://www.docker.org.cn/page/resources.html)
>* [docker-从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)(gitbook)

