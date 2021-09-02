---
title: "Flask Study:Python mgic method and Local\LocalStack\LocalProxy in Werkzeug"
date: 2020-02-10 23:30:33
tags: 
- Flask 
- Python 
- MagicMethod 
- Local
categories: 
- CS
---

> 本篇文章主要记录阅读Flask源码遇到的有关Local以及LocalProxy概念的问题的理解 

<!--more-->
# 起因
FLask源码记录如下(globals.py)：
```python
from functools import partial
from werkzeug.local import LocalProxy
from werkzeug.local import LocalStack

#.......
#somthing here
#.......

# context locals
_request_ctx_stack = LocalStack()
_app_ctx_stack = LocalStack()
current_app = LocalProxy(_find_app)
request = LocalProxy(partial(_lookup_req_object, "request"))
session = LocalProxy(partial(_lookup_req_object, "session"))
g = LocalProxy(partial(_lookup_app_object, "g"))
```


**这里重要的是是声明了几个全局变量，current_app,request,g，这几个变量在flask使用中频繁出现，这里引入这些看似为全局变量的原因，本人理解，一个主要原因主要是为了后续视图函数中可以方便的调用本次如http请求过程中的相关参数，而无需传入参数进视图函数**
这里flask中的路由和视图函数声明过程如下：

```python
@route("/index")
def index():
    pass
```
其中index()函数即为视图函数

到这里，促使我了解上述全局变量的实现原理的原因在于，首先是开头那段代码完全看不懂在做什么，尤其出现了LocalStack、LocalProxy这些对象不知所云，后搜索发现，Werkzeug（Flask一个依赖的基础模块，wsui协议）中的Local模块中完成了一个十分重要的功能：
* 上述的全局变量在多线程/协程环境中，对于各自线程/协程来说，这些全局变量是互相独立互不影响
因此本着学习Python语法的目的，想探究开头代码背后的实现原理

# Python 继承、魔法方法

首先，先看Werkzeug的local模块中的源代码，一共有三个比较重要的类：Local、LocalStack、LocalProxy

首先看最基础的Local类，源代码如下:
```python
class Local(object):
    __slots__ = ('__storage__', '__ident_func__')

    def __init__(self):
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)

    def __iter__(self):
        return iter(self.__storage__.items())

    def __call__(self, proxy):
        """Create a proxy for a name."""
        return LocalProxy(self, proxy)

    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    def __getattr__(self, name):
        try:
            return self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}

    def __delattr__(self, name):
        try:
            del self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)
```

## Object类
Object类，这个在python2中用于区分新式类和旧式类，即继承于Object类的为新式类，反之没有继承的为旧式类。新式类和旧式类的一大区别在于，object类声明了许多魔法属性和方法（magic method）这些方法非常有用，除此之外新式类和旧式类在多重继承上也有区别，具体可以搜索相关内容，这里主要说明下object类的概念。
而在Python3中所有的类都是继承于object类的，不论是否显示声明继承于Object类
可用下述代码测试是类是否继承了Object类的相关属性和方法
```python
class test(object):
    pass

if __name__ == __main__:
    print(dir(test()))
```

回到Local类的代码上，其中Local类的初始化函数中如下：
```python
  def __init__(self):
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)
```
这里有几个问题:
* 通过object类名直接调用函数的含义
* __setattr__函数的含义

## Python继承
对于上述的第一个问题，即是调用父类的相应同名方法，这在子类构造函数需要调用父类的构造函数中尤为常见，但是这应该是比较老的写法，对于Python3来说，推荐使用super的写法
```python
class father():
    def __init__(self):
        pass

## Old Method(python2)
class sonA(father):
    def __init__(self):
        father.__init__(self)
        # son init here
        pass

## Use super
class sonB(father):
    def __init__(self):
        super(sonB,self).__init__()

class sonC(father):
    def __init__(self):
        super().__init__()

```

## Python魔法方法
对于第二个问题，有关__setattr__函数的含义,这个是object类中定义的魔法方法，在某些特定情况下会调用这些函数，这里__setattr__即是在对象进行属性设定会调用该函数进行设定，具体的魔法方法可以参考下述链接：
参考着篇文章[python magic method](https://rszalski.github.io/magicmethods/#appendix1)
里面对python的各种魔法方法都进行了详细介绍

这里又引出了第三个问题:
**Local类这里设置属性的时候为啥非要使用父类的object类？**
这里主要是为了避免无限递归造成的堆栈溢出错误
如果这里采用直接属性赋值的方式，即
```python
    def __init__(self):
        # 这里实际上调用的是 self.__setattr()方法
        self.storage = {}

    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}
```
可以看到，__setattr__方法中又调用了self.__storage__,在这里又会调用__setattr__的方法，陷入无限递归,因此这里必须调用原有的父类Object类中的设置属性的方法


# Local类、LocalStack类、LocalProxy类
到这里，看懂这段Loal类的创建语法上已经没有问题了，接下来就是Local类的设计思路：
Local目的是为了实现在多线程/协程环境下能够提供单线程/协程环境内的的使用的全局变量（同名），因此Local实际上是封装了各个线程内的同名变量，并用线程/协程的id来区分，在获取的时候通过获取当前线程/协程的id来获取对应的变量。
其中可以看到storage即是Local类用于存储各个线程/协程变量的字典数据结构

LocalStack类则是基于Local实现的栈结构

LocalProxy则是实现了对local对象以及可执行函数的一个代理，即代理类会将所有受到的操作都转移给其代理对象进行，但是必须注意的是，这里不能直接对代理对象进行赋值（一开始的理解就卡在这里，以为赋值为其他后依然是对原代理对象进行操作，实际上并不是，因为如果进行赋值后，会在调用该操作的区域生成了同名的局部变量遮盖了原有的代理变量，因此这样的操作违反了本意，也无法对原有代理对象进行修改），一般需要通过原始的Local对象进行获取操作，或者是LocalStack类的对象。
结合具体使用LocalProxy的例子来看
```python

def test_local():
    """ Local实例的代理 """
    class Request(object):
        
        def __init__(self,i):
            self.a = i

    l = Local()
    my_test = LocalProxy(l, 'my_test')


    def job(i):
        # Local().__storage__[线程/协程标识]['my_test'] = A()
        ## 一般来说 每个线程一开始都会有对该变量进行赋值的一个操作,这个操作必须通过Local.对象的方式进行
        l.my_test = Request(i)

        # lp = __storage__[线程/协程标识]['my_test'], 即lp.a = 1 等价于 给当前线程中的A().a对象赋值
        ## 获取值
        print(my_test.a)
        ## 改变值 
        my_test.b = i

    for i in range(3):
        Thread(target=job, args=(i,)).start()

    for k, v in l.__storage__.items():
        print(k)
        for name, obj in v.items():
            print(name, obj.__dict__)

```











