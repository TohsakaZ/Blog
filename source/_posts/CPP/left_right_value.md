---
title: C++中左值、右值概念
p: CPP/left_right_value
date: 2021-01-09 21:02:44
tags:
---

### auto decltype

### 函数指针

### 引用、指针

* 定义的引用变量本身并不是一个对象，因此无法定义引用的引用，但是这个在函数传参的时候怎么理解？

* 四种类型转换

* const 顶层、底层基本用法 以及C++11引入的constexpr的用法，其中 一个典型应用就是将string应用于swtich case中（正常switch 只能接收int 和 enum类型的常量表达式）

### 函数传参
函数重载（传值、传引用无法区分）

### 拷贝控制 乃至 移动语义(增强了值语义的控制变量周期的功能)
使得向面向值语义的面向对象更进一步，即原有的对象语义（不可拷贝），也可以融于现有的值语义体系（通过移动代替拷贝的操作）

不过移动语义最大的功能也就在于（目前看来）提升代码效率,避免一些不必要的临时变量（本质上，C++的class是值语义的，因此，很容易在参数传递和返回等地方多出临时变量的拷贝消耗,但是现在完全可以通过移动语义来避免掉这些拷贝过程）

### 值语义-> 数据抽象

陈硕 CPP工程实践

### C++ 四大模块
《The C++ Programming Language》 [3rd]
C++ is a general-purpose programming language with a bias towards systems
programming that
* is a better C,
* supports data abstraction,
* supports object-oriented programming, and
* supports generic programming.








