---
layout: post
title:  使用GitHub+Hexo搭建个人网站
date:   2019-04-14 20:40:07 +08:00
category: 其他
tags: [GitHub, 安装部署]
comments: true
---


本文主要记录我将 GitHub 的个人博客从 Jekyll 迁移到 Hexo 的一些踩坑实践。

<!-- more -->

## 为什么迁移

- Jekyll搭建博客所需的技术栈（如Liquid语言、Ruby、gem等）我不太熟悉，每次想改点啥都不方便；而 Hexo 是基于Node.js的，js相对熟悉一点，定制起来方便一些。
- Hexo 生成站点的速度更快，且该框架是台湾人贡献的，中文文档很多。

> 网上对比的文章 [How to Choose the Right Static Generator: Jekyll vs. Hugo vs. Hexo](https://www.techiediaries.com/jekyll-hugo-hexo/)


## Hexo和博客主题

### Hexo 简介

Hexo的安装可以参考[Hexo官网](https://hexo.io/)，需要先安装node.js和git，照着文档安装就行了。

常用指令有：

- 清除缓存： `hexo clean`
- 生成静态文件：`hexo generate`，简写`hexo g`
- 启动服务器：`hexo server`，简写`hexo s`
- 部署到远程站点： `hexo deploy`，简写`hexo d`

> 更多指令：https://hexo.io/zh-cn/docs/commands

### 主题选择

Hexo的主题有很多，可以从下面几篇推荐里挑挑：

- [知乎 - 在github中有哪些好的hexo博客模板？](https://www.zhihu.com/question/39388850)
- [Hexo博客主题推荐](https://www.jianshu.com/p/bcdbe7347c8d)

我主要有几个需求：

- **支持搜索**
- **目录显示当前所处章节**：阅读文章时显示目录TOC，且滑动到文章相应位置时，目录同步定位
- 支持赞赏展示和自定义页面
- 支持标签和分类

简单比较了下，选择了[indigo](https://github.com/yscoder/hexo-theme-indigo) 


## 搭建博客

大致以下几个步骤：

1. 安装Hexo
2. 选用主题，并根据主题的安装步骤去更改配置项
3. 本地`hexo clean & hexo g`生成本地文件，`hexo s`启动本地服务器，访问`http://localhost:4000/`查看效果


可以通读以下博客了解细节：

- [基于hexo+github搭建一个独立博客](http://www.cnblogs.com/MuYunyun/p/5927491.html#_label7)
- [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

> indigo 主题的wiki: https://github.com/yscoder/hexo-theme-indigo/wiki


## 我的定制修改

我写了个Java程序把原来Jekyll下博客的md文件copy到了新的文件夹下，并对每个md文件需要适配的地方(如摘要是通过`<!-- more -->`来区分的)进行了批量处理。

### 统计

很多主题都支持访问统计，一般需要在配置文件配置自己对应账号的key，我一般使用Google Analytics+百度统计：

- Google Analytics: https://analytics.google.com/analytics/web/
    - https://support.google.com/analytics/answer/1008080
    - [Analytics Book 中文版](http://cn.analyticsbook.org/google-analytics-tracking-codes/)：16年的，感觉过时了。
- 百度统计: https://tongji.baidu.com/web/homepage/index

其中有官方提供的统计js代码，indigo的代码有些过时了，可通过配置文件里统计配置对应的关键字全局搜索，找到对应代码，并更新成最新的js。

具体地，indigo主题的统计代码在`themes/indigo/layout/_partial/plugins`目录下的`baidu.ejs`和`google-analytics.ejs`，其他主题也可通过类似方法修改



### 搜索

针对无数据库的静态博客搜索方案一般有两种：

- 第三方搜索服务；
- 序列化站点内容作为数据源，然后自己写查询方法。

indigo作者是通过自定义搜索实现的，搜索方案为：[Hexo添加站内搜索功能初步完成](https://yscoder.github.io/20160511/hexo-search.html#%E6%90%9C%E7%B4%A2%E6%96%B9%E6%A1%88)。原理比较简单，简单概括就是：
先通过`hexo-generator-json-content`插件生成json格式的数据文件，然后将这个数据文件作为数据源，对输入进行正则匹配。由于我只需要简单搜索，简单正则就能满足我的要求，该方案基本可以，所有没有增加分词等高级搜索的功能。

但有他的实现有几个问题：

- 没有对输入转义，直接根据输入生成正则，当输入包含正则元字符时前端会报错，且把空格替换为了`或`的逻辑，而不是`且`的逻辑
- 没有权重，标题、正文等只要出现关键字就显示，导致我很多文章都有git地址，只要输入git，全匹配了，没有区分度
- 没有对摘要进行搜索

所以我做了简单的修改：

- 转义用户输入中的正则元字符
- 生成json数据文件时增加摘要字段，去掉正文字段，以减少json文件大小（可选，也可以不去掉）
- 增加权重，权重大小：标题>摘要>标签>正文

修改`themes/indigo/source/js/search.js`即可在本地测试，发布线上的话，需要mimify js文件，然后替换掉原来的`themes/indigo/source/js/search.min.js`，压缩js的工具网站：https://javascript-minifier.com/

*以下是具体实现，没兴趣可略过*

修改前后的正则(注释的为修改前的)：

```js
// var regExp = new RegExp(key.replace(/[ ]/g, '|'), 'gmi');
var regExp = new RegExp(escapeRegExp(key), 'gmi');

function escapeRegExp(string) {
    return string.replace(/([.*+?^=!:${}()|[\]\/\\])/g, '\\$&');
        //$&表示整个被匹配的字符串
}
```

其中`function escapeRegExp(string)`参考**正则escape**：

 - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions
 - https://stackoverflow.com/questions/3561493/is-there-a-regexp-escape-function-in-javascript

同时，匹配结果不再只返回`true/false`，而是返回权重，搜索结果更根据权重排序：

```js
// 匹配文章内容返回结果，返回值代表权重。按照标题，摘要，标签，正文的顺序递减
function matcher(post, regExp) {
    if (regtest(post.title, regExp)) {
        return 10;
    }
    if (regtest(post.excerpt, regExp)) {
        return 5;
    }
    if (post.tags.some(function (tag) {
        return regtest(tag.name, regExp);
    })) {
        return 3;
    }
    if(regtest(post.text, regExp)){
        return 1;
    }
    return 0;
}

// 匹配文章内容返回结果
loadData(function (data) {
    // 结果按照匹配程度过滤并排序
    var result = data.map(post => {
        post.search_order = matcher(post, regExp);
        return post;
    });
    result = result.filter(post => post.search_order && post.search_order > 0);
    result = result.sort((a, b) => b.search_order - a.search_order);

    render(result);
    Control.show();
});
```


## 遇到的问题

### 安装hexo失败

- 报错：`Missing write access to /usr/local/lib/node_modules/hexo-cli`
- 解决： 进入文件夹`/usr/local/lib`，修改权限

```
$ cd /usr/local/lib where the global node_module folder is found
$ sudo chown -R username:group node_modules
```

- 参考：
  - https://github.com/hexojs/hexo/issues/2545
  - https://github.com/npm/npm/issues/10683

### 自定义域名

- 问题：绑定失败
- 解决：
  - 申请并购买一个自定义域名 
  - ping你的github.io域名，得到一个IP
  - 在你的域名的 DNS 配置中添加A类型的记录，主机记录为`@`，记录值为上述IP
  - 在仓库根目录添加`CNAME`文件，并在文件中填写绑定的顶级域名
  - 在对应仓库的设置里`Settings->GitHub Pages->Custom domain`写上你的自定义域名
  - 有的博客说还要更换DNS服务器地址为`f1g1ns1.dnspod.net`和`f1g1ns2.dnspod.net`，我没换也成功了。
- 参考
  - https://www.zhihu.com/question/31377141
  - https://help.github.com/en/articles/using-a-custom-domain-with-github-pages 


由于自定义域名后原来的统计数据就从零重计了，所以就放弃了，还是使用github.io的域名算了。
    
### 清理DNS缓存
    
- 问题：配置自定义域名失败，一直跳空白或者找不到地址，删除DNS解析配置也没用
- 解决：需要清除本地DNS缓存以及chrome缓存

```shell
sudo killall -HUP mDNSResponder; sleep 2;
sudo killall mDNSResponderHelper
sudo dscacheutil -flushcache
```

chrome缓存清理：`设置->高级->隐私设置和安全性->清除浏览数据`或者`右上角三个点->更多工具->清除浏览数据`

- 参考：
 - [How to Clear DNS Cache in macOS Mojave](https://www.hongkiat.com/blog/how-to-clear-dns-cache-in-macos-mojave/)
 - [Flush DNS cache locally in macOS Mojave, Sierra, OSX, Linux and Windows](https://coolestguidesontheplanet.com/clear-the-local-dns-cache-in-osx/)
 - [Mac OS X 清除DNS缓存](https://www.cnblogs.com/qq952693358/p/9126860.html)


### 博客原文的备份

- 问题：使用Hexo后，仓库只有生成的public文件夹下的文件，博客都是html，原来的博客原文没有版本管理
- 解决：新建分支`hexo-public`（名字自取），将hexo的根目录作为仓库根目录，这样发布的内容在`master`分支，而编辑的内容和各种配置等原始信息在`hexo-public`实现版本管理
    - 使用的主题的对应文件夹下会包含`.git`，`git add `时会有一些提示，根据自己需要删除`.git`，或者作为子模块进行管理。最好不要直接`git rm --cached`，否则后续对主题的修改不会被追踪
- 参考：
    - [在Github上备份Hexo博客](https://lrscy.github.io/2018/01/26/Hexo-Github-Backup/#%E5%9C%A8Github%E4%B8%8A%E5%A4%87%E4%BB%BD) 
 


## 参考

- [利用Github Pages + Hexo搭建个人博客](http://threehao.com/2016/08/22/Github%20Pages%20+%20Hexo/)
- [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
- [Hexo 使用总结 & 常见问题](http://www.cylong.com/blog/2016/04/25/hexo-faq/)
- [Hexo常见问题解决方案](https://xuanwo.io/2014/08/14/hexo-usual-problem/#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
- [制作Hexo主题详细教程（2）](http://blog.geekaholic.cn/2017/03/06/%E5%88%B6%E4%BD%9CHexo%E4%B8%BB%E9%A2%98%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B%EF%BC%882%EF%BC%89/)
- [为 Hexo 博客创建本地搜索引擎](https://liam.page/2017/09/21/local-search-engine-in-Hexo-site/)


  

