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
<video id="video" src="rotate_camera.mp4" controls="" preload="none" width="480" height="320">
</video>

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

3.添加摄像机移动按键事件
```C++
void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
{
    ...
    GLfloat cameraSpeed = 0.05f;
    if(key == GLFW_KEY_W)
        cameraPos += cameraSpeed * cameraFront;
    if(key == GLFW_KEY_S)
        cameraPos -= cameraSpeed * cameraFront;
    if(key == GLFW_KEY_A)
        cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
    if(key == GLFW_KEY_D)
        cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;  
}
```
**Important**
**对右向量进行标准化，获得匀速效果**

4.处理按键按下/释放
```C++
bool keys[1024];

[...]

void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
{
  if(action == GLFW_PRESS)
    keys[key] = true;
  else if(action == GLFW_RELEASE)
    keys[key] = false;
  ...
}

void do_movement()
{
  // 摄像机控制
  GLfloat cameraSpeed = 0.01f;
  if(keys[GLFW_KEY_W])
    cameraPos += cameraSpeed * cameraFront;
  if(keys[GLFW_KEY_S])
    cameraPos -= cameraSpeed * cameraFront;
  if(keys[GLFW_KEY_A])
    cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
  if(keys[GLFW_KEY_D])
    cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
}

[...]

while(!glfwWindowShouldClose(window))
{
  // 检测并调用事件
  glfwPollEvents();
  do_movement();  

  // 渲染
  ...
}
```

## 移动速度
利用循环间得帧率差，来计算速度，获取移动流畅效果
```C++
GLfloat deltaTime = 0.0f;   // 当前帧遇上一帧的时间差
GLfloat lastFrame = 0.0f;   // 上一帧的时间

[...]

GLfloat currentFrame = glfwGetTime();
deltaTime = currentFrame - lastFrame;
lastFrame = currentFrame;  

[...]

void do_movement()
{
  GLfloat cameraSpeed = 5.0f * deltaTime;
  ...
}
```

## 视角移动
使用鼠标来旋转视角
### 欧拉角(Euler Angle)
![](camera_pitch_yaw_roll.png)
1.**俯仰角**是往上和往下看得角(图1)，**偏航角**是往左和往右看的角(图2)，**滚转角**代表我们如何翻转摄像机
2.对摄像机系统不关心滚转角，基本得旋转三角公式图如下
![](camera_triangle.png)
3.偏航角例子
![](camera_pitch.png)
4.偏航角代码实现
```C++
direction.y = sin(glm::radians(pitch)); // 注意我们先把角度转为弧度
direction.x = cos(glm::radians(pitch));
direction.z = cos(glm::radians(pitch));
```
5.俯仰角例子
![](camera_yaw.png)
6.俯仰角代码实现


























[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/09%20Camera/)**