---
layout:     post
title:      "JS手撸数据结构系列 (六) ——动态规划与最长公共子序列"
subtitle:   "Keynote: Javascript Lcs Dp"
date:       2017-05-05
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 算法
    - JavaScript
    - 数据结构
---
# LCS (最长公共子序列)
 
* 定义：一个数列 S，如果分别是两个或多个已知数列的子序列，且是所有符合此条件序列中最长的，则 S 称为已知序列的最长公共子序列。

* 例如：输入两个字符串BDCABA和ABCBDAB，字符串BCBA和BDAB都是是它们的最长公共子序列，则输出它们的长度4，并打印任意一个子序列。

# 动态规划
##### 一、基本概念
动态规划过程是：每次决策依赖于当前状态，又随即引起状态的转移。一个决策序列就是在变化的状态中产生出来的，所以，这种多阶段最优化决策解决问题的过程就称为动态规划。

##### 二、基本思想与策略
* 基本思想与分治法类似，也是将待求解的问题分解为若干个子问题（阶段），按顺序求解子阶段，前一子问题的解，为后一子问题的求解提供了有用的信息。在求解任一子问题时，列出各种可能的局部解，通过决策保留那些有可能达到最优的局部解，丢弃其他局部解。依次解决各子问题，最后一个子问题就是初始问题的解。
* 由于动态规划解决的问题多数有重叠子问题这个特点，为减少重复计算，对每一个子问题只解一次，将其不同阶段的不同状态保存在一个二维数组中。
与分治法最大的差别是：适合于用动态规划法求解的问题，经分解后得到的子问题往往不是互相独立的（即下一个子阶段的求解是建立在上一个子阶段的解的基础上，进行进一步的求解）。

* 理论基础:

	设有：

	Xi = [ x1，...，xi ]  即X序列的前i个字符 (1 ≤ i ≤ m)（前缀）

	Yj = [ y1，...，yj ]  即Y序列的前j个字符 (1 ≤ j ≤ n)（前缀）

	 
	假定 Z = [z1，...，zk] ∈ LCS(X , Y)。
	即 Z 为 Xi 和 Yj 的最长公共子序列

	若 Zm = Zn（X序列与Y序列最后一个字符相同），则：该字符必是x与y的任一最长公共子序列Z（设长度为k）的最后一个字符。
	若 Zm ≠ Zn，则 Zk 要么属于 Y[m-1]，要么属于 Y[n-1]
	设 Xi 和 Yj 最长公共子序列的长度为C[ i , j ]，则有（公式1）：
    ![img](/blog/img/in-post/lcs-formula.gif) 
	那么，接下来的目标就是，想办法判断子序列的最后一个字符是只属于X序列还是只属于Y序列，还是既属于X序列又属于Y序列。
	而当c[i,j-1] > c[i-1,j]的时候，最长子序列的最后一个字符存在于Xi中，反之，存在于Yj中。
	 
# 算法流程
* `str1 = "SDFAWEFSDFXZ"`

  `str2 = "DAFWAFSFAEF" `  

  求连个序列的最长公共子序列

* 画出初始表格
如下图所示，`str1.length+2` 行 `str2.length+2` 列的表格，长度都比原始字符串的长度增加了2，一行为了看到清楚，把字符串列举出来
1行为全0，这是因为存在c[i, j] = c[i - 1, j - 1] + 1 这样的运算，如果数组以0为开头，会出现下标为－1的情况，严格来讲，前两行和前两列没有实际意义：和具体算法没关系。


![img](/blog/img/in-post/lcs-init.png)
* 根据状态转移方程填表

	
![img](/blog/img/in-post/lcs-computed.png)
	
	
* 由最后一点回溯到起点，求路径
![img](/blog/img/in-post/lcs-path.png)
	
* 绘制动画

![img](/blog/img/in-post/lcs-common.png)

# 程序代码

* 把LCS算法和绘制DOM动画的代码写在一起了....有些丑  ==！

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>blog</title>
    <style>
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

        .path {
            background-color: aqua;

        }

        .common {
            background-color: red;
        }
        .node {
            background-color: crimson;
        }

        .wall {
            background-color: #777;
        }
    </style>
</head>
<body>
<div id="maze">
    <span>The Longest Common Subsequence.</span>
    <table border="1" class="t">
    </table>
</div>

<script>
     let str2 = "DAFWAFSFAEF";
     let str1 = "SDFAWEFSDFXZ";

    /*let str2 = "GCCCTAGCG";
    let str1 = "GCGCAATG";*/
    function genTable(x, y) {
        let xlen = x.length;
        let ylen = y.length;


        let lengthTable = [];
        // 基础表格
        for (let i = 0; i < xlen + 2; i++) {
            lengthTable[i]=[];
            for (let j = 0; j < ylen + 2; j++) {
                if(i==0){
                    if(y[j-2]){
                        lengthTable[i][j]={value:y[j-2]}
                    }else {
                        lengthTable[i][j]={value:'-'}
                    }
                }else{
                    if(j==0){
                        if(x[i-2]){
                            lengthTable[i][j]={value:x[i-2]}
                        }else {
                            lengthTable[i][j]={value:'-'}
                        }
                    }else{
                        lengthTable[i][j]={value:0}
                    }

                }

            }
        }
        // 填表
         for (let i = 2, ilen = lengthTable.length; i < ilen; i++) {
            for (let j = 2, jlen = lengthTable[i].length; j < jlen; j++) {
                if (x[i - 2] == y[j - 2]) {
                    lengthTable[i][j] = {
                        value: lengthTable[i - 1][j - 1].value + 1,
                        pre: [i - 1, j - 1],
                        index: [i, j]
                    }
                } else {
                    lengthTable[i][j] = lengthTable[i][j - 1].value
                    >= lengthTable[i - 1][j].value
                        ? {
                            value: lengthTable[i][j - 1].value,
                            pre: [i, j - 1],
                            index: [i, j]
                        }
                        : {
                            value: lengthTable[i - 1][j].value,
                            pre: [i - 1, j],
                            index: [i, j]
                        };
                }

            }
        }

       let node = lengthTable[lengthTable.length - 1][lengthTable[lengthTable.length - 1].length - 1];
       let path =[];
       let common =[];
       // 找对角线 生成路径
         while (node.pre) {
            if (node.value !== lengthTable[node.pre[0]][node.pre[1]].value) {
                common.push(node.index);
            }
            path.push(node.index);
            node = lengthTable[node.pre[0]][node.pre[1]];
        }

        return {
            lengthTable:lengthTable,
            common:common,
            path:path
        }
    }

    let res = genTable(str1, str2);
    // 画基础表格
    for (let i = 0, len = res.lengthTable.length; i < len; i++) {
        let tr = document.createElement("tr");
        document.querySelector('table').appendChild(tr);
        for (let j = 0, len = res.lengthTable[i].length; j < len; j++) {
            let td = document.createElement("td");
            // start
            td.innerHTML = res.lengthTable[i][j].value;
            td.setAttribute("class", "wall");
            tr.appendChild(td);
        }
    }
    // 画路径
    let table = document.getElementsByTagName('table');
    res.path.forEach(pathNode=>{
        table[0].rows[pathNode[0]].cells[pathNode[1]].setAttribute("class", "path");
    });
    res.common.forEach(commonNode=>{
        table[0].rows[commonNode[0]].cells[commonNode[1]].setAttribute("class", "node");
        table[0].rows[0].cells[commonNode[1]].setAttribute("class", "common");
        table[0].rows[commonNode[0]].cells[0].setAttribute("class", "common");
    });
    console.log(table[0].rows[0].cells[0])
</script>


</body>

</html>

```