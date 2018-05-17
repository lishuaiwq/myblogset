---
title: 深度探索c++对象模型-第四章Function语意学
date: 2018-02-26 20:52:36
tags: function语意学
categories: 读书笔记

---
此篇文章主要的内容是向我们介绍C++中的一些函数的调用方式，比如成员函数，非常成员函数，静态成员函数，虚函数等等，其中每一种类型的函数的调用方式都不相同。并且除了说明此章节的主题意外还告诉我们了一个小知识点,那就是<!--more-->:①静态成员函数不可能直接存取非静态成员变量，因为它没有this指针，②静态成员函数不可能被声明为const，因为const修饰函数是防止他去修改成员变量的值，而静态成员函数根本不能访问成员，所以修饰它无任何意义，所以规定不能用const 修饰静态成员函数。
## 1.非静态成员函数
作者告诉我们设计非静态成员函数的最起码这个函数需要和一般的非成员函数有同样的效率，假如有如下的两个函数

	float magnitude3d(const Point3d *_this){..};//Point3d是类名
	float Point3d::magnitude3d() const {..};
我们看起来是不是成员函数相对来说没有 带来什么负担，反而效率似乎还能高一点，因为非成员函数中还需要经形参取值才能运用成员呢。其实吧，成员函数看着小清新，其实在真正使用的时候，成员函数也是被内化非成员函数的形式了，下面作者就介绍了这个内化的过程！

1.

改写成员函数的原型，给其安插一个额外的参数到成员函数中，用来提供成员的操作，使得类对象可以将这个函数调用，而这个额外的参数就被称作this指针！代码如下：

	float Point3d::magnitude3d(Point3d *const this) const {..};
如果说成员函数是const的，则会变为:

	float Point3d::magnitude3d(const Point3d *const this) const {..};

2.

将对成员的存取操作变成这个样子

	{
   		this->_x*this->y;
	}

3.将成员函数重新写成一个外部函数（全局函数）。将函数名经过"mangling"处理，使他在程序中独一无二。（这里mangling就是给函数的变量和函数名字经过编译器自己的一些算法，重新起一个独一无二名字，这么做是为了支持C++重载）。致此，函数就内化完成了

	extern magnitude_7Point3dFv(register Point3d* const this);

既然函数都被改的飞起来了，那么函数的调用毋庸置疑也被改掉了
	
	Point obj;
	Point *ptr=&obj;
	obj.magnitude();------>magnitude_7Point3dFv(&obj)
    ptr->magnitude();------>magnitude_7Point3dFv(ptr)
## 2.虚函数
对于虚函数的调用，如果是对象指针调用的话

	Point *ptr=&obj;
    ptr->normalize();//这个函数是虚函数的话
那么可能会是如下形式：

	(*(ptr->vptr[1]))(ptr);
这句话是调用虚函数。，ptr等同于this指针
## 3.静态成员函数
首先先给出两个转换形式，

	obj.normalize();//静态成员函数调用
    normalize_7Point3dSFv();
    ptr->normalize();
    normalize_7Point3dSFv();
会将其转换成非成员函数的调用，即普通的调用。后面即介绍了一下静态成员函数的特点

1.没有this指针，所以不能访问成员变量

2.不能够被声明为volatitle,virtual,const.

3.不需要经过对象就可以调用。

因为其没有this指针的特性，所以其和非成员函数有点类似，所以对其取地址，得到的是非成员函数的指针而非成员函数的指针刑（int (Ponit3d::*)()).
....
然后此章节后面的内容就是介绍虚拟函数包括其一些对象模型。
在这里就不多说了直接附上我的两篇模型剖析的博客，感兴趣的 自己去看看
CSDN:[http://blog.csdn.net/it_iverson/article/details/78206211](http://blog.csdn.net/it_iverson/article/details/78206211 "http://blog.csdn.net/it_iverson/article/details/78206211")
自己的博客：[http://lishuaii.top/2018/02/26/c++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B%E7%9A%84%E5%89%96%E6%9E%90/#more](http://lishuaii.top/2018/02/26/c++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B%E7%9A%84%E5%89%96%E6%9E%90/#more "http://lishuaii.top/2018/02/26/c++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B%E7%9A%84%E5%89%96%E6%9E%90/#more")

后面我在抓抓本章的重点内容写一写吧！嗯......

然后没发现什么哈哈！！
这里在附上一篇inline的详解的博客咯：
[http://blog.csdn.net/it_iverson/article/details/78473778](http://blog.csdn.net/it_iverson/article/details/78473778 "http://blog.csdn.net/it_iverson/article/details/78473778")