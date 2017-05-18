---
layout: post
title:  前端学习笔记(2)-CSS 基础
date:   2017-05-18 21:55:07 +08:00
category: 前端
tags: CSS 前端
comments: true
---

* content
{:toc}

本文主要对 CSS 相关基础知识进行介绍






## 基础知识

CSS 样式由选择符和声明组成，而声明又由属性和值组成。

- 选择符：又称选择器，指明网页中要应用样式规则的元素。
- 声明：在英文大括号`{}`中的的就是声明，属性和值之间用英文冒号`{}`分隔。当有多条声明时，中间可以英文分号`;`分隔。

```
选择器{
    样式;
}
```

从CSS 样式代码插入的形式来看基本可以分为以下3种：内联式、嵌入式和外部式三种。优先级遵循就近原则，一般来说，`内联式 > 嵌入式 > 外部式`。

- 内联式

例子 `<p style="color:red;font-size:12px">这里文字是红色。</p>`

- 嵌入式

```html
<style type="text/css">
    span{
        color:red;
    }
</style>
```

- 外部式

例子：`<link href="base.css" rel="stylesheet" type="text/css" />`



## CSS 选择器

常见的类选择器类型有如下几种：

- 标签选择器，`.标签选择器名称{css样式代码;}`
- 类选择器，`.类选器名称{css样式代码;}`
- ID 选择器，`＃类选器名称{css样式代码;}`
- 子选择器，即大于符号(`>`),用于选择指定标签元素的第一代子元素
- 包含选择器，即加入空格` `,用于选择指定标签元素下的后辈元素
- 通用选择器，匹配html中所有标签元素，`* {css样式代码;}`

类选择器和ID选择器都可以应用于任何元素，但 **ID 选择器只能在文档中使用一次**，可以使用类选择器词列表方法为一个元素同时设置多个样式，ID 选择器是不可以的。

子选择器和包含选择器区别：`>`作用于元素的第一代后代，空格作用于元素的所有后代。

另外还有两种选择符：

- 伪类选择符，允许给 HTML 不存在的标签（标签的某种状态）设置样式。常用的有 `a:hover{color:red;}`
- 分组选择符，为 HTML 中多个标签元素设置同一个样式时，可以使用分组选择符`,`。例如 `h1,span{color:red;}`



## CSS 的继承、层叠和特殊性

- CSS 的某些样式是具有继承性的，继承是一种规则，它允许样式不仅应用于某个特定 HTML 标签元素，而且应用于其后代。
- 特殊性：不同选择器具有不同权值，标签的权值为 1，类选择符的权值为 10，ID选择符的权值最高为 100。
- **层叠** 就是在 HTML 文件中对于同一个元素可以有多个 CSS 样式存在，当有相同权重的样式存在时，会根据这些 CSS 样式的前后顺序来决定，处于最后面的 CSS 样式会被应用。


## CSS 格式化排版

文字排版

- 字体，`body{font-family:"Microsoft Yahei";}`
- 字号、颜色，`body{font-size:12px;color:#666}`
- 粗体，`body{font-weight:bold;}`
- 斜体，`body{font-style:italic;}`
- 下划线，`body{font-style:italic;}`
- 删除线，`body{text-decoration:line-through;}`

段落排版

- 缩进，`p{text-indent:2em;}`
- 行间距（行高），`p{line-height:1.5em;}`
- 中文字间距、字母间距，`letter-spacing:50px;`和`word-spacing:50px;`
- 对齐，`div{text-align:center;}`


## CSS 盒模型

### 元素分类

在 CSS 中，HTML 中的标签元素大体被分为三种不同的类型：块状元素、内联元素(又叫行内元素)和内联块状元素。

- 常用的块状元素有：

```
<div>、<p>、<h1>...<h6>、<ol>、<ul>、<dl>、<table>、<address>、<blockquote> 、<form>
```

块级元素特点：

1. 每个块级元素都从新的一行开始，并且其后的元素也另起一行。
2. 元素的高度、宽度、行高以及顶和底边距都可设置。
3. 元素宽度在不设置的情况下，是它本身父容器的 100%（和父元素的宽度一致），除非设定一个宽度。

设置 `display:block` 就是将元素显示为块级元素，从而使元素具有块状元素特点。

*注：img 标签与 div 层之间会有空隙的解决方法是：使用 `display:block` 就可以消除间隙。*

- 常用的内联元素有：

```
<a>、<span>、<br>、<i>、<em>、<strong>、<label>、<q>、<var>、<cite>、<code>
```

内联元素特点：

1. 和其他元素都在一行上；
2. 元素的**高度、宽度**及顶部和底部边距**不可**设置；
3. 元素的宽度就是它包含的文字或图片的宽度，不可改变。

块状元素也可以通过代码 `display:inline` 将元素设置为内联元素。

- 常用的内联块状元素有：

```
<img>、<input>
```

inline-block 元素特点：

1. 和其他元素都在一行上；
2. 元素的高度、宽度、行高以及顶和底边距都可设置。

内联块状元素（inline-block）就是同时具备内联元素、块状元素的特点，代码 `display:inline-block` 就是将元素设置为内联块状元素。


### 盒模型

- 边框

盒子模型的**边框**就是围绕着**内容**及**补白**的线，这条线你可以设置它的**粗细**、**样式**和**颜色**(边框三个属性)。

```css
div{
    border:2px  solid  red;
}


div{
    border-width:2px;
    border-style:solid;
    border-color:red;
}
```

单独设置下边框的例子 `div{border-bottom:1px solid red;}`

- 宽度和高度

CSS 内定义的宽（width）和高（height），指的是 **填充以里的内容范围**。一个元素实际宽度（盒子的宽度）=左边界+左边框+左填充+内容宽度+右填充+右边框+右边界。

W3C 的标准 Box Model:

```
/*外盒尺寸计算（元素空间尺寸）*/
Element空间高度 = content height + padding + border + margin
Element 空间宽度 = content width + padding + border + margin
/*内盒尺寸计算（元素大小）*/
Element Height = content height + padding + border （Height为内容高度）
Element Width = content width + padding + border （Width为内容宽度）
```

所以有时会设置 `box-sizing: border-box;` 来避免计算内部元素大小

> 参考 [CSS3 Box-sizing](http://www.w3cplus.com/content/css3-box-sizing)

- 填充(padding)

元素内容与边框之间是可以设置距离的，称之为“填充”。填充也可分为上、右、下、左(顺时针)。

例子：

```css
/*上、右、下、左(顺时针)，顺序不要搞混*/
div{padding:20px 10px 15px 30px;}
div{
   padding-top:20px;
   padding-right:10px;
   padding-bottom:15px;
   padding-left:30px;
}
/*上、右、下、左的填充都为10px;*/
div{padding:10px;}
/*上下填充一样为10px，左右一样为20px*/
div{padding:10px 20px;}
```

- 边界(margin)

元素与其它元素之间的距离可以使用边界（margin）来设置，顺序和填充一样是上，右，下，左。padding 在边框里，margin 在边框外。


## CSS 布局模型

CSS 包含 3 种基本的布局模型，用英文概括为：Flow、Layer 和 Float。
在网页中，元素有三种布局模型：

1. 流动模型（Flow）
2. 浮动模型 (Float)
3. 层模型（Layer）

### 流动模型

流动模型，流动（Flow）是默认的网页布局模式。

流动布局模型具有2个比较典型的特征：

1. **块状元素** 都会在所处的包含元素内自上而下按顺序垂直延伸分布，因为在默认状态下，块状元素的宽度都为 100%。实际上，块状元素都会以行的形式占据位置。
2. 在流动模型下，**内联元素** 都会在所处的包含元素内从左到右水平分布显示。


### 浮动模型

任何元素在默认情况下是不能浮动的，但可以用 CSS 定义为浮动。例子：`#div1{float:left;}`

### 层模型

CSS 定义了一组定位（positioning）属性来支持层布局模型。

层模型有三种形式：
1. 绝对定位(`position: absolute`)
2. 相对定位(`position: relative`)
3. 固定定位(`position: fixed`)

可参考这篇文章辅助理解 [《css绝对定位、相对定位和文档流的那些事》](http://www.cnblogs.com/tim-li/archive/2012/07/09/2582618.html)

> 官方文档: [MDN - position](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)

- 绝对定位(`position: absolute`)

如果想为元素设置层模型中的绝对定位，需要设置 `position:absolute`(表示绝对定位)，这条语句的作用将元素从文档流中拖出来，然后使用 left、right、top、bottom 属性相对于其**最接近的一个具有定位属性的父包含块**进行绝对定位。如果不存在这样的包含块，则相对于 body 元素，即相对于**浏览器窗口**。



- 相对定位(`position: relative`)

如果想为元素设置层模型中的相对定位，需要设置 `position:relative`（表示相对定位），它通过 left、right、top、bottom 属性确定元素在**正常文档流中**的偏移位置。相对定位完成的过程是首先按 static(float) 方式生成一个元素(并且元素像层一样浮动了起来)，然后相对于**以前的位置移动**，移动的方向和幅度由left、right、top、bottom属性确定，**偏移前的位置保留不动**。

简单来说，就是相对元素原来的位置进行移动，元素本身所占的位置会保留。

- 固定定位(`position: fixed`)

设置 `position:fixed;`。fixed：表示固定定位，与 absolute 定位类型类似，但它的相对移动的坐标是视图（**屏幕内的网页窗口**）本身。由于视图本身是固定的，它不会随浏览器窗口的滚动条滚动而变化，除非你在屏幕中移动浏览器窗口的屏幕位置，或改变浏览器窗口的显示大小，因此固定定位的元素会始终位于浏览器窗口内视图的某个位置，不会受文档流动影响，这与 `background-attachment:fixed;` 属性功能相同。


Relative 与 Absolute 组合使用,必须遵守下面规范：

1. 参照定位的元素必须是相对定位元素的前辈元素
2. 参照定位的元素必须加入 `position:relative;`
3. 定位元素加入 `position:absolute`，便可以使用 top、bottom、left、right 来进行偏移定位了

例子(HTML 和 CSS 代码分别为)：

```html
<div id="box1"><!--参照定位的元素-->
    <div id="box2">相对参照元素进行定位</div><!--相对定位元素-->
</div>
```

```css
#box1{
    width:200px;
    height:200px;
    position:relative; /*参照定位的元素必须加入position:relative;*/      
}

#box2{
    position:absolute;
    top:20px;
    left:30px;/*定位元素加入position:absolute*/         
}
```

##  颜色和长度

设置颜色的方法也有很多种：

- 英文命令颜色，`p{color:red;}`
- RGB颜色，`p{color:rgb(133,45,200);}` 和 `p{color:rgb(20%,33%,25%);}`
- 十六进制颜色，
这种颜色设置方法是现在比较普遍使用的方法，其原理其实也是 RGB 设置，但是其每一项的值由 0-255 变成了十六进制 00-ff。`p{color:#00ffff;}`(当你设置的颜色是 16 进制的色彩值时，如果每两位的值相同，可以缩写一半，`#0ff`)

RGB 配色表参考 [RGB颜色对照表 - 在线工具 - 开源中国](http://tool.oschina.net/commons?type=3) 或者 [RGB 配色表](http://www.wahart.com.hk/rgb.htm)

长度单位总结一下，目前比较常用到px（像素）、em、% 百分比，要注意其实这三种单位都是相对单位。

- 像素
- em，就是本元素给定字体的 font-size 值
- % 百分比


## 设置小技巧

### 水平居中设置

- 行内元素。如果被设置元素为文本、图片等行内元素时，水平居中是通过给**父元素**设置 `text-align:center` 来实现的。
- 定宽块状元素(块状元素的宽度 width 为固定值)。满足**定宽**和**块状**两个条件的元素是可以通过设置“左右 margin”值为 auto 来实现居中的。
- 不定宽块状元素。
    - 加入 table 标签(包括 `<tbody>、<tr>、<td>`)，为这个 table 设置“左右 margin 居中”
    - 设置 `display: inline` 方法：与第一种类似，显示类型设为 行内元素，然后使用 `text-align:center` 来实现居中效果，进行不定宽元素的属性设置。
    - 给父元素设置 float 和 `position:relative; left:50%`，子元素设置 `position:relative` 和 `left: -50%` 来实现水平居中。

### 垂直居中设置

- 父元素高度确定的单行文本。通过设置父元素的 height 和 line-height 高度一致来实现的。(height: 该元素的高度；line-height: 顾名思义，行高（行间距），指在文本中，行与行之间的 基线间的距离 )。
- 父元素高度确定的多行文本。使用插入 table  (包括 tbody、tr、td)标签，同时设置 `vertical-align：middle`。


另外，**为元素设置以下两个属性之一会隐形改变 display 类型**，元素的display显示类型就会自动变为以 `display:inline-block`（块状元素）的方式显示，当然就可以设置元素的 width 和 height 了，且默认宽度不占满父元素。

 1. `position: absolute`
 2. `float: left` 或 `float:right`

更详细的居中布局技巧可以参考我另外一篇文章：[CSS 布局经典问题初步整理](http://brianway.github.io/2017/05/18/css-layout-classical-problems/)


## 参考资料

>* [MDN CSS入门教程](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Getting_started)
>* [慕课HTML+CSS基础教程视频](http://www.imooc.com/learn/9)
>* [css绝对定位、相对定位和文档流的那些事](http://www.cnblogs.com/tim-li/archive/2012/07/09/2582618.html)
>* [W3School - CSS 参考手册](http://www.w3school.com.cn/cssref/index.asp)
>* [学习CSS布局](http://zh.learnlayout.com/toc.html)
