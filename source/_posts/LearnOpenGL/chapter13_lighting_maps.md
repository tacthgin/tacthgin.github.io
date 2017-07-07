---
title: 光照贴图
date: 2017-7-3 11:21:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 漫反射贴图
在光照场景中，通过纹理来呈现一个物体的diffuse颜色，这个做法被称做**漫反射贴图(Diffuse texture)**
使用下面的图片来演示漫反射贴图：
![](container2.png)
**Attention**
**要记住的是sampler2D也叫做模糊类型，这意味着我们不能以某种类型对它实例化，只能用uniform定义它们。如果我们用结构体而不是uniform实例化（就像函数的参数那样），GLSL会抛出奇怪的错误；这同样也适用于其他模糊类型。**

1.移除材质中的amibient颜色分量，因为ambient绝大多数情况等于diffuse颜色
```C++
struct Material
{
    sampler2D diffuse;
    vec3 specular;
    float shininess;
};
...
in vec2 TexCoords;
```
**Important**
**如果你非把ambient颜色设置为不同的值不可（不同于diffuse值），你可以继续保留ambient的vec3，但是整个物体的ambient颜色会继续保持不变。为了使每个原始像素得到不同ambient值，你需要对ambient值单独使用另一个纹理。**

2.替换原来的diffuse和amibient颜色值
```C++
vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
```

3.在顶点着色器中添加纹理坐标
```C++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 normal;
layout (location = 2) in vec2 texCoords;
...
out vec2 TexCoords;

void main()
{
    ...
    TexCoords = texCoords;
}
```

4.更新材质中的diffuse纹理
```C++
glUniform1i(glGetUniformLocation(lightingShader.getProgram(), "material.diffuse"), 0);
...
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, diffuseMap);
```

得到下列的箱子：
![](diffuse_map_example.png)

## 镜面贴图
由于木头的材质不应该有镜面高光，铁架子部分是会显示镜面高光的，所以用一个中间黑色的纹理来处理镜面高光：
![](container2_specular.png)
一个specular高光的亮度可以通过图片中每个纹理的亮度来获得。specular贴图的每个像素可以显示为一个颜色向量，比如：在那里黑色代表颜色向量vec3(0.0f)，灰色是vec3(0.5f)。在片段着色器中，我们采样相应的颜色值，把它乘以光的specular亮度。像素越“白”，乘积的结果越大，物体的specualr部分越亮。

## 镜面贴图采样
1.更新材质中的镜面属性，并绑定纹理到镜面中
```C++
struct Material
{
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
};
```
```C++
glUniform1i(glGetUniformLocation(lightingShader.getProgram(), "material.specular"), 1);
...
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, specularMap);
```
2.更新片段着色器的镜面光照
```C++
vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
color = vec4(ambient + diffuse + specular, 1.0f);
```
得到具有高光的铁边箱子：
![](specular_map_example.png)
**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/04%20Lighting%20maps/)**