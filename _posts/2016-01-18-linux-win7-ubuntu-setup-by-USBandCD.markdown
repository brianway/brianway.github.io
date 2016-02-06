---
layout: post
title:  光盘和U盘安装win7和ubuntu14.04全步骤
date:   2016-01-18 18:36:10 +08:00
category: linux
tags: [linux, ubuntu]
comments: true
---

* content
{:toc}



说来惭愧，作为一个学IT的，居然自己没重装过系统。一直想自己安装一次双系统，正好这个周日抽空研究了下，折腾了一天，总算如愿以偿。



## 写在前面

- 本文是先安装windows再安装linux，并通过windows引导linux的启动项。这样方便linux的反复重装、折腾等。
- 光盘安装和U盘安装基本差不多，只是U盘安装多了把镜像文件写到U盘制成启动盘的过程，启动时对应选择*从USB启动*/*从CD-ROM启动*即可
- 由于我有win7的光盘，就没研究怎么刻录win7到U盘
- 关于每一步的截图，文末的【参考资料】里别人已经截的很完善了，我就没重复造轮子(*知乎那个李彬的答案关于EasyBCD使用有问题，那是硬盘启动，而非U盘，U盘没那么麻烦*)
- 该文只是综合各个参考资料，按照安装顺序、更全面的把每一步需要注意的关键点用文字整理记录
- 关于原理部分，可以看《鸟叔私房菜-基础学习篇》(第三版)第3、4章相关内容，里面对MBR、分区表、引导加载程序的工作执行等有深入浅出的讲解

------

## 需要的工具

>* [驱动精灵](http://www.drivergenius.com/)--用于安装win7的各种驱动，建议下载**万能网卡版**，把网络解决了才方便后面在线下载其他驱动
>* [WIN7 Activation(Win7激活工具) 1.8 绿色免费版](http://www.xiazaizhijia.com/soft/6387.html)--用于激活win7
>* [EasyBCD](http://www.softpedia.com/get/System/OS-Enhancements/EasyBCD.shtml) （[Community Edition](http://www.softpedia.com/get/System/OS-Enhancements/EasyBCD.shtml) / [中文版下载](http://www.onlinedown.net/soft/58174.htm)）--用于设置由windows引导ubuntu的启动加载程序,
>* [Universal-USB-Installer](http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/#button)--用于制作ubuntu的U盘启动盘
>* [Ubuntu Desktop](http://www.ubuntu.com/download/desktop) (附[kylin(优麒麟)版](http://www.ubuntu.com/download/ubuntu-kylin))--Ubuntu的ISO镜像文件,用于制作U盘启动盘



------


## 步骤概述

- **win7安装**

1. 插入光盘，重启电脑，狂按`F12`
2. 选择从CD-ROM启动
3. 点安装，按提示进行就行了
4. 用驱动精灵安装各种驱动
5. 下载其他普通软件


- **ubuntu 14.04安装**

1. 刻录驱动盘,刻好后插入
2. 重启电脑，狂按`F12`，选择USB启动
3. 按照提示选择语言、分区、键盘布局等
4. 重启电脑，在win7下添加启动菜单
5. 重启电脑，选ubuntu，安装ubuntu的一些软件

------

## 详细说明

这里对上面**【步骤概述】**相应步骤中需要注意的地方进行说明

###  win7安装

第1，2两步

- 没什么好说的。我是戴尔的电脑，不同电脑进入启动界面的按键可能不同，戴尔的是`F12`,其他电脑的可能是`Delete`、`F2`、`F10`等
- 在BIOS里面，硬盘有时候不是 Hard Deice，有时候是HDD等，光驱不一定是CDROM，有时候是DVD-;

第3步

- 提示"进行何种类型的安装"->选择“自定义(高级)”；
- 分区时，如果不想影响其他盘的话，直接点系统盘(C盘)然后下一步即可，这时其他盘数据不会被影响；如果想重新分区，这里可以分区、格式化。点完就直接开始安装了。
- 需要注意的是，我不知道为什么，点分区只能分主分区，不能分逻辑分区。所以如果要安装双系统的话，需要留一个分区给linux，分区时主分区最多3个。
- 记得给linux留点硬盘空间，我这里留了100G
- 期间系统会自动重启，等进入到设置用户名密码、产品密钥等就说明好了，设置完就OK了。这里密钥能填正确最好，填失败了的话，后面还可以通过激活工具激活。
- 关于分区，其实可以在重装系统前进行，"开始->计算机->右键->管理->存储->磁盘管理"

第4步

- 把驱动精灵用U盘拷到电脑上，安装驱动精灵，然后先安装网卡驱动，能连网了再安装别的驱动。期间可能会多次询问重启，选择稍后重启，都安完了一次性重启

第5步

- 安装其他的软件，比如浏览器啊，上面提到的EasyBCD啊等


### ubuntu安装


第1步

- 下载ubuntu镜像，用上面提到的工具刻录U盘(格式化成FAT32的格式)，详情见文末的【参考链接4】
- 依次选择ubuntu版本，镜像文件，要写入的U盘。最后那个可选项可以不管，默认0MB


第2步 

- 没什么特别的

第3步
 
- 选择第二项，安装ubuntu
- 选择语言，中文往下拉，在后面
- 选择“安装这个第三方软件”，其他不管
- 联网可选，有wifi这时可以先连
- **重要**：选择最后一个"其他选项"
- **重要**：分区。这里每个人不一样,我分配的大小都比较富裕，顺序是按表中由上到下分配的。关于分区，可以参考[Linux系统安装时分区的选择](http://www.cnblogs.com/gylei/archive/2011/12/04/2275987.html)

| 挂载点   | 大小     |  类型   |新分区位置  | 用于           |
| -------- | :-----:  | :----:  |  :----:    |:----:          |
| /boot    | 200M     | 逻辑分区|空间起始位置|EXT4日志文件系统|
| /        | 20G      | 主分区  |空间起始位置|EXT4日志文件系统|
| 不设置   | 2G       | 逻辑分区|空间起始位置|     交换空间   |
| /home    | 60G      | 逻辑分区|空间起始位置|EXT4日志文件系统|


- **重要**：上一步中，要记住/boot的设备号，比如我的是`/dev/sda6`,下面的"安装启动引导器的设备"->选择/boot所在的分区。这里不要选错，不然就是linux引导windows

- 后面无非就是一些用户设置了，没什么难的，键盘布局就选英文(美国)就行

第4步

- 安装好后ubuntu，重启电脑，进入win7设置启动菜单
- 打开EasyBCD，"添加新条目->linux/BSD->类型:GRUN(Legacy),设备/驱动器:刚才/boot对应的分区->点添加"
- /boot的分区是以linux开头的，不记得就看大小，比如我的是200M
- 可以在工具栏的"编辑引导菜单"查看启动菜单

第5步

- 首先需要更新一下依赖：

~~~shell
sudo apt-get update
sudo apt-get upgrade
~~~

- 校园网认证使用mentohust,可以在linux公社下载，[mentohust下载地址](http://linux.linuxidc.com/2013%E5%B9%B4%E8%B5%84%E6%96%99/1%E6%9C%88/20%E6%97%A5/Ubuntu%E4%B8%8B%E4%BD%BF%E7%94%A8MentoHUST%E4%BB%A3%E6%9B%BF%E9%94%90%E6%8D%B7%E8%AE%A4%E8%AF%81%E4%B8%8A%E7%BD%91/),用户和密码都是`www.linuxidc.com`
- 下载地址位于`http://linux.linuxidc.com/`的"/2013年资料/1月/20日/Ubuntu下使用MentoHUST代替锐捷认证上网"，或者"pub/ubuntu/Ubuntu 11.04校园网锐捷认证上网"下
- 可参考[Ubuntu下使用MentoHUST搞定 锐捷校园网认证网络](http://www.linuxidc.com/Linux/2013-10/91157.htm)和[Ubuntu下Mentohust的配置](http://www.linuxidc.com/Linux/2013-10/91158.htm)这两篇文章

------

## 参考资料

>1. [知乎：怎样安装 Windows 7 与 Linux 的双系统？](https://www.zhihu.com/question/19867618)
>2. [linux公社：Ubuntu 12.04和Windows 7双系统安装图解](http://www.linuxidc.com/Linux/2012-05/59663.htm)
>3. [Win7下安装CentOS 6.5双系统](http://blog.sina.com.cn/s/blog_86e874d30101e3d8.html)
>4. [win7下制作ubuntu系统安装启动盘和U盘安装ubuntu全过程](http://blog.csdn.net/liangcaiyun2013/article/details/10410797)
>5. [重装Win7 系统(用光盘重装Win7系统)_百度经验](http://jingyan.baidu.com/article/597035520848d98fc00740f1.html)









