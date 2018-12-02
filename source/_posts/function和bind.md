---
title: 'std::function 和 std::bind'
tags: C++
permalink: 'std-:-function-and-std-:-bind'
translate_title: stdfunction-and-stdbind
date: 2017-09-05 21:00:20
---

# 定义
标准库函数function()和bind()定义于头文件<functional>中，
```
template< class >
class function; /* undefined */
template< class R, class... Args >
class function<R(Args...)>;

template< class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );

template< class R, class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );
```
<!--more-->
std::function是可调用对象的包装器。它是一个类模板，可以容纳除了类成员（函数）指针之外的所有可调用对象（s包括普通函数、Lambda表达式、函数指针、以及其它函数对象等）。通过指定它的模板参数，它可以用同一的方式处理函数、函数对象、函数指针，并允许保存和延迟执行他们。

而当std::function和std::bind配合起来使用时，所有的可调用对象（包括类的成员函数指针和类成员指针）都将有统一的调用方式。

std::bind 用来将可调用对象与其参数一起绑定。绑定后的结果可以使用std::function进行保存，并延迟调用到任何我们需要的时候。
它有两大作用：
1）将可调用对象与其参数对象一起绑定成一个仿函数
2）将n元可调用对象转成一元或者n-m元可调用对象，即只绑定部分参数。

# 使用
{% codeblock  lang:cpp %} 
#include <functional>
#include <iostream>
using namespace std;

std::function< int(int)> Functional;

// 普通函数
int TestFunc(int a)
{
    return a;
}

// Lambda表达式
auto lambda = [](int a)->int{ return a; };

// 仿函数
class Functor
{
public:
    int operator()(int a)
    {
        return a;
    }
};

// 1.类成员函数
// 2.类静态函数
class TestClass
{
public:
    int ClassMember(int a) { return a; }
    static int StaticMember(int a) { return a; }
};

int main()
{
    // 普通函数
    Functional = TestFunc;
    int result = Functional(10);
    cout << "普通函数："<< result << endl;

    // Lambda表达式
    Functional = lambda;
    result = Functional(20);
    cout << "Lambda表达式："<< result << endl;

    // 仿函数
    Functor testFunctor;
    Functional = testFunctor;
    result = Functional(30);
    cout << "仿函数："<< result << endl;

    // 类成员函数
    TestClass testObj;
    //对于有参数的函数，使用 bind 时需要将参数补全，不补全就要用占位符
    //对于类的成员函数，第一个参数是类对象本身
    Functional = std::bind(&TestClass::ClassMember, testObj, std::placeholders::_1);
    result = Functional(40);
    cout << "类成员函数："<< result << endl;

    // 类静态函数
    Functional = TestClass::StaticMember;
    result = Functional(50);
    cout << "类静态函数："<< result << endl;

    return 0;
}
{% endcodeblock %}  