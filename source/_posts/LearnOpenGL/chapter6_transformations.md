---
title: 变化
date: 2017-6-2 7:39:00
tags: 
    OpenGL
category: LearnOpenGL
---
这章都是线性代数的知识,本来不想搞的。
## 向量
1.向量是具有大小和方向的量
![](vectors.png)
表达式:
![](vector_expression.png)

2.向量取反
![](vector_negate.png)

3.向量加减
加法:
![](vector_plus.png)

示意图:
![](vectors_addition.png)

减法:
![](vector_minus.png)

示意图:
![](vectors_subtraction.png)

4.向量长度(使用勾股定理)
![](vector_length.png)

示意图:
![](vectors_triangle.png)

单位向量:
![](vector_unit.png)

4.向量相乘
点乘:
![](vector_dot_multipy.png)

叉乘:
![](vector_x_multipy.png)

示意图:
![](vectors_crossproduct.png)

## 矩阵
1.矩阵加减
标量加减:
![](matrix_add_scalar.png)
![](matrix_minus_scalar.png)

矩阵加减:
![](matrix_add_matrix.png)
![](matrix_minus_matrix.png)

2.矩阵乘法
标量乘法:
![](matrix_multipy_scalar.png)

矩阵乘法:
![](matrix_multiplication.png)

m×n矩阵乘以n×p矩阵,得出m×p矩阵
![](matrix_multipy_vector.png)

### 缩放矩阵
![](matrix_scale.png)

### 位移矩阵
![](matrix_pos.png)

**Important**
**齐次坐标(Homogeneous Coordinates)**
**向量的w分量也叫齐次坐标。想要从齐次向量得到3D向量，我们可以把x、y和z坐标分别除以w坐标。我们通常不会注意这个问题，因为w分量通常是1.0。使用齐次坐标有几点好处：它允许我们在3D向量上进行位移（如果没有w分量我们是不能位移向量的），而且下一章我们会用w值创建3D视觉效果。如果一个向量的齐次坐标是0，这个坐标就是方向向量(Direction Vector，因为w坐标是0，这个向量就不能位移（译注：这也就是我们说的不能位移一个方向）。**

### 旋转
1.3D旋转需要定义一个角和一个旋转轴(Rotation Axis)
![](vectors_angle.png)

2.x轴旋转矩阵
![](matrix_rotation_x.png)

3.y轴旋转矩阵
![](matrix_rotation_y.png)

4.z轴旋转矩阵
![](matrix_rotation_z.png)

5.绕x,y,z轴旋转会造成万向节死锁(Gimbal Lock),选择用任意轴旋转(Rx, Ry, Rz)
![](matrix_rotation_anything.png)

## GLM(OpenGL Mathematics)
1.GLM是一个只有头文件的库,从这个[网站](http://glm.g-truc.net/0.9.5/index.html)下载
2.把glm文件夹放在工程include目录下,添加头文件
```c++
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```

3.在顶点着色器添加变换矩阵
```c++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 color;
layout (location = 2) in vec2 texCoord;

out vec3 ourColor;
out vec2 TexCoord;

uniform mat4 transform;

void main()
{
	gl_Position = transform * vec4(position, 1.0);
	ourColor = color;
	TexCoord = vec2(texCoord.x, 1 - texCoord.y);
}
```

4.在画图中，使用sin变化scale大小，随时间来rotate
```c++
glm::mat4 trans;
GLfloat scale = abs(sinf((GLfloat)glfwGetTime()));
trans = glm::scale(trans, glm::vec3(scale, scale, 0.0f));
trans = glm::rotate(trans, (GLfloat)glfwGetTime() * 50.0f, glm::vec3(0.0f, 0.0f, 1.0f));

GLuint transformLoc = glGetUniformLocation(shader.getProgram(), "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```
![](transform.png)

[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/07%20Transformations/)**




