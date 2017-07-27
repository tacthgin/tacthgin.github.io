---
title: 深度测试
date: 2017-7-26 10:17:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 概要
**深度缓冲(Depth Buffer)**像**颜色缓冲(Color Buffer)**存储每个片段的信息(深度值)。深度值一般为16、24或32位浮点数，大部分深度缓冲区为24位。

开启深度测试后，如果测试通过，深度缓冲的值会被设置成新的深度值，测试失败，就丢弃该片段。

1.使用GL_DEPTH_TEST打开深度测试
```C++
glEnable(GL_DEPTH_TEST);
```
2.清除深度缓冲区，不然将保留上一次深度测试的深度值
```C++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```
3.禁用深度缓冲写入，只在深度测试被开启有效
```C++
glDepthMask(GL_FLASE)
```

## 深度测试函数
1.设置glDepthFunc设置比较运算符
```C++
glDepthFunc(GL_LESS);
```
![](comparison_operator.png)
2.使用GL_AWAYS和GL_LESS的比较
GL_AWAYS:
![](depth_testing_func_always.png)
GL_LESS:
![](depth_testing_func_less.png)

## 深度值精度
1.深度缓冲区的深度值在0.0 - 1.0之间，线性深度方程:
![](linear_equation.png)
**Important**
**近平面深度值接近0.0，远平面深度值接近1.0**
![](depth_linear_graph.png)
2.实践中从来不用线性的缓冲区，正确的投影特性的非线性深度方程是和1/z成正比的，这样z很近的时候是高精度，z很远的时候是低精度,非线性方程:
![](no_linear_equation.png)
趋向于远方的时候z值逐渐平稳
![](depth_non_linear_graph.png)
**Tip**
**这个[文章](http://www.songho.ca/opengl/gl_projectionmatrix.html)讲了摄像机的投影矩阵**

## 深度缓冲区的可视化
1.对片段着色器输出片段z值
```C++
void main()
{
    color = vec4(vec3(gl_FragCoord.z), 1.0f);
}  
```
2.较大的z值有较低的精度。该片段的深度值会迅速增加，所以几乎所有顶点的深度值接近1.0。如果我们小心的靠近物体，你最终可能会看到的色彩越来越暗，意味着它们的z值越来越小:
![](depth_testing_visible_depth.png)
3.如果让深度值变回线性，需要让点应用投影变换逆的逆变换(很拗口，我不知道他在讲什么)，成为单独的深度值
进行NDC深度值转换：
```C++
float z = depth * 2.0 - 1.0;
```
z值应用逆转换来检索的线性深度值:
```C++
float linearDepth = (2.0 * near) / (far + near - z * (far - near))
```
这篇[文章](http://www.songho.ca/opengl/gl_projectionmatrix.html)阐述了大量详细的投影矩阵的知识;它还表明了方程是从哪里来的。

能够将屏幕空间的非线性深度值转变为线性深度值的完整的片段着色器如下所示：
```C++
#version 330 core

out vec4 color;

float LinearizeDepth(float depth)
{
    float near = 0.1;
    float far = 100.0;
    float z = depth * 2.0 - 1.0; // Back to NDC
    return (2.0 * near) / (far + near - z * (far - near));
}

void main()
{
    float depth = LinearizeDepth(gl_FragCoord.z);
    color = vec4(vec3(depth), 1.0f);
}
```
图示结果：
![](depth_testing_visible_linear.png)
**PS:这节的计算真是看的一脸懵逼**

## 深度冲突
两个平面或三角形如此紧密相互平行深度缓冲区不具有足够的精度以至于无法得到哪一个靠前。结果是，这两个形状不断似乎切换顺序导致怪异出问题。这被称为**深度冲突(Z-fighting)**。

如果您移动摄像机到容器的里面，那么这个影响清晰可，容器的底部不断切换容器的平面和地板的平面:
![](depth_testing_z_fighting.png)
### 防止深度冲突
1.让物体之间不要离得太近，以至于他们的三角形重叠。通过在物体之间制造一点用户无法察觉到的偏移，可以完全解决深度冲突。
2.尽可能把近平面设置得远一些。前面我们讨论过越靠近近平面的位置精度越高。
3.放弃一些性能来得到更高的深度值的精度。大多数的深度缓冲区都是24位。但现在显卡支持32位深度值，这让深度缓冲区的精度提高了一大节。

**PS:感冒了，拖这么久更新**

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/01%20Depth%20testing/)**