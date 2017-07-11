---
title: 模型
date: 2017-7-11 17:03:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 封装模型类
一个模型包含多个网格(Mesh)，一个网格可能带有多个对象。
```C++
class Model
{
public:
	Model(const char* path);
	void draw(const Shader& shader);
private:
	void loadModel(const std::string& path);
	void processNode(aiNode* node, const aiScene* scene);
	Mesh processMesh(aiMesh* mesh, const aiScene* scene);
	std::vector<Texture> loadMaterialTextures(aiMaterial* mat, aiTextureType type, const std::string& typeName);
	GLint TextureFromFile(const char* path, const std::string& directory);
private:
	std::vector<Mesh> _meshes;
	std::string _directory;
	std::vector<Texture> _loadedTextures;
};
```
draw渲染，调用网格的draw函数
```C++
void Model::draw(const Shader & shader)
{
	for (auto& mesh : _meshes)
		mesh.draw(shader);
}
```
## 导入3D模型到OpenGL
1.使用Assimp的头文件
```C++
#include <assimp/Importer.hpp>
#include <assimp/scene.h>
#include <assimp/postprocess.h>
```
2.加载模型
```C++
void Model::loadModel(const std::string & path)
{
	Importer import;
	const aiScene* scene = import.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);
	if (scene == nullptr || scene->mFlags == AI_SCENE_FLAGS_INCOMPLETE || scene->mRootNode == nullptr)
	{
		cout << "ERROR::ASSIMP::" << import.GetErrorString() << endl;
		return;
	}
	_directory = path.substr(0, path.find_last_of("/"));
	processNode(scene->mRootNode, scene);
}
```
* aiProcess_Triangulate：转换所有的模型的原始几何形状为三角形
* aiProcess_FlipUVs：翻转y轴纹理坐标,使用**Soil库创建图片的时候不需要在顶点着色器翻转了**
* aiProcess_GenNormals:如果模型没有包含法线向量，就为每个顶点创建法线。
* aiProcess_SplitLargeMeshes:把大的网格成几个小的的下级网格，当你渲染有一个最大数量顶点的限制时或者只能处理小块网格时很有用。
* aiProcess_OptimizeMeshes:和上个选项相反，它把几个网格结合为一个更大的网格。以减少绘制函数调用的次数的方式来优化。

3.遍历node，得到所有网格数据
```C++
void Model::processNode(aiNode * node, const aiScene * scene)
{
	for (GLuint i = 0; i < node->mNumMeshes; i++)
	{
		aiMesh* mesh = scene->mMeshes[node->mMeshes[i]];
		_meshes.push_back(processMesh(mesh, scene));
	}

	for (GLuint i = 0; i < node->mNumChildren; i++)
	{
		processNode(node->mChildren[i], scene);
	}
}
```
**Important**
**认真的读者会注意到，我们可能基本忘记处理任何的节点，简单循环出场景所有的网格，而不是用索引做这件复杂的事。我们这么做的原因是，使用这种节点的原始的想法是，在网格之间定义一个父-子关系。通过递归遍历这些关系，我们可以真正定义特定的网格作为其他网格的父（节点）。**
**关于这个系统的一个有用的例子是，当你想要平移一个汽车网格需要确保把它的子（节点）比如，引擎网格，方向盘网格和轮胎网格都进行平移；使用父-子关系这样的系统很容易被创建出来。**
**现在我们没用这种系统，但是无论何时你想要对你的网格数据进行额外的控制，这通常是一种坚持被推荐的做法。这些模型毕竟是那些定义了这些节点风格的关系的艺术家所创建的。**

4.从Assimp的网格数据中，取出顶点数据，索引数据，纹理数据
获取定点数据
```C++
for (GLuint i = 0; i < mesh->mNumVertices; i++)
{
	Vertex vertex;
	aiVector3D aiVertex = mesh->mVertices[i];
	vertex.position = glm::vec3(aiVertex.x, aiVertex.y, aiVertex.z);

	aiVector3D aiNormal = mesh->mNormals[i];
	vertex.normal = glm::vec3(aiNormal.x, aiNormal.y, aiNormal.z);

	aiVector3D* texCoords0 = mesh->mTextureCoords[0];
	if (texCoords0 != nullptr)
	{
		vertex.texCoords = glm::vec2(texCoords0[i].x, texCoords0[i].y);
	}
	else
	{
		vertex.texCoords = glm::vec2(0.0f, 0.0f);
	}

	vertices.push_back(vertex);
}
```
获取索引数据
```C++
for (GLuint i = 0; i < mesh->mNumFaces; i++)
{
	aiFace* face = &(mesh->mFaces[i]);
	for (GLuint j = 0; j < face->mNumIndices; j++)
	{
		indices.push_back(face->mIndices[j]);
	}
}
```
根据材质获取纹理数据
```C++
if (mesh->mMaterialIndex >= 0)
{
	aiMaterial* material = scene->mMaterials[mesh->mMaterialIndex];
	vector<Texture> diffuseMaps = loadMaterialTextures(material, aiTextureType_DIFFUSE, "texture_diffuse");
	textures.insert(textures.end(), diffuseMaps.begin(), diffuseMaps.end());
	vector<Texture> specularMaps = loadMaterialTextures(material, aiTextureType_SPECULAR, "texture_specular");
	textures.insert(textures.end(), specularMaps.begin(), specularMaps.end());
}
```
```C++
std::vector<Texture> Model::loadMaterialTextures(aiMaterial * mat, aiTextureType type, const std::string & typeName)
{
	vector<Texture> textures;
	for (GLuint i = 0; i < mat->GetTextureCount(type); i++)
	{
		aiString str;
		mat->GetTexture(type, i, &str);
		GLboolean skip = false;
		for (auto& loadTexture : _loadedTextures)
		{
			if (loadTexture.path == str.C_Str())
			{
				textures.push_back(loadTexture);
				skip = true;
				break;
			}
		}
		if (!skip)
		{
			Texture texture;
			texture.id = TextureFromFile(str.C_Str(), _directory);
			texture.type = typeName;
			texture.path = str.C_Str();
			textures.push_back(texture);
			_loadedTextures.push_back(texture);
		}
	}
	return textures;
}
```
* _loadedTextures是为了防止创建已经创建过的纹理

使用Soil加载图片
```C++
GLint Model::TextureFromFile(const char * path, const std::string & directory)
{
	string filename = directory + "/" + path;
	GLuint textureId;
	glGenTextures(1, &textureId);
	int width, height;
	unsigned char* image = SOIL_load_image(filename.c_str(), &width, &height, 0, SOIL_LOAD_RGBA);

	glBindTexture(GL_TEXTURE_2D, textureId);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, image);
	glGenerateMipmap(GL_TEXTURE_2D);

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glBindTexture(GL_TEXTURE_2D, 0);
	SOIL_free_image_data(image);

	return textureId;
}
```
[Model类的源码](https://github.com/tacthgin/toy/blob/master/OpenGL/src/Model.h)
## 加载模型
下载孤岛危机的[模型](https://github.com/tacthgin/toy/tree/master/OpenGL/images/nanosuit)
这个模型被输出为obj和mtl文件，mtl包含模型的diffuse和specular以及法线贴图。
加载模型后是这样的:
![](model_example.png)
查看[顶点](https://github.com/tacthgin/toy/blob/master/OpenGL/shaders/model.vs)和[片段](https://github.com/tacthgin/toy/blob/master/OpenGL/shaders/model.frag)以及[源代码](https://github.com/tacthgin/toy/blob/master/OpenGL/src/modelMain.cpp)

**PS:之前发现写好代码无论如何都显示不出模型，各种调试。后来发现是着色器程序忘记使用了=。=**

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/03%20Model%20Loading/03%20Model/)**