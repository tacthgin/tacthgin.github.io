---
title: 语法改进
date: 2017-8-25 10:00:00
tags: 
    C++11
category: C++11
---

## 右>括号的改进
在C++98中，如果模板实例化的时候出现了连续的2个右尖括号>，中间必须加一个空格来进行分隔。
```c++
template <class T>
class A{};

vector<A<int> > v1; //编译成功
vector<A<int>> v2; //编译失败
```
或者使用static_cast、dynamic_cast、reinterpret_cast和const_cast转换的时候也会出现相同情况。
```c++
const vector<int> v = static_cast<vector<int>>(v); //编译失败
```
C++98会将>>符号判断为右移符号。在C++11中，这种限制被取消了，C++11标准要求编译器智能的去判断在哪些情况>>不是右移符号。

## auto类型推导
### 静态类型、动态类型与类型推导
auto类型的基本用法：
```c++
int main()
{
    double foo();
    auto x = 1; //x的类型为int，1是const int，这里const类型限制符被去掉了
    auto y = foo(); //y的类型是double
    struct m { int i; }str;
    auto str1 = str; //str1类型是struct m
    auto z; //无法推导，无法通过编译

    return 0;
}
```
auto并非一种类型声明，而是一个类型声明是的占位符，编译器会在编译时期会将auto替代为变量实际的类型。

## auto的优势
* 简化类型声明，提高可读性
```c++
int main()
{
    vector<int> v;
    for(vector<int>::iterator i = v.begin(); i != v.end(); ++i)
    {
        // 一些处理
    }
    //使用auto后可读性成倍增长
    for(auto i = v.begin(); i != v.end(); ++i)
    {
        // 一些处理
    }
    return 0;
}
```
* 免除类型声明的麻烦和错误
```c++
class PI
{
public:
    double operator* (float v)
    {
        return (double)val * v; //精度被拓展
    }
    const float val = 3.14159265f;
}

int main()
{
    float radius = 1.7e10;
    PI pi;
    auto circumference = 2 * (pi * radius);
}
```
>如果使用float来声明circumference，就享受不到pi带来的精度提升效果，不过auto并不能解决所有的精度问题:
```c++
int main()
{
    unsigned int a = 429467295; //最大无符号整数
    unsigned int b = 1;
    auto c = a + b; //c的类型依然是unsigned int 得到结果0
}
```
* auto自适应性能够在一定程度上支持泛型的编程。
如果把上述的例子中PI的操作符返回long double类型，main函数不需要修改什么，因为auto会自适应。auto应用在模板的定义中，自适应性会得到更加充分的体现：
```c++
template <typename T1, typename T2>
double sum(T1 &t1, T2 &t2)
{
    auto s = t1 + t2; //s的类型会在模板实例化时被推导出来
    return s;
}

int main()
{
    int a = 3;
    int b = 5;
    float c = 1.0f, d = 2.3f;

    auto e = sum<int, long>(a, b); //s类型被推导为long
    auto f = sum<float, float>(c, d); //s类型被推导为float
    return 0;
}
```
>这里模板函数的返回值总是double，后面将使用追踪返回类型的函数声明，来释放sum的能量。

* 在宏中的一些实用
```c++
#define Max1(a, b) ((a) > (b)) ? (a) : (b)
#define Max2(a, b) ({ \
        auto _a = (a); \
        auto _b = (b); \
        (_a > _b) ? _a : _b;})

int mian()
{
    int m1 = MAX1(1 * 2 * 3 * 4, 5 + 6 + 7 + 8); //重复计算a和b的值
    int m2 = MAX2(1 * 2 * 3 * 4, 5 + 6 + 7 + 8);
    return 0;
}
```
>由于a和b的类型无法获得，所以无法定义Max2这样高性能的宏。

### auto使用细则
* auto可以与指针引用结合使用。
```c++
int x;
int *y = &x;
double foo();
int& bar();

auto *a = &x; //int*
auto &a = x; //int&
auto c = y; //int*
auto *d = y; //int*
auto *e = &foo(); //编译失败，指针不用只想一个临时变量
auto &f = foo(); //编译失败， nonconst左值引用不能和一个临时变量绑定
auto g = bar(); //int
auto &h = bar(); //int&
```
>1.a、c、d都是指针类型，对于这3个变量来说，auto*和auto并没有啥区别
>2.如果是变量的引用，必须使用auto&

* auto与volatile和const结合使用(cv限制符)，声明为auto的变量不能从初始化表达式中带走cv限制符。
```c++
double foo();b
float* bar();

const auto a = foo(); //a: const double
const auto &b = foo(); //b: const double&
volatile auto *c = foo(); //a: volatile float*

auto d = a; //d: double
auto &e = a; //e: const double&
auto f = c; //f: float*
volatile auto &g = c; //a: volatile float*&
```
>d、f无法带走a和c的常量性和易失性，声明引用的变量都保持了其引用的对象相同的属性。

* auto可以用来声明多个变量，不过这些变量的类型必须相同
```c++
auto x = 1, y = 2; //x和y都是int类型

const auto *m = &x, n = 1; //m是指向一个const int类型变量的指针， n是int类型的变量

auto i = 1, j = 3.14f; //编译失败

auto o = 1, &p = o, *q = &p; //从左向右推导
```
>第三个中，由于i = 1，对于j来说int j = 3.14f会导致精度的缺失。

* C++11新引入的初始化列表，以及new都可以使用auto关键字。
```c++
#include <initializer_list>

auto x = 1;
auto x1(1);

auto y{1};
auto z = new auto(1);
```

* auto也有使用上的限制(语法的二义性)
```c++
#include <vector>

using namespace std;

void fun(auto x = 1) {} //auto函数参数，无法通过编译

struct str
{
    auto var = 10; //类的非静态成员变量，无法通过编译
};

int main()
{
    char x[3];
    auto z[3] = x; //auto数组，无法通过编译

    vector<auto> v = {1}; //auto模板参数(实例化)，无法通过编译
    return 0;
}
```
>虽然人为的观察很容易推导出auto所在位置的类型，但是C++11标准还没有支持这样的使用方式。

## decltype
### typeid与decltype
C++在C++98中就部分支持动态类型了(RTTI)，RTTI的机制是为每个类型产生一个type_info类型的数据，可以在程序中使用typeid查询一个变量的类型。typeid会返回变量的type_info数据。type_info的name成员函数返回类型的名字，C++11增加了hash_code这个函数，返回该类型唯一的哈希值。
```c++
#include <iostream>
#include <typeinfo>

using namespace std;

class White {};
class Black {};

int main()
{
    White a;
    Black b;

    cout << typeid(a).name() << endl; //class White
    cout << typeid(b).name() << endl; //class Black

    bool sameType = typeid(a).hash_code() == typeid(b).hash_code();
    cout << "a and b ? " << sameType << endl; //0

    White c;
    sameType = typeid(a).hash_code() == typeid(c).hash_code();
    cout << "a and c ? " << sameType << endl; //1

    return 0;
}
```
除了typeid，RTTI还包括了C++中的dynamic_cast等特性。不过RTTI会带来一些运行时的开销，所以编译器会让用户选择性的关闭该特性(GCC的选项-fno-rttion，微软编译器/GR-)

在C++的发展中，类型推导式随着模板和泛型编程的广泛使用而引入的。在上面的auto例子中：
```c++
template <typename T1, typename T2>
double sum(T1 &t1, T2 &t2)
{
    auto s = t1 + t2; //s的类型会在模板实例化时被推导出来
    return s;
}
```
t1 + t2的返回值是不确定类型的，这种写法会限制模板的使用范围和编写方式。C++11制定了auto和decltype，decltype也可以进行类型推导，不过两者的使用方式有一定的区别。
```c++
int main()
{
    int i;
    decltype(i) j = 0;
    cout << typeid(j).name() << endl; //int

    float a;
    double b;
    decltype(a + b) c;
    cout << typeid(c).name() << endl; //double

    return 0;
}
```
decltype总是一个普通的表达式作为参数，返回该表达式的类型。与auto相同，decltype类型推导也是在编译时进行的。

### decltype的应用
* 配合using/typedef使用，在C++11的头文件经常看到下列代码：
```c++
using size_t = decltype(sizeof(0));
using ptrdiff_t = decltype((int*)0 - (int*)0);
using nullptr_t = decltype(nullptr);
```
* 增加可读性
```c++
int main()
{
    vector<int> vec;
    typedef decltype(vec.begin()) vectype;
    for(vectype i = vec.begin(); i != vec.end(); ++i)
    {
        // 一些代码
    }

    for(decltype(vec)::iterator i = vec.begin(); i != vec.end(); ++i)
    {
        // 一些代码
    }

    return 0;
}
```
* 使用匿名类型
```c++
enum class{K1, K2, K3}anon_e; //匿名的强类型枚举

union 
{
    decltype(anon_e) key;
    char* name;
}anon_u;
```
* 扩大模板泛型的能力,改造auto例子的模板相加
```c++
template <typename T1, typename T2>
void sum(T1 &t1, T2 &t2, decltype(t1 + t2) &s)
{
    s = t1 + t2;
}
```
>**Tip:**
>**函数的参数是从右到左入栈的，入栈前会把表达式参数算一遍**

* C++11标准也会依赖decltype的类型推导(result_of推导函数的返回类型)
```c++
typedef double (*func)();

int main()
{
    result_of<func()>::type f; 
    cout << typeid(f).name() << endl; //double
    return 0;
}
```
### decltype推导四规则
如果程序员使用decltype(e)来获取类型时：
* 如果e是一个没有带括号的标记符表达式(id-expression)或者类成员访问表达式，那么decltype(e)就是e所命名的实体的类型。此外，如果e是一个被重载的函数，则会导致编译时错误。
* 否则，假设e的类型是T，如果e是一个将亡值(xvalue)，那么decltype(e)为T&&。
* 否则，假设e的类型是T，如果e是一个左值，则decltype(e)为T&。
* 否则，假设e的类型是T，则decltype(e)为T。
```c++
struct A
{

};

int main()
{
    int i = 0;
    decltype(i) j = 0; //int，规则1
    decltype((i)) k = i; //int&，(i)是左值表达式(可以有具名的地址)，规则3

    A a;
    cout << is_rvalue_reference<decltype(move(a))>::value << endl; //int&&， 1，move(a)是将亡值，规则2

    decltype(++i) b = i; // ++i返回i的左值，规则3
    decltype(i++) c; //int，i++返回右值，规则4

    return 0;
}
```

### cv限制符的继承和冗余的符号
* 与auto类型推导不能带走cv限制符不同，decltype能带走表达式的cv限制符。如果对象的定义中有const或volatile限制符，使用decltype进行推导时，成员不会继承const或volatile限制符。
```c++
int main()
{
    const int ic = 0;
    volatile int iv;

    cout << is_const<decltype(ic)>::value << endl; //1
    cout << is_volatile<decltype(iv)>::value << endl; //1

    return 0;
}
```
* 与auto相同，decltype从表达式推导出类型后，进行类型定义时，也会允许一些冗余的符号。比如cv限制符和引用符号&。如果推导出的类型已经有了这些符号，冗余的符号则会被忽略。
```c++
int mian()
{
    int i = 1;
    int &j = i;
    int *p = &i;

    decltype(j) &a = i; //冗余的&

    auto *b = &i; //int*
    decltype(p) *c = &p; //int** 编译器不会忽略decltype后面的*号
    return 0;
}
```

## 追踪返回类型
### 追踪返回类型的引入
追踪返回类型配合auto与decltype会真正释放C++11中泛型编程的能力。

如果一个函数模板的返回类型依赖于实际的入口参数类型，那么该返回类型在模板实例化之前可能都无法确定。
```c++
template<typename T1， typename T2>
decltype(t1 + t2) sum(T1 &t1, T2 &t2) //编译器推导decltype(t1 + t2)，t1和t2都未声明，编译器只会从左到右读入符号
{
    return t1 + t2;
}
```
上面的问题，C++11引入新语法**追踪返回类型**，来声明和定义这样的函数。
```c++
template<typename T1， typename T2>
auto sum(T1 &t1, T2 &t2) -> decltype(t1 + t2)
{
    return t1 + t2;
}
```
auto占位符合->return_type是构成追踪返回类型函数的两个基本元素。

### 使用追踪返回类型的函数
* 追踪类型的一个好处就是模板类型推导，上面的sum例子。
* 另外一个好处就是简化函数的定义
```c++
int (*(*pf())())() //这是面试题
{
    return nullptr;
}

//auto (*)() -> int (*)() 一个返回函数指针的函数(假设为a函数)
//auto pf1() -> auto (*)() -> int(*)() 一个返回a函数的指针的函数

auto pf1() -> auto (*)() -> int(*)()
{
    return nullptr;
}

int main()
{
    cout << is_same<decltype(pf), decltype(pf1)>::value << endl; //1
    return 0;
}
```
>**PS:说实话那个面试题的什么鬼我都不知道**

* 用在转发函数
```c++
template <class T>
auto Fowrad(T t) -> decltype(foo(t))
{
    return foo(t);
}
```

