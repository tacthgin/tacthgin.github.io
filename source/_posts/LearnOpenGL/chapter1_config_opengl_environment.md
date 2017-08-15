---
title: 配置OpenGL环境
date: 2016-10-17 10:00:37
tags: 
    OpenGL
category: LearnOpenGL
---

## GLFW
GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入。

## 构建GLFW
GLFW可以从它官方网站的[下载页](http://www.glfw.org/download.html)上获取。解压下载的包可以得到：
1.编译生成的库
2.include文件夹
我们需要让IDE知道库和头文件的位置。有两种方法：
1.找到IDE或者编译器的/lib和/include文件夹，添加GLFW的include文件夹里的文件到IDE的/include文件夹里去。用类似的方法，将glfw3.lib添加到/lib文件夹里去。
2.推荐的方式是建立一个新的目录包含所有的第三方库文件和头文件，并且在你的IDE或编译器中指定这些文件夹。
在Virtual Studio中配置:
1.我们首先进入Project Properties(工程属性，在解决方案窗口里右键项目)，然后选择VC++ Directories(VC++ 目录)选项卡（如下图）。在下面的两栏添加目录：($(SolutionDir):vs工程目录路径)
![](1.png)
2.在Linker(链接器)选项卡里的Input(输入)选项卡里添加glfw3.lib这个文件：
![](2.png)
3.添加头文件
```c++
#include <GLFW\glfw3.h>
```
## GLEW
因为OpenGL只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡实现的。由于OpenGL驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。
任务就落在了开发者身上，开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用。取得地址的方法因平台而异，在Windows上会是类似这样：
```c++
// 定义函数原型 
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);
// 找到正确的函数并赋值给函数指针 
GL_GENBUFFERS glGenBuffers = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers"); 
// 现在函数可以被正常调用了 
GLuint buffer; 
glGenBuffers(1, &buffer);
```
(PS:GLEW对OpenGL函数指针api进行了封装，使之更容易使用)

## 编译和链接GLEW
我们使用GLEW的静态版本glew32s.lib（注意这里的“s”），将库文件添加到你的库目录，将include内容添加到你的include目录。接下来，在VS的链接器选项里加上glew32s.lib。注意GLFW3（默认）也是编译成了一个静态库。
如果你希望静态链接GLEW，必须在包含GLEW头文件之前定义预处理器宏GLEW_STATIC：
```c++
#define GLEW_STATIC
#include <GL/glew.h>
```
如果你希望动态链接，那么你可以省略这个宏。但是记住使用动态链接的话你需要拷贝一份.DLL文件到你的应用程序目录

我们现在成功编译了GLFW和GLEW库，我们已经准备好将进入下一节去真正使用GLFW和GLEW来设置OpenGL上下文并创建窗口。记得确保你的头文件和库文件的目录设置正确，以及链接器里引用的库文件名正确。

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/02%20Creating%20a%20window/)**