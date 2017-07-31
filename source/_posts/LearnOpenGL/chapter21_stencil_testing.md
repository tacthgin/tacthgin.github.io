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
glStencilOp函数默认设置为 (GL_KEEP, GL_KEEP, GL_KEEP) ，所以任何测试的任何结果，模板缓冲都会保留它的值。默认行为不会更新模板缓冲，所以如果你想写入模板缓冲的话，你必须像任意选项指定至少一个不同的动作。

## 物体轮廓
**物体轮廓(Object Outlining)**是给物体创建一个有颜色的边，加上轮廓的步骤如下：
1.在绘制物体前，把模板方程设置为GL_ALWAYS，用1更新物体将被渲染的片段。
2.渲染物体，写入模板缓冲。
3.关闭模板写入和深度测试。
4.每个物体放大一点点。
5.使用一个不同的片段着色器用来输出一个纯颜色。
6.再次绘制物体，但只是当它们的片段的模板值不为1时才进行。
7.开启模板写入和深度测试。
先画出正常的箱子：
```C++
glStencilFunc(GL_ALWAYS, 1, 0xFF); //所有片段都要写入模板缓冲
glStencilMask(0xFF); // 设置模板缓冲为可写状态
normalShader.Use();
DrawTwoContainers();
```
然后模板缓冲更新为1了，绘制放大的箱子，关闭模板缓冲的写入：
```C++
glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
glStencilMask(0x00); // 禁止修改模板缓冲
glDisable(GL_DEPTH_TEST);
shaderSingleColor.Use();
DrawTwoScaledUpContainers();
```
shaderSingleColor为外框的颜色，定义如下：
```C++
void main()
{
    outColor = vec4(0.04, 0.28, 0.26, 1.0);
}
```
我们把模板方程设置为GL_NOTEQUAL，它保证我们只箱子上不等于1的部分，这样只绘制前面绘制的箱子外围的那部分。注意，我们也要关闭深度测试，这样放大的的箱子也就是边框才不会被地面覆盖。
总体绘制方法类似这样:
```C++
glEnable(GL_DEPTH_TEST);
glStencilOp(GL_KEEP, GL_KEEP, GL_REPLACE);  

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);

glStencilMask(0x00); // 绘制地板时确保关闭模板缓冲的写入
normalShader.Use();
DrawFloor()  

glStencilFunc(GL_ALWAYS, 1, 0xFF);
glStencilMask(0xFF);
DrawTwoContainers();

glStencilFunc(GL_NOTEQUAL, 1, 0xFF);
glStencilMask(0x00);
glDisable(GL_DEPTH_TEST);
shaderSingleColor.Use();
DrawTwoScaledUpContainers();
glStencilMask(0xFF);
glEnable(GL_DEPTH_TEST);
```
例子：
![](stencil_scene_outlined.png)

**PS：说实话我有点懵逼**

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/02%20Stencil%20testing/)**