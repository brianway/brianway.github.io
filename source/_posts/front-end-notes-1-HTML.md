---
layout: post
title:  前端学习笔记(1)-HTML
date:   2017-05-18 21:34:07 +08:00
category: 基础教程
tags: [HTML, 前端]
comments: true
---

本文主要对 HTML 标签进行介绍和归纳

<!-- more -->

## 基本概念

1. HTML 是网页内容的载体。内容就是网页制作者放在页面上想要让用户浏览的信息，可以包含文字、图片、视频等。
2. CSS 样式是表现。比如，标题字体、颜色变化，或为标题加入背景图片、边框等，所有这些用来改变内容外观的东西称之为表现。
3. JavaScript 是用来实现网页上的特效效果。如：鼠标滑过弹出下拉菜单，或鼠标滑过表格的背景颜色改变，还有焦点新闻（新闻图片）的轮换。有动画的，有交互的一般都是用 JavaScript 来实现的。


## 常用标签

主要需要留意，标签的用途、标签在浏览器中的默认样式

*注：现在一般使用 xhtml1.0 的版本（其它标签也是），这种版本比较规范*

- 强调语气：`<em>`是斜体，`<strong>`是加粗
- 引用：`<q>`短文本引用，`<blockquote>`长文本引用
- 换行 `<br />`
- 水平横线 `<hr />`
- 空格 `&nbsp;`
- 表格 `<table>`
    - `<tbody>` 加上后表格内容全部下载完才会显示
    - 行 `<tr>`
    - 列 `<td>`
    - 表格表头 `<th>`
    - 标题 `<caption>`
- 超链 `<a>`
    - 例子：`<a  href="目标网址"  title="鼠标滑过显示的文本">链接显示的文本</a>`
    - 新标签打开：`target="_blank"`
- 图片 `<img>`，图像可以是 GIF，PNG，JPEG 格式的图像文件
    - 例子：`<img src="图片地址" alt="下载失败时的替换文本" title = "提示文本">`
- 表单 `<form>`
- 文本域 `<textarea>`
    - 例子 `<textarea rows="行数" cols="列数">文本</textarea>`
    - cols 多行输入域的列数；rows 多行输入域的行数。这两个属性可用 CSS 样式的 width 和 height 来代替：col 用 width、row 用 height 来代替
- 输入框 `<input type="text/password" name="名称" value="文本" />`
    - 当 type="text" 时，输入框为文本输入框
    - 当 type="password" 时, 输入框为密码输入框
- 单/复选框 `<input type="radio/checkbox" value="值"    name="名称"   checked="checked"/>`
    - 当 type="radio" 时，控件为单选框，**同一组单选框 name 命名要一致**
    - 当 type="checkbox" 时，控件为复选框
- 提交按钮 `<input type="submit" value="提交">`
- 重置按钮 `<input type="reset" value="重置">`
- 下拉列表框  ` <select><option value="看书">看书</option></select>`
    - value  `<option value="提交值">选项</option>`
    - 选中 `selected="selected"`
    - 多选 `multiple="multiple"`
- 标签 `<label for="控件id名称">`，标签的 for 属性中的值应当与相关控件的 **id 属性** 值一定要相同
