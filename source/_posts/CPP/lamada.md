---
title: 函数对象
p: CPP/lamada
date: 2021-01-10 22:38:34
tags:
---

### 泛型算法 与 迭代器
* algorithms 里面提供的泛型算法一般都不直接改变原有的元素，都是通过迭代器进行操作
* 插入迭代器
<!--more-->
### 传入给定关键字比较类型函数
* 对于algorithms中的算法，使用时直接传入函数指针或lamada表达式等可调用对象
```CPP
vector<int>k;
sort(k.begin(),k.end(),[](const int&a,const int&b)->bool{return a>b;});
```
* 对于定义时需要的，需要给定函数指针类型 一般直接使用decltype进行推导，这里注意推导出来的函数需要加指针
(这里也很好理解 因为定义模板类的时候，不能像上述模板函数算法直接通过参数推导模板类型，因此需要强制指定模板类型，因此具体的调用对象应该通过构造函数的方式进行传递)
(不过实际上一般是直接使用标准库定义的函数对象就可以解决大部分的问题)
```CPP
bool compare(const int&a,const int&b){
    return a>b;
}
class mycompare
{
public:
    bool operator()(const int&a,const int& b){
      return a > b;
    }
};
set<int,decltype(compare)*> s(compare);
priority_queue<int,vector<int>,decltype(compare)*> q(compare);
priority_queue<int,vector<int>,mycopmare> q(mycompare());
priority_queue<int,vector<int>,greater<int>> qq;
```

### 可调用对象 
* 可调用对象 
函数、函数指针、lamada表达式、重载可执行符号函数的类、bind返回的对象

* 函数指针
  * 传参和返回
    ```CPP
    void func1(){
        print("func1");
    }

    //
    void func2(void(*f)()){
        f(); 
    }

    void (*func3())()
    {
        return func1;
    }

    auto func3_2()-> void(*)()
    {
        return func1;
    }
    ```

* lamada 基本用法
  *  定义(lambda实际上定义的是一个匿名的函数对象)
  ```CPP
  int wc;
  int ac;
  auto f =  [&wc,ac](const int& a,const int&b)->bool {cout << wc << ac << endl;return a> b;};
  bool t = f(1,2);
  ```
  这里需要注意以下所定义的lamada表达式中,参数捕获的方式,默认为值捕获（即拷贝）,在中括号中的变量前加&表示引用的方式捕获
  另外，一般lambda尽量用于一次性使用的地方，多次使用的函数还是应该定义函数名
  
* 重载可执行函数的类
  * 即直接重载 operator() 函数即可
  * 标准库中定义了一系列的模板类，可用于执行的如 greater<T>,less<T>等，这也解释了为什么传递给泛型算法中需要带括号，即需要调用这些模板类的构造函数返回一个类对象，同时在传递给其他自定义比较对象（如set、priority_queue等）的时候不需要带括号，因为这个时候指定的是类型

* function对象(bind的返回值）
  * function使用(绑定上述几种常见的函数对象)
  ```CPP
  // function example
    #include <iostream>     // std::cout
    #include <functional>   // std::function, std::negate
    // a function:
    int half(int x) {return x/2;}
    // a function object class:
    struct third_t {
      int operator()(int x) {return x/3;}
    };
    // a class with data members:
    struct MyValue {
      int value;
      int fifth() {return value/5;}
      int fouth(int a) {return a/4;}
    };

    int main () {
        using namespace std::placeholders;
      std::function<int(int)> fn1 = half;                    // function
      std::function<int(int)> fn2 = &half;                   // function pointer
      std::function<int(int)> fn3 = third_t();               // function object
      std::function<int(int)> fn4 = [](int x){return x/4;};  // lambda expression
      std::function<int(int)> fn5 = std::negate<int>();      // standard function object
      Myvalue m{60};
      std::function<int(int)> fn6 = std::bind(MyValue::fouth,&m,_1); // using bind member fucntion
      // stuff with members:
      std::function<int(MyValue&)> value = &MyValue::value;  // pointer to data member
      std::function<int(MyValue&)> fifth = &MyValue::fifth;  // pointer to member function
      return 0;
    }
  ```
  * 可以看到引入function最大的好处在于统一了不同调用对象（但返回参数、调用参数一致的）的类型,从而方便了接口的定义和调用。也因此，通过funciton、bind的方式完全可以取代oo中多态的方式，尤其多态中的虚函数只有定义少数部分的时候。

