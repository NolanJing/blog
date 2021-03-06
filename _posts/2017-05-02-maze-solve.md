---
layout:     post
title:      "JS手撸数据结构系列 (五) ——图的遍历与迷宫求解"
subtitle:   "Keynote: Javascript Maze Graph"
date:       2017-05-02
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 前端开发
    - JavaScript
    - 数据结构
---


## 迷宫问题求解
在[上一篇](http://blog.csdn.net/scargtt/article/details/71078275 "optional title")文章中实现了生成随机迷宫的算法，这一节当然要实现迷宫求解啦，先上效果图。[源代码及在线预览](https://nolanjing.github.io/maze "github")

<img src="http://img.blog.csdn.net/20170502232135778?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="30%"  alt="迷宫最短路径csdn."/>
 
## 广度优先遍历（BFS）
广度优先搜索算法（Breadth-First-Search），又译作宽度优先搜索，或横向优先搜索，简称BFS，是一种图形搜索算法。使用队列实现。简单的说，BFS是从根节点开始，沿着树的宽度遍历树的节点。如果所有节点均被访问，则算法中止。 
我们使用类似的思想，先找到所有与起点距离为1的节点，再找到所有距离为2的点.....直到发现终点，也就是迷宫出口。我们在遍历时，记录每个节点的父节点，最后的迷宫路径就可以从迷宫终点一路沿着父节点找到起点啦。

```javascript
// 遍历节点，寻找路径
Maze.prototype.findPath = function () {
    // 先将所有格子的isVisited 置为 false
    for (let i = this.mazeDataArray.length - 1; i >= 0; i--) {
        for (let j = this.mazeDataArray[i].length - 1; j >= 0; j--) {
            this.mazeDataArray[i][j].isVisited = false;
        }
    }
    // 路径数组
    this.path = [];
    let node = this.mazeDataArray[this.start[0]][this.start[1]]; // 迷宫的出口
    // let pathTree = node;
    let queue = []; // 辅助队列
    node.isVisited = true;
    queue.unshift(node); // 入队
    while (queue.length) { // 队列非空
        let firstItem = queue.shift(); // 队首元素 出队
        firstItem.neighbor = [];
        if (this.mazeDataArray[node.i - 1] && this.mazeDataArray[node.i - 1][node.j].value) {// 上
            if (!this.mazeDataArray[node.i - 1][node.j].isVisited) {
                firstItem.neighbor.push(this.mazeDataArray[node.i - 1][node.j]);
                // 记录前置节点的坐标，以便最后打印出路径
                this.mazeDataArray[node.i - 1][node.j].pre = [firstItem.i, firstItem.j];
            }

        }
        if (this.mazeDataArray[node.i][node.j + 1] && this.mazeDataArray[node.i][node.j + 1].value) {// 右
            if (!this.mazeDataArray[node.i][node.j + 1].isVisited) {
                firstItem.neighbor.push(this.mazeDataArray[node.i][node.j + 1]);
                this.mazeDataArray[node.i][node.j + 1].pre = [firstItem.i, firstItem.j];
            }

        }
        if (this.mazeDataArray[node.i + 1] && this.mazeDataArray[node.i + 1][node.j].value) {// 下
            if (!this.mazeDataArray[node.i + 1][node.j].isVisited) {
                firstItem.neighbor.push(this.mazeDataArray[node.i + 1][node.j]);
                this.mazeDataArray[node.i + 1][node.j].pre = [firstItem.i, firstItem.j];
            }

        }
        if (this.mazeDataArray[node.i][node.j - 1] && this.mazeDataArray[node.i][node.j - 1].value) {// 左
            if (!this.mazeDataArray[node.i][node.j - 1].isVisited) {
                firstItem.neighbor.push(this.mazeDataArray[node.i][node.j - 1]);
                this.mazeDataArray[node.i][node.j - 1].pre = [firstItem.i, firstItem.j];
            }
        }
        // 遍历邻居节点
        for (neighborNode of firstItem.neighbor) {
            if (!neighborNode.isVisited) {
                neighborNode.isVisited = true;
                queue.push(neighborNode);
            }

        }
        node = queue[0];
        if (node && node.i === this.end[0] && node.j === this.end[1]) {
            this.pre = node;
            break;
        }
    }// while

    let item = this.pre;

    while (item.pre) {
        this.path.unshift([item.pre[0], item.pre[1]]);
        // 生成路径数组，path[0]为起点坐标
        item = this.mazeDataArray[item.pre[0]][item.pre[1]];
    }
};
```

## 制作动画

```html
<!--HTML-->
<div id="maze">
    <span>a Maze Generated by JS.</span>
    <table border="1" class="t">
    </table>
    <button onclick="reset()">reset</button>
</div>
```

```javascript
// js
// 操作DOM，以表格的形式绘制迷宫。
Maze.prototype.drawDom = function () {

    for (let i = 0, len = this.mazeDataArray.length; i < len; i++) {
        let tr = document.createElement("tr");
        document.querySelector('table').appendChild(tr);
        for (let j = 0, len = this.mazeDataArray[i].length; j < len; j++) {
            let td = document.createElement("td");
            // start
            if(i===this.start[0] && j===this.start[1] ){
                td.setAttribute("class", "startNode");
                td.innerHTML= 's';
            }
            // end
            if(i===this.end[0] && j===this.end[1] ){
                td.setAttribute("class", "endNode");
                td.innerHTML= 'e';
            }
            // wall
            if (!this.mazeDataArray[i][j].value){
                td.setAttribute("class", "wall");
            }

            tr.appendChild(td);
        }
    }
};
// 延时动画，默认为10ms
Maze.prototype.Animation = function () {
    let count = 0;

    function animation(pathArray) {
        if (count < pathArray.length) {
            let trArr = document.getElementsByTagName('tr');
            let tdArr = trArr[pathArray[count][0]].getElementsByTagName('td');
            let cell = tdArr[pathArray[count][1]];
            cell.setAttribute("class", "path");
            count++
        } else {
            clearInterval(temp);
            console.log('finded the path')
        }
    }
    let temp = setInterval(animation, this.speed, this.path);
};
```

```CSS
<!--CSS-->
        #maze {
            margin: 0 auto;
            margin-top: 100px;
            text-align: center;
        }
        #maze table {
            margin: 0 auto;
            margin-top: 10px;
        }
        #maze span {
            font-family: "Bodoni MT";
            font-size: 20px;
            font-weight: 700;
            color: brown;
        }
        td {
            width: 20px;
            height: 20px;
            text-align: center;
            color: white;
        }
       .startNode {
            background-color: aqua;
        }
        .endNode {
            background-color:red;
        }
        .wall {
            background-color: #777;
        }
        .path {
            background-color: aqua;
        }
```

 

> ***scargtt***

