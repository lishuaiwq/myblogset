---
title: set和map简单模拟实现
date: 2018-05-16 09:13:36
tags:
---

#  铺垫

---
### 底层

>> 首先set和map的底层都是基于红黑树实现的，只需要将红黑树封装起来就可以了，所以在这里给出我之前博客的红黑树的实现

[https://blog.csdn.net/it_iverson/article/details/79299039](https://blog.csdn.net/it_iverson/article/details/79299039)

### 迭代器

>> 其实迭代器和智能指针一样，将原类的遍历，++，--等操作用一个类封装起来，然后将其迭代器类型typedef到原类中方便其使用（一般typedef 成iterator类型），封装后直接对这个对象进行操作就可以了，这个类就是迭代器。

```c++
//红黑树的迭代器
template<class V,class Ref,class Ptr>//分别传V V* V&
struct _RBTreeIterator
{
	typedef _RBTreeIterator<V,Ref,Ptr> Self;
	typedef RBTreeNode<V> Node;
	Node* _node;//用来接头结点的指针，其实迭代器就是一个智能指针只不过他的功能是++和--还有一些解引用而已
	_RBTreeIterator(Node* node)
		:_node(node){}//这里只需要 浅拷贝就可以了，和迭代器的功能有关
	_RBTreeIterator()
	{}
	Ref operator*();//返回引用
	Ptr operator->();//返回指针,只是用模板来看更加的灵活
	Self operator++();//前置++
	Self operator++(int);
	bool operator!=(const Self& s);
	Self operator--();
	Self operator--(int);
};
```
### 改造后的红黑树

```c++
//Myset->RBtree<K,K>
//Myset->RBtree<K,pair<K,V>>
template< class V,class KeyOfValue>
class RBTree
{
public:
	typedef RBTreeNode< V> Node;
	typedef Node* PNode;
	typedef _RBTreeIterator<V, V&, V*> Iterator;
	typedef _RBTreeIterator<V, const V&, const V*> ConstIterator;//让在红黑树中可以使用迭代器
private:
	PNode _pRoot;
public:
	RBTree()
		:_pRoot(NULL){}

	Iterator Begin()
	{
		Node* left=_pRoot;
		while (left->_pLeft)
		{
			left = left->_pLeft;
		}
		return left;
	}
	Iterator End()
	{
		return NULL;
	}
	pair<Iterator,bool> Insert(const V& value)
	{
		if (NULL == _pRoot)//根节点为黑色  
		{
			_pRoot = new Node( value);
			_pRoot->_color = Black;
			return make_pair(Iterator(_pRoot),true);//make_pair<>生成一个pair对象返回去
		}
		KeyOfValue kof;
		PNode cur = _pRoot;
		PNode pParent = NULL;
		while (cur)//找插入的位置  
		{
			if (kof(cur->_v)==kof(value))
				return make_pair(Iterator(cur),false);
			else if (kof(cur->_v)>kof(value))//这里用仿函数巧妙的 处理了 set和map中值不同的问题
			{
				pParent = cur;
				cur = cur->_pLeft;
			}
			else
			{
				pParent = cur;
				cur = cur->_pRight;
			}
		}
		PNode pNewNode = new Node(value);//新插入的结点的颜色默认是红的  
		PNode ret = pNewNode;
		//找到插入的位置以后开始插入  
		
		if (kof(pParent->_v)>kof(value))
		{
			pParent->_pLeft = pNewNode;
			pNewNode->_pParent = pParent;
		}
		else
		{
			pParent->_pRight = pNewNode;
			pNewNode->_pParent = pParent;
		}
		////当结点插入好了以后我们就要根据红黑树的性质来判断结点是否满足，从而调整结点  
		while (pParent)
		{
			if (pParent->_color == Black)//父结点是黑的,不用调整直接退出  
			{
				break;
			}
			//记录祖父结点和定义保存叔叔结点  
			PNode gparent = pParent->_pParent;
			PNode uncle = NULL;
			if (pParent == gparent->_pLeft)//在其左子节点上  
			{
				uncle = gparent->_pRight;
				if (uncle&& uncle->_color == Red)//叔叔结点存在切为红情况3,4  
				{
					gparent->_color = Red;
					uncle->_color = pParent->_color = Black;
					pNewNode = gparent;//保存gg结点  
					pParent = gparent->_pParent;
					continue;
				}
				else if (uncle == NULL || uncle->_color == Black)//叔叔不存在或者为黑  
				{
					if (pNewNode == pParent->_pLeft)//外侧插入  
					{
						_RBRolateR(gparent);
						gparent->_color = Red;
						pParent->_color = Black;
					}
					else//内测插入  
					{
						_RBRolateL(pParent);
						std::swap(pParent, pNewNode);//交换pParent和插入结点指针的值  
						_RBRolateR(gparent);
						gparent->_color = Red;
						pParent->_color = Black;
					}
				}
				break;//直接跳出循环  
			}
			else//右边的情况  
			{
				uncle = gparent->_pLeft;
				if (uncle&&uncle->_color == Red)//叔叔存在而且为红色  
				{
					gparent->_color = Red;
					uncle->_color = pParent->_color = Black;
					pNewNode = gparent;
					pParent = pNewNode->_pParent;
					continue;
				}
				else if (uncle == NULL || uncle->_color == Black)//叔叔不存在或者为黑  
				{
					if (pNewNode == pParent->_pRight)
					{
						_RBRolateL(gparent);
						gparent->_color = Red;
						pParent->_color = Black;
					}
					else
					{
						_RBRolateR(pParent);
						std::swap(pParent, pNewNode);
						_RBRolateL(gparent);
						gparent->_color = Red;
						pParent->_color = Black;
					}
				}
				break;
			}
		}
		_pRoot->_color = Black;
		return make_pair(Iterator(ret), true);
	}
	void InOrder()
	{
		_InOrder(_pRoot);
	}
	bool IsRBTree()
	{
		if (_pRoot == NULL)//根节点 为空是红黑树  
			return true;
		if (_pRoot->_color == Red)//根节点为红色肯定不是红黑树  
			return false;
		int count = 0;//统计黑色结点的数目  
		PNode cur = _pRoot;
		while (cur)//根出一条参考路径的黑色结点的数目  
		{
			if (cur->_color == Black)
				++count;
			cur = cur->_pLeft;
		}
		int k = 0;
		return _IsRBTree(_pRoot, count, k);
	}
	Iterator Find(const V& key)
	{
		KeyOfValue kov;
		PNode cur = _pRoot;
		while (cur)
		{
			if (kov(cur->_v > key))
			{
				cur = cur->_pLeft;
			}
			else if (kov(cur->_v) < key)
			{
				cur = cur->_pRight;
			}
			else
			{
				//return cur;
				return Iterator(cur);
			}
		}
	}
private:
	bool _IsRBTree(PNode pRoot, int& count, int k)//这里的K不能传引用  
	{
		if (pRoot == NULL)
			return true;
		//出现两个连续的红色的结点  
		if (pRoot->_pParent&&pRoot->_color == Red&&pRoot->_pParent->_color == Red)
			return false;
		//如果是黑色结点k++  
		if (pRoot->_color == Black)
			k++;
		if (pRoot->_pLeft == NULL&&pRoot->_pRight == NULL)//如果是叶子结点的话进行判断k和count  
		{
			if (k == count)
				return true;
			else
				return false;
		}
		return _IsRBTree(pRoot->_pLeft, count, k) && _IsRBTree(pRoot->_pRight, count, k);
	}
	//右单旋转  
	void _RBRolateR(PNode parent)
	{
		if (NULL == parent)
			return;
		PNode SubL = parent->_pLeft;// 
		
		PNode SubLR = SubL->_pRight;

		parent->_pLeft = SubLR;
		if (SubLR != NULL)
			SubLR->_pParent = parent;
		PNode pParent = parent->_pParent;
		SubL->_pRight = parent;
		parent->_pParent = SubL;
		if (pParent == NULL)
		{
			_pRoot = SubL;
			SubL->_pParent = NULL;
		}
		else
		{
			if (pParent->_pLeft == parent)
			{
				pParent->_pLeft = SubL;
				SubL->_pParent = parent;
			}
			else
			{
				pParent->_pRight = SubL;
				SubL->_pParent = parent;
			}
		}
	}
	void _RBRolateL(PNode parent)
	{
		if (NULL == parent)
			return;
		PNode SubR = parent->_pRight;
		PNode SubRL = SubR->_pLeft;
		parent->_pRight = SubRL;
		if (SubRL)
		{
			SubRL->_pParent = parent;
		}
		PNode pParent = parent->_pParent;
		SubR->_pLeft = parent;
		parent->_pParent = SubR;
		if (pParent == NULL)
		{
			_pRoot = SubR;
			SubR->_pParent = NULL;
		}
		else
		{
			if (pParent->_pLeft == parent)
			{
				pParent->_pLeft = SubR;
				SubR->_pParent = pParent;
			}
			else
			{
				pParent->_pRight = SubR;
				SubR->_pRight = pParent;
			}
		}
	}
	void _InOrder(PNode pRoot)
	{
		if (pRoot)
		{
			_InOrder(pRoot->_pLeft);
		//	cout << pRoot->_key << " ";
			_InOrder(pRoot->_pRight);
		}
	}
};
```

# Set 

## set的特性

> 所有的元素都会根据元素的键值自动被排序。set的元素不像map那样可以同时拥有实值(value)和键值(key),set元素的键值就是实值，实值就是键值，set不允许两个元素有相同的值。我们也不允许通过set的迭代器改变set元素，因为set元素值就是其关键值，关系到set元素的排列规则。如果任意改变set的元素值得话，会严重破坏set的组织。所以在set的底层RBtree中定义的是const_iterator,用来防止修改值的，在上述的红黑树中也有体现
```c++
typedef _RBTreeIterator<V, const V&, const V*> ConstIterator
```

## set实现

> 因为红黑树是一种平衡二叉搜索树，自动排序的效果很不错，所以标准的STL set以红黑树为底层。由于set所开放的接口，红黑树都提供了，所以几乎所有的set的行为都是红黑树接口的调转。

### 如何区分set和map呢？

> 因为如果是set的话，其key就是键值也是实值,而map的话插入的是pair<k,v>结构`后面map中详细介绍`，而我们需要其中的键值key来进行相关的操作，所以这里我们使用仿函数就可以很容易的实现。
```c++
    //set的仿函数,因为直接在类中，所以少了模板
	struct SetKeyOfValue
	{
		const K& operator()(const K& key)
		{
			return key;
		}
	};

//map的仿函数,因为写在类外所以就带了模板参数
template<class K,class V>
struct MapKeyOfvalue
{
	const K& operator()(const pair<K, V>& kv)
	{
		return kv.first;
	}
};
```

### set的简单模拟实现代码

```c++
#pragma once

#include "RBTree.h"


//template<class K>
template<typename K>
class Set
{
public:
	struct SetKeyOfValue
	{
		const K& operator()(const K& key)
		{
			return key;
		}
	};
	typedef typename RBTree< K, SetKeyOfValue>::Iterator Iterator;
	pair<Iterator, bool> Insert(const K& key)
	{
		return _t.Insert(key);
	}

	Iterator Find(const K& key)
	{
		return _t.Find(key);
	}

	Iterator Begin()
	{
		return _t.Begin();
	}

	Iterator End()
	{
		return _t.End();
	}

protected:

	RBTree<K, SetKeyOfValue> _t;
};

void TestMySet()
{
	Set<string> s;
	s.Insert("sort");
	s.Insert("insert");
	s.Insert("set");
	s.Insert("map");
	s.Insert("iterator");
	s.Insert("value");
	s.Insert("value");
	Set<string>::Iterator it = s.Begin();
	while (it != s.End())
	{
		cout << *it << "  ";
		++it;
	}
	cout << endl;
}
```
 
# map

## map的特性
> 所有的元素都会根据元素的键值(key)自动被排序.map的所有元素都是pair，同时拥有实值(value)和键值(key),pair的第一个元素被视为键值,第二元素被视为实值，map不允许有两个相同的键值。

## pair的结构

`给出pair的源代码`

```c++
#ifndef __SGI_STL_INTERNAL_PAIR_H
#define __SGI_STL_INTERNAL_PAIR_H

__STL_BEGIN_NAMESPACE

template <class T1, class T2>
struct pair {
  typedef T1 first_type;
  typedef T2 second_type;

  T1 first;
  T2 second;
  pair() : first(T1()), second(T2()) {}
  pair(const T1& a, const T2& b) : first(a), second(b) {}

#ifdef __STL_MEMBER_TEMPLATES
  template <class U1, class U2>
  pair(const pair<U1, U2>& p) : first(p.first), second(p.second) {}
#endif
};

template <class T1, class T2>
inline bool operator==(const pair<T1, T2>& x, const pair<T1, T2>& y) { 
  return x.first == y.first && x.second == y.second; 
}

template <class T1, class T2>
inline bool operator<(const pair<T1, T2>& x, const pair<T1, T2>& y) { 
  return x.first < y.first || (!(y.first < x.first) && x.second < y.second); 
}

template <class T1, class T2>
inline pair<T1, T2> make_pair(const T1& x, const T2& y) {
  return pair<T1, T2>(x, y);
}

__STL_END_NAMESPACE

#endif /* __SGI_STL_INTERNAL_PAIR_H */

// Local Variables:
// mode:C++
// End:
```
> 其实简单的理解就这这个样子的，其实就是一个自定义的类型
```c++
template<class K,class V>
struct pair
{
	K first;
	V second;
	pair()//无参构造函数
		:_first(K())
		, second(V()){}
	pair(const K& a, const V& b)//有构造函数
		:_first(a)
		, second(b){}
};
```

## map简单模拟的实现
```c++
#pragma once
#include"RBTree.h"
template<class K,class V>
struct MapKeyOfvalue
{
	const K& operator()(const pair<K, V>& kv)
	{
		return kv.first;
	}
};
template<class K, class V, class MapKeyOfvalue = MapKeyOfvalue<K,V> >
class Map
{
protected:
	RBTree</*K,*/ pair<K, V>,MapKeyOfvalue> _t;//定义红黑树的对象
public:
	typedef typename RBTree</*K,*/ pair<K, V>, MapKeyOfvalue>::Iterator Iterator;
	pair<Iterator, bool> Insert(const pair<K,V>& kv)
	{
		return _t.Insert(kv);
	}
	V& operator[](const K& key)
	{
		pair<Iterator, bool> ret = Insert(make_pair(key, V()));
		return (ret.first)->second;
	}
	Iterator Find(const K& key)
	{
		return _t.Find(key);
	}
	Iterator Begin()
	{
		return _t.Begin();
	}
	Iterator end()
	{
		return _t.End();
	}
};

void TestMyMap()
{
	Map<string, string> dict;
	dict["hehe"] = "haha";
	dict.Insert(make_pair("left", "左边"));

	Map<string, string>::Iterator it = dict.Begin();
	while (it != dict.end())
	{
		cout << (*it).first <<" "<<(*it).second<<endl;
		++it;
	}
}
```
> 在次梳理一下，在红黑树中如果是set的则V直接就是set的key,而如果是map的话，那么V的类型就是pair这个自定义的类型，这里自己注意区分。还有就是set的迭代器使用constiterator而map使用iterator就可以了。


### map中的operator[]



