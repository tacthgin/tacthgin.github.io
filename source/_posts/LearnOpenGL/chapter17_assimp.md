---
title: Assimp
date: 2017-7-11 16:12:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 模型加载库
Assimp(Open Asset Import Library)支持倒入几十种不同格式的模型文件。
Assimp模型文件数据结构：
![](assimp_structure.png)
* 所有的模型、场景数据都包含在scene对象中，如所有的材质和Mesh。同样，场景的根节点引用也包含在这个scene对象中
* 场景的根节点可能也会包含很多子节点和一个指向保存模型点云数据mMeshes[]的索引集合。根节点上的mMeshes[]里保存了实际了Mesh对象，而每个子节点上的mMesshes[]都只是指向根节点中的mMeshes[]的一个引用
* 一个Mesh对象本身包含渲染所需的所有相关数据，比如顶点位置、法线向量、纹理坐标、面片及物体的材质
* 一个Mesh会包含多个面片。一个Face（面片）表示渲染中的一个最基本的形状单位，即图元（基本图元有点、线、三角面片、矩形面片）。一个面片记录了一个图元的顶点索引，通过这个索引，可以在mMeshes[]中寻找到对应的顶点位置数据。顶点数据和索引分开存放，可以便于我们使用缓存（VBO、NBO、TBO、IBO）来高速渲染物体。（详见Hello Triangle）
* 一个Mesh还会包含一个Material（材质）对象用于指定物体的一些材质属性。如颜色、纹理贴图（漫反射贴图、高光贴图等）
**Important**
**网格(Mesh)**
**用建模工具构建物体时，美工通常不会直接使用单个形状来构建一个完整的模型。一般来说，一个模型会由几个子模型/形状组合拼接而成。而模型中的那些子模型/形状就是我们所说的一个网格。例如一个人形模型，美工通常会把头、四肢、衣服、武器这些组件都分别构建出来，然后在把所有的组件拼合在一起，形成最终的完整模型。一个网格（包含顶点、索引和材质属性）是我们在OpenGL中绘制物体的最小单位。一个模型通常有多个网格组成。**
## 构建Assimp
从[Assimp页面下载](http://assimp.sourceforge.net/main_downloads.html),用cmake编译选择vs2015，生成lib和dll(在code/Debug目录)，lib放在工程链接，dll放在可执行，头文件从include获取

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/03%20Model%20Loading/01%20Assimp/)**