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
使用is_move_constructible、is_trivially_move_constructible、is_nothrow_move_constructible来判断一个类型是否是可以移动的。
```c++
cout << is_move_constructible<T>::value;
```
#### 完美转发
完美转发(perfect forwarding)，是指在函数模板中，完全依照模板的参数的类型，将参数传递给函数模板中调用的另外一个函数：
```c++
template <typename T>
void IamForwording(T t) //转发函数模板
{
    IrunCodeActually(t); //真正执行目标代码
}
```
转发函数将参数按照传入IamForwording时的类型传递(传入的是左值对象，目标函数就能获得左值对象。传入的是右值对象，目标函数就能获得右值对象)，不产生额外的开销。上面的例子会产生拷贝构造，谈不上完美转发。

C++11引入一个"引用折叠"的新语言规则，来实现完美转发。
![](reference_folding.png)
可以把转发函数写成如下形式：
```c++
template <typename T>
void IamForwording(T &&t)
{
    IrunCodeActually(static_cast<T&&>(t));
}
```
**Tip:**
**这里的static_cast是用来传递右值的，得出的结果会被转化为右值**
```c++
static_cast<T&& &&>
```

在C++11中，实现完美转发的函数叫做**forward**。虽然std::move也可以用来完美转发，但这并不是推荐的做法（C++11用专门的foward来实现，或者考虑未来拓展）。

### 显式转换操作符
C++11将explicit范围扩展到自定义的类型转换操作符上。
```c++
template <class T>
class Ptr
{
public:
    Ptr(T* p):_p(p) {} 
    operator bool() const
    {
        return _p != 0;
    }
private:
    T* _p;
};

int main()
{
    int a;
    Ptr<int> p(&a); 
    if(p) //转换为bool类型
    {
        cout << "valid pointer" << endl;
    }

    Ptr<double> pd(0);
    cout << p + pd << endl; //得到1.相加语义上没有意义，如果添加explicit编译错误
    return 0;
}
```
添加完explicit，if(p)可以通过编译，因为可以通过p直接构造出bool型变量，p + pd无法通过编译是因为全局的operator+不支持bool类型作为参数(之前强转)。

### 列表初始化
#### 初始化列表
C++11添加了集合的初始化，称为"初始化列表"。
```c++
#include <vector>
#include <map>
using namespace std;

int main()
{
    int a[] = {1, 3, 5}; //C++98通过，C++11通过
    int b[] {2, 4, 6}; //C++98失败，C++11通过
    vector<int> c{1, 3, 5}; //C++98失败，C++11通过
    map<int float> d = {{1, 1.0f}, {2, 2.0f}}; //C++98失败，C++11通过
    return 0;
}
```
C++11支持以下几种初始化方式:
* 等号 "=" 加上赋值表达式(assignment-expression)，比如int a = 3 + 4。
* 等号 "=" 加上花括号的初始化列表，比如int a = {3 + 4}。
* 圆括号的表达式列表(expression-list)，比如int a(3 + 4)。
* 花括号式的初始化列表，比如int a{3 + 4}。

后2种形式可以用于new操作符：
```c++
int *i = new int(1);
double *d = new double(1.2f);
```
C++11中，可以使用<initializer_list>中的initialize_list<T>模板类来使自定义的类使用列表初始化。
```c++
#include <vector>
#include <string>

using namespace std;

enum Gender{boy, girl};

class People
{
public:
    People(initializer_list<pair<string, Gender>> l)
    {
        for(auto i = l.begin(); i != l.end(); ++i)
            _data.push_back(*i);
    }
private:
    vector<pair<string, Gender>> _data;
};

int main()
{
    People ship2012 = {{"Garfield", boy}, {"HelloKitty", girl}};
    return 0;
}
```
initializer_list在普通的函数和操作符都可以使用:
```c++
#include <iostream>
#include <vector>

using namespace std;

class Mydata
{
public:
    Mydata& operator[](initializer_list<int> l)
    {
        for(auto i = l.begin(); i != l.end(); ++i)
        {
            _idx.push_back(*i);
        }
        return *this;
    }
    Mydata& operator=(int v)
    {
        if(!_idx.empty())
        {
            for(auto i = _idx.begin(); i != _idx.end(); ++i)
            {
               _data.resize((*i > _data.size()) ? *i : _data.size());
               _data[*i - 1] = v;
            }

            _idx.clear();
        }

        return *this;
    }
private:
    vector<int> _idx;
    vector<int> _data;
};

int main()
{
    Mydata d;
    d[{2, 3, 5}] = 7;
    d[{1, 4, 5, 8}] = 4;
    return 0;
}
```
#### 防止类型收窄
类型收窄指的是使得数据变化或者精度丢失的隐式类型转换。
* 浮点数隐式转化为整型。
* 高精度的浮点数转化为低精度的浮点数。
* 整型转化为浮点型(整型大到浮点数无法精确表示)
* 整型转化为较低长度的整型(unsigned char = 1024)

在C++11中，使用初始化列表会检查是否发生类型收窄：
```c++
const int x = 1024;
const int y = 10;

char a = x; //收窄，可以通过编译
char *b = new char(1024); //收窄，可以通过编译

char c = {x}; //收窄，无法通过编译
char d = {y}; //收窄，可以通过编译
```
### POD类型
对于c的那套api兼容就不说了，C++11对POD划分2个基本概念的合集：平凡的(trivial)和标准布局的(standard layout)
trivial的定义：
* 拥有trivial constructor和trivial destructor。(不能定义构造函数，编译器会自己生成)
* 拥有trivial copy constructor和trivial move constructor。
* 拥有trivial assignment operator和trivial move operator。
* 不能包含虚函数以及虚基类
* 如果成员变量拥有no trivial构造，那么这个类就不是trivial(C++对象模型书本看的)

在C++中可以使用类模板来判断是不是trivial:
```c++
template<typename T>
struct std::is_trivial;

is_trivial<T>::value;
```

标准布局的定义：
* 所有非静态成员有相同的访问权限(public, private, prtected)
* 在类或者结构体继承时，满足以下2种情况之一：
1.派生类中有非静态成员，且只有一个仅包含静态成员的基类。
2.基类有非静态成员，派生类没有。
```c++
struct B1 {static int a;}
struct D1 : B1 {int d;}

struct B2 {int a;}
struct D2 : B2 {static int d;}

struct D3 : B2, B1 {static int d}
struct D4 : B2 {int d;}
struct D5 : B2, d1{};
```
>D1、D2和D3是标准布局，D4、D5不是，说明只要非静态成员同时出现在基类和子类中就不属于标准布局。

* 类中的第一个非静态成员的类型与其基类不同
```c++
//不是标准布局
struct B{};

struct A : B
{
    B b;
}; 

//标准布局
struct C : B
{
    int a;
    B b;
};
```
>**Tip**
>**如果基类没有成员，C++11允许派生类的第一个成员与基类共享地址，派生类的地址总是"堆叠"在基类之上，表明基类没有占据任何的实际空间(可以节省点数据)。如果第一个成员仍然是基类，编译器会为基类分配1个字节的空间**

![](fisrt_member_is_base.png)

* 没有虚函数和虚基类
* 所有非静态数据成员符合标准布局模型，基类也符合标准布局。

标准布局可以使用模板类来判断：
```c++
template <typename T>
struct std::is_standard_layout;

is_standard_layout<T>::value;
```

对于POD类型来说，C++也可以用模板类来判断：
```c++
template <typename T>
struct std::is_pod;

is_pod<T>::value;
```

使用POD类型有如下好处：
* 字节赋值，可以安全的使用memset和memcpy对POD类型进行初始化和拷贝等操作。
* 提供对C内存布局兼容。C++程序可以与C函数进行相互操作。
* 保证了静态初始化的安全有效。静态初始化在很多时候能提高程序的性能，而POD类型对象的初始化往往更简单(放入目标文件的.bbs段，在初始化中直接被赋值0)。

### 非受限联合体
在C++98中，如果一个联合体有一个不是POD类型的类，无法通过编译：
```c++
struct A 
{
    int a = {1};
    A() {}
};

union T
{
    A a; //无法通过编译
    int b;
};
```
经过长期实践应用证明，C++98标准对于联合体的限制是完全没有必要的：
```c++
union
{
    double d;
    complex<double> cd; //错误，不是POD类型
};

union
{
    double d;
    complex c; //c语言的复数通过编译
};
```
由于C++的复数是用模板实现，这样联合体保持和C的兼容就形同虚设。

在C++11标准中，取消了联合体对于数据成员类型的限制，任何非引用类型都可以成为联合体的数据成员，这样的联合体叫做**非受限联合体(Unrestricted Union)**。
* 不允许静态成员变量的存在(否则该类型的联合体将共享一个值)。
* 拥有静态成员函数，返回常数。
* 如果有非POD成员类型，联合体的默认构造函数将被删除，其他的特殊成员函数(默认拷贝构造、拷贝赋值操作符和析构函数)也遵从此规则。
```c++
union T
{
    string s; //非POD类型
    int n;
}

int main()
{
    T t; //构造失败，因为T的构造函数被删除了
    return 0;
}
```
* 解决构造函数被删除的问题是，自己定义非受限联合体的构造函数。使用placement new来调用非POD类型的构造函数。
```c++
union T
{
    string s;
    int n;
public:
    T() { new (&s) string; }
    ~T() { s.~string(); }
}

int main()
{
    T t; //构造析构成功
    return 0;
}
```
>**Tip：**
**析构的时候union T也必须是一个string对象，否则可能导致析构的错误**

* 匿名非受限联合体可以运用在类的声明中，这样的类被称为**枚举式的类(union-like class)**。
```c++
#include <cstring>
using namespace std;

struct Student
{
    Student(bool g, int a) : gender(g), age(a) {}
    bool gender;
    int age;
};

class Singer
{
public:
    enum Type{ STUDENT, NATIVE, FOREIGNER};

    Singer(bool g, int a) : s(g, a) { t = STUDENT;}
    Singer(int i) : id(i) { t = NATIVE;}
    Singer(const char* n, int s)
    {
        int size = (s > 9) ? 9 : s;
        memcpy(name, n, size);
        name[s] = "\0";
        t = FOREIGNER;
    }
private:
    Type t;
    union //匿名的非受限联合体
    {
        Student s;
        int id;
        char name[10];
    };
};

int main()
{
    Singer(true, 13);
    Singer(310217);
    Singer("Hello world", 9);
    return 0;
}
```
>非受限联合体成为类Singer的**变长成员(variant member)**。

### 用户自定义字面量
C++11允许制定一个后缀标识的操作符，来实现自定义的类型。比如120W，200N这种物理单位：
```c++
struct Watt{ unsigned int v;}

Watt operator "" _w(unsigned long long v)
{
    return {(unsigned int) v};
}

int main()
{
    Watt capacity = 1024_w;
    return 0;
}
```
在C++11中，标准要求声明字面量操作符有一定的规则：
* 如果字面量为整型数，那么字面量操作符函数参数只可接受unsigned long long或者const char\* 为其参数。当unsigned long long无法容纳该字面量的时候，编译器会自动将该字面量转化为以\0为结束符的字符串，并调用const char\* 为参数的版本进行处理。
* 如果字面量为浮点型数，则字面量操作符函数参数只可接受long double或者const char\* 为其参数。调用规则同整型数一样。
* 如果字面量为字符串，则字面量操作符函数参数只可接受const char\*， size_t为其参数(已知长度的字符串)。
* 如果字面量为字符，则字面量操作符函数参数只可接受一个char作为参数。

使用自定义字面量应该注意一下几点：
* 在字面量操作符函数的声明中，operator""与用户自定义后缀之前必须要有空格。
* 后缀建议以下划线开始，否则会被编译器警告。