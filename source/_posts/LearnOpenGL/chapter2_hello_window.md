---
title: 你好，窗口
date: 2016-10-19 16:00:15
tags: 
    OpenGL
category: LearnOpenGL
---

## 创建窗口
这里我们直接用代码展示如何创建一个窗口：

```C++
	#include <iostream>
	#define GLEW_STATIC    //静态链接GLEW
	#include <GL/glew.h>
	#include <GLFW\glfw3.h>   
	 
	//键盘事件回调
	void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
	{
		if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
		{
			glfwSetWindowShouldClose(window, GL_TRUE);
		}
	}
	
	int main()
	{
		glfwInit();  //初始化GLFW
		glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
		glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
		glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
		glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);
		
		GLFWwindow *window = glfwCreateWindow(800, 600, "你好，窗口", nullptr, nullptr);   //创建窗口对象
		if (window == nullptr)
		{
			std::cout << "Failed to create GLFW window" << std::endl;
			glfwTerminate();
		    return -1;
		}
	
		glfwMakeContextCurrent(window);
	
		glfwSetKeyCallback(window, key_callback); //加载窗口按键输入回调
	
		glewExperimental = GL_TRUE;//设置TRUE时，GLEW在管理OpenGL的函数指针时更多地使用现代化的技术
		if (glewInit() != GLEW_OK)//GLEW初始化
		{
			std::cout << "Failed to initialize GLEW" << std::endl;
			return -1;
		}
	
		glViewport(0, 0, 800, 600);//处理渲染窗口大小
	
		while (!glfwWindowShouldClose(window))//游戏循环
		{
			 //检查有没有触发什么事件（比如键盘输入、鼠标移动等），然后调用对应的回调函数（可以通过回调方法手动设置）。我们一般在游戏循环的开始调用事件处理函数。
			glfwPollEvents();
			glfwSwapBuffers(window);
		}
	
		glfwTerminate();
		return 0;
	}
```

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/03%20Hello%20Window/)**