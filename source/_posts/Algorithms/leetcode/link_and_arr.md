---
title: post
p: Algorithm/leetcode/link_and_arr.md
date: 2021-03-10 22:20:41
tags:
---
## 链表、数组（滑动窗口）总结
    主要是针对一些常见算法面试题总结常见的解法，坑   

<!--more-->

### 链表
#### 链表反转
* 迭代版
```C
// 返回链接的头指针
linknode* reverse(linknode* head)
{
    linknode* cur=head,*pre=nullptr,*post=nullptr;
    while (cur!=nullptr){
        post = cur->next;
        cur->next = pre;
        pre = cur;
        cur = post;
    }
}
```
* 递归版
```C
//返回链接的尾指针
linknode *reverse(linknode* head)
{
    if (head == nullptr || head->next==nullptr){
        return head;
    }
    reverse(head->next)->next = head;
    return head;
}
```

##### 快慢指针的应用
    快慢指针感觉目前用的地方还比特别，LeetCode227 231
* 寻找链表中的中位数，O(N)复杂度）
```C
//若链表个数为奇数，返回的为正中间位置指针
//若链表个数为偶数，返回的为中间两个数考前的一个指针
linknode *middle(linknode* head){
    liknode *fast=head->next,*slow=head;
    while (fast!=nullptr || fast->next !=nullptr){
        fast = fast->next->next;
        slow = slow->next;
    }
    return slow;
}
```
* 另一个应用是根据快慢指针判断环，原理即如果链表中有成环状的，那么快慢指针一定会在环中相遇。

##### 数组链表的形式


### 数组

#### 位运算的运用（查找重复元素、缺失元素）

#### 哈希表使用（这个比较常见，即使用uordered_map或map快速查找）


### 二叉树、堆等树型结构应用

#### 二叉树（遍历、递归）

#### 堆（维护中间数，这个比较生疏）

