---
layout:     post
title:      "JS手撸数据结构系列 (三) ——子序列、幂集与递归"
subtitle:   "Keynote: JavaScript Subsequence Powerset"
date:       2017-04-24
author:     "Nolan"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - JavaScript
    - 数据结构
    - 算法
---
  
## 穷举所有子序列
当时的情况是这样的，本来想用最蠢的方法写LCS（最长公共子序列），穷举A、B的所有子序列，然后循环套循环逐一比较......

不过.....穷举所有子序列.....貌似也不是那么一下就能解决.....

人脑遍历的结果如下所示：一共 $2^4 =16$ 种结果，归纳成公式就是$ 2^{ str.length} $。现在我们需要解决的问题就变成了如何穷举出所有的子序列。

```javascript
 // 一个字符串，也就是一个序列
 str='abcd';

 subSeq0 = [' ']； // 没有元素，也就是集合为空，这和幂集的定义有关。

 subSeq1 = ['a', 'b', 'c', 'd']； // 一个元素的情况

 subSeq2 = ['ab ', 'bc', 'ac']； // 两个元素的情况

 subSeq3 = ['abc', 'bcd', 'acd']； // 三个元素的情况

 subSeq4 = ['abcd']； // 四个元素 
  
```

## 幂集
通过上面的分析，发现穷举子序列这个问题等价于求一个元素的幂集。

- **幂集定义**：集合A的幂集就是所有A的子集所组成的集合。比如集合[1,2,3],它的幂集B就是[{1}，{2}，{3}，{1,2}，{2,3}，{1,3}，{空集]

##  状态二叉树
可以设想这么一个过程，幂集中的每个元素都是一个集合，它或者是空集，或含有集合A中的一个元素，或两个元素.....或者全部n个元素。
另一个角度，从 $A$ 中每个元素来看，它只有两种状态['属于幂集元素集', '不属于幂集元素集']，则求幂集所有元素的过程可以看成是依次对集合 $A$ 中的元素进行 '取' 或 '舍' 的过程，并可用下图的二叉树来表示。最下层的所有叶子节点就是所求幂集的所有元素。

![二叉树](http://img.blog.csdn.net/20170424114649018?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


 根据这颗状态树，可以写出如下代码： 
``` javascript
var arr = [1, 2, 3];
 

 function getPowerSet(i, listA) {

   let listB = [];

   function recurse(i, listA) {

       if(i > listA.length - 1){
          //输出当前Ｂ值，即幂集里的一个元素
          console.log('set: ' + listB);
       } else {
         var x = listA[i];
         listB.push(x);
         recurse(i+1, listA);
         listB.pop(x);
         recurse(i+1, listA);
       }
    }
      
  recurse(i, listA);

 }
 
getPowerSet(0, arr)


```
##  重构一下下
上面的是数据结构书上的内容，理解起来很别扭。我试着重构了一下下让代码更简单。
把上述的树存在一个数组中

| arr[0] = ' '  | 空集 |
| ------------- |-----:|
| arr[1] = 'a', 代表'取a' | arr[2] = ' ', 代表'舍a' |
| arr[3] = 'ab', 代表 '取b', arr[4] = 'a', 代表 '舍b' | arr[5] = 'b', 代表 '取b'，arr[6] = 'a',  代表 '舍b' |

然后我们发现，所有表示舍弃的状态，因为你没有元素在末尾追加新元素，都和以前的重复了。
所以我们只保留 '取'的这一种分支。

- 新算法思路：
1.  首先有一个空数组list[];
 2. 遍历数组list，在list中每一个元素末尾追加新字符，然后把新生成的元素push到list的末尾。生成新的list
 3. 重复第二步，直到没有新字符需要判断。

``` javascript
let arr = ['a', 'b', 'c', 'd'];
let list = [''];

for(var i = 0, len = arr.length; i<len ; i++ ){
  list.forEach(x => {
    list.push(x + arr[i]);
  });
}

console.log(list.sort()); // 排序一下，方便查看
```
##  运行结果
![powerSet](http://img.blog.csdn.net/20170424124204958?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>***《 论被自己写的代码美哭是什么感觉 》***

