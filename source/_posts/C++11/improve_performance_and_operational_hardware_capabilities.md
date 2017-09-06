---
title: 提高性能及操作硬件的能力
date: 2017-8-31 14:25:00
tags: 
    C++11
category: C++11
---

## 常量表达式
### 运行时常量性与编译时常量性
在C++中，常量用const关键字来修饰，不过const描述的是一些运行时常量性的概念，如果我们需要得到编译时期的常量性，const关键字无法保证的。
```c++
#include <iostream>

const int getConst() { return 1; }

int main()
{
    int arr[getConst()] = {0}; //无法通过编译
    enum { e1 = getConst(), e2}; //无法通过编译

    switch(cond)
    {
        case getConst(): //无法通过编译
            break;
        default:
            break;
    }

    return 0;
}
```
C++11使用constexpr关键字来表示编译期常量。可以把getConst函数改成：
```c++
constexpr int getConst() { return 1; }
```
### 常量表达式函数
在返回类型加入关键字constexpr会使函数成为常量表达式函数。常量表达式函数要求一下几点：
* 函数体只有单一的return返回语句。(不产生实际代码的可以编译通过，比如static_assert)
```c++
constexpr int func(int x)
{
    static_assert(false, "assert fail");
    return x;
}
```
* 函数必须有返回值。
* 函数在使用前已经定义过了。
* return返回语句表达式中不能使用非常量表达式的函数、全局数据，且必须是一个常量表达式。

### 常量表达式值
* 常量表达式值必须被常量表达式赋值。
* 常量表达式值使用前必须被初始化。
* 如果没有代码显式使用常量表达式值，编译器可以选择不为它生成数据。
* 浮点常量表达式值得精度至少等于(或者高于)运行时的浮点数常量的精度。
* 自定义数据常量表达式值必须要有自定义常量构造函数
```c++
struct MyType
{
    constexpr MyType(int x) : i(x) {}
    int i;
}
```
>常量表达式的构造也有使用上的约束：
>* 函数体必须为空。
>* 初始化列表只能有常量表达式来赋值。

* 常量表达式不能作用于virtual函数

### 常量表达式的其他应用
* 可以用在模板函数，不过如果模板函数不满足常量表达式的需求，编译器会忽略constexpr，变成普通函数
* 常量表达式函数支持递归

## 变长模板
### 变长函数
传统的变长函数的例子：
```c++
double sum(int count, ...)
{
    va_list ap;
    double sum = 0;

    va_start(ap, count);

    for(int i = 0; i < count; ++i)
        sum += va_arg(ap, double);

    va_end(ap)
    return sum;
}
```
局限比较大，只能通过count获得函数参数个数，通过va_arg(ap, double)获取函数参数类型
### 变长模板：模板参数包和函数参数包
以标准库中tuple为例：
```c++
template <typename... Elements> class tuple;

tuple<int, char, double>
```
...表示该参数是变长的，Elements是一个**模板参数包(template parameter pack)**。类模板tuple接受任意多个参数作为模板参数。

模板参数包可以是非类型的：
```c++
template<int...A> class Test{};
Test<1, 0, 2> test;
```
一个模板参数包在模板推导时会被认为是模板的单个参数，为了使用模板参数包，需要将其**解包(unpack)**，通过一个名为**包扩展(pack expansion)**的表达式来完成。
```c++
template <typename T1, typename T2> class B{};
template<typename... A> class Test : private B<A...>{};
Test<X, Y> xy;
```
A...就是一个包扩展,上面的例子中如果我们定义Test&lt;X, Y, Z>就会出现错误。如果想要变长的参数，可以参考下tuple的例子：
```c++
template <typename... Elements> class tuple;

template<typename Head, typename... Tail>   //递归的偏特化定义
class tuple<Head, Tail...> : private tuple<Tail...>
{
    Head head;
};

template<> class tuple<> {}; //边界条件
```
Head作为tuple<Head, Tail...>的第一个成员，tuple<Tail...>作为tuple<Head, Tail>的私有基类。当实例化一个tuple&lt;double, int, char, float>类型时候，会引起基类的递归构造，递归在tuple的参数包为0的时候会结束。编译器将从tuple<>构造出tuple&lt;float>，然后tuple&lt;char, float>、tuple&lt;int, char, float>，最后tuple&lt;double, int, char, float>。
![](template_param.png)

除了变长的模板类，在C++11中，我们还可以声明变长模板的函数。同样的，变长的函数参数也可以声明为函数参数包(function parameter pack)。
```c++
template<typename... T> void f(T... args);
```
C++11标准要求函数参数包必须唯一，并且是函数的最后一个参数。
```c++
template<typename T>
T f(T first)
{
    return first;
}

template<typename T, typename... Args>
T f(T first, Args... args)
{
    return first + f(args...);
}

int main()
{
    cout << f(12, 3, 2, 3); // 20
    return 0;
}
```
### 变长模板：进阶