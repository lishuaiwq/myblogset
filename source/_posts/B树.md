---
title: B树
date: 2018-05-14 11:10:47
tags:
categories: 数据结构随笔
---

## B树的基本概念

- B树是R.Bayer和E.mccreight提出的一种适合外查找的树（在磁盘上查找），他是一种平衡的多叉树，成为B树（有些地方写的是B-树，注意不要读成了"B减树"）

### 一个B数应该满足下列的性质

- 根节点至少有两个孩子，[2,M]个孩子
- 每个非根节点有[M/2,M]个孩子
- 每个非根节点有[M/2-1,M-1]个关键字，并且以升序排列
- 每个结点孩子的数量比关键字的数量多一个
- key[i]和key[i+1]之间的孩子结点的值介于key[i]、key[i+1]之间
- 所有的叶子结点都在同一层

### 给出B树的结构框架
```c++
template<class K,class V,size_t M>
struct BTreeNode
{
	pair<K, V> _kvs[M - 1];//关键字
	BTreeNode<K, V, M>* subs[M];//孩子的指针集
	size_t _size;//关键字的数量
};

template<class K,class V,size_t M>
class BTree
{
public:
	typedef BTree<K, V, M> Node;
	typedef Node* PNode;
private:
	PNode _pRoot;
};
```
>这只是简单的最基础的框架，后面我们还要在此基础上进行修改！

## B树的代码实现

## B树的应用

B+树是B树的变形，在实际中B+树用的最多，数据库这里剩下的东西需要自己去了解。