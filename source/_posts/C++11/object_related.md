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

## 移动语义
### 移动构造
如果拷贝一个临时对象的数据，并且这个对象在堆中申请了内存，那么拷贝的时候也要在堆中在申请一份内存，然后复制临时对象的数据(深拷贝)。
因为临时对象马上会被释放掉，如果在堆中的内存非常大，拷贝构造的过程就会变得非常昂贵。可以使用移动构造函数，直接把指针指向临时对象的内存。
![](move_construct.png)

### 左值、右值与右值引用
在传统的左值、右值判断中，比如一个赋值表达式中，等号左边的是左值，右边的是右值
```c++
a = b + c; //a是左值，b + c是右值
```
在C++的判断中，可以取地址、有名字的叫做左值。不能取地址、没有名字的是右值。
```c++
&(b + c) //编译不过
```
在C++11中，右值是由2个概念组成，一个是**将亡值(xvalue, eXpiring Value)**，一个是**纯右值(prvalue, Pure Rvalue)。**

**纯右值**是C++98的概念，临时变量和一些不跟对象关联的值。比如 1 + 3产生的临时变量值，2、'c'、true不跟对象关联的。还有类型转换的返回值，lambda表达式都是右值。

**将亡值**是C++11跟右值引用相关的表达式，比如返回右值引用T&&的函数返回值、std::move的返回值，或者转换为T&&的类型转换函数的返回值。

#### 右值引用
在C++11中，右值引用就是对一个右值进行引用的类型。由于右值通常不具有名字，我们也只能通过引用的方式找到它的存在：
```c++
T &&a = rturnRvalue();
```
在C++98中，右值不能绑定到左值：
```c++
T &e = returnRvalue(); //编译过不了
const T &f = returnRvalue();
```
在C++98中，用常量左值绑定右值不少见：
```c++
const bool &judgement = true; //为右值续命
const bool judgement = true; //赋值完右值释放
```
能够绑定右值的引用类型，都能够延长右值的生命期。
![](bind_rvalue.png)
判断一个类型是否是引用类型，可以用<type_traits>提供的模板类：is_rvalue_reference、is_lvalue_reference、is_reference
```c++
cout << is_rvalue_reference<string &&>::value;
```
#### std::move
使用std::move将左值转右值:
```c++
static_cast<T&&>(lvalue);
```
```c++
Copyable s;
Copyable news = s; //调用拷贝构造
Copyable news = std::move(s); //调用移动构造
```
