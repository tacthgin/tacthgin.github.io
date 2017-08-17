---
title: 对象相关
date: 2017-8-15 12:23:00
tags: 
    C++11
category: C++11
---

## 继承构造函数
派生类使用基类的构造函数，需要在构造函数中显式声明：
```c++
struct A { A(int i) {} };
struct B : A { B(int i):A(i) {} };
```
在C++11中使用using声明来声明继承构造函数:
```c++
struct A 
{
    A(int i) {}
    A(double d, int i) {}
    A(float f, int i, const char* c) {}
};

struct B : A 
{
    using A::A;    
}
```
把基类中的构造函数继承到了子类B中，而且C++11标准继承构造函数被设计为跟派生类中的各种类默认函数(默认构造，析构，拷贝构造等)是隐式声明的。如果继承构造函数不被相关代码使用，编译器不会为其产生真正的函数代码(POD类型)。

继承构造冲突情况：
```c++
struct A { A(int) {} };
struct B { B(int) {} };

struct C : A, B
{
    using A::A;
    using B::B;
};
```
子类使用显式定义冲突的构造函数来解决：
```C++
struct C : A, B
{
    using A::A;
    using B::B;
    C(int) {}
};
```

## 委托构造函数
委托构造函数目的是减少程序员的书写构造函数的时间。
example：
```c++
class Info
{
public:
    Info() { initRest(); }
    Info(int i) : type(i) { initRest(); }
    Info(char e) : name(e) { initRest(); }
private:
    void initRest() { /*其他初始化*/ }
    int type {1};
    char name {'a'};
};
```
使用委托构造函数后:
```c++
class Info
{
public:
    Info() { initRest(); }
    Info(int i) : Info() { type = i; }
    Info(char e) : Info() { name = e; }
private:
    void initRest() { /*其他初始化*/ }
    int type {1};
    char name {'a'};
};
```
**Tip:**
**可以使用黑科技， 用placement new来实现在构造函数中调用构造函数**
```c++
Info(int i) { new (this) Info(); type = i;}
```
委托构造函数不能与初始化列表一起使用：
```c++
class Info
{
public:
    
    Info(int i) : type(i) {}
    Info(): Info(40), i(1) {} // 无法通过编译
private:
    int type {1};
};
```
避免形成委托环(delegation cycle):
```c++
class Info
{
public:
    Info():Info(1) {}
    Info(int i) : Info('a') {}
    Info(char e) : Info(1) {}
private:
    int type {1};
    char name {'a'};
};
```
委托构造函数比较实用的应用就是使用构造模板函数产生目标构造函数
```c++
class TDContructed
{
    template<class T> TDContructed(T first, T last) : l(first, last) {}
    list<int> l;
public:
    TDContructed(vector<short>& v) : TDContructed(v.begin(), v.end()) {}
    TDContructed(deque<int>& d) : TDContructed(d.begin(), d.end()) {}
}
```
**Tip:**
**目标构造函数抛出的异常可以在委托构造函数中捕获到**