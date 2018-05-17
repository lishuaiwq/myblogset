---
title: sizeof详解
date: 2018-02-24 15:42:50
tags: sizeof
categories: c/c++
---
首先对于sizeof做一个简单的介绍让你知道什么是sizoef，从心里对它有个大致的印象。
<!--more-->

# 定义： #

----------

sizeof是C/C++中的一个操作符（operator），简单的说其作用就是返回一个对象或者类型所占的内存字节数。

MSDN上的解释为：
The sizeof keyword gives the amount of storage, in bytes, associated with a variable or a type(including aggregate types). This keyword returns a value of type size_t.

其返回值类型为size_t，在头文件stddef.h中定义。这是一个依赖于编译系统的值，一般定义为

    typedef unsigned int size_t;

好了现在你应该对于sizoef有了一个大概的认识了，现在我们来看看sizoef的基本用法：

1. sizeof(对象）；
2. sizeof(类型）；
3. sizeof 对象；//这个主意和1区分

对于上面的三种用法，其实你只需要掌握sizoef()的这种用法就够了，另外一个自己知道就可以了，因为根本没人去那么用。好了划重点了：**记住sizeof计算对象的大小也是转换成对应的类型来计算的，**这句话的言外之意就是不同的对象如果类型一样的话，sizoef计算的值是一样的。

----------
# 用法： #

----------

接下来我就所说sizeof的一些应用场景和用法

## 1 sizeof对一个表达式求值

注意：对于表达式求值，编译器会根据表达式结果的类型来计算，一般不会对表达式进行计算

举例如下：

    	  int fun(int a)
	{
	cout << "fun::sizoef" << endl;
	return a;
	}
	void test()
	{
	cout << sizeof(fun(4));
	}

此例子的输出就是 4，不会输出fun::sizeof,证明函数没有执行。		

## 2.sizoef的常量性
	
sizeof的计算是放生在编译期间的，所以其可以被当做常量表达式使用

举例如下：

	int n=10;
    int arr[sizeof(n)];

## 3.sizeof和指针变量

注意：当sizoef的对象是指针变量的时候，它的结果和指针所指的内容没有任何的关系，所以在32位计算机中指针变量是4,64位计算机中是8

## 4.sizeof和数组

sizoef的数组的值等于数组所占的内存字节数，这里我在CSDN的博客写的很清楚了，感兴趣的可以自己去看
[http://blog.csdn.net/it_iverson/article/details/74733426](http://blog.csdn.net/it_iverson/article/details/74733426 "http://blog.csdn.net/it_iverson/article/details/74733426")

这里还要强调的一个重点就是数组传参的时候。

	void fun(int arr[])
	{
	cout << sizeof(arr);
	}
	void test()
	{
	int arr[10] = { 0 };
	fun(arr);
	}

这里的sizoef的输出是4而不是40，因为在函数的形参那里arr已经不是数组类型了，而是蜕变成了指针。相当于int *arr,因为数组是传址传参的，只是把数组首地址传过去了，所以接受地址的自然就是指针变量了，那么正如上文所说，指针变量的大小和其指向的内容没有关系，所以大小为4

## 5.sizeof和结构体

这里主要强调的是结构体内存对其的一些知识在CSDN的博客中也有详细说明

[http://blog.csdn.net/it_iverson/article/details/74790127](http://blog.csdn.net/it_iverson/article/details/74790127 "http://blog.csdn.net/it_iverson/article/details/74790127")

## 6.含有位字段的结构体

   注意：单独的位字段成员不能被sizeof求值

那么什么是为字段呢？

位字段是C语言中一种存储结构，不同于一般结构体的是它在定义成员的时候需要指定成员所占的位数。主要应用于嵌入开发

如下：

	struct stu
	{
	char a : 4;//占4位
	char b : 3;//占3位
	char  c : 8;//占8位
	};//大小为两个字节

在这里要注意的是：
 
1.如果相邻的位字段的类型相同，且其位数之和小于自身类型数，则后面的字段和紧挨着前面的字段存储，直到这个不能容纳为止

2.如果相邻的位字段类型相同，且其位数之和大于自身的类型数，则后面的字段从新的存储单元开始。

3.如果相邻位字段类型不同，则根据编译器的不同，是否采取压缩存储就不一定了。

4.如果位字段之间穿插着非为字段，则不进行压缩存储
 
## 7.sizeof和联合体

因为联合体是重叠式存储，各成员共享一段内存，所以整个联合体的大小就是最大成员所占的空间的大小，假如联合体中有一个结构体成员，那么这个联合体的大小就是这个结构体的大小。

----------






