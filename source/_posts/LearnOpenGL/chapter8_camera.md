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
```c++
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
```

### 摄像机方向
摄像机指向的方向，**方向向量(Direction Vector)**是指向z的正方向，与摄像机面朝的方向相反
```c++
glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
```
**Think**
**我不太懂为什么是指的反方向，可能跟坐标系有关**

### 右轴
**右向量(Right Vector)**，代表摄像机空间的x轴的正方向，定义一个**上向量(Up Vector)**与摄像机方向叉乘得到
```c++
glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f); 
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
```
**Think**
**叉乘也符合右手定律，因为叉乘只需要一个平面就能得到右向量，所以随意指定一个上向量就好**

### 上轴
使用右向量(x轴)和方向向量(z轴)叉乘得到
```c++
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
```

**Think**
**右向量加上方向向量得到垂直摄像机的上向量**

## Look At
根据三个向量可以得到Look At矩阵，R是右向量，U是上向量，D是方向向量，P是摄像机位置向量
![](look_at_matrix.png)
**Think**
**我又懵逼了，这里位置向量是相反的，原文章说把世界平移到与我们自身移动的相反方向**

```c++
glm::mat4 view;
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), 
glm::vec3(0.0f, 0.0f, 0.0f), 
glm::vec3(0.0f, 1.0f, 0.0f));
```
<video id="video" src="rotate_camera.mp4" controls="" preload="none" width="480" height="320">
</video>

## 自由移动
1.自定义相机的向量
```c++
glm::vec3 cameraPos   = glm::vec3(0.0f, 0.0f,  3.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp    = glm::vec3(0.0f, 1.0f,  0.0f);
```
2.得出LookAt函数
```c++
view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
```
**Think**
**cameraPos + cameraFront是摄像机的方向向量**

3.添加摄像机移动按键事件
```c++
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
```c++
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
```c++
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

<video id="video" src="move_camera.mp4" controls="" preload="none" width="480" height="320">
</video>

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
```c++
direction.y = sin(glm::radians(pitch)); // 注意我们先把角度转为弧度
direction.x = cos(glm::radians(pitch));
direction.z = cos(glm::radians(pitch));
```
5.俯仰角例子
![](camera_yaw.png)
**译注**
**这里的球坐标与笛卡尔坐标的转换把x和z弄反了，如果你去看最后的源码，会发现作者在摄像机源码那里写了yaw = yaw – 90，实际上在这里x就应该是sin(glm::radians(yaw))，z也是同样处理，当然也可以认为是这个诡异的坐标系，但是在这里使用球坐标转笛卡尔坐标有个大问题，就是在初始渲染时，无法指定摄像机的初始朝向，还要花一些功夫自己实现这个；此外这只能实现像第一人称游戏一样的简易摄像机，类似Maya、Unity3D编辑器窗口的那种摄像机还是最好自己设置摄像机的位置、上、右、前轴，在旋转时用四元数对这四个变量进行调整，才能获得更好的效果，而不是仅仅调整摄像机前轴。**

6.俯仰角代码实现
```c++
direction.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw));//译注：direction代表摄像机的“前”轴，但此前轴是和本文第一幅图片的第二个摄像机的direction是相反的
direction.y = sin(glm::radians(pitch));
direction.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));
```

### 鼠标输入
1.鼠标水平移动影响偏航角，垂直移动影响俯仰角。
2.使用GLFW捕捉鼠标
```c++
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
```
3.设置鼠标回调事件
```c++
void mouse_callback(GLFWwindow* window, double xpos, double ypos);
[...]
glfwSetCursorPosCallback(window, mouse_callback);
```
4.设置鼠标偏移量，初始值设为屏幕中心(800 * 600)
```c++
GLfloat lastX = 400, lastY = 300;
...
GLfloat xoffset = xpos - lastX;
GLfloat yoffset = lastY - ypos; // 注意这里是相反的，因为y坐标的范围是从下往上的
lastX = xpos;
lastY = ypos;

GLfloat sensitivity = 0.05f;
xoffset *= sensitivity;
yoffset *= sensitivity;
```
5.加到偏航角与俯仰角
```c++
yawAngle += xoffset;
pitchAngle += yoffset;
```
6.设置角度限制，不能高于89度(到90度视角逆转)
```c++
if(pitchAngle > 89.0f)
  pitchAngle =  89.0f;
if(pitchAngle < -89.0f)
  pitchAngle = -89.0f;
```
7.得到方向向量
```c++
glm::vec3 front;
front.x = cos(glm::radians(pitchAngle)) * cos(glm::radians(yawAngle));
front.y = sin(glm::radians(pitchAngle));
front.z = cos(glm::radians(pitchAngle)) * sin(glm::radians(yawAngle));
cameraFront = glm::normalize(front);
```

8.鼠标右键控制摄像机旋转
```c++
void mouse_button_callback(GLFWwindow* window, int button, int action, int mods)
{
  if (button == GLFW_MOUSE_BUTTON_RIGHT)
  {
    if (action == GLFW_PRESS)
    {
      firstMouse = true;
      mouseDown = true;
    }
    else if (action == GLFW_RELEASE)
      mouseDown = false;
  }
}
[...]
glfwSetMouseButtonCallback(window, mouse_button_callback);
```

9.处理第一次鼠标进入窗口事件
```c++
if(!mouseDown)return;
if(firstMouse) // 这个bool变量一开始是设定为true的
{
  lastX = xpos;
  lastY = ypos;
  firstMouse = false;
}
```

### 缩放
1.注册鼠标滚轮事件
```c++
glfwSetScrollCallback(window, scroll_callback);
```
2.使用鼠标滚轮事件来缩放摄像机视野
```c++
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
{
  if(aspect >= 1.0f && aspect <= 45.0f)
    aspect -= yoffset;
  if(aspect <= 1.0f)
    aspect = 1.0f;
  if(aspect >= 45.0f)
    aspect = 45.0f;
}
```
3.添加到观察矩阵
```c++
projection = glm::perspective(aspect, (GLfloat)WIDTH/(GLfloat)HEIGHT, 0.1f, 100.0f);
```

### 完整初始化代码
```c++
int screenWidth = 800;
int screenHeight = 600;

bool keys[1024];
bool mouseDown = false;

GLfloat aspect = 45.0f;
GLboolean firstMouse = true;
GLfloat lastX = screenWidth / 2;
GLfloat lastY = screenHeight / 2;
GLfloat pitchAngle = 0.0f;
GLfloat yawAngle = -90.0f;
vec3 cameraPos = vec3(0.0f, 0.0f, 3.0f);
vec3 cameraFront = vec3(0.0f, 0.0f, -1.0f);
vec3 cameraUp = vec3(0.0f, 1.0f, 0.0f);
```

**Think**
**欧拉角实现的摄像机会遇到万向节死锁问题，使用四元数比较好**

<video id="video" src="move_rotate_camera.mp4" controls="" preload="none" width="480" height="320">
</video>

### 摄像机类
使用一个类来封装摄像机
[源代码](https://github.com/tacthgin/toy/blob/master/OpenGL/src/Camera.h)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/09%20Camera/)**