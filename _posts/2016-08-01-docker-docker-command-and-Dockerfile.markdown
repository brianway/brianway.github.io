---
layout: post
title:  Docker commands和Dockerfile
date:   2016-08-01 10:08:07 +08:00
category: 分布式系统
tags: Docker
comments: true
---

* content
{:toc}


本文主要对Docker commands和Dockerfile的相关知识进行整理






## Docker commands

官网传送门：

>* [Docker run reference](https://docs.docker.com/engine/reference/run/)
>* [Docker commands](https://docs.docker.com/engine/reference/commandline/)

首先，当然是配置命令自动补全，只需要把一个文件用curl下载copy到特定路径即可，具体操作参考[Command-line Completion](https://docs.docker.com/compose/completion/)

其实docker有很完备的命令帮助提示，对哪个指令不清楚，只需要在后面加`--help`就能看到帮助说明。例如：

- 输入`docker --help`可以看到所有可执行的命令。
- 随便挑一个，比如`run`命令，则输入`docker run --help`又能看到`run`的相关帮助了。

常用命令：

- 查看本地images:

```
docker images
```

- 运行image

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
//i.e.
docker run image
docker run -it image /bin/bash
```

常用的一些参数：

- `--rm`:container退出后自动删除
- `-i`和`-t`常常一起用，`-it`:以超级管理员权限打开一个命令行窗口
- `-d`: 后台运行container
- `--name`:给container命名


- 查看当前container

```
docker ps -a
```

- 删除所有状态的container

```
docker rm $(docker ps -a -q)
```

- 通过另外的tty查看已经运行的容器

```
docker exec -it ${container_id} /bin/bash
```

- 查看容器的信息

```
docker inspect ${container_id}
```


另外， 在以上指令中，容器名和容器的container_id都是可以使用的，如果用户没有指定容器名，docker会默认给每个容器分配一个比较友好的随机名称，像fervent_perlman,high_galileo等等



## Dockerfile

官网传送门：

>* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
>* [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

感觉文档里说了很全了，这里稍微提几个容易困惑的点

1.exec form vs shell form

在`CMD`和`ENTRYPOINT`都涉及到着两种形式(`CMD`多一种完全作为参数的形式)，例如:

- `CMD ["executable","param1","param2"]`(exec形式,推荐)
- `CMD command param1 param2` (shell形式)

至于两种形式的区别，官方的几点说明挺详细的，主要就是变量替换，脚本环境等方面有差别：

>* Note: If CMD is used to provide default arguments for the ENTRYPOINT instruction, both the CMD and ENTRYPOINT instructions should be specified with the JSON array format.
>* Note: The exec form is parsed as a JSON array, which means that you must use double-quotes (“) around words not single-quotes (‘).
>* Note: Unlike the shell form, the exec form does not invoke a command shell. This means that normal shell processing does not happen. For example, CMD [ "echo", "$HOME" ] will not do variable substitution on $HOME. If you want shell processing then either use the shell form or execute a shell directly, for example: CMD [ "sh", "-c", "echo $HOME" ].

2.ENTRYPOINT vs CMD

读完官方的[Understand how CMD and ENTRYPOINT interact
](https://docs.docker.com/engine/reference/builder/#/understand-how-cmd-and-entrypoint-interact)，觉得这两者特别相似，对这两者有什么区别和联系还是有些困惑，阅读下面这篇文章：

> [Dockerfile: ENTRYPOINT vs CMD](https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/)

**简而言之，ENTRYPOINT更像一个写死的可执行指令，CMD更像默认的一个可选项。**

一个image只做一个单一的用途，就像一个可执行的命令时，建议使用ENTRYPOINT，把CMD作为默认参数(第三种形式`CMD ["param1","param2"] (as default parameters to ENTRYPOINT)`)。因为一般而言，ENTRYPOINT是不被覆盖的(除非在run时显式使用--entrypoit),而CMD是defaults的选项，从前文的run命令格式`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`可知，用户可以在运行images时输入自己的COMMAND来覆盖默认的CMD。


3.ADD vs COPY

这两个好像都是把东西从host拷贝到docker的container里,官方比较如下：

> Although ADD and COPY are functionally similar, generally speaking, COPY is preferred. That’s because it’s more transparent than ADD. COPY only supports the basic copying of local files into the container, while ADD has some features (like local-only tar extraction and remote URL support) that are not immediately obvious. Consequently, the best use for ADD is local tar file auto-extraction into the image, as in ADD rootfs.tar.xz /.

简单来说，主要就两点区别：

- COPY只能把本地文件拷贝到container里；ADD还支持从远程拷贝(remote URL support)
- ADD可以自动解压本地压缩文件

官方建议用COPY(preferred)

参考链接

>* [Reference - ADD or COPY](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/add-or-copy)
>* [Stackoverflow - Docker COPY vs ADD](http://stackoverflow.com/questions/24958140/docker-copy-vs-add)



----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
