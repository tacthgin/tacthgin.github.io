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
Head作为tuple&lt;Head, Tail...>的第一个成员，tuple&lt;Tail...>作为tuple&lt;Head, Tail>的私有基类。当实例化一个tuple&lt;double, int, char, float>类型时候，会引起基类的递归构造，递归在tuple的参数包为0的时候会结束。编译器将从tuple<>构造出tuple&lt;float>，然后tuple&lt;char, float>、tuple&lt;int, char, float>，最后tuple&lt;double, int, char, float>。
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
C++11标准定义了7中参数包可以展开的位置：
* 表达式
* 初始化列表
* 基类描述列表
* 类成员初始化列表
* 模板参数列表
* 通用属性列表
* lambda函数的捕捉列表

其他一些有趣的包扩展表达式：
* 声明Arg为参数包，使用Arg&&的包扩展表达式，解包后等价于Arg1&&，...，Argn&&
* 继承的解包
```c++
template <typename... A> class T : privateB<A>...{};
//相当于 class T<X, Y> : private B<X>, private B<Y>{};

template <typename... A> class T : private B<A...>{}
//相当于 class T<X, Y> : private B<X, Y>{};
```
* 函数参数的解包
```c++
template <typename T>
T pr(T t)
{
	cout << t;
	return t;
}

template <typename... Args>
void dumpWrapper(Args... args)
{

}

template <typename... Args>
void dump(Args... args)
{
    dumpWrapper(pr(args)...); //调用pr(arg1), pr(arg2)
}

int main()
{
    dump("a", "b", "c", "d"); //dcba 
    return 0;
}
```
* 操作符sizof...,计算变长包的长度

## 原子类型与原子操作
### 原子操作与C++11原子类型
原子操作就是多线程程序中最小的且不可并行化的操作。如果对一个共享的资源操作是原子操作，那么多个线程访问该资源，仅有一个线程在对这个资源进行操作。通常情况原子操作都是通过互斥的访问来保证的(互斥锁)。
在C++11中，通过对并行编程更为良好的抽象，实现同样的功能就简单了很多。
* 原子类型
```c++
#include <iostream>
#include <atomic>
#include <thread>

using namespace std;

atomic_llong total { 0 };

void func(int)
{
    for (long long i = 0; i < 100000000LL; ++i)
    {
        total += i;
    }
}

int main()
{
    thread t1(func, 0);
    thread t2(func, 0);

    t1.join();
    t2.join();

    cout << total << endl; //9999999900000000 如果使用传统的long long类型在vs2015上是5634540629934525

    return 0;
}
```
* &lt;cstdatomic>头文件中的原子类型
![](atomic_type.png)

* atomic类模板
```c++
std::atomic<T> t;

atomic<float> af {1.2f};
atomic<float> af1 {af}; //无法通过编译

float f = af;
float f1 = {af};
```
* 原子类型能够保持的原子性的原因是编译器能够保证对原子类型的操作都是原子操作，在C++11中，将原子操作定义为atomic模板类的成员函数。
![](atomic_handle.png)
atomic-integral-type和integral-type指的是表6-1中所有原子类型的整型，class-type指的是自定义类型。对于大多数原子类型而言，都可以执行读(load)、写(store)、交换(exchange)、比较交换(compare-exchange_weak/compare_exchange_stronge)。
```c++
atomic<int> a;
int b = a; //a.load()

atomic<int> a;
a = 1; //a.store(1)
```
* atomic_flag是无锁的，线程对其访问不需要加锁，不需要使用load、store等成员函数进行读写。通过atomic_flag的成员test_and_set以及clear，可以实现一个自旋锁(spin lock)。
```c++
#include <iostream>
#include <atomic>
#include <thread>
#include <windows.h>

using namespace std;

atomic_flag lock = ATOMIC_FLAG_INIT; //设置初始状态false

void f(int n)
{
    while (lock.test_and_set(memory_order_acquire)) //尝试获得锁，不断设置为true
        cout << "Waiting from thread" << n << endl; //自旋
    cout << "Thread " << n << " strats working" << endl;
}

void g(int n)
{
    cout << "Thread " << n << " is going to start" << endl;
    lock.clear(); //设置标志位false
    cout << "Thread " << n << " strats working" << endl;
}


int main()
{
    lock.test_and_set(); //设置为true
    thread t1(f, 1);
    thread t2(g, 2);

    t1.join();
    Sleep(1000);
    t2.join();

    return 0;
}
```

### 内存模型，顺序一致性与memory_order
线程间看到的内存数据被改变的顺序与机器指令中的一致就是强顺序的，反之就是弱顺序的。
**Tip：**
**弱顺序的内存模型可以使得处理器进一步发掘指令中的并行性，使得指令执行的性能更高。**

C++11中的内存模型要保证代码的顺序一致性，就必须同时做到以下几点：
* 编译器保证原子操作的指令间顺序不变，既保证产生的读写原子类型的变量的机器指令与代码编写者看到的是一致的。
* 处理器对原子操作的汇编指令的执行顺序不变。这对于x86这样的强顺序的体系结构来说，不成问题；对于PowerPC这样的弱顺序的体系结构而言，则要求编译器在每次原子操作后加入内存栅栏。

在C++11中，程序员可以为原子操作指定所谓的内存顺序：memory_order。比如松散的内存模型(relaxed memory model)：
```c++
atomic<int> a {0};
int t = 1;
a.store(t, memory_order_relaxed);
```
C++11定义了7种memory_order的枚举值：
![](memory_order.png)

## 线程局部存储
**线程局部存储(TLS,thread local storage)**是拥有线程生命期及线程可见性的变量。TLS变量可以在每个线程拥有自己各自的值，在C++11中，使用thread_local修饰符声明变量即可：
```c++
int thread_local errCode;
```
TLS变量在线程开始时候被初始化，在线程结束的时候失效。对thread_local变量取地址值，也只能获得当前线程中的TLS变量的地址值。

## 快速退出：quick_exit与at_quick_exit
* terminate函数，没有被捕捉的异常就会导致terminate函数的调用。此外noexcept关键字声明的函数，如果抛出了异常，也会调用terminate函数。terminate函数在默认情况，是去调用abort函数，不过用户可以通过set_terminate函数来改变默认的行为。
* abort函数不会调用任何的析构函数(默认的terminate也是如此)，abort会抛出一个信号：SIGABRT，操作系统将默认的释放进程所有的资源。如果被终止的应用与其他应用有些交互(硬件驱动程序，网络通信程序，假设这些程序设计的不那么健壮)，可能会使这些交互进程处于一些中间状态，进而出现一些问题。
* exit属于正常退出，它会正常调用自动变量的析构函数，还会调用atexit注册的函数。这跟main函数结束时的清理工作是一样的。注册的函数被调用的次序与注册顺序相反。不过exit结束程序的方式也不那么令人满意：
* 类在堆上分配了大量的零散内存，exit函数调用会导致析构函数将这些零散的内存还给操作系统，是一件费时的工作。实际上，堆内存在进程结束就由操作系统统一回收(相当快，操作系统除了释放一些进程相关的数据结构外，直接把物理内存标记为未使用就可以了)。
* 多线程情况，使用exit函数退出程序，需要向线程发一个信号，并等待线程结束后执行析构函数，atexit注册的函数，在一些复杂的情况下，可能会因为信号顺序而导致死锁的状况，程序就会卡死无法退出。
* C++11引入quick_exit，该函数不执行析构函数而只是使程序终止。quick_exit与exit同属于正常退出，at_quick_exit注册的函数也可以在quick_exit的时候被调用。