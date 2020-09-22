---
title: Data Struct Summary(Algorithms in C Chapter 2)
date: 2020-09-22 00:01:41
tags:
- CS
---

关系图:


<!--more-->
#### 基本数据结构
这里介绍了C语言中提供的最基本的数据结构，其是后续构造更复杂数据结构的基础，实际可以理解为也是一种C基于机器语言上提供的一个抽象（这里抽象的概念在计算机体系中十分通用，表明了一种层次性，对于每一层上都有一个相应的抽象模型）


* 主要介绍了数组、链表、复合数据结构（struct）
* 数组就很基本用法
* 链表 基于指针和复合数据结构即可以实现 或者 基于数组实现(本质上是一种抽象)
* 关于链表的相关操作以及概念
    * 链表遍历、删除、增加
    * 链表求逆（需要同时维护三个节点）
    * 注意有无哑元节点情况下的操作
* 书中还提到基于链表这种动态变化的特点实现的一种简单版本的malloc函数

```C
// 基于指针的实现
#include <stdio.h>
typedef node * link;
struct node {
    //Item Means any data struct
    Item item;
    link next;
};

// init
link beg_link = (link)malloc(sizeof(node));

```
* 字符串（这里主要注意 C中操作字符串主要是处理字符串指针）
* 复合数据结构（
  * 多维矩阵（多维数组或是多维链表的实现）
  * 图（邻接矩阵 或是邻接表的实现） 
* 最后是一个二维数组的链表的实例（降低算法时间复杂度）


#### 抽象数据类型

这里涉及了一个ADT的概念，即抽象数据类型（也就是前述的一种抽象层次上的描述，表征一种抽象操作）
* ADT一般来说都通过提供的接口函数对对象集合进行操作，而不是直接操作内部对象
* 同一个ADT本质抽象概念相同，但是实现可以不同

之后主要介绍了堆栈以及相应的FIFO队列
* stack （集合先进后出）
  * 基本操作（init、push、pop())
  * 实现（数组和列表的实现)
  * 应用（后缀表达式的求解 、中缀表达式转后缀表达式）


* FIFO （队列先进先出）
    * 实现（数组实现，链表实现）

* 广义队列的概念（
    * 本质上堆栈和队列都可以看成一种广义队列，只是具有不同的删除规则
    * 优先队列、双端队列
    * 符号表（map）

数组可以看成是一个特殊的符号表：项本身就是索引值

实际上后续学习的这些数据结构都是一簇ADT，有着不同定以的操作接口，我们需要找寻他们的一种高效实现。


#### 递归、树、图（仅遍历）

* 这里将递归、树、图放在一起的原因我觉得是树这种数据结构与递归很类似（大问题基于小问题构成，树拆分其中一部分也还是一颗树，因此基于树的数据结构的一些算法常常可以用递归的方式进行实现，当然也能非递归实现，比如基于前述的栈、队列）

* 链表的一些操作也适合用递归完成，链表可以看成一个特殊的树，每个结点只有一个分支

* 递归的一些应用
  * 求公约数
  * 前缀表达式求值

* 分治法 （hanoi塔）
  * hanoi 本质上会产生2^N - 1 的移动 ，由其递推公式可以得到(TN = 2 * TN-1  +1)
  * 基于分治算法的范型包括如快速排序、归并排序、二分搜索等算法
  * 分治算法揭示了一个非常基础的原理在于将问题分半（或者多个），将大问题分解为子问题进行求解

##### 动态规划（本质也是将一个问题分解为多个子问题进行求解，但是不同之处在于分解的子问题可能存在重复）
  * 以斐波那契数列数列和背包问题为例
  * 动态规划关键在于分解子问题（找出递推关系 及 原问题和子问题之间的联系）以及 存储已求子问题的解防止重复求解
  * 自底向上的动态规划（非低轨，循环外推）
  * 自顶向上的动态规划（递归版本）
	(斐波那契数列问题)
```C
// no recursion version
int fibonaci1(int i)
{
  // f46 have the max value can be storaged by int
  int F[46];
  F[0] =1;
  F[1] =1;
  for (int i =2;i<=i;i++){
    F[i] = F[i-1]+F[i-2];
  }
  return F[i];
}

// recursion version
int known[46];
memset(known,-1,sizeof(known));
int fibnoci2(int i){
    if (known[i]!=-1){
      return known[i];
    }
    if (i==0 || i==1){
      known[i] = 1;
    }
    else{
      known[i] = fibnoci2(i-1) + fibnoci2(i-2);
    }
    return known[i];
}
```

	背包问题
```C
struct item{
  int  value;
  int  size;
};

int N=100;
item Item[N];

int MaxKnown[Max_M];
memset(Memset,-1,sizeof(MaxKnown));
// recursive version
int knap(int M){
	if(MaxKnown[M]!=-1){
		return MaxKnown[M];
	}
	MaxKnown[M]=0;
	for (int i =0;i<N;i++){
		if (M-Item[i].size > 0){
			MaxKnown[M] = max(MaxKnown[M],knap(M-Item[i].size)+Item[i].value);
		}
	}

	return MaxKnown[M];
}

// no recursive version
int knap_no_recursive(int M){
	int* MaxKnown = (int*)malloc(M*sizeof(int));
	memset(MaxKnown,0,sizeof(MaxKnown));
	for (int i =0;i<M;i++){
		for (int j =0;j<N;j++){
			int space = i+Item[j].size;
			if ( space<=M ){
				MaxKnown[space] = max(MaxKnown[space],MaxKnown[i]+Item[j].value);
			}
		}
	}
	return MaxKnown[M];
	
}
```
  * 这里 自顶向下的动态规划的优势在于 算法描述相对于解决问题十分自然，且可以自动确定待求解的子问题，个人觉得问题在于其一般采用递归实现，在解决较大递归深度情况下能力有限，如背包量十分大的背包问题，
  * 自底向上的动态规划则采用循环实现，递归深度一般对于应数组存储，无函数调用开销，感觉能支持较大递归深度下的动态规划问题。

##### 树
* 基本概念
  * 根节点、叶节点、二叉树
  * 实现一般采用双向链表的方式表示
  ```C
  typedef node *link;
  struct node{
	  int value;
	  link father;
	  link left_child;
	  link right_child;
	  // or multi
	  link[10] child_list;
  }
  ```
  * 这里实现不采用类似前面的ADT的方式在于 树本身是用于实现其他高级ADT的基础
  * 二叉树的数学性质：
    * 具有N个内部节点的树具有N+1个的外部节点(外部节点是指不包含子节点的节点，剩余为内部节点)
    * N个内部节点的二叉树有2N个连接（实际就等于总节点数-1）N-1个内部链接 N+1个外部链接
    * N个内部节点的二叉树外部路径长度 比内部路径长度大2N （路径长度即为所有节点的层数之和，根节点为0层，层数最大值称为这颗树的高度）
    * N个内部节点的二叉树高度至多为N-1（感觉是N 感觉书上漏算了最后的外部节点的高度） 至少为lgN(2为底) 
  * 树的遍历
    * 首先是三种遍历 前序 中序 后序 区分这三种顺序方式就是判断访问某个节点时候 本身节点与其子节点之间的访问顺序（其中前中后 分别表示本身节点的访问顺序）
      * 前序 (本 左 右)
      * 中序（左 中 右)
      * 后序（左 右 中）
    * 实现（同样可以基于递归与非递归的方式，其中非递归基于栈数据结构即可以实现）
      * 非递归的实现（在实现中、后序的时候 需要考虑将本节点的子节点压入堆栈后 指向空
	* 层序遍历（按层次进行节点的访问）
    	* 实现 基于队列的数据结构即可
	* 一些具体的应用：
    	* 求树节点树、高度（递归很容易完成）
    	* 中序 打印树的形状  前序 则遍历类似文件树或者目录树的结构

##### 图的遍历
	* 深度优先搜索
