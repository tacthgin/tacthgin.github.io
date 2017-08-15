---
title: 材质
date: 2017-7-2 22:49:00
tags: 
    OpenGL
category: LearnOpenGL
---

在真实世界里，每个物体会对光产生不同的反应。钢看起来比陶瓷花瓶更闪闪发光，一个木头箱子不会像钢箱子一样对光产生很强的反射。每个物体对镜面高光也有不同的反应。有些物体不会散射(Scatter)很多光却会反射(Reflect)很多光，结果看起来就有一个较小的高光点(Highlight)，有些物体散射了很多，它们就会产生一个半径更大的高光。如果我们想要在OpenGL中模拟多种类型的物体，我们必须为每个物体分别定义材质(Material)属性。
## 定义材质
```c++
#version 330 core
struct Material
{
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
};
uniform Material material;
```
1.ambient材质向量定义了在环境光照下这个物体反射的是什么颜色
2.diffuse材质向量定义了在漫反射光照下物体的颜色
3.specular材质向量设置的是物体受到的镜面光照的影响的颜色(或者可能是反射一个物体特定的镜面高光颜色)
4.shininess影响镜面高光的散射/半径
下图展示了几种材质属性
![](materials_real_world.png)
## 设置材质
1.在着色器插入材质颜色
```c++
void main()
{
    // 环境光
    vec3 ambient = lightColor * material.ambient;

    // 漫反射光
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = lightColor * (diff * material.diffuse);

    // 镜面高光
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);  
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = lightColor * (spec * material.specular);  

    vec3 result = ambient + diffuse + specular;
    color = vec4(result, 1.0f);
}
```

2.传递数据给材质uniform
```c++
GLint matAmbientLoc = glGetUniformLocation(lightingShader.getProgram(), "material.ambient");
GLint matDiffuseLoc = glGetUniformLocation(lightingShader.getProgram(), "material.diffuse");
GLint matSpecularLoc = glGetUniformLocation(lightingShader.getProgram(), "material.specular");
GLint matShineLoc = glGetUniformLocation(lightingShader.getProgram(), "material.shininess");

glUniform3f(matAmbientLoc, 1.0f, 0.5f, 0.31f);
glUniform3f(matDiffuseLoc, 1.0f, 0.5f, 0.31f);
glUniform3f(matSpecularLoc, 0.5f, 0.5f, 0.5f);
glUniform1f(matShineLoc, 32.0f);
```
得到下图的效果
![](material_example.png)

## 光的属性
1.这个物体太亮了。物体过亮的原因是环境、漫反射和镜面三个颜色任何一个光源都会去全力反射。代码类似这样：
```c++
vec3 ambient = vec3(1.0f) * material.ambient;
vec3 diffuse = vec3(1.0f) * (diff * material.diffuse);
vec3 specular = vec3(1.0f) * (spec * material.specular);
```
2.光源应该对环境、漫反射和镜面元素同时具有不同的强度。使用独立的光属性影响每个光照元素，创建一个光源：
```c++
struct Light
{
    vec3 position;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
uniform Light light;
```
3.更新片段着色器，并且设置光的亮度
```c++
vec3 ambient = light.ambient * material.ambient;
vec3 diffuse = light.diffuse * (diff * material.diffuse);
vec3 specular = light.specular * (spec * material.specular);
```
```c++
GLint lightAmbientLoc = glGetUniformLocation(lightingShader.getProgram(), "light.ambient");
GLint lightDiffuseLoc = glGetUniformLocation(lightingShader.getProgram(), "light.diffuse");
GLint lightSpecularLoc = glGetUniformLocation(lightingShader.getProgram(), "light.specular");

glUniform3f(lightAmbientLoc, 0.2f, 0.2f, 0.2f);
glUniform3f(lightDiffuseLoc, 0.5f, 0.5f, 0.5f);// 让我们把这个光调暗一点，这样会看起来更自然
glUniform3f(lightSpecularLoc, 1.0f, 1.0f, 1.0f);
```
![](material_with_light.png)

## 不同的光源颜色
使用sin和glfwGetTime动态改变光的环境和漫反射颜色:
```c++
vec3 lightColor;
lightColor.x = sin(glfwGetTime() * 2.0f);
lightColor.y = sin(glfwGetTime() * 0.7f);
lightColor.z = sin(glfwGetTime() * 1.3f);

vec3 diffuseColor = lightColor * vec3(0.5f);
vec3 ambientColor = diffuseColor * vec3(0.2f);

glUniform3f(lightAmbientLoc, ambientColor.x, ambientColor.y, ambientColor.z);
glUniform3f(lightDiffuseLoc, diffuseColor.x, diffuseColor.y, diffuseColor.z);
```
<video id="video" src="dynamic_lighting.mp4" controls="" preload="none" width="480" height="320" />
**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/03%20Materials/)**