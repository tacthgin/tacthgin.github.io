---
title: Lua的Astar实现
date: 2017-9-30 9:47:00
tags: 
    Astar
category: Toy
---

## 用Lua实现了一个简单的方块A\*寻路。
A\*寻路的原理是会计算路径，产生一个估值f值，这个估值是由起点到走过的路径g值和当前的位置到终点的估值h组成的。取这个估值的最小值来进行寻路，所以A*寻路的效率是由估值函数的效率决定的。
代码分析：
* 创建一个节点，该节点保存位置和估值信息，并保留父节点，相当于整个路径是由链式节点组成。
```lua
local node = {}
node.__index = node
function node.new(pos, h, p)
    local n = {}
    n.pos = {x = pos.x, y = pos.y}
    n.h = h
    n.parent = p
    n.g = 0
    if p ~= nil then
        n.g = p.g + 1
    end
    n.f = n.g + n.h
    return setmetatable(n, node)
end

function node:get_pos()
    return self.pos
end

function node:get_g()
    return self.g
end

function node:get_f()
    return self.f
end
```
* 开始寻路的时候，会把当前节点的四周节点加入开放列表中（寻找路径只会在开放列表中），把障碍物或者已经走过的路径放入关闭列表中（防止重复行走）
```lua
local astar = {}
astar.close = {}
astar.open = {}

--估值函数，方块地图简单的使用终点位置到起点的估值
local function estimate_hvalue(srcPos, dstPos)
    return math.abs(srcPos.x - dstPos.x) + math.abs(srcPos.y - dstPos.y);
end

--f值排序
local function nodeSort(l, r)
    if l:get_f() == r:get_f() then
        return l:get_g() > r:get_g()
    end

    return l:get_f() < r:get_f()
end

local function contain(t, pos)
    for _, v in ipairs(t) do
        if v:get_pos().x == pos.x and v:get_pos().y == pos.y then
            return v
        end
    end
end

function astar:fitNode(pos)
    if self.map[pos.x] ~= nil and self.map[pos.x][pos.y] == 0 then
        return true
    end
    return false
end

--把周围4个节点不是墙或者没走过的节点加入开放列表中，开放列表中按f值从少到多排序
function astar:addOtherNode(curNode)
    if curNode == nil then return end

    local d = {
        {x = 0, y = 1},
        {x = 0, y = -1},
        {x = -1, y =  0},
        {x = 1, y = 0}
    }

    table.insert(self.close, curNode)

    local newPos, h = {x = 0, y = 0}, 0
    for _, v in ipairs(d) do
        newPos.x = curNode:get_pos().x + v.x
        newPos.y = curNode:get_pos().y + v.y
        h = estimate_hvalue(newPos, self.dstPos)
        if contain(self.close, newPos) == nil and self:fitNode(newPos) then
            table.insert(self.open, node.new(newPos, h, curNode))
        end
    end

    table.sort(self.open, nodeSort)
end

--重复从开放列表中取f值低的节点，直到终点，然后把终点的节点和一连串的父节点组成路径
function astar:makePath()
    self.close = {}
    self.open = {}
    local path = {}

    local h = estimate_hvalue(self.srcPos, self.dstPos)
    if h == 0 then return end

    local n = node.new(self.srcPos, h, nil)
    table.insert(self.open, n)

    local last = nil
    while #self.open ~= 0 and last == nil do
        n = table.remove(self.open, 1)
        self:addOtherNode(n)
        last = contain(self.open, self.dstPos)
    end

    while last ~= nil do
        table.insert(path, 1, last)
        last = last.parent
    end

    return path
end
```
* 加载地图等资源进行寻路
```lua
local astar = require "astar"

local map = {
	{0, 0, 0, 0, 0, 0},
	{0, 0, 1, 1, 1, 0},
	{0, 0, 1, 0, 1, 0},
	{0, 1, 1, 0, 1, 0},
	{0, 1, 0, 0, 1, 0},
	{0, 0, 0, 0, 0, 0}
}

function printMap()
    for i=1, #map do
        local s = ""
        for j=1, #map[i] do
            s = s..string.format("%d ", map[i][j])
        end
        print(s)
    end
end

function main()
    local path = astar:getPath({x = 1, y = 1}, {x = 3, y = 4}, map)

    if path ~= nil then
        print("before")
        printMap()
        
        local n = nil
        for i=1, #path do
            n = path[i]
            map[n:get_pos().x][n:get_pos().y] = 2
        end

        print("after")
        printMap()
    end
end

main()
```
![](astar_result.png)
0是空地，1是墙壁，2是行走的路径

## A\*寻路的优化
* 当地图比较大的时候，使用A*寻路的效率就会变低，这时候不用再一次函数计算中得到结果，可以在游戏帧中分几帧来计算。
* 像魔兽或者帝国中，一大堆兵团进行寻路时候，可以使用队列中间的一个来进行计算，其他兵种靠偏移量来确定位置。