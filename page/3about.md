---
layout: page
title: About
permalink: /about/
icon: heart
---

* content
{:toc}


## 关于我


> 2017年我就要找工作了，希望做服务器/数据分析相关方向

在校硕士研究生一枚，就读于华中科技大学电子信息与通信学院，2015级学硕(三年)   

爱技术，爱吹逼，热爱生活。

最喜欢的游戏是《炉石传说》,天梯传说+竞技场12胜,[武汉高校星联赛季军](http://7xsna4.com2.z0.glb.clouddn.com/heartstone-prize.png)

---

## 联系我


{% for i in site.data.author.follow %}[{{i.text}}]({{ i.link }})|{% endfor %}

---

## 我的社区

{% for i in site.data.author.community %}[{{i.text}}]({{ i.link }})|{% endfor %}




---

## 关于本站  


这个博客主要用于记录一个菜鸟的成长之路

2016.03.08 | 学习springmvc+mybatis的使用，整理了系列学习笔记博客近40篇(已发布20篇)，还被清华大学出版社编辑联系出书[(附图片)](http://7xph6d.com1.z0.glb.clouddn.com/%E9%9A%8F%E7%AC%94_%E6%B8%85%E5%8D%8E%E5%A4%A7%E5%AD%A6%E5%87%BA%E7%89%88%E7%A4%BE%E8%81%94%E7%B3%BB%E6%88%91.png)，婉拒了。这个月开始补统计学的基础，着手机器学习。平时抽空深入学习spring(依赖注入、AOP、IOC等等)。
2016.02.08 | 复习和巩固了javase,入门了javaweb基础;并把学习笔记整理成系列博客发布在技术社区(oschina,有两篇文章上了推荐，两篇浏览破千，一篇被76人收藏)和自己的网站(升级到jekyll 3.0.2)。准备开始学习spring以及机器学习基本算法
2016.01.08 | 开始建立个人博客,github+jekyll,把jekyll官网通读了一遍  -_-
2015.12.01 | 转投intellj IDEA,学习ant+ivy,maven,git的基本使用,搭建hadoop环境
2015.10.01 | 开始接触网络爬虫,小demo。调研主流爬虫，在eclise下开发修改nutch的parser插件，


通过搭建这个静态博客，对git的基本用法更熟了，同时对jekyll的目录结构，文件作用大致了解了。  

---


Comment below to exchange link with me.  


## Comments

{% if site.duoshuo_shortname %}
<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="{{ site.url }}{{ page.url }}" data-title="{{page.title}}" data-url="{{ site.url }}{{ page.url }}"></div>
<!-- 多说评论框 end -->
{% endif %}

{% if site.disqus_shortname %}
<div id="disqus_thread"></div>
<script>
/**
* RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
* LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
*/

var disqus_config = function () {
this.page.url = '{{ site.url }}{{ page.url }}'; // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = '{{ site.url }}{{ page.url }}'; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');

s.src = '//{{site.disqus_shortname}}.disqus.com/embed.js';

s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}

<script>
/**
 * target _blank
 */
(function() {
    var aTags = document.querySelectorAll('.left a')
    for (var i = 0; i < aTags.length; i++) {
        aTags[i].setAttribute('target', '_blank')
    }
}());
</script>


