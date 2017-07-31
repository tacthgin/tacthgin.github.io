---
title: 模板测试
date: 2017-7-26 11:36:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 概要
当片段着色器处理完片段之后，模板测试(Stencil Test) 就开始执行了，和深度测试一样，它能丢弃一些片段。仍然保留下来的片段进入深度测试阶段。
模板测试基于**模板缓冲(Stencil Buffer)**，模板缓冲中的**模板值(Stencil Value)**通常是8位的。因此每个片段/像素共有256种不同的模板值。
**Important**
**每个窗口库都需要为你设置模板缓冲。GLFW自动做了这件事，所以你不必告诉GLFW去创建它，但是其他库可能没默认创建模板库，所以一定要查看你使用的库的文档。**
模板缓冲例子：
![](stencil_buffer.png)
模板缓冲先清空模板缓冲设置所有片段的模板值为0，然后开启矩形片段用1填充。场景中的模板值为1的那些片段才会被渲染（其他的都被丢弃）。
改变模板缓冲的内容实际上就是对模板缓冲进行写入。在同一次（或接下来的）渲染迭代我们可以读取这些值来决定丢弃还是保留这些片段。当使用模板缓冲的时候，需要遵守下面的原则：
* 开启模板缓冲写入。
* 渲染物体，更新模板缓冲。
* 关闭模板缓冲写入。
* 渲染（其他）物体，这次基于模板缓冲内容丢弃特定片段。

1.使用GL_STENCIL_TEST开启模板测试
```C++
glEnable(GL_STENCIL_TEST);
```
2.清空模板缓冲
```C++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
```
3.给模板值设置一个**位遮罩(Bitmask)**，它与模板值进行按位与(AND)运算决定缓冲是否可写。
```C++
// 0xFF == 0b11111111
//此时，模板值与它进行按位与运算结果是模板值，模板缓冲可写
glStencilMask(0xFF); 

// 0x00 == 0b00000000 == 0
//此时，模板值与它进行按位与运算结果是0，模板缓冲不可写
glStencilMask(0x00); 
```
## 模板函数
1.配置模板测试：glStencilFunc和GlStencilOp
void glStencilFunc(GLenum func, GLint ref, GLuint mask)函数有三个参数：
* func：设置模板测试操作。这个测试操作应用到已经储存的模板值和glStencilFunc的ref值上，可用的选项是：GL_NEVER、GL_LEQUAL、GL_GREATER、GL_GEQUAL、GL_EQUAL、GL_NOTEQUAL、GL_ALWAYS。它们的语义和深度缓冲的相似。
* ref：指定模板测试的引用值。模板缓冲的内容会与这个值对比。
* mask:指定一个遮罩，在模板测试对比引用值和储存的模板值前，对它们进行按位与（and）操作，初始设置为1。
例子：
```C++
glStencilFunc(GL_EQUAL, 1, 0xFF);
```
glStencilOp描述我们如何更新缓冲
void glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)
* sfail： 如果模板测试失败将采取的动作。
* sfaidpfaill： 如果模板测试通过，但是深度测试失败时采取的动作。
* dppass： 如果深度测试和模板测试都通过，将采取的动作。
每个选项都可以使用下列任何一个动作：
![](stencil_action.png)
















**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/02%20Stencil%20testing/)**