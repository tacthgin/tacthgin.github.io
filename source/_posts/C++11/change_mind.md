---
title: 为改变思考方式而改变
date: 2017-9-8 16:37:00
tags: 
    C++11
category: C++11
---

## 指针空值-nullptr
* NULL宏被定义为0或者(void*)0，在C++98的重载函数中容易引起歧义。
* nullptr_t是由关键字nullptr推导出来的。
* 所有nullptr_t类型的数据都是等价的，nullptr_t可以隐式转换为任何指针类型。
* nullptr_t不能转换为其他非指针类型，既使用reinterpret是不允许的。
* nullptr_t的内存大小和void*是一样的。
* 不能对nullptr_t取地址，它被定义为右值，但是可以对其引用取地址。

## 默认函数的控制
* 在C++中，如果一个类是POD类型的，那么编译器会帮我们自动生成构造函数、赋值操作符那些函数。
* 在C++11中，可以使用=default，显式的指示编译器生成该函数的默认版本
```c++
class Test
{
public:
    Test() = default; //依然还是pod类型
};
```
* 使用=delete，来禁止使用者使用函数。
```c++
class Test
{
public:
    Test() = default;
    Test(const Test& ) = delete; //禁止使用拷贝构造
}
```
* 显式的删除并非局限于成员函数。
* C++11不推荐explicit关键字和显式删除合用，只会引起一些混乱性。
* 如果显式删除自定义类型的operator new操作符，就可以避免在堆上分配该class的对象
```c++
class NoHeapAlloc
{
public:
    void* operator new(std::size_t) = delete;
};
```
* 

## lambda函数
C++11的lambda函数定义如下：
```c++
[capture](parameters) mutable ->return-type{statement}
```
* [capture]：捕捉列表，能够捕捉上下文中的变量以供lambda使用。
* (parameters)：参数列表，与普通的函数参数列表一样。
* mutable：mutable修饰符，默认情况lambda函数总是一个const函数，mutable可以取消常量性。在使用该修饰符时，参数列表不可省略(即使参数为空)。
* ->return-type：返回类型，用追踪返回类型式声明函数的返回类型。不需要返回值的时候可以连同符号->一起省略。在返回类型明确的时候也可以省略，让编译器对返回类型进行推导。
* {statement}：函数体，跟普通函数一样，不过除了使用参数以外，还可以使用所有捕获的变量。

捕捉列表有如下几种形式：
* [var]传值方式捕捉变量var。
* [=]表示值传递方式捕捉所有父作用域的变量(包括this)。
* [&var]表示引用传递捕捉变量var。
* [&]表示引用传递捕捉所有父作用域的变量(包括this)。
* [this]表示值传递方式捕捉当前的this指针。
* [=, &a, &b]引用方式捕获a和b，值传递方式捕获其他所有变量。
* [&, a, this]以值传递方式捕获a和this，引用方式捕获其他所有变量。
* [=, a]和[&, &this]不允许变量重复传递。