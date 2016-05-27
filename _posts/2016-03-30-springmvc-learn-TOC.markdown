---
layout: post
title:  springmvc+mybatis学习笔记(汇总)
date:   2016-03-30 17:40:00 +08:00
category: web开发
tags: springmvc 总结
comments: true
---

* content
{:toc}

笔记分为两大部分：mybatis和springmvc

- [mybatis](http://blog.csdn.net/h3243212/article/category/6110387)
- [springmvc](http://blog.csdn.net/h3243212/article/category/6110387)

笔记内容主要是mybatis和springmvc的一些基本概念和使用方法，涉及概念介绍、环境搭建、编程细节、运行调试等方面。

这套笔记整体偏入门和应用，适合快速上手，对底层实现和机理并未做过多分析。我后续会研读spring源码，并把学习的收获写成博客分享出来，根据情况再开一个仓库或者贴博客链接。





-----

github:

- [springmvc-mybatis-learning](https://github.com/brianway/springmvc-mybatis-learning)
- git-clone:`git@github.com:brianway/springmvc-mybatis-learning.git`



**如果觉得不错，请先在这个仓库上点个star吧**，这也是对我的肯定和鼓励，谢谢了。不定时进行调整和补充，需要关注更新的请 Watch、Star、Fork




-----

# 目录

  - [mybatis](/mybatis)
    - [mybatis学习笔记(1)-对原生jdbc程序中的问题总结.md](http://blog.csdn.net/h3243212/article/details/50756617)
    - [mybatis学习笔记(2)-mybatis概述.md](http://blog.csdn.net/h3243212/article/details/50756622)
    - [mybatis学习笔记(3)-入门程序一.md](http://blog.csdn.net/h3243212/article/details/50756631)
    - [mybatis学习笔记(3)-入门程序二.md](http://blog.csdn.net/h3243212/article/details/50756635)
    - [mybatis学习笔记(4)-开发dao方法.md](http://blog.csdn.net/h3243212/article/details/50756808)
    - [mybatis学习笔记(5)-配置文件.md](http://blog.csdn.net/h3243212/article/details/50759845)
    - [mybatis学习笔记(6)-输入映射.md](http://blog.csdn.net/h3243212/article/details/50765375)
    - [mybatis学习笔记(7)-输出映射.md](http://blog.csdn.net/h3243212/article/details/50765422)
    - [mybatis学习笔记(8)-动态sql.md](http://blog.csdn.net/h3243212/article/details/50766105)
    - [mybatis学习笔记(9)-订单商品数据模型分析.md](http://blog.csdn.net/h3243212/article/details/50770013)
    - [mybatis学习笔记(10)-一对一查询.md](http://blog.csdn.net/h3243212/article/details/50770023)
    - [mybatis学习笔记(11)-一对多查询.md](http://blog.csdn.net/h3243212/article/details/50770026)
    - [mybatis学习笔记(12)-多对多查询.md](http://blog.csdn.net/h3243212/article/details/50770032)
    - [mybatis学习笔记(13)-延迟加载.md](http://blog.csdn.net/h3243212/article/details/50770050)
    - [mybatis学习笔记(14)-查询缓存之一级缓存.md](http://blog.csdn.net/h3243212/article/details/50774921)
    - [mybatis学习笔记(15)-查询缓存之二级缓存.md](http://blog.csdn.net/h3243212/article/details/50778927)
    - [mybatis学习笔记(16)-mybatis整合ehcache.md](http://blog.csdn.net/h3243212/article/details/50778933)
    - [mybatis学习笔记(17)-spring和mybatis整合.md](http://blog.csdn.net/h3243212/article/details/50778934)
    - [mybatis学习笔记(18)-mybatis逆向工程.md](http://blog.csdn.net/h3243212/article/details/50778937)
  - [springmvc](/springmvc)
    - [springmvc学习笔记(1)-框架原理和入门配置.md](http://blog.csdn.net/h3243212/article/details/50828141)
    - [springmvc学习笔记(2)-非注解的处理器映射器和适配器.md](http://blog.csdn.net/h3243212/article/details/50829777)   
    - [springmvc学习笔记(3)-注解的处理器映射器和适配器.md](http://blog.csdn.net/h3243212/article/details/50834272)
    - [springmvc学习笔记(4)-前端控制器.md](http://blog.csdn.net/h3243212/article/details/50834276)
    - [springmvc学习笔记(5)-入门程序小结.md](http://blog.csdn.net/h3243212/article/details/50834278)
    - [springmvc学习笔记(6)-springmvc整合mybatis(IDEA中通过maven构建).md](http://blog.csdn.net/h3243212/article/details/50837187)
    - [springmvc学习笔记(7)-springmvc整合mybatis之mapper.md](http://blog.csdn.net/h3243212/article/details/50837878)
    - [springmvc学习笔记(8)-springmvc整合mybatis之service.md](http://blog.csdn.net/h3243212/article/details/50843840)
    - [springmvc学习笔记(9)-springmvc整合mybatis之controller.md](http://blog.csdn.net/h3243212/article/details/50845546)
    - [springmvc学习笔记(10)-springmvc注解开发之商品修改功能.md](http://blog.csdn.net/h3243212/article/details/50845549)
    - [springmvc学习笔记(11)-springmvc注解开发之简单参数绑定.md](http://blog.csdn.net/h3243212/article/details/50854748)
    - [springmvc学习笔记(12)-springmvc注解开发之包装类型参数绑定.md](http://blog.csdn.net/h3243212/article/details/50854757)
    - [springmvc学习笔记(13)-springmvc注解开发之集合类型参数绑定.md](http://blog.csdn.net/h3243212/article/details/50854765)
    - [springmvc学习笔记(14)-springmvc校验.md](http://blog.csdn.net/h3243212/article/details/50864737)
    - [springmvc学习笔记(15)-数据回显.md](http://blog.csdn.net/h3243212/article/details/50864744)
    - [springmvc学习笔记(16)-异常处理器.md](http://blog.csdn.net/h3243212/article/details/50864745)
    - [springmvc学习笔记(17)-上传图片.md](http://blog.csdn.net/h3243212/article/details/50885274)
    - [springmvc学习笔记(18)-json数据交互.md](http://blog.csdn.net/h3243212/article/details/50885288)
    - [springmvc学习笔记(19)-RESTful支持.md](http://blog.csdn.net/h3243212/article/details/50885293)
    - [springmvc学习笔记(20)-拦截器.md](http://blog.csdn.net/h3243212/article/details/50894887)
    - [springmvc学习笔记(21)-springmvc整合mybatis遇到的问题及解决小结.md](http://blog.csdn.net/h3243212/article/details/50894901)
    - [springmvc学习笔记(22)-springmvc开发小结.md](http://blog.csdn.net/h3243212/article/details/50894913)


-----


# sourcecode说明

该文件下是涉及到的源码，其中mybatis部分都是直接新建的web工程，springmvc部分都是使用maven构建的。

我使用的IDE是intellij IDEA 15.0.2,以下每个子文件夹对应一个project。

- [mybatis](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/mybatis):mybatis部分前16篇笔记用到的源码
- [mybatis-spring](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/mybatis-spring):mybatis部分笔记(17)对应的源码
- [mybatis-generator](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/mybatis-generator):逆向工程的源码
- [springmvcfirst](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/springmvcfirst):springmvc部分前两篇笔记对应的非注解方式配置的源码
- [springmvcsecond](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/springmvcsecond):springmvc部分前几篇笔记对应的注解方式配置的源码
- [**learnssm-firstssm**](https://github.com/brianway/springmvc-mybatis-learning/tree/master/sourcecode/learnssm-firstssm):核心代码，springmvc和mybatis整合部分的笔记几乎所有的源码


-----

# 联系作者

- [Brian's Personal Website](http://brianway.github.io/)
- [oschina](http://my.oschina.net/brianway)
- [CSDN](http://blog.csdn.net/h3243212/)


-----

**All Copyright Reserved**



