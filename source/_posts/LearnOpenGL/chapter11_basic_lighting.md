---
title: 光照基础
date: 2017-6-30 11:46:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 冯氏光照模型
![](basic_lighting_phong.png)
* 环境光照(Ambient Lighting):即使在黑暗的情况下，世界上也仍然有一些光亮(月亮、一个来自远处的光)，所以物体永远不会是完全黑暗的。我们使用环境光照来模拟这种情况，也就是无论如何永远都给物体一些颜色。
* 漫反射光照(Diffuse Lighting):模拟一个发光物对物体的方向性影响(Directional Impact)。
* 镜面光照(Specular Lighting):模拟有光泽物体上面出现的亮点。镜面光照的颜色，相比于物体的颜色更倾向于光的颜色。

光通常都不是来自于同一光源，而是来自散落于我们周围的很多光源，即使它们可能并不是那么显而易见。光的一个属性是，它可以向很多方向发散和反弹，所以光最后到达的地点可能并不是它所临近的直射方向；光能够像这样**反射(Reflect)**到其他表面，一个物体的光照可能受到来自一个非直射的光源影响。考虑到这种情况的算法叫做**全局照明(Global Illumination)**算法，但是这种算法既开销高昂又极其复杂。
## 环境光
使用一个很小的常量颜色添加物体片段的最终颜色中，即使没有直射光源也存在发散的光
```C++
void main()
{
    float ambientStrength = 0.1f;
    vec3 ambient = ambientStrength * lightColor;
    vec3 result = ambient * objectColor;
    color = vec4(result, 1.0f);
}
```
![](ambient.png)
## 漫反射光照
![](diffuse_light.png)
图左上方有一个光源，它所发出的光线落在物体的一个片段上。我们需要测量这个光线与它所接触片段之间的角度。如果光线垂直于物体表面，这束光对物体的影响会最大化(译注：更亮)。为了测量光线和片段的角度，我们使用一个叫做法向量(Normal Vector)的东西，它是垂直于片段表面的一种向量(这里以黄色箭头表示)。两个向量之间的角度就能够根据点乘计算出来。

### 法向量
1.法向量(Normal Vector)是垂直于顶点表面的(单位)向量
2.利用顶点周围的顶点计算出这个顶点的表面
3.由于使用立方体，使用简单的法线数据添加到顶点数据
```C++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 normal;
...
```
```C++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid * )0);
glEnableVertexAttribArray(0);
```
**Important**
**发光物着色器顶点数据的不完全使用看起来有点低效，但是这些顶点数据已经从立方体对象载入到GPU的内存里了，所以GPU内存不是必须再储存新数据。相对于重新给发光物分配VBO，实际上却是更高效了。**

4.传递法线数据到片段着色器
```C++
out vec3 Normal;
void main()
{
    gl_Position = projection * view * model * vec4(position, 1.0f);
    Normal = normal;
}
```
在片段着色器定义输入值
```C++
in vec3 Normal;
```
### 计算漫反射光照
1.在片段着色器添加光源位置
```C++
uniform vec3 lightPos;
```
更新uniform
```C++
GLint lightPosLoc = glGetUniformLocation(lightingShader.Program, "lightPos");
glUniform3f(lightPosLoc, lightPos.x, lightPos.y, lightPos.z);
```
2.计算片段的位置,把顶点乘以模型矩阵得到世界空间的坐标
```C++
out vec3 FragPos;
out vec3 Normal;

void main()
{
    gl_Position = projection * view * model * vec4(position, 1.0f);
    FragPos = vec3(model * vec4(position, 1.0f));
    Normal = normal;
}
```
3.在片段着色器输入片段位置，计算光照
```C++
in vec3 FragPos;
```
```C++
vec3 norm = normalize(Normal);
vec3 lightDir = normalize(lightPos - FragPos);
```
**Important**
**当计算光照时我们通常不关心一个向量的“量”或它的位置，我们只关心它们的方向。所有的计算都使用单位向量完成，因为这会简化了大多数计算(比如点乘)。**

计算当前片段的散射影响，如果向量角度大于90，结果变为负数颜色，使用max来避免这种情况
```C++
float diff = max(dot(norm, lightDir), 0.0);
vec3 diffuse = diff * lightColor;
```
对环境光和漫射光进行相加，得出最终的光照颜色，乘以物体颜色，得出片段颜色
```C++
vec3 result = (ambient + diffuse) * objectColor;
color = vec4(result, 1.0f);
```
![](diffuse.png)

### 法线向量注意事项

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/02%20Basic%20Lighting/)**