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
