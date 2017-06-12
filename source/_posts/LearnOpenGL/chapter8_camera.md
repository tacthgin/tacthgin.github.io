---
title: 摄像机
date: 2017-6-7 7:21:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 摄像机/观察空间(Camera/View Space)
![](camera_axes.png)
### 摄像机位置
世界坐标中的摄像机位置，正z轴的位置就是摄像机后退的位置
```C++
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
```

### 摄像机方向
摄像机指向的方向，**方向向量(Direction Vector)**是指向z的正方向
```C++
glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
```
**Think**
**我不太懂为什么是指的反方向，难道还是跟OpenGL矩阵相乘有关系**

### 右轴
**右向量(Right Vector)**，代表摄像机空间的x轴的正方向，定义一个**上向量(Up Vector)**与摄像机方向叉乘得到
```C++
glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f); 
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
```
**Think**
**叉乘也符合右手定律**

### 上轴
使用右向量(x轴)和方向向量(z轴)叉乘得到
```C++
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
```

## Look At
根据三个向量可以得到Look At矩阵，R是右向量，U是上向量，D是方向向量，P是摄像机位置向量
![](look_at_matrix.png)
**Think**
**我又懵逼了，这里位置向量是相反的，原文章说把世界平移到与我们自身移动的相反方向**

```C++
glm::mat4 view;
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), 
glm::vec3(0.0f, 0.0f, 0.0f), 
glm::vec3(0.0f, 1.0f, 0.0f));
```
<video id="video" src="rotate_camera.mp4" controls="" preload="none" width="480" height="320" />

## 自由移动
1.自定义相机的向量
```C++
glm::vec3 cameraPos   = glm::vec3(0.0f, 0.0f,  3.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp    = glm::vec3(0.0f, 1.0f,  0.0f);
```
2.得出LookAt函数
```C++
view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
```



























[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/09%20Camera/)**