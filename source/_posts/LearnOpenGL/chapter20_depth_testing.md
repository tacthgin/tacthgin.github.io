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
2.较大的 z 值有较低的精度。该片段的深度值会迅速增加，所以几乎所有顶点的深度值接近 1.0。如果我们小心的靠近物体，你最终可能会看到的色彩越来越暗，意味着它们的 z 值越来越小:
![](depth_testing_visible_depth.png)


**PS:感冒了，拖这么久更新**

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/01%20Depth%20testing/)**