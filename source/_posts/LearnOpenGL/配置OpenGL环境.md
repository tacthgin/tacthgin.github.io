---
title: 配置OpenGL环境
date: 2016-10-17 10:00:37
tags: 
    OpenGL
    Visual Studio
category: LearnOpenGL
---

# GLFW
GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入。
# 构建GLFW
GLFW可以从它官方网站的[下载页](http://www.glfw.org/download.html)上获取。解压下载的包可以得到：
1.编译生成的库
2.include文件夹
我们需要让IDE知道库和头文件的位置。有两种方法：
1.找到IDE或者编译器的/lib和/include文件夹，添加GLFW的include文件夹里的文件到IDE的/include文件夹里去。用类似的方法，将glfw3.lib添加到/lib文件夹里去。
2.推荐的方式是建立一个新的目录包含所有的第三方库文件和头文件，并且在你的IDE或编译器中指定这些文件夹。
在Virtual Studio中配置:
1.我们首先进入Project Properties(工程属性，在解决方案窗口里右键项目)，然后选择VC++ Directories(VC++ 目录)选项卡（如下图）。在下面的两栏添加目录：
![img_1](1.png)