---
layout:     post
title:      "JS手撸数据结构系列 (四) ——Prim算法与迷宫生成"
subtitle:   "Keynote: Javascript Maze Prim"
date:       2017-04-27
author:     "Nolan"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 前端开发
    - JavaScript
    - 数据结构
---




## 迷宫问题

*一个100*100的方格的迷宫，障碍未知，只知道起点和终点，以第一视角进入，求最短距离路径。*

这是当时腾讯二面的面试官给我留的题目，当时只要求写出了BFS求最短路径的算法，那么就会很自然的想到如何生成迷宫呢？
    迷宫可以看成是一个图，也可以看成是一个二维数组，其中数组元素的值为1，代表可以走通，代表路，其中数组元素的值为0的话，代表不能走通，代表墙壁。要想生成随机迷宫，难点就在如何确保在墙壁和道路随机的情况下，能保证从起点到终点的路线是连通的。就算碰巧（概率应该很低了）能保证存在路线，但是这样的迷宫也好丑，障碍都是离散的.....

##Prim算法

 这个问题困扰了我好久，网上的资料一般关于如何寻找迷宫路径的算法比较多，直到有一天看到数据结构中的图的最小生成树一章，可以发现迷宫问题就是一种最小生成树。如下图所示，左图代表起始状态，白色路径表示所有值为1的点，我们要做的就是制造一颗连续生长的树，最后把所有值为1点的点都添加到这颗树中， 只不过书上的Prim算法每步选取新节点的原则是最小边，我们改成随机选取就好了。

 ![prim](http://img.blog.csdn.net/20170501233751883?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)           =>     ![prim](http://img.blog.csdn.net/20170502100417524?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 
 
##迷宫算法实现
（1）如上图图左所示为一个7x7的迷宫，先假设迷宫中所有的通路都是完全封闭的，白色的格子表示可以通过，黑色的格子表示墙壁不能通过。
（2）随机选择一个白色的格子作为当前正在访问的格子，同时把该格子放入一个已经访问的列表中。
（3）循环以下操作，直到所有的格子都被访问到。
　　1.得到当前访问格子的四周（上下左右）的格子，在这些格子中随机选择一个没有在访问列表中的格子，如果找到，则把该格子和当前访问的格子中间的墙打通(置为0)，把该格子作为当前访问的格子，并放入访问列表。
　　2.如果周围所有的格子都已经访问过，则从已访问的列表中，随机选取一个作为当前访问的格子。

通过以上的迷宫生成算法，我们就可以生成一个路径分布很自然随机迷宫 [源代码与在线预览](https://nolanjing.github.io/maze/"optional title"  )
```javascript
  function Maze(col, row, start, end) {
        this.col = col;
        this.row = row;
        this.start = start;
        this.end = end;
    }
    // 向下取整，生成随机数
    Maze.prototype.random = function (k) {
        return Math.floor(Math.random() * k);
    };


    Maze.prototype.generate = function () {
        // 生成 2R+1 行 2R+1 列数组
        this.mazeDataArray = [];
        for (let i = 0; i < 2 * this.col + 1; i++) {
            let arr = [];
            for (let j = 0; j < 2 * this.row + 1; j++) {
                // 设置墙和初始格子
                if (i % 2 == 0 || j % 2 == 0) {
                    arr.push({
                        value: 0,
                        x: j,
                        y: i
                    });
                } else {
                    arr.push({
                        value: 1,
                        isVisited: false,
                        x: j,
                        y: i
                    });
                }
            }
            this.mazeDataArray[i] = arr;
        }
        // 随机选择一点作为 currentNode
        let currentNode = this.mazeDataArray[2 * this.random(this.row) + 1][2 * this.random(this.col) + 1];
        currentNode.isVisited = true;
        // 访问过的节点列表
        let visitedList = [];
        visitedList.push(currentNode);
        // 循环以下操作，直到所有的格子都被访问到。
        while (currentNode.isVisited) {
            // 得到当前访问格子的四周（上下左右）的格子
            let upNode = this.mazeDataArray[currentNode.y - 2] ? this.mazeDataArray[currentNode.y - 2][currentNode.x] : {isVisited: true};
            let rightNode = this.mazeDataArray[currentNode.x + 2] ? this.mazeDataArray[currentNode.y][currentNode.x + 2] : {isVisited: true};
            let downNode = this.mazeDataArray[currentNode.y + 2] ? this.mazeDataArray[currentNode.y + 2][currentNode.x] : {isVisited: true};
            let leftNode = this.mazeDataArray[currentNode.x - 2] ? this.mazeDataArray[currentNode.y][currentNode.x - 2] : {isVisited: true};

            let neighborArray = [];
            if (!upNode.isVisited) {
                neighborArray.push(upNode);
            }
            if (!rightNode.isVisited) {
                neighborArray.push(rightNode);
            }
            if (!downNode.isVisited) {
                neighborArray.push(downNode);
            }
            if (!leftNode.isVisited) {
                neighborArray.push(leftNode);
            }
            // 在这些格子中随机选择一个没有在访问列表中的格子，
            // 如果找到，则把该格子和当前访问的格子中间的墙打通(置为0)，
            if (neighborArray.length !== 0) { // 如果找到
                let neighborNode = neighborArray[this.random(neighborArray.length)];
                this.mazeDataArray[(neighborNode.y + currentNode.y) / 2][(neighborNode.x + currentNode.x) / 2].value = 1;
                neighborNode.isVisited = true;
                visitedList.push(neighborNode);
                currentNode = neighborNode;
            } else {
                // 把该格子作为当前访问的格子，并放入访问列表。
                // 如果周围所有的格子都已经访问过，则从已访问的列表中，随机选取一个作为当前访问的格子。
                currentNode = visitedList[this.random(visitedList.length)];
                if (!currentNode) {
                    // visitedList为空时 跳出循环
                    break;
                }
                currentNode.isVisited = true;
                // 从 visitedList 中删除随机出来的当前节点
                let tempArr = [];
                visitedList.forEach(item => {
                    if (item !== currentNode) {
                        tempArr.push(item);
                    }
                });
                visitedList = tempArr;
            }
        }
        this.mazeDataArray[this.start[0]][this.start[1]]={
            x:this.start[0],
            y:this.start[1],
            value:1
        };
        this.mazeDataArray[this.end[0]][this.end[1]]={
            x:this.end[0],
            y:this.end[1],
            value:1};
    };


    let maze = new Maze(3, 3, [1, 0], [5, 6]);
    maze.generate();
```


> ***I know H.T.M.L （How to meet ladies）***
>                             --***Silicon Valley***  
>  《硅谷》第四季 强烈推荐


