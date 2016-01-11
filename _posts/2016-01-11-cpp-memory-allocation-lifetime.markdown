---
layout: post
title:  一个例子记住C++对象的生存周期
date:   2016-01-11 22:41:10
category: c++
tags: [c++, examples]
comments: true
---

* content
{:toc}


最近要考C++，复习过程中遇到一些问题，总结记录一下。文中代码均在[ideone在线编译器](http://ideone.com)中运行的。

## 代码
代码说明：

- 类A，含构造函数和析构函数
- 普通函数fun，函数体里新建了类A的局部自动对象`FunObj`和局部静态对象`InStaObj`
- main方法新建了类A的局部自动对象`MainObj`,调用`fun`方法
- 外面新建了A的的外部静态对象`ExStaObj`和外部对象`GblObj`

~~~cpp
#include <iostream>
#include <string.h>
using namespace std;
class A {
  char string[50];
public :
  A(char * st);
  ~A( );
};

A::A(char * st)
{ 
   strcpy(string, st);
   cout << string << "被创建时调用构造函数 ! " << endl;
}
A::~A( )
{  
	cout << string << 
    "被撤消时调用析构函数 ! " << endl;
}


void fun( )
{ 
	cout << "在fun( )函数体内 : \n" << endl; 
	A FunObj("fun( )函数体内的自动对象FunObj");
			  
	static A InStaObj("内部静态对象InStaObj");
}

int main( )
{ 
	A MainObj("主函数体内的自动对象MainObj");
	cout<<"主函数体内，调用fun()函数前: \n";
	fun( );
	cout << "\n主函数体内，调用fun()函数后:\n";
	return 0;
}

static A ExStaObj("外部静态对象ExStaObj");
A GblObj("外部对象GblObj"); 
~~~

输出：

~~~
外部静态对象ExStaObj被创建时调用构造函数 ! 
外部对象GblObj被创建时调用构造函数 ! 
主函数体内的自动对象MainObj被创建时调用构造函数 ! 
主函数体内，调用fun()函数前: 
在fun( )函数体内 : 

fun( )函数体内的自动对象FunObj被创建时调用构造函数 ! 
内部静态对象InStaObj被创建时调用构造函数 ! 
fun( )函数体内的自动对象FunObj被撤消时调用析构函数 ! 

主函数体内，调用fun()函数后:
主函数体内的自动对象MainObj被撤消时调用析构函数 ! 
内部静态对象InStaObj被撤消时调用析构函数 ! 
外部对象GblObj被撤消时调用析构函数 ! 
外部静态对象ExStaObj被撤消时调用析构函数 ! 
~~~

若将`A GblObj("外部对象GblObj"); `写在`static A ExStaObj("外部静态对象ExStaObj");`前面，则输出时两者顺序也颠倒。

## 分析

- 创建顺序
  
  外部静态对象or外部对象优先于main函数

- 销毁顺序
  
  和创建顺序相反，**注意**静态对象会在main函数执行完才会销毁

  

## 内存的三种分配方式

- 从静态存储区分配：此时的内存在程序编译的时候已经分配好，并且在程序的整个运行期间都存在。全局变量，static变量等在此存储
- 在栈区分配：相关代码执行时创建，执行结束时被自动释放。局部变量在此存储。栈内存分配运算内置于处理器的指令集中，效率高，但容量有限
- 在堆区分配：动态分配内存。用new/malloc时开辟，delete/free时释放。生存期由用户指定，灵活。但有内存泄露等问题


![内存分配](http://7xph6d.com1.z0.glb.clouddn.com/cpp_%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D.png)
