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
enum Category { General, Pistol, MachineGun, cannon };
```
>因为General是全局的，所以编译会报错。

* 由于C中枚举被设计为常量数值的别名，所以枚举成员总是可以隐式的转换为整形。很多时候，这是不安全的。
```c++
enum Type { General, Light, Medium, Heavy};
enum Category { Pistol, MachineGun, cannon };

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
enum class Category { General = 1, Pistol, MachineGun, cannon };

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