---
layout: post
title:  在github搭建个人网站
date:   2016-01-07 22:35:11 +08:00
category: 其他
tags: [GitHub, 安装部署]
comments: true
---

先搜篇中文博客了解下流程，再根据下面的官方文档**按顺序**看一遍就差不多了。

这里不得不吐槽windows太垃圾了，linux下三行指令解决的问题，windows还要一个个安装、改配置文件。

<!-- more -->

## 参考教程

>* [GitHub Help](https://help.github.com/categories/github-pages-basics/)
>* [GitHub Pages](https://pages.github.com/)
>* [Jekyll](http://jekyllrb.com/)*（附上[中文版](http://jekyllcn.com/))*
>* [Run Jekyll on Windows](http://jekyll-windows.juthilo.com/)


## 流程
首先默认你已经拥有了自己github账号，并会基本的git操作。

1. 新建一个和github用户名(username)同名的仓库，`username.github.io`
2. 在本地电脑安装
- Ruby和devkit
- jekyll
- python

3. 找一个好的模板
4. 修改模板，导入自己的文章

具体步骤懒得写了，不定期完善。

- Mac下安装

1.安装ruby

由于我之前安装过Xcode还有Xcode command line tools,所以直接就有ruby了，Terminal里输入`ge`+`Tab键`能看到有`gem`指令

2.安装jekyll和bundle

参考：[Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)


## 感想
主要是我完全不懂ruby和python,前端也只有少的可怜的一点常识。搭建环境不复杂，基本安装好了后`jekyll new myblog`就有个基础模板了，不过很难看。所以我主要花了一整天时间去找主题、P图、了解jekyll的目录结构和用法、测试显示效果。总算搞定了，不过markdown的语法高亮我还是不满意，比较喜欢SegmentFault的高亮。有空再说这部分。

## 遇到的问题
1.windows下安装ruby的devkit遇到问题

```
Invalid configuration or no Rubies listed. Please fix 'config.yml'
and rerun 'ruby dk.rb install'
```


解决：[How do I configure config.yml so that I can install devkit?](http://stackoverflow.com/questions/20810653/how-do-i-configure-config-yml-so-that-i-can-install-devkit)

2.kunka主题jekyll build问题

```
Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration fil e.
```

解决：[jekyll-paginate gem](https://teamtreehouse.com/community/jekyllpaginate-gem)

3.markdown显示问题

这里很蛋疼，我写markdown的习惯是代码都是前后各三个反单引号包起来的。使用kramdown解析markdown不能对含三个反单引号的代码块进行识别，默认按单行代码处理，缩进都没了；使用redcarpet能识别代码块，但没高亮，rouge又提示什么要下1.3版本，反正神烦。

参考：

- [Syntax highlighting markdown code blocks in Jekyll (without using liquid tags)](http://stackoverflow.com/questions/8648390/syntax-highlighting-markdown-code-blocks-in-jekyll-without-using-liquid-tags)
- [Jekyll kramdown配置](http://blog.javachen.com/2015/06/30/jekyll-kramdown-config.html)



4.jekyll build失败报错(多版本冲突)

解决：[Jekyll/Ruby Kramdown Missing Dependency](http://stackoverflow.com/questions/31417469/jekyll-ruby-kramdown-missing-dependency)

5.bundle install SSL接连不上(其实是网速不好，多试几次。下载前`gem update --system`)

查阅：[SSL Error When installing rubygems, Unable to pull data from 'https://rubygems.org/](http://stackoverflow.com/questions/10246023/bundle-install-fails-with-ssl-certificate-verification-error)

6.jekyll升级到3.0

>* [GitHub Pages now faster and simpler with Jekyll 3.0](https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0)
>* [Upgrading from 2.x to 3.x](http://jekyllrb.com/docs/upgrading/2-to-3/)

7.gem install出现error，伟大的防火墙问题

```
brian@brianway:~$ gem install jekyll
ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://api.rubygems.org/quick/Marshal.4.8/jekyll-3.1.6.gemspec.rz)
```

解决：换成淘宝的镜像 https://ruby.taobao.org/

8.MAC下`gem install jekyll`失败

```
ERROR:  While executing gem ... (Gem::FilePermissionError)
You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
```

好像不建议使用`sudo`,需要另外弄一套包管理rbenv或者RVM，图省事的话，可以直接安装在本地用户目录,指令格式：`gem install *** --user-install`，`***`是要安装的包名，具体可以输入`gem help install`查看参数意义


参考：

- [Installing gem or updating RubyGems fails with permissions error](http://stackoverflow.com/questions/14607193/installing-gem-or-updating-rubygems-fails-with-permissions-error)
- [Can't install gems on OS X “El Capitan”](http://stackoverflow.com/questions/31972968/cant-install-gems-on-os-x-el-capitan)

9.mac下bundle执行失败

```
brian@brianway:~/mygit/brianway.github.io (master)$ bundle exec jekyll serve
Configuration file: /Users/brian/mygit/brianway.github.io/_config.yml
Source: /Users/brian/mygit/brianway.github.io
Destination: /Users/brian/mygit/brianway.github.io/_site
Incremental build: disabled. Enable with --incremental
Generating...
ERROR: YOUR SITE COULD NOT BE BUILT:
------------------------------------
Invalid date '<%= Time.now.strftime('%Y-%m-%d %H:%M:%S %z') %>': Document 'vendor/bundle/ruby/2.0.0/gems/jekyll-3.1.6/lib/site_template/_posts/0000-00-00-welcome-to-jekyll.markdown.erb' does not have a valid date in the YAML front matter.
```

解决：在`_config.yml`文件添加`exclude: ["vendor"]`

参考：

- [How to set up a Jekyll blog on Heroku](http://www.markcampbell.me/jekyll/heroku/2013/05/18/how-to-set-up-jekyll-on-heroku.html)
- [Error building site: Post 0000-00-00... does not have a valid date (hurtstotouchfire 的回答)](https://github.com/benbalter/jekyll-auth/issues/23)
- [Fix Travis issue due to --deployment flag during bundle ](https://github.com/nparry/nparry.com/commit/2642e799a9b28ec866c405cccac379ae3770f0fe)


## 主题

好多主题太闪，我跑不通，或者怕不好改，挑了几个简单的，我跑通的主题(不要吐槽我的审美)

>* [so-simple-theme](http://mmistakes.github.io/so-simple-theme/theme-setup/)
>* [hpstr-jekyll-theme](http://mmistakes.github.io/hpstr-jekyll-theme)
>* [陈俊的网：浮生志](http://chenjun.com/links.html)
>* [kunka](https://github.com/pizn/kunka)
