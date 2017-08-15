---
title: 纹理
date: 2017-6-1 21:07:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 纹理
1.使用顶点的纹理坐标(Texture Coordinate)来指定在纹理的哪个部分采样(Sampling, 用纹理坐标获取纹理颜色)
2.纹理坐标从左下角(0, 0)到右上角(1,1),剩余的片段会进行片段插值(Fragment Interpolation)
纹理坐标:
![](tex_coords.png)
```c++
GLfloat texCoords[] = {
    0.0f, 0.0f, // 左下角
    1.0f, 0.0f, // 右下角
    0.5f, 1.0f // 上中
};
```

## 纹理环绕方式
1.如果纹理坐标设置在0 - 1范围之外,Opengl提供一些处理方式(默认是重复这个纹理图像)
![](texture_wrap.png)
效果:
![](texture_wrapping.png)

2.使用glTexParameter*对单独坐标轴设置(s, t, r(3D纹理使用)类似x, y, z)
```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```

3.使用glTexParameterfv指定GL_TEXTURE_BORDER_COLOR方式的颜色
```c++
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

## 纹理过滤
1.纹理坐标不依赖分辨率,对低分辨率纹理处理采用GL_NEAREST和GL_LINEAR
2.GL_NEAREST(邻近过滤,Nearest Neighbor Filtering),OpenGL选择中心点最接近纹理坐标的像素(OpenGL默认使用过滤方式)
![](filter_nearest.png)

3.GL_LINEAR（也叫线性过滤，(Bi)linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。
![](filter_linear.png)

效果:
![](texture_filtering.png)

4.进行放大(Magnify)和缩小(Minify)设置
```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### 多级渐远纹理
1.多级渐远纹理(Mipmap)用来处理远处的物体只产生很少的片段问题
![](mipmaps.png)

2.多级渐远纹理只在纹理缩小的时候才能使用
```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

## 加载与创建纹理
### SOIL(Simple OpenGL Image Library)
1.使用SOIL库,从[主页](http://www.lonesock.net/soil.html)下载,使用vs生成lib,在include文件夹添加src文件夹的内容,并把lib添加到链接选项,导入头文件
```c++
#include <SOIL.h>
```

2.加载一张图片
```c++
int width, height;
unsigned char* image = SOIL_load_image("../images/timg.jpg", &width, &height, 0, SOIL_LOAD_RGBA);
```
第四个参数指定图片的通道(Channel)数量，但是这里我们只需留为0

### 生成纹理
1.创建纹理ID
```c++
GLuint texture;
glGenTextures(1, &texture);
```

2.绑定纹理
```c++
glBindTexture(GL_TEXTURE_2D, texture);
```

3.加载纹理
```c++
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, image);
glGenerateMipmap(GL_TEXTURE_2D);
```
* 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。这里我们填0，也就是基本级别。
* 第六个参数应该总是被设为0（历史遗留问题）。

4.释放资源
```c++
SOIL_free_image_data(image);
glBindTexture(GL_TEXTURE_2D, 0);
```

5.例子
```c++
GLuint texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
//纹理坐标范围超出设置
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
//纹理过滤
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

unsigned char* image = SOIL_load_image("../images/timg.jpg", &width, &height, 0, SOIL_LOAD_RGBA);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, image);
glGenerateMipmap(GL_TEXTURE_2D);
SOIL_free_image_data(image);
glBindTexture(GL_TEXTURE_2D, 0);
```

### 应用纹理
1.把纹理坐标更新到顶点数据中
```c++
	GLfloat vertices[] = {
//     ---- 位置 ----        ---- 颜色 ----       - 纹理坐标 -
		 0.5f,  0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  1.0f, 1.0f, //右上
		 0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  1.0f, 0.0f, //右下
		-0.5f, -0.5f, 0.0f,  0.0f, 0.0f, 1.0f,  0.0f, 0.0f, //左下
		-0.5f,  0.5f, 0.0f,  1.0f, 1.0f, 0.0f,  0.0f, 1.0f  //左上
	};
```

2.使用纹理顶点属性
![](vertex_attribute_pointer_interleaved_textures.png)
```c++
glVertexAttribPointer(2, 2, GL_FLOAT,GL_FALSE, 8 * sizeof(GLfloat), (GLvoid*)(6 * sizeof(GLfloat)));
glEnableVertexAttribArray(2);
```

3.修改顶点着色器
```c++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 color;
layout (location = 2) in vec2 texCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(position, 1.0f);
    ourColor = color;
    TexCoord = texCoord;
}
```

4.修改片段着色器
```c++
#version 330 core
in vec3 ourColor;
in vec2 TexCoord;

out vec4 color;

uniform sampler2D ourTexture;

void main()
{
    color = texture(ourTexture, TexCoord);
}
```
texture函数来采样纹理的颜色，它第一个参数是纹理采样器，第二个参数是对应的纹理坐标。texture函数会使用之前设置的纹理参数对相应的颜色值进行采样。这个片段着色器的输出就是纹理的（插值）纹理坐标上的(过滤后的)颜色。

5.绑定纹理,画出图形
```c++
glBindTexture(GL_TEXTURE_2D, texture);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
glBindVertexArray(0);
```

![](texture.png)

6.纹理混合颜色
```c++
color = texture(ourTexture, TexCoord) * vec4(ourColor, 1.0f);
```

![](texture_mix_color.png)

### 纹理单元(Texture Unit)
1.通过分配给纹理采样器一个位置值,可以在片段着色器设置多个纹理
2.一个纹理的位置值称为一个纹理单元,默认的激活纹理单元是0
3.激活纹理单元
```c++
glActiveTexture(GL_TEXTURE0); //在绑定纹理之前先激活纹理单元
glBindTexture(GL_TEXTURE_2D, texture);
```

**Important**
**OpenGL至少保证有16个纹理单元供你使用，也就是说你可以激活从GL_TEXTURE0到GL_TEXTRUE15。它们都是按顺序定义的，所以我们也可以通过GL_TEXTURE0 + 8的方式获得GL_TEXTURE8，这在当我们需要循环一些纹理单元的时候会很有用。**

4.在片段着色器中添加采样器
```c++
#version 330 core
...

uniform sampler2D ourTexture1;
uniform sampler2D ourTexture2;

void main()
{
    color = mix(texture(ourTexture1, TexCoord), texture(ourTexture2, TexCoord), 0.2);
}
```
mix第三个参数表示返回80%的第一个输入颜色和20%的第二个输入颜色

5.使用例子:
```c++
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glUniform1i(glGetUniformLocation(shader.getProgram(), "ourTexture1"), 0);

glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);
glUniform1i(glGetUniformLocation(shader.getProgram(), "ourTexture2"), 1);

glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
glBindVertexArray(0);
```

**Tip**
**SOIL_load_image加载图形的时候是颠倒的,所以在顶点着色器中修改纹理坐标TexCoord = vec2(texCoord.x, 1 - texCoord.y);,最好的方式是换个加载图像的库。**

![](texture_mix_texture.png)

[源代码](https://github.com/tacthgin/toy/tree/master/OpenGL)在这

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/06%20Textures/)**