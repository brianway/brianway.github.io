---
layout: post
title:  在intellij IDEA中为web应用创建图片虚拟目录(详细截图)
date:   2016-03-30 14:28:00 +08:00
category: web开发
tags: IntelliJ-IDEA SpringMVC
comments: true
---

* content
{:toc}

本文主要展示如何在intellij IDEA中为web应用添加虚拟目录映射,并附上步骤截图




## 工程配置和环境

我使用的版本为

- tomcat 8.0.30
- intellij 15.0.2
- jdk 1.8.0_25

已经部署好了一个web应用，并且已经在IDEA中添加好了tomcat容器，现在想为这个web应用添加一个图片虚拟目录


## 操作步骤

- 1.点击工具栏的运行配置`Edit Configurations`

![Edit Configurations](http://7xph6d.com1.z0.glb.clouddn.com/IDEA_web-%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%9501.png)


- 2.在弹出的`Run/debug Configurations`中选中tomcat容器，选择`deployment`这个tab

![deployment](http://7xph6d.com1.z0.glb.clouddn.com/IDEA_web-%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%9502.png)

- 3.添加物理目录和并设置虚拟目录路径

![添加物理目录和并设置虚拟目录路径](http://7xph6d.com1.z0.glb.clouddn.com/IDEA_web-%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%9503.png)

这里我选择了D盘下面的tmp文件夹作为物理目录，虚拟目录设为了`/pic`,我试了下，虽然斜杠少了也没什么影响，一样能访问，不过还是建议加上吧。

- 4.运行web应用，访问图片资源

附上博主帅照一张

![访问图片资源](http://7xph6d.com1.z0.glb.clouddn.com/IDEA_web-%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%9504.png)


这里需要接上具体访问资源的文件名，不然后访问不到的，如下图

![访问不到](http://7xph6d.com1.z0.glb.clouddn.com/IDEA_web-%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%9505.png)


## 在非IDE环境下配置虚拟目录

怎么为tomcat配置虚拟目录映射可以参考下面的博客：

> [tomcat配置虚拟目录映射](http://blog.csdn.net/xiazdong/article/details/7215052)


