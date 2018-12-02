---
title: 数据结构之 Skip List
tags: 数据结构
permalink: skip-list-for-data-structures
translate_title: data-structure-skip-list
date: 2017-08-05 21:00:20
---

# 介绍
Skip List 在 leveldb 以及 lucence 中都广为使用，是比较高效的数据结构。由于它的代码以及原理实现的简单性，更为人们所接受。我们首先看看SkipList的定义，为什么叫跳跃表？
> Skip lists  are data structures  that use probabilistic  balancing rather  than  strictly  enforced balancing. As a result, the algorithms  for insertion  and deletion in skip lists  are much simpler and significantly  faster  than  equivalent  algorithms  for balanced trees.    
跳跃表使用概率均衡技术而不是使用强制性均衡，因此，对于插入和删除结点比传统上的平衡树算法更为简洁高效。    
<!--more-->
Skip List 是一种随机化的数据结构，基于并联的链表，其效率可比拟于二叉查找树（对于大多数操作需要O(log n)平均时间）。基本上，跳跃列表是对有序的链表增加上附加的前进链接，增加是以随机化的方式进行的，所以在列表中的查找可以快速的跳过部分列表(因此得名)。所有操作都以对数随机化的时间进行，因此随机函数在此格外重要。Skip List可以很好解决有序链表查找特定值的困难。    

# 特征
一个跳表，应该具有以下特征：
* 一个跳表应该有几个层（level）组成；
* 跳表的最底层包含所有的元素；
* 每一层都是一个有序的链表；
* 如果元素 x 出现在第 i 层，则所有比 i 小的层都包含 x；    

随机生成的一个跳跃表如下：
![](http://osac5rsex.bkt.clouddn.com/skiplist.jpg?imageslim)

# 实现
## 结点和链表定义
每一个结点都由3部分组成，key（关键字）、value（存放的值）以及forward数组（指向后续结点的数组，这里只保存了首地址）。通过这些结点，我们就可以创建跳跃表List，它是由两个元素构成，首结点以及level（当前跳跃表内最大的层数或者高度）    
{% codeblock  lang:cpp %}
//定义key和value的类型
typedef int KeyType;
typedef int ValueType;

//定义结点
typedef struct nodeStructure* Node;
struct nodeStructure
{
	KeyType key;
	ValueType value;
	Node forward[0]; //柔性数组
};

//定义跳跃表
typedef struct listStructure* List;
struct listStructure
{
	int level;
	Node header;
};
{% endcodeblock %}   

## 初始化链表
初始化表主要包括两个方面，header 节点和 NIL 结点的创建，其次就是 List 的创建和初始化。
{% codeblock  lang:cpp %}
void SkipList::NewList()
{
    //设置NIL结点
    NewNodeWithLevel(0, NIL_);
    NIL_->key = 0x7fffffff;
    //设置链表List
    list_ = (List)malloc(sizeof(listStructure));
    list_->level = 0;
    //设置头结点
    NewNodeWithLevel(MAX_LEVEL,list_->header);
    for(int i = 0; i < MAX_LEVEL; ++i)
    {
        list_->header->forward[i] = NIL_;
    }
    //设置链表元素的数目
    size_ = 0;
}

void SkipList::NewNodeWithLevel(const int& level , Node& node)
{
    //新结点空间大小
    int total_size = sizeof(nodeStructure) + level*sizeof(Node);
    //申请空间
    node = (Node)malloc(total_size);
    assert(node != NULL);
}
{% endcodeblock %}   
其中，NewNodeWithLevel 是申请结点所需的内存空间，总共 level + 1 层，level 从 0 开始。NIL_节点会在后续全部代码实现中可以看到。  

## 查找
查找就是给定一个 key，查找这个 key 是否出现在跳跃表中，如果出现，则返回其值，如果不存在，则返回不存在。我们结合一个图就是讲解查找操作，如下图所示：
![](http://osac5rsex.bkt.clouddn.com/skiplistsearch1.jpg?imageslim)
如果我们想查找 19 是否存在？如何查找呢？我们从头结点开始，首先和 9 进行判断，此时大于 9，然后和 21 进行判断，小于 21，此时这个值肯定在 9 结点和 21 结点之间，此时，我们和 17 进行判断，大于 17，然后和 21 进行判断，小于 21，此时肯定在 17 结点和 21 结点之间，此时和 19 进行判断，找到了。具体的示意图如下图所示：
![](http://osac5rsex.bkt.clouddn.com/skiplistsearch2.jpg?imageslim)
{% codeblock lang:cpp %}
bool SkipList::Search(const KeyType& key , ValueType& value)
{
    Node x = list_->header;
    int i;
    for(i = list_->level; i >= 0; --i)
    {
        while(x->forward[i]->key < key)
            x = x->forward[i];
    }
    x = x->forward[0];
    if(x->key == key)
    {
        value = x->value;
        return true;
    }
    else
    {
        return false;
    }
}
{% endcodeblock %}

## 插入
插入包含如下几个操作：1、查找到需要插入的位置   2、申请新的结点    3、调整指针。
我们结合下图进行讲解，查找如下图的灰色的线所示，申请新的结点如17结点所示， 调整指向新结点17的指针以及17结点指向后续结点的指针。这里有一个小技巧，就是使用update数组保存大于17结点的位置，这样如果插入17结点的话，就指针调整update数组和17结点的指针、17结点和update数组指向的结点的指针。update数组的内容如红线所示，这些位置才是有可能更新指针的位置。
![](http://osac5rsex.bkt.clouddn.com/skiplistinsert.png?imageslim)
{% codeblock  lang:cpp %}
bool SkipList::Insert(const KeyType& key, const ValueType& value)
{
	Node update[MAX_LEVEL];
	int i;
	Node x = list_->header;
	//寻找key所要插入的位置
	//保存大于key的位置信息
	for (i = list_->level; i >= 0; --i)
	{
		while (x->forward[i]->key < key)
			x = x->forward[i];
		update[i] = x;
	}
	x = x->forward[0];
	//如果key已经存在
	if (x->key == key)
	{
		x->value = value;
		return false;
	}
	else
	{
		//随机生成新结点的层数
		int level = RandomLevel();
		//为了节省空间，采用比当前最大层数加1的策略
		if (level > list_->level)
		{
			level = ++list_->level;
			update[level] = list_->header;
		}
		//申请新的结点
		Node newNode;
		NewNodeWithLevel(level, newNode);
		newNode->key = key;
		newNode->value = value;
		//调整forward指针
		for (int i = level; i >= 0; --i)
		{
			x = update[i];
			newNode->forward[i] = x->forward[i];
			x->forward[i] = newNode;
		}
		//更新元素数目
		++size_;
		return true;
	}
}
{% endcodeblock %} 

## 删除
 删除操作类似于插入操作，包含如下3步：1、查找到需要删除的结点 2、删除结点  3、调整指针
 ![](http://osac5rsex.bkt.clouddn.com/skiplistdelete.png?imageslim)
{% codeblock  lang:cpp %} 
bool SkipList::Delete(const KeyType& key, ValueType& value)
{
	Node update[MAX_LEVEL];
	int i;
	Node x = list_->header;
	//寻找要删除的结点
	for (i = list_->level; i >= 0; --i)
	{
		while (x->forward[i]->key < key)
			x = x->forward[i];
		update[i] = x;
	}
	x = x->forward[0];
	//结点不存在
	if (x->key != key)
	{
		return false;
	}
	else
	{
		value = x->value;
		//调整指针
		for (i = 0; i <= list_->level; ++i)
		{
			if (update[i]->forward[i] != x)
				break;
			update[i]->forward[i] = x->forward[i];
		}
		//删除结点
		free(x);
		//更新level的值，有可能会变化，造成空间的浪费
		while (list_->level > 0
			&& list_->header->forward[list_->level] == NIL_) {
			--list_->level;
		}
		//更新链表元素数目
		--size_;
		return true;
	}
}
{% endcodeblock %}

## 随机数生成器
再向跳跃表中插入新的结点时候，我们需要生成该结点的层数，使用的就是随机数生成器，随机的生成一个层数。这部分严格意义上讲，不属于跳跃表的一部分。可以采用c语言中srand以及rand函数，也可以自己设计随机数生成器。    
levelDB 随机数生成器如下所示：
{% codeblock lang:cpp %}
// Copyright (c) 2011 The LevelDB Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file. See the AUTHORS file for names of contributors.
#include<stdint.h>

//typedef unsigned int           uint32_t;
//typedef unsigned long long     uint64_t;

// A very simple random number generator.  Not especially good at
// generating truly random bits, but good enough for our needs in this
// package.
class Random {
private:
	uint32_t seed_;
public:
	explicit Random(uint32_t s) : seed_(s & 0x7fffffffu) {
		// Avoid bad seeds.
		if (seed_ == 0 || seed_ == 2147483647L) {
			seed_ = 1;
		}
	}
	uint32_t Next() {
		static const uint32_t M = 2147483647L;   // 2^31-1
		static const uint64_t A = 16807;
		// bits 14, 8, 7, 5, 2, 1, 0
		// We are computing
		//       seed_ = (seed_ * A) % M,    where M = 2^31-1
		// seed_ must not be zero or M, or else all subsequent computed values
		// will be zero or M respectively.  For all other values, seed_ will end
		// up cycling through every number in [1,M-1]
		uint64_t product = seed_ * A;
		// Compute (product % M) using the fact that ((x << 31) % M) == x.
		seed_ = static_cast<uint32_t>((product >> 31) + (product & M));
		// The first reduction may overflow by 1 bit, so we may need to
		// repeat.  mod == M is not possible; using > allows the faster
		// sign-bit-based test.
		if (seed_ > M) {
			seed_ -= M;
		}
		return seed_;
	}
	// Returns a uniformly distributed value in the range [0..n-1]
	// REQUIRES: n > 0
	uint32_t Uniform(int n) { return (Next() % n); }

	// Randomly returns true ~"1/n" of the time, and false otherwise.
	// REQUIRES: n > 0
	bool OneIn(int n) { return (Next() % n) == 0; }

	// Skewed: pick "base" uniformly from range [0,max_log] and then
	// return "base" random bits.  The effect is to pick a number in the
	// range [0,2^max_log-1] with exponential bias towards smaller numbers.
	uint32_t Skewed(int max_log) {
		return Uniform(1 << Uniform(max_log + 1));
	}
};
{% endcodeblock %}

## 释放
释放表的操作比较简单，只要像单链表一样释放表就可以，示意图如下：
![](http://osac5rsex.bkt.clouddn.com/skiplistfree.jpg?imageslim)
{% codeblock lang:cpp %}
void SkipList::FreeList()
{
    Node p = list_->header;
    Node q;
    while(p != NIL_)
    {
        q = p->forward[0];
        free(p);
        p = q;
    }
    free(p);
    free(list_);
}
{% endcodeblock %}
## 性能对比
网上有一张跳跃表、平衡树的对比图，如下图所示：
![](http://osac5rsex.bkt.clouddn.com/skiplistvsavl.jpg?imageslim)
从中可以看出，随机跳跃表表现性能很不错。

## 概率分析
这块与随机数生成器密切相关。假设我们采用以下随机数生成器：
{% codeblock lang:cpp %}
int randomLevel()
{
    int k=1;
    while (rand()%2)
        k++;
    k=(k<MAX_LEVEL)?k:MAX_LEVEL;
    return k;
}
{% endcodeblock %}
则概率分析如下图所示：
![](http://osac5rsex.bkt.clouddn.com/skiplistgailv1.jpg?imageslim)
![](http://osac5rsex.bkt.clouddn.com/skiplistgailv2.jpg?imageslim)
![](http://osac5rsex.bkt.clouddn.com/skiplistgailv3.jpg?imageslim)

# 总结
* skip list 的复杂度和红黑树一样，但实现起来更简单。
* 在并发环境下skip list 有另外一个优势，红黑树在插入和删除的时候可能需要做一些 rebalance 的操作，这样的操作可能会涉及到整个树的其他部分，而 skip list 的操作显然更加局部性一些，锁需要盯住的节点更少，因此在这样的情况下性能好一些。    

# 参考链接  
[浅析SkipList跳跃表原理及代码实现](http://blog.csdn.net/ict2014/article/details/17394259)    
[Skip List（跳跃表）原理详解与实现](http://dsqiu.iteye.com/blog/1705530)     
[Choose Concurrency-Friendly Data Structures](http://www.drdobbs.com/parallel/choose-concurrency-friendly-data-structu/208801371)
