---
layout: post
title:  3D Rigid Object Tracking
date:   2016-05-16 20:58:00 +08:00
category: project
tags: project
comments: true
---

* content
{:toc}


华中科技大学图像分析与理解课程项目--3D Rigid Object Tracking.
每周最新进展可参看[Time Line](#time-line)和[To Do List](#to-do-list)








组员： [魏楚阳](http://brianway.github.io/)，邵滨峰，郑琪，付鼎


## Introduction

The objective of 3D rigid object tracking is to associate 3D target objects in consecutive video frames and meanwhile estimate the relative pose (3D translation and 3D rotation) between the 3D target and the camera.Rigid means the relative position among object components do not change. For instance, a cup, a book and a car are rigid object while a human face and a cat are non-rigid object.It has a variety of uses, some of which are: humancomputer interaction, security and surveillance, video communication and compression, augmented reality, traffic control, medical imaging and video editing.

3D object tracking can be especially difficult when the objects are moving fast relative to the frame rate. Another situation that increases the complexity of the problem is when the tracked object changes orientation over time. For these situations the tracking system usually employs a motion model which describes how the image of the target might change for different possible motions of the object.



## Time Line

| Time        | details |  
| :--------:  | :----- |
| 2016.05.16  | choose the project 5,create the projetct link |



## To Do List

计划逐步扩充并实现下面的任务

- [x] 选定项目题目，组队确定组员，创建项目链接
- [ ] 阅读相关论文，确定实现方案
- [ ] 待更新



## Video

暂时展示一个旅游视频

<embed src="http://player.youku.com/player.php/sid/XOTQ3MTAwMzMy/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash" />



## Reference

> [1]. Multiple 3D Object Tracking for Augmented Reality, In ISMAR, 2008.
> [2]. https://www.ssontech.com/tutes/tuteobj.html
> [3]. Manipulator and Object Tracking for In-Hand 3D Object Modeling, IN IJRR, 2011.
> [4]. Robust Statistics for 3D Object Tracking, In ICRA 2006.
> [5]. Real-time 3D Object Pose Estimation and Tracking for Natural Landmark Based Visual Servo. In IROS, 2008.
