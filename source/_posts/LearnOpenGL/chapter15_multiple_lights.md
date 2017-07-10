---
title: 多光源
date: 2017-7-6 11:11:00
tags: 
    OpenGL
category: LearnOpenGL
---

使用多光源完成1个平行光(Directional light)和4个点光源(Point lights)以及一个手电筒(Flashlight)
多光源的代码结构:
```C++
out vec4 color;

void main()
{
	vec3 result = calcDirectionLight();

	for(int i = 0; i < NR_POINT_LIGHTS; i++)
		result += calcPointLight();

	result += calcSpotLight();

	color = vec4(result, 1.0f);
};
```
## 平行光
1.封装平行光
```C++
struct DirectionLight
{
	vec3 direction;
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
};
```
2.计算平行光的输出颜色
```C++
vec3 calcDirectionLight(DirectionLight light, vec3 normal, vec3 viewDir)
{
	vec3 lightDir = normalize(-light.direction);
	float diff = max(dot(normal, lightDir), 0.0);
	vec3 reflectDir = reflect(-lightDir, normal);
	float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);

	vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
	vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
	vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));

	return ambient + diffuse + specular;
}
```
## 点光源
1.封装点光源
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
2.计算点光源的输出颜色
```C++
vec3 calcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
	vec3 lightDir = normalize(light.position - fragPos);
	float diff = max(dot(normal, lightDir), 0.0);
	vec3 reflectDir = reflect(-lightDir, normal);
	float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);

	vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
	vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
	vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));

	float distance = length(light.position - fragPos);
	float attenuation = 1.0f / (light.constant + light.linear * distance + light.quaderatic * (distance * distance));

	ambient *= attenuation;
	diffuse *= attenuation;
	specular *= attenuation;

	return ambient + diffuse + specular;
}
```
## 手电筒
1.封装聚光
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
2.计算聚光输出颜色
```C++
vec3 calcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
	vec3 lightDir = normalize(light.position - fragPos);
	float theta = dot(lightDir, normalize(-light.direction));
	float epsilon = light.cutOff - light.outerCutOff;
	float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);

	vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));

	vec3 norm = normalize(Normal);
	float diff = max(dot(norm, lightDir), 0.0);
	vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
	diffuse *= intensity;

	vec3 reflectDir = reflect(-lightDir, norm);
	float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
	vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
	specular *= intensity;

	float distance = length(light.position - FragPos);
	float attenuation = 1.0f / (light.constant + light.linear * distance + light.quaderatic * (distance * distance));

	ambient *= attenuation;
	diffuse *= attenuation;
	specular *= attenuation;

	return ambient + diffuse + specular;
}
```
## 放在一起
```C++
void main()
{
	vec3 norm = normalize(Normal);
	vec3 viewDir = normalize(viewPos - FragPos);

	vec3 result = calcDirectionLight(directionLight, norm, viewDir);

	for(int i = 0; i < NR_POINT_LIGHTS; i++)
		result += calcPointLight(pointLight[i], norm, FragPos, viewDir);

	result += calcSpotLight(spotLight, norm, FragPos, viewDir);

	color = vec4(result, 1.0f);
};
```
1.给点光源一个位置
```C++
vec3 pointLightPositions[] = {
    vec3( 0.7f,  0.2f,  2.0f),
    vec3( 2.3f, -3.3f, -4.0f),
    vec3(-4.0f,  2.0f, -12.0f),
    vec3( 0.0f,  0.0f, -3.0f)
};  
```
2.对多光源进行赋值
```C++
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "directionLight.direction"), -0.2f, -1.0f, -0.3f);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "directionLight.ambient"), 0.05f, 0.05f, 0.05f);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "directionLight.diffuse"), 0.4f, 0.4f, 0.4f);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "directionLight.specular"), 0.5f, 0.5f, 0.5f);

string front;
for(int i = 0; i < 4; i++)
{
    front = "pointLight[" + to_string(i) + "]";
    glUniform3f(glGetUniformLocation(lightingShader.getProgram(), (front + ".position").c_str()), pointLightPositions[i].x, pointLightPositions[i].y, pointLightPositions[i].z);
    glUniform3f(glGetUniformLocation(lightingShader.getProgram(), (front + ".ambient").c_str()), 0.05f, 0.05f, 0.05f);
    glUniform3f(glGetUniformLocation(lightingShader.getProgram(), (front + ".diffuse").c_str()), 0.8f, 0.8f, 0.8f);
    glUniform3f(glGetUniformLocation(lightingShader.getProgram(), (front + ".specular").c_str()), 1.0f, 1.0f, 1.0f);
    glUniform1f(glGetUniformLocation(lightingShader.getProgram(), (front + ".constant").c_str()), 1.0f);
    glUniform1f(glGetUniformLocation(lightingShader.getProgram(), (front + ".linear").c_str()), 0.09f);
    glUniform1f(glGetUniformLocation(lightingShader.getProgram(), (front + ".quadratic").c_str()), 0.032f);
}

vec3 cameraFront = camera.getFront();
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.position"), cameraPos.x, cameraPos.y, cameraPos.z);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.direction"), cameraFront.x, cameraFront.y, cameraFront.z);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.cutOff"), cos(radians(12.5f)));
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.outerCutOff"), cos(radians(15.0f)));
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.ambient"), 0.0f, 0.0f, 0.0f);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.diffuse"), 1.0f, 1.0f, 1.0f);
glUniform3f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.specular"), 1.0f, 1.0f, 1.0f);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.constant"), 1.0f);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.linear"), 0.09);
glUniform1f(glGetUniformLocation(lightingShader.getProgram(), "spotLight.quadratic"), 0.032);
```
得出多光源的例子:
1.未添加手电筒
![](multi_light_example_no_flask.png)
2.添加手电筒
![](multi_lights_example_with_flask.png)
**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/02%20Lighting/06%20Multiple%20lights/)**