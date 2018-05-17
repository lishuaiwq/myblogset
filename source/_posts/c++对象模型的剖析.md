---
title: C++对象模型的剖析
date: 2018-02-26 15:11:42
tags: 对象模型剖析
categories: c/c++

---
## 1.单继承对象模型（含有虚函数）
首先阐述对象模型：
<!--more-->
1.子类和父类都拥有各自的虚函数表

2.如果子类重写了父类的虚函数，则在子类的虚函数表中替换同名的父类虚函数，如果没有重写，则子类的虚函数表中是父类的虚函数（注意在这里只要子类的函数 中有和父类一样的，不管子类加不加vartual，都是重写父类的虚函数）

3.如果子类有自己新写的虚函数，则该虚函数放在虚函数表的后面

	class Base
	{
	public:
		int a;
	public:
		virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base::fun2()" << endl;
	}
	};
	class Dervice :public Base
	{
	public:
		int b;
	public:
		virtual void fun2()
	{
		cout << "Dervice::fun2()" << endl;
	}
		virtual void fun3()
	{
		cout << "Dervice::fun3()" << endl;
	}
	};
	void test()
	{
	Base b;
	Dervice d;
	Base *p = &d;
	b.a = 10;
	d.b = 20;
	d.a = 30;
	}
首先来看base类的对象模型：
![](https://i.imgur.com/CX0gNv7.jpg)
然后看d的对象模型
![](https://i.imgur.com/dNiCO7p.jpg)
下面来验证一下 ：

	typedef void(*fun)();
	fun *pp = (fun*)(*(int*)&d);
	(*pp)();
	pp++;
	(*pp)();
	pp++;
	(*pp)();
![](https://i.imgur.com/1rYOHkz.jpg)
## 2.简单多继承对象模型
简单描述：

1.如果子类新增虚函数，则放在声明的第一个父类的虚函数表中（理解成继承下来的虚表比较好理解），

2.如果子类重写了父类的虚函数（两个父类中都有的那个虚函数），所有父类虚函数表都要改变。

3.子类内存布局中父类按照其声明顺序排列

	class Base1
	{
	public:
	int base1;
	public:
	virtual void fun1()
	{
		cout << "base1::fun1()" << endl;
	}
	};
	class Base2
	{
	public:
	int base2;
	public:
	virtual void fun1()
	{
		cout << "base2::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base2::fun2()" << endl;
	}
	};
	class Dervice :public Base1,public Base2
	{
	public:
	int b;
	public:
	virtual void fun1()
	{
		cout << "Dervice::fun1()" << endl;
	}
	virtual void fun3()
	{
		cout << "Dervice::fun3()" << endl;
	}
	};
	void test()
	{
	Base1 b1;
	Base2 b2;
	Dervice d;
	b1.base1 = 10;
	b2.base2 = 20;
	d.b = 30;
	d.base1 = 40;
	d.base2 = 50;
	}
![](https://i.imgur.com/H5A3DzY.jpg)
验证输出：

	typedef void(*fun)();
	fun *pp = (fun*)(*(int*)&d);
	(*pp)();
	pp++;
	(*pp)();
	pp += 2;
	/*pp++;*/
	(*pp)();
	pp++;
	(*pp)();
![](https://i.imgur.com/CEoQgzq.jpg)
## 3.简单虚继承对象模型
简单阐述：

1.虚继承的子类，如果本身定义了新的虚函数，则编译器为其生成一个新的vptr指针和和虚表，将其新定义的虚函数放进去，并且这个vptr位于对象的最前面

2.虚继承的子类也保留了父类的vptr和虚表

3.虚继承的子类有虚基类表指针vbptr，虚基类表中放的第一个是基类表指针到到对象首地址的偏移地址，后面的则放的是到第二个，第三个虚继承父类的偏移值。

	class Base1
	{
	public:
	int base1;
	public:
	virtual void fun1()
	{
		cout << "base1::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base1::fun2()" << endl;
	}
	};
	class Dervice :virtual public Base1
	{
	public:
	int b;
	public:
	virtual void fun1()
	{
		cout << "Dervice::fun1()" << endl;
	}
	virtual void fun3()
	{
		cout << "Dervice::fun3()" << endl;
	}
	};
	void test()
	{
	Base1 b1;
	Dervice d;
	b1.base1 = 10;
	d.b = 20;
	d.base1 = 30;
	}
![](https://i.imgur.com/ynnFjGK.jpg)
## 4.菱形继承对象模型
菱形继承是多继承和虚继承的复合
	class A
	{
	public:
	int _a;
	virtual void fun1()
	{
		cout << "A::fun1" << endl;
	}
	virtual void fun2()
	{
		cout << "A::fun2" << endl;
	}
	};
	class B1 : virtual public A
	{
	public:
	int _b1;
	virtual void fun1()
	{
		cout << "B1::fun1" << endl;
	}
	virtual void fun3()
	{
		cout << "B1::fun3" << endl;
	}

	};
	class B2 :virtual public A
	{
	public:
	int _b2;
	virtual void fun1()
	{
		cout << "B2::fun1" << endl;
	}
	virtual void fun4()
	{
		cout << "B2::fun4" << endl;
	}
	};
	class C :public B1, public B2
	{
	public:
	int _c;
	virtual void fun1()
	{
		cout << "C::fun1" << endl;
	}
	virtual void fun3()
	{
		cout << "C::fun3" << endl;
	}
	virtual void fun4()
	{
		cout << "C::fun4" << endl;
	}

	virtual void fun5()
	{
		cout << "C::fun5" << endl;
	}
	};void test()
	{

	cout << "A:" << sizeof(A) << endl;
	cout << "B1:" << sizeof(B1) << endl;
	cout << "B2:" << sizeof(B2) << endl;
	cout << "C:" << sizeof(C) << endl;
	A a;
	B1 b1;
	B2 b2;
	C c;
	a._a = 10;
	b1._a = 10;
	b1._b1 = 20;
	b2._b2 = 30;
	b2._a = 10;
	c._c = 40;
	c._a = 10;
	c._b1 = 20;
	c._b2 = 30;
	}
![](https://i.imgur.com/0Hu7rQV.jpg)
![](https://i.imgur.com/iGVt8Hh.jpg)