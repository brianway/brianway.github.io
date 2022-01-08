---
layout: post
title:  前端学习笔记(3)-DOM 基础
date:   2017-05-18 22:27:07 +08:00
category: 基础教程
tags: [前端]
comments: true
---

本文主要介绍 HTML DOM。第一部分 "DOM 概述" 主要参考 MDN，第二部分 "HTML DOM" 主要参考 W3School。

<!-- more -->

## DOM 概述

> 文档对象模型 (DOM) 是HTML和XML文档的编程接口。它提供了对文档的结构化的表述，并定义了一种方式可以使从程序中对该结构进行访问，从而改变文档的结构，样式和内容。DOM 将文档解析为一个由节点和对象（包含属性和方法的对象）组成的结构集合。简言之，它会将web页面和脚本或程序语言连接起来。
>
> 所有操作和创建web页面的属性，方法和事件都会被组织成对象的形式（例如， document 对象表示文档本身， table 对象实现了特定的 HTMLTableElement DOM 接口来访问HTML 表格等）。
>
> 引自 [MDN - DOM 概述](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)

JavaScript 可以访问和操作存储在 DOM 中的内容:

`API (web 或 XML 页面) = DOM + JS (脚本语言)`

一些重要的数据类型：

|document|每个载入浏览器的 HTML 文档都会成为 Document 对象。[MDN - document](https://developer.mozilla.org/en-US/docs/Web/API/document)|
|:---|:---|
|element|element 是指由 DOM API 中成员返回的类型为 element 的一个元素或节点。|
|nodeList|nodeList 是一个元素的数组|
|attribute|DOM 中的属性也是节点，就像元素一样|
|namedNodeMap|namedNodeMap 和数组类似，但是条目是由 name 或 index 访问的|


--------------


## HTML DOM

下面用一张树状图图来介绍 HTML DOM。

文档对象模型 DOM（Document Object Model）定义访问和处理HTML文档的标准方法。DOM 将 HTML 文档呈现为带有元素、属性和文本的树结构（节点树）

![HTML DOM 树](http://blog.qiniu.brianway.site/dom_ct-htmltree.gif)


HTML 文档可以说由节点构成的集合,常见的 DOM 节点:

1. 元素节点：上图中`<html>、<body>、<p>`等都是元素节点，即标签
2. 文本节点：向用户展示的内容，如 `<li>...</li>`中的 JavaScript、DOM、CSS 等文本。
3. 属性节点：元素属性，如 `<a>` 标签的链接属性 `href="http://www.imooc.com"`

几个简单的 DOM 操作：

- innerHTML 属性用于获取或替换 HTML 元素的内容，`Object.innerHTML`
- 改变 HTML 样式，`Object.style.property=new style;`
- 隐藏和显示 `Object.style.display = value`，value 可取 `none` 或者 `block`

> 参考 [慕课网 - JavaScript 入门篇(第三章)](http://www.imooc.com/learn/36)

------------

HTML DOM 定义了所有 HTML 元素的对象和属性，以及访问它们的方法。换言之，HTML DOM 是关于如何获取、修改、添加或删除 HTML 元素的标准。

> 参考 [W3School - HTML DOM 教程](http://www.w3school.com.cn/htmldom/index.asp)

### 方法和属性

所有 HTML 元素被定义为对象，而编程接口则是 **对象方法** 和 **对象属性**。

- 方法是您能够执行的动作（比如添加或修改元素）。
- 属性是您能够获取或设置的值（比如节点的名称或内容）

一些常用的 HTML DOM 方法：

- getElementById(id) - 获取带有指定 id 的节点（元素）
- appendChild(node) - 插入新的子节点（元素）
- removeChild(node) - 删除子节点（元素）

一些常用的 HTML DOM 属性：

- innerHTML - 节点（元素）的文本值
- parentNode - 节点（元素）的父节点
- childNodes - 节点（元素）的子节点
- attributes - 节点（元素）的属性节点
- nodeName - nodeName 属性规定节点的名称。
    - nodeName 是只读的
    - 元素节点的 nodeName 与标签名相同
    - 属性节点的 nodeName 与属性名相同
    - 文本节点的 nodeName 始终是 `#text`
    - 文档节点的 nodeName 始终是 `#document`
- nodeValue - 性规定节点的值。
    - 元素节点的 nodeValue 是 `undefined` 或 `null`
    - 文本节点的 nodeValue 是文本本身
    - 属性节点的 nodeValue 是属性值
- nodeType - 返回节点的类型。nodeType 是只读的。

DOM 根节点有两个特殊的属性，可以访问全部文档：

- document.documentElement - 全部文档
- document.body - 文档的主体

### 访问和修改

访问 HTML 元素等同于访问节点,能够以不同的方式来访问 HTML 元素：

- `getElementById()` 方法
- `getElementsByTagName()` 方法
- `getElementsByClassName()` 方法

修改 HTML DOM 意味着许多不同的方面：

- 改变 HTML 内容
- 改变 CSS 样式
- 改变 HTML 属性
- 创建新的 HTML 元素
- 删除已有的 HTML 元素
- 改变事件（处理程序）

### 事件

HTML 事件的例子：

- 当用户点击鼠标时，onmousedown、onmouseup 以及 onclick 事件
- 当网页已加载时，onload 和 onunload 事件
- 当图片已加载时
- 当鼠标移动到元素上时，onmouseover 和 onmouseout 事件
- 当输入字段被改变时，onchange 事件
- 当 HTML 表单被提交时
- 当用户触发按键时


## 参考资料

>* [W3School - HTML DOM 教程](http://www.w3school.com.cn/htmldom/index.asp)
>* [W3School - HTML DOM Document 对象](http://www.w3school.com.cn/jsref/dom_obj_document.asp)
>* [MDN - Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
