---
title: 关键字
date: 2017-8-15 10:44:00
tags: 
    C++11
category: C++11
---
* long long整形最低64位
* static_assert提供在编译期断言
* final防止函数被子类重写。
* override标明是虚函数重载，如果基类中没有此函数，编译不通过。
* template <typename T = int> void func(T a){}模板函数可以添加默认参数
example:
```c++
template <class T, class U = double>
void f(T t = 0, U u = 0);
int main()
{
    f(1); // f<int, double>(1, 0),使用默认参数double
    f<int>(); // f<int, double>(0, 0),使用默认参数double
    f(); //无法推导T是什么类型
    f<int, char>(); // f<int, char>(0, 0)
    return 0;
}
```
* 使用模板外部声明防止模板因为数据不同生成多份的实例代码
未使用声明前：
![](no_extern_template.png)
声明后：
```c++
#include "test.h"
template void fun<int>(int); //显示的实例化
void test1() { fun(3); }

//test2.cpp中
#include "test.h"
extern template void fun<int>(int); //外部模板声明
void test1() { fun(4); }
```
![](extern_template.png)

* 局部和匿名类型可以做模板实参
example:
```c++
template <class T> class X{};
template <class T> void func(T t) {}
struct A{} a;
struct {int i;}b; // b是匿名类型变量
typedef struct {int i;}B; //B是匿名类型
int main()
{
    struct C{} c;
    X<A> x1; //C++98通过，C++11通过
    X<B> x1; //C++98错误，C++11通过
    X<C> x2; //C++98错误，C++11通过
    func(a); //C++98通过，C++11通过
    func(b); //C++98错误，C++11通过
    func(c); //C++98错误，C++11通过
    return 0;
}
```
> **Tip**
```c++
template <class T> struct MyTemplate{};
int main()
{
    MyTemplate<struct {int a;}> t; //无法编译通过，匿名类型的生命不能再模板实参位置
    return 0;
} 
```