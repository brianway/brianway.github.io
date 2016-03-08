---
layout: post
title:  简单谈谈C++中的函数形参与浅拷贝
date:   2016-01-11 21:35:10 +08:00
category: cpp
tags: [cpp, examples]
comments: true
---

* content
{:toc}



最近要考C++，复习过程中遇到一些问题，总结记录一下。文中代码均在[ideone在线编译器](http://ideone.com)中运行的

字面上都知道，函数传递参数有值传递和引用传递，但具体区别是什么呢？除了一个传对象拷贝，一个传对象本身之外，还有哪些影响？

这里定义一个`str`类，只有一个`private char*st`变量；有几个基本的函数，重载了`=`和`==`运算符，`str & operator=(str const & a)`和`str & operator==(str a)`,用于用不同方式实现赋值。代码如下：

## 代码

~~~cpp
#include <iostream> 
#include <string.h> 
using namespace std; 
class str 
{
private: 
	char *st; 
public:  
  str(char *a) {
	set(a);
	cout<<"构造函数：str addr "<<this<<"  st addr "<<&st<<"  st point to  "<<(void *)st<<"  st  "<<st<<endl; 
  }  
  str & operator==(str  a) {//二元操作符“==”，参数类型和后者一致
	cout<<"op:形参 str a addr  "<<&a<<"  a.st addr  "<<&(a.st)<<"  a.st point to  "<<(void *)(a.st)<<" a.st content  "<<(a.st)<<endl;
	delete st; 
	set(a.st);
	return *this;
  }  
  str & operator=(str const &  a) {//二元操作符“=”，参数类型和后者一致
   //str & operator=(str  &  a) {//二元操作符“=”，参数类型和后者一致
	cout<<"op:形参 str const & a addr  "<<&a<<"  a.st addr  "<<&(a.st)<<"  a.st point to  "<<(void *)(a.st)<<" a.st content  "<<(a.st)<<endl;
	delete st; 
	set(a.st);
	return *this;
  }  

  void show(){cout<<"show func: "<<st<<endl;} 
  ~str(){
	cout<<"~str:before str addr "<<this<<"  st addr "<<&st<<"  st point to  "<<(void *)(st)<<"  st content "<<st<<endl; 
	delete st;
	//cout<<"~str:after  str addr "<<this<<"  st addr "<<&st<<"  st point to  "<<(void *)(st)<<"  st content "<<st<<endl; 
  }  
  void set(char *s)//初始化st 
  {   
	cout<<"set:before new st addr  "<<&st<<"  st point to  "<<(void *)(st)<<endl;
	st = new char[strlen(s)+1];//先分配空间，否则野指针,+1存"\0"
	cout<<"set:after new st addr  "<<&st<<"  st point to  "<<(void *)(st)<<endl;
	if(st)
	strcpy(st,s); 
  } 
};  

int  main()  
{
	str s1("he"),
	s2("she"); 
	s1.show(),
	s2.show(); 
	//s2=s1;  
	s2==s1;
	s1.show(),
	s2.show();
	return 0;
} 
~~~

-  使用`str & operator==(str  a)`重载的输出：

~~~
set:before new st addr  0xbfce5374  st point to  0xb785c680
set:after new st addr  0xbfce5374  st point to  0x9bffa10
构造函数：str addr 0xbfce5374  st addr 0xbfce5374  st point to  0x9bffa10  st  he
set:before new st addr  0xbfce5378  st point to  0x804abc4
set:after new st addr  0xbfce5378  st point to  0x9bffa20
构造函数：str addr 0xbfce5378  st addr 0xbfce5378  st point to  0x9bffa20  st  she
show func: he
show func: she
op:形参 str a addr  0xbfce537c  a.st addr  0xbfce537c  a.st point to  0x9bffa10 a.st content  he
set:before new st addr  0xbfce5378  st point to  0x9bffa20
set:after new st addr  0xbfce5378  st point to  0x9bffa20
~str:before str addr 0xbfce537c  st addr 0xbfce537c  st point to  0x9bffa10  st content he
show func: 
show func: he
~str:before str addr 0xbfce5378  st addr 0xbfce5378  st point to  0x9bffa20  st content he
~str:before str addr 0xbfce5374  st addr 0xbfce5374  st point to  0x9bffa10  st content 
~~~

注意第二次两行`show`之前的那句`~str:before str addr 0xbfce537c  st addr 0xbfce537c  st point to  0x9bffa10  st content he`

- 使用`str & operator=(str const &  a)`重载,将`main`中`s2==s1;`注释掉，使用`s2=s1;`, 输出：

~~~
set:before new st addr  0xbff47b38  st point to  0x804aba0
set:after new st addr  0xbff47b38  st point to  0x9693a10
构造函数：str addr 0xbff47b38  st addr 0xbff47b38  st point to  0x9693a10  st  he
set:before new st addr  0xbff47b3c  st point to  0x8049302
set:after new st addr  0xbff47b3c  st point to  0x9693a20
构造函数：str addr 0xbff47b3c  st addr 0xbff47b3c  st point to  0x9693a20  st  she
show func: he
show func: she
op:形参 str const & a addr  0xbff47b38  a.st addr  0xbff47b38  a.st point to  0x9693a10 a.st content  he
set:before new st addr  0xbff47b3c  st point to  0x9693a20
set:after new st addr  0xbff47b3c  st point to  0x9693a20
show func: he
show func: he
~str:before str addr 0xbff47b3c  st addr 0xbff47b3c  st point to  0x9693a20  st content he
~str:before str addr 0xbff47b38  st addr 0xbff47b38  st point to  0x9693a10  st content he
~~~

## 分析

- 使用`str & operator==(str a)`重载时，s1的地址为`0xbfce5374`，a的地址为`0xbfce537c`,确实是另外创建了新变量a，但是，两者的st均指向同一个地址`0x9bffa10`
- 使用`str & operator=(str const & a)`重载时，s1和a的地址均为`0xbff47b38`
- 两次输出的区别在于使用值传递时，即第一个输出结果多了一句输出：`~str:before str addr 0xbfce537c st addr 0xbfce537c st point to 0x9bffa10 st content he`


可以看出，主要区别在于,参数类型不是引用时，形参为值传递，**默认的复制构造函数就是简单的把成员变量依次赋值**，所以`str a`的st和`s1`的st指向的同一段内存，函数`str & operator==(str a)`执行完，自动变量a会销毁，调用析构函数，释放st所指向的内存并设指针为空，所以为null。而使用函数`str & operator=(str const & a)`，则不会代用析构函数。由于这里未对变量a做修改，所以去掉const不影响，不过最好保留。

## 关于拷贝

- 对象间赋值(=)是一个拷贝过程
- 浅拷贝
  - 实现对象间数据元素的一一对应复制。
- 深拷贝
  - 当被复制的对象数据成员是指针类型时，不是复制该指针成员本身，而是将指针所指的对象进行复制。
系统提供的拷贝(如默认拷贝构造函数等)只能实现浅拷贝，深拷贝必须自定义


----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
