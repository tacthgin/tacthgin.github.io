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
1.把一个局部坐标转换到裁剪坐标
![](multipy_matrix.png)
**Think**
**之前一直不明白,为什么OpenGL的矩阵是右向左乘,然后查了资料才发现OpenGL顶点坐标是列向量**

## 进入3D
1.创建模型矩阵
```C++
glm::mat4 model;
model = glm::rotate(model, -55.0f, glm::vec3(1.0f, 0.0f, 0.0f));
```

2.创建观察矩阵
```C++
glm::mat4 view;
// 注意，我们将矩阵向我们要进行移动场景的反向移动。
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f)); 
```

3.创建投影矩阵
```C++
glm::mat4 projection;
projection = glm::perspective(45.0f, screenWidth / screenHeight, 0.1f, 100.0f);
```

4.修改顶点着色器
```C++
#version 330 core
layout (location = 0) in vec3 position;
...
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    // 注意从右向左读
    gl_Position = projection * view * model * vec4(position, 1.0f);
    ...
}
```

5.给矩阵赋值
```C++
GLuint modelLoc = glGetUniformLocation(shader.getProgram(), "model");
glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));
[...]
```
![](easy_rotate.png)

## 更多的3D
1.创建一个立方体
```C++
float vertices[] = {
	-0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
	0.5f, -0.5f, -0.5f,  1.0f, 0.0f,
	0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
	0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
	-0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
	-0.5f, -0.5f, -0.5f,  0.0f, 0.0f,

	-0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
	0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
	0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
	0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
	-0.5f,  0.5f,  0.5f,  0.0f, 1.0f,
	-0.5f, -0.5f,  0.5f,  0.0f, 0.0f,

	-0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
	-0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
	-0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
	-0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
	-0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
	-0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

	0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
	0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
	0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
	0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
	0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
	0.5f,  0.5f,  0.5f,  1.0f, 0.0f,

	-0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
	0.5f, -0.5f, -0.5f,  1.0f, 1.0f,
	0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
	0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
	-0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
	-0.5f, -0.5f, -0.5f,  0.0f, 1.0f,

	-0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
	0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
	0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
	0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
	-0.5f,  0.5f,  0.5f,  0.0f, 0.0f,
	-0.5f,  0.5f, -0.5f,  0.0f, 1.0f
};
[...]
glDrawArrays(GL_TRIANGLES, 0, 36);
```

2.随时间变化旋转
```C++
model = glm::rotate(model, (GLfloat)glfwGetTime() * 50.0f, glm::vec3(0.5f, 1.0f, 0.0f));
```
![](easy_rotate_with_time.png)

### Z缓冲
1.由于没有开启深度测试,立方体后面的绘制会重叠前面
2.z缓冲(也叫深度缓冲区(Depth Buffer))，保存深度信息
3.深度存储在每个片段中(z值),跟z缓冲进行对比，如果当前片段在其他片段之后，会被丢弃
4.开启深度测试
```C++
glEnable(GL_DEPTH_TEST);
```
5.渲染前清除深度缓冲区(否则前一个片段的深度信息仍然保存在缓冲区中)
```C++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```
![](easy_rotate_with_depth_test.png)

### 更多立方体
1.定义10个立方体
```C++
glm::vec3 cubePositions[] = {
  glm::vec3( 0.0f,  0.0f,  0.0f), 
  glm::vec3( 2.0f,  5.0f, -15.0f), 
  glm::vec3(-1.5f, -2.2f, -2.5f),  
  glm::vec3(-3.8f, -2.0f, -12.3f),  
  glm::vec3( 2.4f, -0.4f, -3.5f),  
  glm::vec3(-1.7f,  3.0f, -7.5f),  
  glm::vec3( 1.3f, -2.0f, -2.5f),  
  glm::vec3( 1.5f,  2.0f, -2.5f), 
  glm::vec3( 1.5f,  0.2f, -1.5f), 
  glm::vec3(-1.3f,  1.0f, -1.5f)  
};
```

2.改变不同角度,位置
```C++
for (GLuint i = 0; i < 10; i++)
{
	glm::mat4 model;
	model = glm::translate(model, cubePositions[i]);
	GLfloat angle = 20.0f * (i + 1);
	model = glm::rotate(model, angle * (float)glfwGetTime(), glm::vec3(1.0f, 0.3f, 0.5f));
	glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

	glDrawArrays(GL_TRIANGLES, 0, 36);
}
```

![](more_cube.png)


[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/08%20Coordinate%20Systems/)**