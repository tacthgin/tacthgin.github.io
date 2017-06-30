---
title: 颜色
date: 2017-6-28 11:49:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 颜色
1.使用向量定义一个颜色(RGB)
```C++
glm::vec3 coral(1.0f, 0.5f, 0.31f);
```
2.光源和物体颜色的反射运算

![](light_reflection.png)

```C++
glm::vec3 lightColor(1.0f, 1.0f, 1.0f);
glm::vec3 toyColor(1.0f, 0.5f, 0.31f);
glm::vec3 result = lightColor * toyColor; // = (1.0f, 0.5f, 0.31f);
```
## 光照场景
1.创建一个光照VAO
```C++
GLuint lightVAO;
glGenVertexArrays(1, &lightVAO);
glBindVertexArray(lightVAO);
// 只需要绑定VBO不用再次设置VBO的数据，因为容器(物体)的VBO数据中已经包含了正确的立方体顶点数据
glBindBuffer(GL_ARRAY_BUFFER, VBO);
// 设置灯立方体的顶点属性指针(仅设置灯的顶点数据)
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
glBindVertexArray(0);
```
2.修改片段着色器，受光照影响
```C++
#version 330 core
out vec4 color;

uniform vec3 objectColor;
uniform vec3 lightColor;

void main()
{
    color = vec4(lightColor * objectColor, 1.0f);
}
```
3.赋值物体颜色和光照颜色
```C++
GLint objectColorLoc = glGetUniformLocation(lightingShader.Program, "objectColor");
GLint lightColorLoc  = glGetUniformLocation(lightingShader.Program, "lightColor");
glUniform3f(objectColorLoc, 1.0f, 0.5f, 0.31f);// 我们所熟悉的珊瑚红
glUniform3f(lightColorLoc,  1.0f, 1.0f, 1.0f); // 依旧把光源设置为白色
```
4.创建光源shader，纯白色的光
```C++
#version 330 core
out vec4 color;

void main()
{
    color = vec4(1.0f); //设置四维向量的所有元素为 1.0f
}
```
5.设置光源位置,大小
```C++
glm::vec3 lightPos(1.2f, 1.0f, 2.0f);
[...]
model = glm::mat4();
model = glm::translate(model, lightPos);
model = glm::scale(model, glm::vec3(0.2f));
```
6.绘制立方体
```C++
lampShader.Use();
// 设置模型、视图和投影矩阵uniform
...
// 绘制灯立方体对象
glBindVertexArray(lightVAO);
glDrawArrays(GL_TRIANGLES, 0, 36);
glBindVertexArray(0);
```
![](light_scene.png)
**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/01%20Colors/)**