---
title: 提高类型安全
date: 2017-8-28 12:10:00
tags: 
    C++11
category: C++11
---

## 强类型枚举
### 枚举：分门别类与数值的名字
C++定义数值的名字有以下3种方式实现：
* 宏，不过宏的弱点在于其定义的只是预处理阶段的名字，如果代码中有相同的字符串，无论什么位置都会被替代，所以程序员会让宏全部以大写字母来命名，以区别于正常的代码。
* 匿名枚举，匿名枚举的成员都是编译时期的名字，会得到编译器的检查。
* 静态变量，同样得到编译时期的检查，并且名字作用域局限于文件内，不过会增加一点存储空间。

### 有缺陷的枚举类型
* C++的具名enum类型的名字，以及enum成员的名字都是全局可见。容易出现如下的问题：
```c++
enum Type { General, Light, Medium, Heavy};
enum Category { General, Pistol, MachineGun, Cannon };
```
>因为General是全局的，所以编译会报错。

* 由于C中枚举被设计为常量数值的别名，所以枚举成员总是可以隐式的转换为整形。很多时候，这是不安全的。
```c++
enum Type { General, Light, Medium, Heavy};
enum Category { Pistol, MachineGun, Cannon };

int main()
{
    Type t = General;
    if(t > Pistol) //比较错类型
    {
        //一些代码
    }
    return 0;
}
```
* 枚举的占用空间，普通的枚举占用4个字节空间，需要的时候会扩展到8字节。

### 强类型枚举以及C++11对原有枚举类型的扩展
声明强类型枚举只要在enum后面加上关键字class。具有以下几点优势：
* 强作用域，枚举成员的名称不会被输出到其父作用域空间。
* 转换限制，枚举成员的值不可以与整型隐式的相互转换。
* 可以指定底层类型，强类型枚举默认的底层类型为int，但可以指定底层类型，在枚举名称后面加上: type，type可以是除wchar_t以外的任何整型。
```c++
enum class Type { General, Light, Medium, Heavy};
enum class Category { General = 1, Pistol, MachineGun, Cannon };

int main()
{
    Type t = Type::Light;
    t = General; //编译失败，必须用强类型的名称

    if(t == Category::General) //编译失败，必须使用Type中的General
    {
        //一些代码
    }
        
    if(t > Type::General) //编译通过
    {
        //一些代码
    }

    if(t > 0) //编译失败，无法转换为int类型
    {
        //一些代码
    }

    if((int)t > 0) //编译通过
    {
        //一些代码
    }

    cout << is_pod<Type>::value << is_pod<Category>::value << endl; //11
    return 0;
}
```
* 节省内存空间
```c++
enum class C : char { C1 = 1, C2 = 2 };
enum class D : unsigned int { D1 = 1, D2 = 2, Dbig = 0xFFFFFFF0U };

int main()
{
    cout << sizeof(C::C1) << endl; //1
    cout << (unsigned int)D::Dbig << endl; //4294967280
    cout << sizeof(D::D1) << endl; //4
    cout << sizeof(D::Dbig) << endl; //4
    return 0;
}
```
* 匿名的强类型枚举可能什么都做不了。
```c++
enum class { General, Light, Medium, Heavy} weapon; //无法编译通过

int main()
{
    weapon = General; //无法编译通过
    bool b = (weapon == weapon::General); //无法编译通过
    return 0;
}
```

C++11对原有的枚举类型有做了一些改动。
* 原有的枚举类型也可以指定除了wchar_t之外的整型。
* 枚举成员的名字除了会自动输出到父作用域，也可以在枚举类型定义的作用域有效。
```c++
enum Type { General, Light, Medium, Heavy };
Type t1 = General;
Type t2 = Type::General;
```

## 堆内存管理：智能指针与垃圾回收
### 显式内存管理
由于C/C++允许程序员自由的管理内存，但是一旦使用不当，容易出现以下几个问题：
* 野指针：指针指向的内存已经被释放，但是这个指针依然被使用。
* 重复释放：重复释放已经被释放过的内存
* 内存泄漏：不需要使用的内存单元没有被释放，并且反复的进行此类操作，导致大量内存泄漏

### C++11的智能指针
#### unque_ptr
unque_ptr只绑定一份对象的内存，不能与其他unque_ptr共享。
* 没有拷贝构造函数(拷贝赋值运算符同理)，不允许拷贝智能指针。
* 有移动构造函数(移动赋值运算符同理)，允许移动智能指针。

```c++
#include <memory>

using namespace std;

int main()
{
    unique_ptr<int> p(new auto(1));
    unique_ptr<int> p1 = p; //不能通过编译
    unique_ptr<int> p2 = move(p);

    int i = *p; //运行时错误
    p2.reset(); //显式释放内存
    p.reset(); //不会导致运行时错误

    return 0;
}

```
#### shared_ptr
shared_ptr允许多个指针共享一份内存。使用引用计数来控制释放内存的时机。
```c++
#include <memory>

using namespace std;

int main()
{
    shared_ptr<int> sp(new auto(1));
    shared_ptr<int> sp1 = sp;
    cout << sp1.use_count() << endl; //2

    return 0;
}
```
shared_ptr要注意一点就是不能陷入循环引用，不然会引起内存泄漏

#### weak_ptr
weak_ptr可以接收一个shared_ptr判断指针是否已经被释放过了。
```c++
#include <memory>

using namespace std;

int main()
{
    shared_ptr<int> sp(new auto(1));
    sp.reset();

    weak_ptr<int> wp(sp);
    if (wp.lock()) //Pointer is invalid
    {
        cout << "Pointer is not invalid" << endl;
    }
    else
    {
        cout << "Pointer is invalid" << endl; 
    }

    return 0;
}
```
