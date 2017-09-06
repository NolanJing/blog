---
layout:     post
title:      "再谈递归之全排列"
subtitle:   "Keynote: Javaascript Permutation Recursion"
date:       2017-09-1
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 算法
    - JavaScript
    - 数据结构
---
# 全排列
 
花了一晚上的时间苦思冥想，终于是彻底理解了字符串全排列了。自己还是太渣了.....

所谓全排列，就是打印出字符串中所有字符的所有排列。例如输入字符串abc，则打印出 a、b、c 所能排列出来的所有字符串 abc、acb、bac、bca、cab 和 cba。根据排列组合的数学公式也可知道给定 n 个字符，全排列一共 n! 项。

# 全排列的递归实现
递归方法的全排列思想挺简单的，就是从第一个字符起，将它与其后面的每个字符
  进行交换。每交换一次就算一种新的字符串（不包括重复的字符）。这些字符串的集合就是全排列了。

这句话听起来是有点摸不到头脑，但是回想一下我们自己手动求全排列的过程,可以分为下面三种情况：
  1. 第一位为 a , 则后面有两种情况，一种 a 后接 bc, 一种 a 后接 cb
  2. 第一位为 b , 则后面有两种情况，一种 b 后接 ac, 一种 b 后接 ca
  2. 第一位为 c , 则后面有两种情况，一种 b 后接 ac, 一种 b 后接 ca





``` javascript
/*
全排列（递归交换）算法
1、将第一个位置分别放置各个不同的元素；
2、对剩余的位置进行全排列（递归）；
3、递归出口为只对一个元素进行全排列。
*/

// 交换arr数组的第i位和第j位
function swap(arr,i,j) {
    if(i!=j) {
        var temp=arr[i];
        arr[i]=arr[j];
        arr[j]=temp;
    }
}

function allRange(arr, index) { // 为第index个位置选择元素
    if(index == arr.length-1) { // 交换到最后一位
         console.log(arr)
    } else {
        for(var i = index; i < arr.length; i++) {
            swap(arr, i, index, 1);
            allRange(arr, index + 1)
            swap(arr, i, index, 0);
        }
    }
};

```

*  然后在需要用到的组件中直接调用就好了。

``` javascript
 imgLoadingAlt.init(this.$el);
```