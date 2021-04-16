---
title: 方格地图A*的TS实现
date: 2021-4-16 10:09:00
tags: 
    Astar
category: Toy
---

## 回顾
之前用Lua实现了一个简单的A*，参考这个文章([Lua的Astar实现](2017/09/30/Toy/astar_lua/))。然后自己做魔塔后，就用ts实现了一个版本。
## A*的TS代码
```TypeScript 
import { Vec2 } from "cc";

/** A*节点 */
class AstarNode {
    private _parent: AstarNode = null;

    private _position: Vec2 = null;
    /** 当前结点离起始点的路程 */
    private _gValue: number = null;
    /** 该结点的总路程，f = g + h */
    private _fValue: number = null;

    /**
     *
     * @param position 当前节点位置
     * @param hValue 预估值
     * @param parentNode 父结点
     */
    constructor(position: Vec2, hValue: number, parentNode: AstarNode = null) {
        this._parent = parentNode;
        this._position = position;
        this._gValue = parentNode ? parentNode.gValue + 1 : 0;
        this._fValue = this._gValue + hValue;
    }

    get parent() {
        return this._parent;
    }

    get position() {
        return this._position;
    }

    get gValue() {
        return this._gValue;
    }

    get fValue() {
        return this._fValue;
    }

    equals(position: Vec2) {
        return this._position.equals(position);
    }

    add(position: Vec2) {
        return this._position.add(position);
    }
}

export interface AstarMap {
    getRow(): number;
    getColumn(): number;
    isEmpty(tile: Vec2, endTile: Vec2): boolean;
}

/** 方格类的A*算法 */
export class Astar {
    /** 关闭列表的节点不会在用到 */
    private closeList: any = {};
    /** 从开放列表取估值最小的路径来行走 */
    private openList: AstarNode[] = [];

    private map: AstarMap = null;
    /** 4方向 */
    static SquarePositions: Readonly<Vec2[]> = [new Vec2(0, 1), new Vec2(0, -1), new Vec2(1, 0), new Vec2(-1, 0)];

    constructor(map: AstarMap = null) {
        this.setMap(map);
    }

    setMap(map: AstarMap) {
        this.map = map;
    }
    /**
     * 估值函数
     * @param beginPos 当前节点位置
     * @param endPos 终点位置
     */
    private estimateHValue(beginPos: Vec2, endPos: Vec2) {
        return Math.abs(beginPos.x - endPos.x) + Math.abs(beginPos.y - endPos.y);
    }

    private contain(list: AstarNode[], position: Vec2) {
        for (let i = 0; i < list.length; i++) {
            if (list[i].equals(position)) {
                return true;
            }
        }
        return false;
    }

    /** 获取地块唯一id **/
    private getUniqeIndex(position: Vec2) {
        return position.y * this.map.getColumn() + position.x;
    }

    /** 边界判断 */
    private inBoundary(position: Vec2) {
        return position.x >= 0 && position.x < this.map.getColumn() && position.y >= 0 && position.y < this.map.getRow();
    }

    /** 添加进关闭列表 */
    private addToClose(position: Vec2) {
        this.closeList[this.getUniqeIndex(position)] = true;
    }

    /** 插入进开放列表按f值排序 */
    private addToOpen(node: AstarNode) {
        let index = 0;
        for (let i = 0; i < this.openList.length; i++) {
            if (node.fValue < this.openList[i].fValue) {
                index = i;
                break;
            }
        }
        this.openList.splice(index, 0, node);
    }

    private isInCloseList(position: Vec2) {
        return this.closeList[this.getUniqeIndex(position)];
    }

    private addSquareNode(currentNode: AstarNode, endPos: Vec2) {
        if (currentNode) {
            //走过的节点放入关闭列表
            this.addToClose(currentNode.position);

            Astar.SquarePositions.forEach((position) => {
                let newPos = currentNode.add(position);
                if (this.inBoundary(newPos) && !this.isInCloseList(newPos)) {
                    //如果关闭列表和开放列表不包含新的节点，并且该位置可以走，加入开放列表，否则加入关闭列表
                    if (this.map.isEmpty(newPos, endPos) && !this.contain(this.openList, newPos)) {
                        this.addToOpen(new AstarNode(newPos, this.estimateHValue(newPos, endPos), currentNode));
                    } else {
                        this.addToClose(newPos);
                    }
                }
            });
        }
    }

    makePath(beginPos: Vec2, endPos: Vec2) {
        let hValue = this.estimateHValue(beginPos, endPos);
        if (hValue == 0) return;

        this.openList = [];
        this.closeList = {};

        //起点
        let node = new AstarNode(beginPos, hValue, null);
        this.openList.push(node);

        let last = null;
        while (this.openList.length > 0) {
            node = this.openList.shift();
            if (node.equals(endPos)) {
                last = node;
                break;
            }
            this.addSquareNode(node, endPos);
        }

        if (!last) return null;

        let path = [];

        // 判断父亲是不包含起点
        while (last && last.parent) {
            path.push(last.position);
            last = last.parent;
        }

        path.reverse();

        return path;
    }

    getPath(map: AstarMap, beginPos: Vec2, endPos: Vec2) {
        this.setMap(map);
        return this.makePath(beginPos, endPos);
    }
}

export let CommonAstar = new Astar();

```
## 代码对比分析
* 关闭列表的结构修改，之前lua采用的是数组结构，ts采用的是map结构，ts版本从关闭列表查找节点更快。
* ts版本加入开放列表的时候采用插入排序，因为对已经拍好序的数组，插入一个新的节点，只要O(n)的复杂度。
* ts版本可以选择四方向或者八方向的方格行走(代码还暂未添加八方向)。

## 总结
方格A*算法的核心原理基本不变，因为估值H相对简单。
