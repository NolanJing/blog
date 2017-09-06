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

# 递归实现
递归方法的全排列思想挺简单的，就是从第一个字符起，将它与其后面的每个字符
进行交换。每交换一次就算一种新的字符串（不包括重复的字符）。这些字符串的集合就是全排列了。

这句话听起来是有点摸不到头脑，但是回想一下我们自己手动求全排列的过程, 可以分为下面三种情况：

  * 第一位为 a , 则后面就是bc的全排列，一种 a 后接 bc, 一种 a 后接 cb

  * 第一位为 b , 则后面就是ac的全排列，一种 b 后接 ac, 一种 b 后接 ca

  * 第一位为 c , 则后面就是ab的全排列，一种 b 后接 ab, 一种 b 后接 ba

我们的选第一位这个操作，就相当于把a字符和它后面的字符（包含它本身）交换。
  * abc 中 a 和 a 交换  => abc （第一位为a）

  * abc 中 a 和 b 交换  => bac （第一位为c）

  * abc 中 a 和 c 交换  => cba （第一位为c）

第一位已经确定为a了，那么怎样选第二位呢? 同样的思路:

  * bc 中 b 和 b 交换  => bc
  * bc 中 b 和 b 交换  => cb

现在第一位已经确定为 a 了，第二位确定为 b ，只剩下个 c ，不用再进行挑选字符了，直接输出abc就得出一种结果。

显然，上面的算法是行得通的，敲代码！


``` javascript
/*
全排列（递归交换）算法
1、将第一个位置分别放置各个不同的元素；
2、对剩余的位置进行全排列（递归）；
3、递归出口为只对一个元素进行全排列。
*/

/**
* 将数组第i位和第j位交换
**/
function swap(arr,i,j) {
    if(i!=j) {
        var temp=arr[i];
        arr[i]=arr[j];
        arr[j]=temp;
    }
}

/**
* @index 就是选第index位的字符
**/
function allRange(arr, index) {
    if(index == arr.length-1) {
         console.log(arr)
    } else {
        for(var i = index; i < arr.length; i++) {
            swap(arr, i, index); // 把 index 位的字符与其后面的字符依次交换
            allRange(arr, index + 1) // 对后面未选中的字符做全排列
            swap(arr, i, index); //关键！！！
        }
    }
};

allRange(['a','b','c'], 0)

/*
注：可以看到，在对后面未选中的字符做全排列之后，又重新交换了一次！
    因为递归过程中 swap()函数是会改变数组内元素顺序的。我们的本意是想让 a，b，c分别为头部一次
    如果我们把第二次交换的代码去掉，等递归终止回溯回来的时候，整个字符串已经顺序打乱了
    我们需要再次交换，回归初试状态。
*/

```

# 非递归实现

理解并完成上面的递归实现浪费了两个小时，发现上面的思路还不是很直观，有那么一丢丢绕。然后我发现还真有一个相对好理解的思路。
比如几个人排队，除了根据位置选人。也可以根据人选位置啊！拿abc三人排队举例:
  * a 来的最早 , 则队伍里只有一人 'a'

  * b 过来插队 , 我们把原来的队列表示为 '_ a _'  下划线代表空位置，则有两种插队方式 ab 和 ba

  * 现在 c 过来插队 , 假设之前的队伍为 '_a_b_', 一共有三种插队方式。

 根据这个插队的思路可以写出下面的代码:


``` javascript
/**
* @queueSet 所有可能队列的集合 例如[[1]],[[1,2],[2,1]]
* @key 待插入的值
**/
function insert(queueSet, key) {
    var newQueueSet = [] // 新的队列集合
    queueSet.forEach(queue => {
        for(var j = 0; j < queue.length + 1; j++){
           // 插队后的新队列
           let newQueue =  queue.slice(0,j).concat(key, queue.slice(j));
           newQueueSet.push(newQueue)
        }
    })
    return newQueueSet ;
}

function permutation(arr){
     var queueSet = [arr[0]];
     for (var i = 1; i < arr.length; i++) {
         queueSet = insert(queueSet,arr[i])
     }
     return queueSet
}

console.log(permutation(['a','b','c','d']))
```

