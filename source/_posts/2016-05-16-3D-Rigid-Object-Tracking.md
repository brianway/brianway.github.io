---
layout: post
title:  3D Rigid Object Tracking
date:   2016-05-16 20:58:00 +08:00
category: project
tags: [project]
comments: true
---

华中科技大学图像分析与理解课程项目--3D Rigid Object Tracking.

每周最新进展概况可参看[Time Line](#time-line),详细进度可参看[Details](#details)

<!-- more -->

组员： [魏楚阳](http://brianway.github.io/)，邵滨峰，郑琪，付鼎


## Introduction

The objective of 3D rigid object tracking is to associate 3D target objects in consecutive video frames and meanwhile estimate the relative pose (3D translation and 3D rotation) between the 3D target and the camera.Rigid means the relative position among object components do not change. For instance, a cup, a book and a car are rigid object while a human face and a cat are non-rigid object.It has a variety of uses, some of which are: humancomputer interaction, security and surveillance, video communication and compression, augmented reality, traffic control, medical imaging and video editing.

3D object tracking can be especially difficult when the objects are moving fast relative to the frame rate. Another situation that increases the complexity of the problem is when the tracked object changes orientation over time. For these situations the tracking system usually employs a motion model which describes how the image of the target might change for different possible motions of the object.


## To Do List

计划逐步扩充并实现下面的任务

- [x] 选定项目题目，组队确定组员，创建项目链接(2016.5.13~2016.5.19)
- [x] 阅读相关论文，确定实现方案(1~2周)
- [x] 代码实现，PC上验证和测试方案(2周)
- [x] 移植到移动端，在安卓设备上实现(1周)


## Time Line

| Time        | details |  
| :--------:  | :----- |
| 2016.05.16  | choose the project 5,create the projetct link |
| 2016.05.26  | meet OpenCV,read two references,test one method |
| 2016.06.02  | Android  preparing |
| 2016.06.10  | Monocular camera calibration and find corner on the object by Checkerboard|
| 2016.06.16  | Implement PC demo all by our own and complete the android version with Vuforia|


## Video

- PC端演示视频


<embed src="http://player.youku.com/player.php/sid/XMTYxMTIyMzU0MA==/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash" />


- Android端演示视频


<embed src="http://player.youku.com/player.php/sid/XMTYxMDQyODg2OA==/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash" />



## Reference

>* [1]. [Multiple 3D Object Tracking for Augmented Reality, In ISMAR, 2008.](http://delivery.acm.org/10.1145/1610000/1605357/04637336.pdf)
>* [2]. https://www.ssontech.com/tutes/tuteobj.html
>* [3]. [Manipulator and Object Tracking for In-Hand 3D Object Modeling, IN IJRR, 2011.](http://rse-lab.cs.washington.edu/papers/ijrr-11-tracking.pdf)
>* [4]. Robust Statistics for 3D Object Tracking, In ICRA 2006.
>* [5]. Real-time 3D Object Pose Estimation and Tracking for Natural Landmark Based Visual Servo. In IROS, 2008.
>* [6]. [OpenCV Tutorial C++](http://opencv-srf.blogspot.my/2010/09/object-detection-using-color-seperation.html)


--------------------


## Details

项目详细进展

### week 1

因为之前从来都没有接触过3D object tracking，所以这一周我们主要是从算法入手，选择性地详细阅读了两篇参考文献，了解一下实现步骤。

为了熟悉一下openCV，我们运行了一个tracking ball的小程序，没有考虑物体的3D信息：

![http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_trackingball.jpg](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_trackingball.jpg)


3D object tracking的复杂性在于要估计物体姿态，逼近物体表面，而不是简单地用一个方形或者圆形的框把物体框住。由于没有采集深度信息的设备，如Kinect等，我们只能利用基于模型的方法来实现。

> [1] Youngmin Park, Vincent Lepetit, Woontack Woo. Multiple 3D Object Tracking for Augmented Reality.

Proposed 方法把object detection和tracking结合起来了，可以满足实时性要求，并且同时可以tracking多个object，这是当时其他方法做不到的。

Proposed 方法由两个可以并行的模型组成，object detection和object tracking，如下图所示：

![图像处理课_reference1-proposed方法.png](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_reference1-proposed%E6%96%B9%E6%B3%95.png)


用于表示物体的数据结构：

包括几何信息和目标物体的外形。几何信息是一个存储在一系列三角形中的标准CAD 3D模型；外形信息则是由一些关键帧(keyframe)集合提供，这些关键帧主要是从不同视角拍摄物体，一般3、4个关键帧就足以360°覆盖整个物体。在每一个关键帧中，提取出特征点，也被称为keypoints，然后通过在3D模型上进行back-projecting的方式确定这些keypoints的3D位置，keypoints及其3D位置也被存储起来。

来自所有objects的所有N个keyframes被分到多个不同的子集中，每个子集中含有f个keyframe.

- (1) object detection：用reference[8]中的方法来匹配输入帧和关键帧，会得到两者之间特征点的匹配数，然后用RANSAC和non-iterative P-n-P算法来估测物体的pose。**[这里得到的是matched keypoints]**
- (2) frame-by-frame tracking：有两个目的：一是只要物体出现了就会被检测出来，以此达到tracking的目的；二是消除了单独的object detection可能产生的抖动。具体做法是：在每一帧输入中提取特征点，用基于cross-correlation和local search的方法将这些特征点和前面一帧中提取的特征点进行匹配。**[这里得到的是“temporal keypoints”]**
- (3) 将temporal keypoints和matched keypoints进行融合来估测物体的姿态


第一篇文章是2008年的，proposed方法可以实现多目标tracking，但是在实时性上可能效果不太好，作者也说的比较委婉。于是我们阅读了下面一篇文献，是2012年的。

> [2] Changhyun Choi and Henrik I. Christensen. Real-time 3D Model-based Tracking Using Edge and Keypoint Features for Robotic Manipulation.

主要步骤：

- 先用keypoints完成Global Pose Estimation(GPE)中的estimate the initial pose步骤
- 然后用edge做tracking, 即Local Pose Estimation(LPE)

![图像处理课_reference2-LPE.png](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_reference2-LPE.png)

和[1]一样，这里也用到了事先存储好的关键帧（keyframe）集合；先得到当前帧的SURF keypoints，再利用这些keypoints将当前帧和关键帧match；keypoints的3D坐标也是通过在CAD model上进行back-projecting得到的，从而可以进行pose estimation。

这篇文章的实验部分写的比较详细，尤其是我们在第一篇文章中不知道CAD模型怎么获取等，在这篇文章中都有讲到。相比之下，这篇文章的方法更复杂，速度非常快，我们准备尝试实现该文章的算法，具体的细节下次再更新~


### week 2


本周主要完成安卓的准备工作

1.功能需求：


- 调用手机的相机进行拍照
- 对拍照得到的图像进行轮廓识别和描点（由于目前图像3D识别相关的c算法还未完成，这里先用google的人脸识别功能代替）
- 小组主页的展示


2.实现效果：

菜单见下图（菜单中包含照相、分析、关于我们三部分）

![图像处理课_week2-菜单](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week2-%E8%8F%9C%E5%8D%95.jpg)



3.相关知识点：

- 菜单的动画效果：

这里的动画效果采用的是属性动画（ValueAnimator），相比于原始动画，属性动画的点击效果会随动画的位置改变而改变这更加符合响应时的逻辑，而且属性动画的可定制性更高可以
做出更酷炫的效果。属性动画的原理是基于TimeInterpolator和TypeEvaluator的。如果将属性值的变化过程看做一个数学函数的话，从动画效果上来看它是连续的，但实际上它还是
离散的，因为它实际上也就是通过插入中间值（简称插值）从而”一帧一帧”完成动画的，那每一帧在哪里取，取多少呢？这也就是ValueAnimator类主要完成的作用。
TimeInterpolator用来控制在哪里取，而TypeEvaluator用来控制取多少。

- 调用相机功能：

调用相机的原理是通过使用startActivityForResult来启动相机组件，拍照完成后通过onActivityResult方法可以获取到拍照得到的图像，进而可以对图像进行处理。在这里需要做一个
适配。需要判断当前android手机的版本是6.0以前的还是6.0以后的（>=6.0）,因为android6.0以后采用的是运行时的权限机制，需要在运行时由用户自行决定是否开启某项
权限（这里主要是两个权限：调用相机和访问存储空间），这就需要在代码中加入额外的逻辑。


- 人脸识别：

这里调用的是google人脸识别的api，其识别原理是先对人眼进行识别，然后再得到其余的相关点。人脸识别的过程相对比较耗时，因此我们通过使用AsyncTask将其放入异步线程中执行
防止其对主线程的阻塞。在AsyncTask类中有三个主要的方法，分别是onPreExecute（）、doInBackground（）、onPostExecute（）。首先在onPreExecute中初始化加载对话框提示用户
正在进行加载，然后将人脸识别的任务放在异步的doInBackground方法中进行执行，最后onPostExecute方法回到主线程来取消加载对话框并显示人脸识别后的图像。

![图像处理课_week2-识别人脸](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week2-%E8%AF%86%E5%88%AB%E4%BA%BA%E8%84%B8.jpg)


- app中内嵌web页面：

这里我们使用的是WebView来做的内嵌页面。通过对WebView设置加载客户端和访问Url地址可以使其显示相关网页上的内容。这里需要注意的是要开启JavaScript配置，这样显示出来的
页面才具有交互性。并且设置加载客户端时需要覆盖shouldOverrideUrlLoading方法，这样页面才能在app程序中运行，而不是调用系统的浏览器运行。最后还需要覆盖该activity
界面的onKeyDown方法，设置按下返回键时判断WebView能否返回上一页面，若能返回则返回上一页面，否则退出这个activity。

好吧，这周的工作就到此为止啦，下周将继续研究和实现java使用jni调用c算法的部分~


### week 3

由于端午过节，更新晚了。


上周完成了两个任务，一是单目摄像机的标定，主要是利用棋盘格子和opencv的`calibrateCamera()`函数，计算摄像机的内参矩阵和RT矩阵。那么对于图像平面上的二维点，可以求出对应的三维坐标，找到物体平面和图像平面之间的关系，如图所示：

![week3-平面](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week3-%E5%B9%B3%E9%9D%A2.png)


可以看出，与视线垂直的棋盘面上的物体是垂直于棋盘面的。

另外一个就是以棋盘为参照物，在棋盘旁放置物体，可以找到该物体上的角点，用于后面的三维重建：

![week3-棋盘格](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week3-%E6%A3%8B%E7%9B%98%E6%A0%BC.png)



### week 4

俗话说的好，deadline是第一生产力。这周我们完成了PC端的简单demo以及android端的程序，演示视频已在[Video](#video)部分更新。


- PC部分

在之前阅读了一些参考文献后，对3d物体追踪的步骤有了基本了解，但由于对CAD等3维模型不熟悉，加上时间比较紧迫，我们降低了实验难度和实现结果的预期，对实验条件进行了简化(如:采用规则物体)并只使用长方体框出物体并进行姿态估计，而不是做到实时勾勒出物体的轮廓。

效果如图(tracking的框画漏了)，具体可看视频部分：

![PC](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week4-PC.png)


实现思路是选择一帧图像作为参考，具体而言就是，在棋盘图像上放置物体，利用棋盘对摄像头进行标定，并计算出棋盘坐标系。然后找出图中物体的特征点，计算ORB特征描述。对物体的特征点进行kmeans聚类，可以得到物体的中心点，在中心点处的棋盘坐标系上以长方体给出物体的姿态估计。

后面对物体的追踪部分就是找出物体特征点，计算其ORB特征描述(正好我们组之前论文阅读汇报作业里读的论文就是`BRIEF`，也算学以致用吧)，并与参考帧进行特征点匹配。计算匹配特征点之间的单应性矩阵，然后对初始姿态的长方体角点进行变换，更新物体姿态。


环境配置：Python 2.7.6 + openCV 2.4.10，Ubuntu 14.04




- Android部分

本来计划是在PC端实现算法，然后封装起来，在移动端通过JNI调用，但由于时间不够App最后的功能实现使用的是Vuforia的SDK中的算法（也是基于openGL实现的，封装成接口，可以直接调用）。

3D物体追踪主要涉及两个方面：

1. 物体的特征提取
2. 根据1中得到的物体特征进行物体识别和追踪

其中物体的特征识别使用的是Vuforia的Scanner软件，将物体放在定标纸可以定标的区域，然后对物体进行360度的拍摄扫描得到物体的特征信息如图（示例中扫描的物体为一个鼠标）

![scanner](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week4-scanner.png)


特征信息会保存到一个od文件中。在Vuforia的开发者平台上，开发者可以创建自己的数据库，在数据库中添加刚刚得到的od文件，之后便可以下载得到这个od文件对应的xml数据文件，这个xml数据文件会在后面我们设计的app中用到。

接下来就是我们自己app的部分了，首先需要进行一些环境的配置：

1. 将开发者网站上下载的sdk文件放置到libs文件夹下，并在build.gradle文件依赖中设置编译这个sdk；
2. 新建一个jniLibs文件夹，将算法主体的libVuforia.so（封装了调用openGL的算法）放置到该文件夹下，在build.gradle文件配置.so文件使其可以被正确引用；
3. 将开发者网站中注册得到的license key配置到Vuforia的初始代码中；
4. 将先前得到的xml数据文件放到项目的assets文件中。

配置完成以后就是界面和实现的部分了。进入物体追踪界面时首先会进行任务的初始化，初始化包括：

1. 初始化框住物体的openGL view，这个view是基于android的GLSurfaceView实现的；
2. 渲染器的初始化，这里的渲染器是基于GLSurfaceView里的内部类Renderer来实现的；
3. Vuforia任务初始化，初始化后将其绑定到2的渲染器上，在渲染器上通过设置Texture可以设置渲染器的渲染材质，将渲染器绑定到openGL view上。

初始化时界面背景为黑色，且中间会显示一个进度条代表正在进行初始化，初始化完毕以后显示摄像头界面并取消进度条。初始化完成以后就可开始物体的识别和追踪了。该activity设置了GestureListener实现了手势监听，单击屏幕中要追踪的物体则会显示一个正方体块将目标物体包住（这里需要借助定标纸），与此同时会初始化一个Object Tracker来跟踪物体实时位置变化使包络能和物体一起移动。现在拖动定标纸，就可以看到正方形包络和物体一起发生移动，效果如下图，具体可看视频部分。


![android](http://7xph6d.com1.z0.glb.clouddn.com/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E8%AF%BE_week4-Android.png)
