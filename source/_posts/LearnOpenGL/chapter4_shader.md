---
title: 着色器
date: 2017-5-25 7:48:00
tags: 
    OpenGL
category: LearnOpenGL
---

## GLSL
1.着色器使用GLSL语言编写,代码结构如下
```C++
#version version_number

in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

2.顶点着色器中,每个输入变量叫做顶点属性(Vertex Attribute),使用GL_MAX_VERTEX_ATTRIBS查询支持的顶点属性上限
```C++
GLint nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes << std::endl;
```

## 数据类型
像C语言一样包含技术数据类型:int、float、double、uint、bool

### 向量
1.包含几个基础类型的分量的容器
![](vec.png)

2.重组(Swizzling)
```C++
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

```C++
vec2 vect = vec2(0.5f, 0.7f);
vec4 result = vec4(vect, 0.0f, 0.0f);
vec4 otherResult = vec4(result.xyz, 1.0f);
```

### Uniform
1.uniform是一种从CPU向GPU着色器发送数据方式
2.uniform是全局的,在着色器对象中是唯一的
3.可以被着色器程序的任意着色器在任意阶段访问
4.unifrom可以保存数据,除非被重置或更新
```C++
#version 330 core
out vec4 color;

uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量

void main()
{
    color = ourColor;
}  
```

**Attention**
**如果你声明了一个uniform却在GLSL代码中没用过，编译器会静默移除这个变量，导致最后编译出的版本中并不会包含它，这可能导致几个非常麻烦的错误，记住这点！**

5.使用uniform,动态改变三角形颜色
```C++
GLfloat timeValue = glfwGetTime();
GLfloat greenValue = (sin(timeValue) / 2) + 0.5;
GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

整体渲染例子:
```C++
while(!glfwWindowShouldClose(window))
{
    // 检测并调用事件
    glfwPollEvents();

    // 渲染
    // 清空颜色缓冲
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    // 记得激活着色器
    glUseProgram(shaderProgram);

    // 更新uniform颜色
    GLfloat timeValue = glfwGetTime();
    GLfloat greenValue = (sin(timeValue) / 2) + 0.5;
    GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
    glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

    // 绘制三角形
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);
    glBindVertexArray(0);
}
```

## 更多属性！
1.在VBO中添加颜色顶点属性
```C++
GLfloat vertices[] = {
    // 位置              // 颜色
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
};
```

2.在顶点着色器中添加颜色输入
```C++
#version 330 core
layout (location = 0) in vec3 position; // 位置变量的属性位置值为 0 
layout (location = 1) in vec3 color;    // 颜色变量的属性位置值为 1

out vec3 ourColor; // 向片段着色器输出一个颜色

void main()
{
    gl_Position = vec4(position, 1.0);
    ourColor = color; // 将ourColor设置为我们从顶点数据那里得到的输入颜色
}
```

3.片段着色器使用顶点着色器的颜色输出
```C++
#version 330 core
in vec3 ourColor;
out vec4 color;

void main()
{
    color = vec4(ourColor, 1.0f);
}
```

4.顶点属性在VBO中的内存布局
![](vertex_attribute_pointer_interleaved.png)

5.使用glVertexAttribPointer更新顶点格式,因为添加了颜色顶点属性,所以要更新步长值为6
```C++
// 位置属性
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
// 颜色属性
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3* sizeof(GLfloat)));
glEnableVertexAttribArray(1);
```
![](color_vertex_attribute.png)

## 自己的着色器类
```C++
#ifndef _SHADER_H_
#define _SHADER_H_

#include <GL/glew.h>

class Shader
{
public:
	Shader(const GLchar* vertexPath, const GLchar* fragmentPath);
	//使用着色器程序对象
	void use();
	GLint getProgram();
private:
	GLuint loadShader(GLint shaderType, const GLchar* sharderSource);
	void linkProgram(const GLchar* vertexShaderSource, const GLchar* fragmentShaderSource);
private:
	//程序id
	GLint _program;
};

#endif // !_SHADER_H_
```

##从文件读取
```C++
Shader::Shader(const GLchar * vertexPath, const GLchar * fragmentPath)
:_program(0)
{
	if (vertexPath == nullptr || fragmentPath == nullptr) return;
	string vertexCode;
	string fragmentCode;
	ifstream vShaderFile;
	ifstream fShaderFile;
	vShaderFile.exceptions(ifstream::badbit);
	fShaderFile.exceptions(ifstream::badbit);
	try
	{
		vShaderFile.open(vertexPath);
		fShaderFile.open(fragmentPath);
		stringstream vShaderStream, fShaderStream;
		vShaderStream << vShaderFile.rdbuf();
		fShaderStream << fShaderFile.rdbuf();
		vShaderFile.close();
		fShaderFile.close();
		vertexCode = vShaderStream.str();
		fragmentCode = fShaderStream.str();
	}
	catch (const ifstream::failure& e)
	{
		cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << endl;
	}

	[...]
}
```

使用例子:
```C++
	Shader shader("shaders/shader.vs", "shaders/shader.frag");
	
	GLfloat vertices[] = {
		0.0f, 0.5f, 0.0f, 1.0f, 0.0f, 0.0f,
		-0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f,
		0.5f, -0.5f, 0.0f, 0.0f, 0.0f, 1.0f,
	};

	GLuint VBO, VAO, IBO;
	glGenVertexArrays(1, &VAO); //创建vao
	glGenBuffers(1, &VBO); //创建vbo
	glGenBuffers(1, &IBO); //创建ibo

	glBindVertexArray(VAO); //绑定vao

	glBindBuffer(GL_ARRAY_BUFFER, VBO);//绑定缓冲对象
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); //复制顶点数据到缓冲对象

	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)0); //设置顶点属性
	glEnableVertexAttribArray(0);

	glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3 * sizeof(GLfloat))); //设置顶点属性
	glEnableVertexAttribArray(1);

	glBindBuffer(GL_ARRAY_BUFFER, 0);
	glBindVertexArray(0); //解绑vao

	while (!glfwWindowShouldClose(window))
	{
		glfwPollEvents();//处理触发事件(键盘,鼠标)

		glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
		glClear(GL_COLOR_BUFFER_BIT);

		shader.use();
		glBindVertexArray(VAO);
		glDrawArrays(GL_TRIANGLES, 0, 3);
		glBindVertexArray(0);

		glfwSwapBuffers(window);
		
	}
	glDeleteVertexArrays(1, &VAO);
	glDeleteBuffers(1, &VBO);
```

[源代码](https://github.com/tacthgin/toy/blob/master/OpenGL/src/Shader.cpp)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/05%20Shaders/)**