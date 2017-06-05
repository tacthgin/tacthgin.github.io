---
title: 坐标系统
date: 2017-6-4 11:10:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 概述
![](coordinate_systems.png)
1.局部空间是对象所在的坐标空间,比如模型的各个顶点坐标
2.世界空间是对象在游戏世界的坐标(通过**模型矩阵(Model Matrix)**转换),利用世界坐标平移
3.观察空间是摄像机观察到的坐标(通过**观察矩阵(View Matrix)**转换),利用摄像机的位置、观察方向来做矩阵平移和旋转
4.如果对象在观察空间之外,那么对象将被剔除或者裁剪,这个空间叫做裁剪空间(通过投影矩阵转换)
投影矩阵创建的**观察区域(Viewing Box)**叫做**平截头体(Frustum)**
### 正射投影(Orthographic Projection)
1.正射投影矩阵定义了一个立方体的平截头体
![](orthographic_frustum.png)
2.使用glm创建一个正射投影矩阵
```C++
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
```
前两个参数指定了平截头体的左右坐标，第三和第四参数指定了平截头体的底部和上部，最后2个参数指定近平面和远平面

### 透视投影(Perspective Projection)
1.透视投影会实现一个透视的效果
![](perspective_frustum.png)
2.投影矩阵会修改每个顶点的W分量，使得离观察者越远的坐标w分量越大,具体投影矩阵计算在这篇[文章](http://www.songho.ca/opengl/gl_projectionmatrix.html)
![](projection_w.png)
3.使用glm创建一个透视矩阵
```C++
glm::mat4 proj = glm::perspective(45.0f, (float)width/(float)height, 0.1f, 100.0f);
```
第一个参数指定**视野(Field of View)**
第二个参数指定宽高比
第三个、第四个指定近平面和远平面

4.正射投影和透视投影对比
![](perspective_orthographic.png)
使用透视投影的话，远处的顶点看起来比较小，而在正射投影中每个顶点距离观察者的距离都是一样的。

### 组合



[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/08%20Coordinate%20Systems/)**




