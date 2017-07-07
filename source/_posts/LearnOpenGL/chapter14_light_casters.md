---
title: 投光物
date: 2017-7-4 11:16:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 定向光
当一个光源很远的时候，来自光源的每条光线接近于平行。这看起来就像所有的光线来自于同一个方向，无论物体和观察者在哪儿。当一个光源被设置为无限远时，它被称为定向光(Directional Light)，因为所有的光线都有着同一个方向；它会独立于光源的位置。太阳就是一个方向光：
![](light_casters_directional.png)
1.所有的光都是平行的，所以只需要光的方向向量
```C++
struct DirectionLight
{
	vec3 direction;
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
};
...
void main()
{
    vec3 lightDir = normalize(-directionLight.direction);
    ...
}
```
**Tip**
**现在使用的光照计算需要光的方向作为一个来自片段朝向的光源的方向，所以对光的方向取反**

2.创建10个箱子的物体，并给方向光一个方向
```C++
for(GLuint i = 0; i < 10; i++)
{
    model = mat4();
    model = translate(model, cubePositions[i]);
    GLfloat angle = 20.0f * i;
    model = rotate(model, angle, vec3(1.0f, 0.3f, 0.5f));
    glUniformMatrix4fv(modelLoc, 1, GL_FALSE, value_ptr(model));
    glDrawArrays(GL_TRIANGLES, 0, 36);
}
```
```C++
GLint lightDirPos = glGetUniformLocation(lightingShader.getProgram(), "directionLight.direction");
glUniform3f(lightDirPos, -0.2f, -1.0f, -0.3f);
```
得到定向光的例子：
![](direction_example.png)
**Tip**
**这里要对法向量进行共轭转置，因为箱子进行旋转了。**

## 点光源
点光是一个在时间里有位置的光源，它向所有方向发光，光线随距离增加逐渐变暗。想象灯泡和火炬作为投光物，它们可以扮演点光的角色。
![](light_casters_point.png)
### 衰减
随着光线穿越距离的变远使得亮度也相应地减少的现象，通常称之为**衰减(Attenuation)。**
衰减公式：
![](attenuation_formula.png)
* d是光源和片段的距离
* 常数项Kc通常是1.0
* 一次项Kl与距离值相乘，线性的方式减少亮度
* 二次项Kq与距离的平方相乘，为光源设置一个亮度的二次递减。二次项在距离比较近的时候相比一次项会更小，但是当距离更远的时候比一次项更大

由于二次项的光会以线性方式减少，指导距离足够大的时候，就会超过一次项，之后，光的亮度会减少的更快。最后的效果就是光在近距离时，非常量，但是距离变远亮度迅速降低，最后亮度降低速度再次变慢。下面的图展示了在100以内的范围，这样的衰减效果。
![](attenuation.png)
### 选择系数值
Ogre3D维基所配的系数：
![](attenuation_table.png)
* 常数项Kc一直是1.0
* 选择32到100的距离对大多数光通常就足够了

### 实现衰减
1.定义点光源,赋值衰减系数
```C++
struct PointLight
{
	vec3 position;
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
	float constant;
	float linear;
	float quaderatic;
};
```
```C++
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "pointLight.constant"), 1.0f);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "pointLight.linear"), 0.09);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "pointLight.quadratic"), 0.032);
```
2.计算衰减值
```C++
float distance = length(light.position - FragPos);
float attenuation = 1.0f / (light.constant + light.linear*distance +light.quadratic*(distance*distance));
```
**Important**
**我们可以可以把ambient元素留着不变，这样amient光照就不会随着距离减少，但是如果我们使用多余1个的光源，所有的ambient元素会开始叠加，因此这种情况，我们希望ambient光照也衰减。简单的调试出对于你的环境来说最好的效果。**
```C++
ambient *= attenuation;
diffuse *= attenuation;
specular *= attenuation;
```
得到点光源的例子：
![](pointlight_example.png)

## 聚光
聚光是一种位于环境中某处的光源，它不是向所有方向照射，而是只朝某个方向照射。结果是只有一个聚光照射方向的确定半径内的物体才会被照亮，其他的都保持黑暗。聚光的好例子是路灯或手电筒。
OpenGL中的聚光用世界空间位置，一个方向和一个指定了聚光半径的切光角来表示。我们计算的每个片段，如果片段在聚光的切光方向之间（就是在圆锥体内），我们就会把片段照亮。下面的图可以让你明白聚光是如何工作的：
![](light_casters_spotlight_angles.png)
* LightDir:从片段指向光源的向量
* SpotDir:聚光所指向的方向
* Φ:定义聚光半径的切光角。每个落在这个角度之外的，聚光都不会照亮
* θ:LightDir向量和SpotDir向量之间的角度。θ值应该比Φ值小，这样才会在聚光内

### 手电筒
一个手电筒是一个普通的聚光，但是根据玩家的位置和方向持续的更新它的位置和方向。
1.定义聚光
```C++
struct SpotLight
{
	vec3 position;
	vec3 direction;
	float cutOff;
	float outerCutOff;
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
	float constant;
	float linear;
	float quaderatic;
};
```
2.赋值位置、方向和切光角
```C++
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.position"), cameraPos.x, cameraPos.y, cameraPos.z);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.direction"), cameraFront.x, cameraFront.y, cameraFront.z);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.cutOff"), cos(radians(12.5f)));
```
3.计算θ值
```C++
float theta = dot(lightDir, normalize(-light.direction));
if(theta > light.cutOff)
{
    // 执行光照计算
}
else // 否则使用环境光，使得场景不至于完全黑暗
    color = vec4(light.ambient*vec3(texture(material.diffuse,TexCoords)), 1.0f);
```
**Important**
**你可能奇怪为什么if条件中使用>符号而不是<符号。为了在聚光以内，theta不是应该比光的切光值更小吗？这没错，但是不要忘了，角度值是以余弦值来表示的，一个0度的角表示为1.0的余弦值，当一个角是90度的时候被表示为0.0的余弦值，你可以在这里看到：**
![](light_casters_cos.png)
**现在你可以看到，余弦越是接近1.0，角度就越小。这就解释了为什么θ需要比切光值更大了。切光值当前被设置为12.5的余弦，它等于0.9978，所以θ的余弦值在0.9979和1.0之间，片段会在聚光内，被照亮。**
得到手电筒的例子：
![](spotlight_example.png)
### 平滑/软化边缘
为创建聚光的平滑边，我们希望去模拟的聚光有一个内圆锥和外圆锥。我们可以把内圆锥设置为前面部分定义的圆锥，我们希望外圆锥从内边到外边逐步的变暗。
为创建外圆锥，我们简单定义另一个余弦值，它代表聚光的方向向量和外圆锥的向量（等于它的半径）的角度。然后，如果片段在内圆锥和外圆锥之间，就会给它计算出一个0.0到1.0之间的亮度。如果片段在内圆锥以内这个亮度就等于1.0，如果在外面就是0.0。
我们可以使用下面的公式计算这样的值：
![](inter_outer_light_strength.png)
这里ϵ是内部（Φ）和外部圆锥（γ）（\epsilon = \phi - \gamma）的差。结果I的值是聚光在当前片段的亮度。
很难用图画描述出这个公式是怎样工作的，所以我们尝试使用一个例子：
![](spotlight_strength.png)
1.添加外切光角outerCutOff
```C++
struct SpotLight
{
	vec3 position;
	vec3 direction;
	float cutOff;
	float outerCutOff;
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
	float constant;
	float linear;
	float quaderatic;
};
```
2.计算漫射光和镜面光的外切光强度
```C++
float theta = dot(lightDir, normalize(-light.direction));
float epsilon = light.cutOff - light.outerCutOff;
float intensity = clamp((theta - light.outerCutOff) / epsilon,0.0, 1.0);
...
// We’ll leave ambient unaffected so we always have a little
light.diffuse* = intensity;
specular* = intensity;
...
```
3.内切光角12.5f，外切光角17.5f得出的聚光例子：
![](flask_light.png)
4.手电筒完整片段着色器
```C++
void main()
{
	vec3 lightDir = normalize(spotLight.position - FragPos);

	float theta = dot(lightDir, normalize(-spotLight.direction));
	float epsilon = spotLight.cutOff - spotLight.outerCutOff;
	float intensity = clamp((theta - spotLight.outerCutOff) / epsilon, 0.0, 1.0);

	vec3 ambient = spotLight.ambient * vec3(texture(material.diffuse, TexCoords));
	
	vec3 norm = normalize(Normal);
	float diff = max(dot(norm, lightDir), 0.0);
	vec3 diffuse = spotLight.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
	diffuse *= intensity;

	vec3 viewDir = normalize(viewPos - FragPos);
	vec3 reflectDir = reflect(-lightDir, norm);
	float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
	vec3 specular = spotLight.specular * spec * vec3(texture(material.specular, TexCoords));
	specular *= intensity;

	float distance = length(spotLight.position - FragPos);
	float attenuation = 1.0f / (spotLight.constant + pointLight.linear * distance + pointLight.quaderatic * (distance * distance));

	//ambient *= attenuation;
	diffuse *= attenuation;
	specular *= attenuation;

	color = vec4(specular + ambient + diffuse, 1.0f);
};
```
**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/05%20Light%20casters/)**