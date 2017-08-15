---
title: 网格
date: 2017-7-11 16:12:00
tags: 
    OpenGL
category: LearnOpenGL
---

## 封装网格类
一个网格(Mesh)代表一个可绘制实体，一个网格应该至少需要一组顶点，每个顶点包含一个位置向量，一个法线向量，一个纹理坐标向量。一个网格也应该包含一个索引绘制用的索引，以纹理（diffuse/specular map）形式表现的材质数据。
1.定义顶点类
```c++
class Vertex
{
public:
	glm::vec3 position;
	glm::vec3 normal;
	glm::vec2 texCoords;
};
```
2.定义纹理类
```c++
class Texture
{
public:
	GLuint id;
	std::string type; //判断是diffuse纹理还是specular纹理
	std::string path; //路径用来判断是否重复导入
};
```
3.定义网格类
```c++
class Mesh
{
public:
	Mesh(const std::vector<Vertex>& vertices, const std::vector<GLuint>& indices, const std::vector<Texture>& textures);
	~Mesh();
	void draw(const Shader& shader);
private:
	void setupMesh();
private:
	GLuint _VAO;
	GLuint _VBO;
	GLuint _EBO;
	std::vector<Vertex> _vertices;
	std::vector<GLuint> _indices;
	std::vector<Texture> _textures;
};
```
## 初始化
```c++
Mesh::Mesh(const std::vector<Vertex>& vertices, const std::vector<GLuint>& indices, const std::vector<Texture>& textures)
:_vertices(vertices)
,_indices(indices)
,_textures(textures)
{
	setupMesh();
}
```
```c++
void Mesh::setupMesh()
{
	glGenVertexArrays(1, &_VAO);
	glGenBuffers(1, &_VBO);
	glGenBuffers(1, &_EBO);

	glBindVertexArray(_VAO);

	glBindBuffer(GL_ARRAY_BUFFER, _VBO);
	glBufferData(GL_ARRAY_BUFFER, _vertices.size() * sizeof(Vertex), &_vertices[0], GL_STATIC_DRAW);

	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _EBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, _indices.size() * sizeof(GLuint), &_indices[0], GL_STATIC_DRAW);

	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (GLvoid*)0);
	glEnableVertexAttribArray(0);

	glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (GLvoid*)offsetof(Vertex, normal));
	glEnableVertexAttribArray(1);

	glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex), (GLvoid*)offsetof(Vertex, texCoords));
	glEnableVertexAttribArray(2);

	glBindVertexArray(0);
}
```
* 因为数组的内存排列是连续的，所以可以用&_vertices[0]首地址
* offsetof可以获取到成员变量的偏移地址
```c++
#define offsetof(s,m) ((size_t)&reinterpret_cast<char const volatile&>((((s*)0)->m)))
```
## 渲染
1.由于不知道一个网格需要的diffuse贴图和specular贴图个数，可以使用格式化纹理名字的方式来传递,或者采用定义一个较大的数组，然后赋值的方式
```c++
uniform sampler2D texture_diffuse1;
uniform sampler2D texture_diffuse2;
uniform sampler2D texture_diffuse3;
uniform sampler2D texture_specular1;
uniform sampler2D texture_specular2;
```
2.实现draw函数
```c++
void Mesh::draw(const Shader & shader)
{
	GLuint diffuseNr = 1;
	GLuint specularNr = 1;
	for (GLuint i = 0; i < _textures.size(); i++)
	{
		glActiveTexture(GL_TEXTURE0 + i);
		stringstream ss;
		const auto& name = _textures[i].type;
		if (name == "texture_diffuse")
			ss << diffuseNr++;
		else if (name == "texture_specular")
			ss << specularNr++;
		string number = ss.str();

		glUniform1f(glGetUniformLocation(shader.getProgram(), (name + number).c_str()), i);
		glBindTexture(GL_TEXTURE_2D, _textures[i].id);
	}

	glBindVertexArray(_VAO);
	glDrawElements(GL_TRIANGLES, _indices.size(), GL_UNSIGNED_INT, 0);
	glBindVertexArray(0);

	for (GLuint i = 0; i < _textures.size(); i++)
	{
		glActiveTexture(GL_TEXTURE0 + i);
		glBindTexture(GL_TEXTURE_2D, 0);
	}
}
```
[Mesh类的源码](https://github.com/tacthgin/toy/blob/master/OpenGL/src/Mesh.h)

**源文章出处[LearnOpenGL](http://learnopengl-cn.readthedocs.io/zh/latest/03%20Model%20Loading/02%20Mesh/)**