---
title: Hello, Irrlicht
date: 2016-11-09 11:20:23
tags: Irrlicht
category: Irrlicht
---

## 配置Irrlicht
从Irrlicht官方网站[下载](http://irrlicht.sourceforge.net/?page_id=10)引擎包：
![](1.png)
我们需要创建Virtual Studio工程来使用Irrlicht，配置如下:
1.把引擎包的include和lib文件夹复制到工程下面，在VC++目录配置路径：
![在工程下包含include，lib文件夹](2.png)![配置路径](3.png)
2.在输出的Debug文件夹中加入Irrlicht.dll(在引擎包的bin文件夹下)
3.在连接器中输入Irrlicht.lib
![](link_lib.png)
到这里就配置成功了，接下来写一个简易的demo

## Hello,Irrlicht
```C++
#include <irrlicht.h>

using namespace irr;
using namespace core;
using namespace video;
using namespace scene;
using namespace gui;

int main()
{
	IrrlichtDevice *device = createDevice(video::EDT_SOFTWARE, dimension2d<u32>(640, 480), 16,
			false, false, false, 0);
	if (!device)
		return 1;
	device->setWindowCaption(L"Hello Irrlicht!");
	IVideoDriver* driver = device->getVideoDriver();
	ISceneManager* smgr = device->getSceneManager();
	IGUIEnvironment* guienv = device->getGUIEnvironment();
	video::ITexture* cat = driver->getTexture("../resources/images/cat.jpg");
	guienv->addImage(cat, vector2d<s32>(100, 100));

	while (device->run())
	{
		driver->beginScene(true, true, SColor(255, 25, 25, 25));
		smgr->drawAll();
		guienv->drawAll();
		driver->endScene();
	}

	device->drop();
	return 0;
}
```
使用Irrlicht画一张图：
![](cat.png)