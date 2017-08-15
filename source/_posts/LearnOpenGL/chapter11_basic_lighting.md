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
```c++
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
```c++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 normal;
...
```
```c++
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid * )(3 * sizeof(float)));
glEnableVertexAttribArray(1);
```
**Important**
**发光物着色器顶点数据的不完全使用看起来有点低效，但是这些顶点数据已经从立方体对象载入到GPU的内存里了，所以GPU内存不是必须再储存新数据。相对于重新给发光物分配VBO，实际上却是更高效了。**

4.传递法线数据到片段着色器
```c++
out vec3 Normal;
void main()
{
    gl_Position = projection * view * model * vec4(position, 1.0f);
    Normal = normal;
}
```
在片段着色器定义输入值
```c++
in vec3 Normal;
```
### 计算漫反射光照
1.在片段着色器添加光源位置
```c++
uniform vec3 lightPos;
```
更新uniform
```c++
GLint lightPosLoc = glGetUniformLocation(lightingShader.getProgram(), "lightPos");
glUniform3f(lightPosLoc, lightPos.x, lightPos.y, lightPos.z);
```
2.计算片段的位置,把顶点乘以模型矩阵得到世界空间的坐标
```c++
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
```c++
in vec3 FragPos;
```
```c++
vec3 norm = normalize(Normal);
vec3 lightDir = normalize(lightPos - FragPos);
```
**Important**
**当计算光照时我们通常不关心一个向量的“量”或它的位置，我们只关心它们的方向。所有的计算都使用单位向量完成，因为这会简化了大多数计算(比如点乘)。**

计算当前片段的散射影响，如果向量角度大于90，结果变为负数颜色，使用max来避免这种情况
```c++
float diff = max(dot(norm, lightDir), 0.0);
vec3 diffuse = diff * lightColor;
```
对环境光和漫射光进行相加，得出最终的光照颜色，乘以物体颜色，得出片段颜色
```c++
vec3 result = (ambient + diffuse) * objectColor;
color = vec4(result, 1.0f);
```
![](diffuse.png)

### 法线向量注意事项
1.法线向量是方向向量，不能表达空间种的特定位置
2.法线向量没有齐次坐标(顶点位置的w分量),平移不应该影响到法向量
3.法向量只能应用缩放(Scale)和旋转(Rotation)变换
4.如果模型矩阵应用不等比缩放，法向量就不再垂直表面，光照就会被扭曲
![](basic_lighting_normal_transformation.png)
5.利用正规矩阵(Normal Matrix)来修复法线不垂直于表面的情况，正规矩阵被定义为"模型矩阵左上角的逆矩阵(Inverse Matrix)的转置矩阵(Transpose Matrix)"，然后强制转换为3×3矩阵，避免平移的影响
```c++
Normal = mat3(transpose(inverse(model))) * normal;
```
**Tip**
**正规矩阵计算可以看[这篇文章](http://www.lighthouse3d.com/tutorials/glsl-12-tutorial/the-normal-matrix/)**

**Attention**
**计算逆矩阵的开销比较大，应该在cpu中计算完毕，再通过uniform传送给着色器**

## 镜面光照
![](basic_lighting_specular_theory.png)
镜面光照依据光的方向向量、物体的法向量和玩家的观察方向得出，利用反射法向量周围光源的方向计算光源的反射向量，计算反射向量和观察向量的夹角，
角度越小，镜面光照的强度越大。当看到物体表面的时候，会出现高光效果

**Important**
**我们选择在世界空间(World Space)进行光照计算，但是大多数人趋向于在观察空间(View Space)进行光照计算。在观察空间计算的好处是，观察者的位置总是(0, 0, 0)，所以这样你直接就获得了观察者位置。可是，我发现出于学习的目的，在世界空间计算光照更符合直觉。如果你仍然希望在视野空间计算光照的话，那就使用观察矩阵应用到所有相关的需要变换的向量(不要忘记，也要改变正规矩阵)。**

1.传递观察坐标
```c++
uniform vec3 viewPos;

GLint viewPosLoc = glGetUniformLocation(lightingShader.getProgram(), "viewPos");
vec3 cameraPos = camera.getPos();
glUniform3f(viewPosLoc, cameraPos.x, cameraPos.y, cameraPos.z);
```
2.给镜面光照一个镜面强度，计算发射向量
```c++
float specularStrength = 0.5f;
vec3 viewDir = normalize(viewPos - FragPos);
vec3 reflectDir = reflect(-lightDir, norm);
```
3.计算镜面光分量，32次幂是高光的**发光值(Shininess)**。发光值越高，反射光能力越强，散射的越少，高光点越小
```c++
float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
vec3 specular = specularStrength * spec * lightColor;
```
![](basic_lighting_specular_shininess.png)
4.整合环境光和漫反射得出最终物体颜色
```c++
vec3 result = (ambient + diffuse + specular) * objectColor;
color = vec4(result, 1.0f);
```
![](specular.png)
**Important**
**早期的光照着色器，开发者在顶点着色器中实现冯氏光照。在顶点着色器中做这件事的优势是，相比片段来说，顶点要少得多，因此会更高效，所以(开销大的)光照计算频率会更低。然而，顶点着色器中的颜色值是只是顶点的颜色值，片段的颜色值是它与周围的颜色值的插值。结果就是这种光照看起来不会非常真实，除非使用了大量顶点。**
![](basic_lighting_gouruad.png)
**在顶点着色器中实现的冯氏光照模型叫做Gouraud着色，而不是冯氏着色。记住由于插值，这种光照连起来有点逊色。冯氏着色能产生更平滑的光照效果。**

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/02%20Basic%20Lighting/)**