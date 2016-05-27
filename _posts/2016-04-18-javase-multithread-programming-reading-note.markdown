---
layout: post
title:  java多线程核心技术梳理(附源码)
date:   2016-04-18 16:18:01 +08:00
category: 编程语言
tags: java 多线程 总结
comments: true
---

* content
{:toc}

本文对多线程基础知识进行梳理，主要包括多线程的基本使用，对象及变量的并发访问，线程间通信，lock的使用，定时器，单例模式，以及线程状态与线程组。




## 写在前面

花了一周时间阅读《java多线程编程核心技术》(高洪岩 著)，本文算是此书的整理归纳，书中几乎所有示例，我都亲手敲了一遍，并上传到了我的github上，有兴趣的朋友可以到我的github下载。源码采用maven构建，多线程这部分源码位于`java-multithread`模块中。

>* 仓库地址：[java-learning](https://github.com/brianway/java-learning)
>* git clone： `git@github.com:brianway/java-learning.git`


## java多线程

基础知识

- 创建线程的两种方式：1.继承Thread类，2.实现Runnable接口。具体两者的联系可以参考我之前的博文[《java基础巩固笔记(5)-多线程之传统多线程》](http://blog.csdn.net/h3243212/article/details/50659389)
- 一些基本API：isAlive(),sleep(),getId(),yield()等。
   - `isAlive()`测试线程是否处于活动状态
   - `sleep()`让“正在执行的线程”休眠
   - `getId()`取得线程唯一标识
   - `yield()`放弃当前的CPU资源
- 弃用的API:`stop()`,`suspend()`,`resume()`等，已经弃用了，因为可能产生数据不同步等问题。
- 停止线程的几种方式：
  - 使用退出标识，使线程正常退出，即run方法完成。
  - 使用interrupt方法中断线程
- 线程的优先级:继承性，规则性，随机性
  - 线程的优先级具有继承性. 如,线程A启动线程B，则B和A优先级一样
  - 线程的优先级具有规则性. CPU尽量倾向于把资源优先级高的线程
  - 线程的优先级具有随机性. 优先级不等同于执行顺序，二者关系不确定
- java中的两种线程：用户线程和守护(Daemon)线程。
  - 守护线程：进程中不存在非守护线程时，守护线程自动销毁。典型例子如：垃圾回收线程。 

比较和辨析

- 某个线程与当前线程：当前线程则是指正在运行的那个线程，可由`currentThread()`方法返回值确定。例如，直接在main方法里调用run方法，和调用线程的start方法，打印出的当前线程结果是不同的。
- `interrupted()`和`isInterrupted()`
  - `interrupted()`是类的静态方法，测试当前线程是否已经是中断状态，执行后具有将状态标志清除为false的功能。
  - `isInterrupted()`是类的实例方法，测试Thread对象是否已经是中断状态，但不清楚状态标志。
- `sleep()`和`wait()`区别：
  - sleep()是Thread类的static(静态)的方法；wait()方法是Object类里的方法
  - sleep()睡眠时，保持对象锁，仍然占有该锁；wait()睡眠时，释放对象锁
  - 在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级；wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程
  - wait()必须放在synchronized block中，否则会在runtime时扔出`java.lang.IllegalMonitorStateException`异常

 

| 方法|	是否释放锁|	备注|
|:----:|:----:|:----:|
|wait |	是	|wait和notify/notifyAll是成对出现的, 必须在synchronize块中被调用|
|sleep|	否	|可使低优先级的线程获得执行机会|
|yield|	否	|yield方法使当前线程让出CPU占有权, 但让出的时间是不可设定的|
  
 
## 对象及变量的并发访问

- `synchronized`关键字
  - 调用用关键字synchronized声明的方法是排队运行的。但假如线程A持有某对象的锁，那线程B异步调用非synchronized类型的方法不受限制。
  - synchronized锁重入:一个线程得到对象锁后，再次请求此对象锁时是可以得到该对象的锁的。同时，子类可通过“可重入锁”调用父类的同步方法。
  - 同步不具有继承性。
  - synchronized使用的“对象监视器”是一个，即必须是同一个对象
- synchronized同步方法和synchronized同步代码块。
  - 对其他synchronized同步方法或代码块调用呈阻塞状态。
  - 同一时间只有一个线程可执行synchronized方法/代码块中的代码
- synchronized(非this对象x)，将x对象作为“对象监视器”
  - 当多个线程同时执行`synchronized(x){}`同步代码块时呈同步效果
  - 当其他线程执行x对象中synchronizd同步方法时呈同步效果
  - 当其他线程执行x对象方法里的synchronized(this)代码块时呈同步效果
- 静态同步synchronized方法与synchronized(class)代码块：对当前对应的class类进行持锁。

	
*线程的私有堆栈图*

![javaSE_多线程-线程的私有堆栈](http://7xph6d.com1.z0.glb.clouddn.com/javaSE_%E5%A4%9A%E7%BA%BF%E7%A8%8B-%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%A7%81%E6%9C%89%E5%A0%86%E6%A0%88.png)


- volatile关键字：主要作用是使变量在多个线程间可见。**加volatile关键字可强制性从公共堆栈进行取值,而不是从线程私有数据栈中取得变量的值**
   - 在方法中while循环中设置状态位(不加volatile关键字)，在外面把状态位置位并不可行，循环不会停止，比如JVM在-server模式。
   - 原因：是私有堆栈中的值和公共堆栈中的值不同步
   - volatile增加了实例变量在多个线程间的可见性，但不支持原子性
- 原子类:一个原子类型就是一个原子操作可用的类型，可在没有锁的情况下做到线程安全。但原子类也不是完全安全，虽然原子操作是安全的，可方法间的调用却不是原子的，需要用同步。


*读取公共内存图*

![javaSE_多线程-读取公共内存.png](http://7xph6d.com1.z0.glb.clouddn.com/javaSE_%E5%A4%9A%E7%BA%BF%E7%A8%8B-%E8%AF%BB%E5%8F%96%E5%85%AC%E5%85%B1%E5%86%85%E5%AD%98.png)

辨析和零散补充

- synchronized静态方法与非静态方法：synchronized关键字加static静态方法上是给Class类上锁，可以对类的所有实例对象起作用；synchronized关键字加到非static静态方法上是给对象上锁，对该对象起作用。这两个锁不是同一个锁。
- synchronized和volatile比较
  -  1)关键字volatile是线程同步的轻量级实现，性能比synchronized好，且volatile只能修饰变量，synchronized可修饰方法和代码块。
  -  2)多线程访问volatile不会发生阻塞，synchronized会出现阻塞
  -  3)volatile能保证数据可见性，不保证原子性;synchronized可以保证原子性，也可以间接保证可见性，因为**synchronized会将私有内存和公共内存中的数据做同步**。
  - 4)volatile解决的是变量在多个线程间的可见性，synchronized解决的是多个线程访问资源的同步性。
- String常量池特性，故大多数情况下，synchronized代码块都不适用String作为锁对象。
- 多线程死锁。使用JDK自带工具，jps命令+jstack命令监测是否有死锁。
- 内置类与静态内置类。
- 锁对象的的改变。
- 一个线程出现异常时，其所持有的锁会自动释放。

*变量在内存中的工作过程图*

![javaSE_多线程-变量在内存中的工作过程.png](http://7xph6d.com1.z0.glb.clouddn.com/javaSE_%E5%A4%9A%E7%BA%BF%E7%A8%8B-%E5%8F%98%E9%87%8F%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.png)


## 线程间通信

- 等待/通知机制：`wait()`和`notify()`/`notifyAll()`。wait使线程停止运行，notify使停止的线程继续运行。
  - `wait()`：将当前执行代码的线程进行等待，置入"预执行队列"。
     - 在调用wait()之前，线程必须获得该对象的对象级别锁；
     - 执行wait()方法后，当前线程立即释放锁；
     - 从wait()返回前，线程与其他线程竞争重新获得锁
     - 当线程呈wait()状态时，调用线程的interrup()方法会出现InterrupedException异常
     - `wait(long)`是等待某一时间内是否有线程对锁进行唤醒，超时则自动唤醒。
  - `notify()`：通知可能等待该对象的对象锁的其他线程。随机挑选一个呈wait状态的线程，使它等待获取该对象的对象锁。
     - 在调用notify()之前，线程必须获得该对象的对象级别锁；
     - 执行完notify()方法后，不会马上释放锁，要直到退出synchronized代码块，当前线程才会释放锁。
     - notify()一次只随机通知**一个**线程进行唤醒
  - `notifyAll()`和`notify()`差不多，只不过是使所有正在等待队中等待同一共享资源的“全部”线程从等待状态退出，进入可运行状态。
- 每个锁对象有两个队列：就绪队列和阻塞队列。
   - 就绪队列：存储将要获得锁的线程
   - 阻塞队列：存储被阻塞的的线程
- 生产者/消费者模式
   - “假死”：线程进入WAITING等待状态，呈假死状态的进程中所有线程都呈WAITING状态。
     - 假死的主要原因：有可能连续唤醒同类。notify唤醒的不一定是异类，也许是同类，如“生产者”唤醒“生产者”。 
     - 解决假死：将notify()改为notifyAll()
  - wait条件改变，可能出现异常，需要将if改成while
- 通过管道进行线程间通信：一个线程发送数据到输出管道，另一个线程从输入管道读数据。
  - 字节流：`PipedInputStream`和`PipedOutputStream`
  - 字符流：`PipedReader`和`PipedWriter`
- `join()`：等待线程对象销毁，具有使线程排队运行的作用。
  - join()与interrupt()方法彼此遇到会出现异常。
  - `join(long)`可设定等待的时间
- `join`与`synchronized`的区别：join在内部使用wait()方法进行等待;synchronized使用的是“对象监视器”原理作为同步
- `join(long)`与`sleep(long)`的区别：join(long)内部使用wait(long)实现，所以join(long)具有释放锁的特点;Thread.sleep(long)不释放锁。
- `ThreadLocal`类：每个线程绑定自己的值
  -  覆写该类的`initialValue()`方法可以使变量初始化，从而解决get()返回null的问题
  -  `InheritableThreadLocal`类可在子线程中取得父线程继承下来的值。


## Lock的使用

- `ReentrantLock`类：实现线程之间的同步互斥，比synchronized更灵活
  - `lock()`，调用了的线程就持有了“对象监视器”，效果和synchronized一样
- 使用`Condition`实现等待/通知：比wait()和notify()/notyfyAll()更灵活，比如可实现多路通知。
  - 调用condition.await()前须先调用lock.lock()获得同步监视器

Object与Condition方法对比

|Object| Condition|
|:---: |:---:| 
|wait()|await()|
|wait(long timeout)|await(long time,TimeUnit unit)|
|notify()|signal()|
|notifyAll()|signalAll()|



一些API

| 方法|	说明|
|:----|:----|
| `int getHoldCount()`|查询当前线程保持此锁定的个数，即调用lock()方法的次数|
| `int getQueueLength()`|返回正在等待获取此锁定的线程估计数|
| `int getWaitQueueLength(Condition condition)`|返回等待与此锁定相关的给定条件Conditon的线程估计数|
| `boolean hasQueueThread(Thread thread)`|查询指定的线程是否正在等待获取此锁定|
| `boolean hasQueueThreads()`|查询是否有线程正在等待获取此锁定|
| `boolean hasWaiters(Condition)`|查询是否有线程正在等待与此锁定有关的condition条件|
| `boolean isFair()`|判断是不是公平锁|
| `boolean isHeldByCurrentThread()`|查询当前线程是否保持此锁定|
| `boolean isLocked()`|查询此锁定是否由任意线程保持|
| `void lockInterruptibly()`|如果当前线程未被中断，则获取锁定，如果已经被中断则出现异常|
| `boolean tryLock()`|仅在调用时锁定未被另一个线程保持的情况下，才获取该锁定|
| `boolean tryLock(long timeout,TimeUnit unit)`|如果锁定在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁定|


- 公平锁与非公平锁
   - 公平锁表示线程获取锁的顺序是按照加锁的顺序来分配的，即FIFO先进先出。
   - 非公平锁是一种获取锁的抢占机制，随机获得锁。
- `ReentrantReadWriteLock`类
   - 读读共享
   - 写写互斥
   - 读写互斥
   - 写读互斥


## 定时器

常用API

|方法|说明|
|:---|:----|
|schedule(TimerTask task, Date time)|在指定的日期执行某一次任务|
|scheduleAtFixedRate(TimerTask task, Date firstTime, long period) |在指定的日期之后按指定的间隔周期，无限循环的执行某一任务|
|schedule(TimerTask task, long delay)|以执行此方法的当前时间为参考时间，在此时间基础上延迟指定的毫秒数后执行一次TimerTask任务|
|schedule(TimerTask task, long delay, long period)|以执行此方法的当前时间为参考时间，在此时间基础上延迟指定的毫秒数，再以某一间隔时间无限次数地执行某一TimerTask任务|


- `schedule`和`scheduleAtFixedRate`的区别:schedule不具有追赶执行性;scheduleAtFixedRate具有追赶执行性



## 单例模式与多线程

- 立即加载/“饿汉模式”：调用方法前，实例已经被创建了。通过静态属性new实例化实现的
- 延迟加载/“懒汉模式”：调用get()方法时实例才被创建。最常见的实现办法是在get()方法中进行new实例化
  - 缺点：多线程环境中，会出问题
  - 解决方法
     - 声明synchronized关键字，但运行效率非常低下
     - 同步代码块，效率也低
     - 针对某些重要代码(实例化语句)单独同步，效率提升，但会出问题
     - 使用DCL双检查锁
     - 使用enum枚举数据类型实现单例模式


## 拾遗补增


*方法与状态关系示意图*

![javaSE_多线程-方法与状态关系示意图.png](http://7xph6d.com1.z0.glb.clouddn.com/javaSE_%E5%A4%9A%E7%BA%BF%E7%A8%8B-%E6%96%B9%E6%B3%95%E4%B8%8E%E7%8A%B6%E6%80%81%E5%85%B3%E7%B3%BB%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

- 线程的状态：`Thread.State`枚举类,参考官网API[Enum Thread.State](https://docs.oracle.com/javase/8/docs/api/index.html?java/lang/Thread.State.html) 
- 线程组：线程组中可以有线程对象，也可以有线程组，组中还可以有线程。可批量管理线程或线程组对象。
- `SimpleDateFormat`非线程安全，解决办法有：
  - 创建多个SimpleDateFormat类的实例
  - 使用ThreadLocal类
- 线程组出现异常的处理
  - `setUncaughtExceptionHandler()`给指定线程对线设置异常处理器
  - `setDefaultUncaughtExceptionHandler()`对所有线程对象设置异常处理器






## 参考资料

>* [浅谈Java中的锁](http://zhwbqd.github.io/2015/02/13/lock-in-java.html)
>* [java synchronized关键字的用法](http://zhh9106.iteye.com/blog/2151791)
>* [Java Thread(线程)案例详解sleep和wait的区别](http://www.cnblogs.com/DreamSea/archive/2012/01/16/2263844.html)




----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)

