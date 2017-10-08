---
layout:     post
title:      "理解异步(一)——LazyMan（上）"
subtitle:   "Keynote: LazyMan Asynchronous Flow-Control"
date:       2017-10-3
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - JavaScript
    - 流程控制
---
# 前言
最近在看Promise的原理，仔细找了点资料发现这些流程控制类的问题都可以归为一大类模式，可以让我们深入理解异步是个什么东西？

我们可以从一个问题入手。如下面的的LazyMan，实现函数，在相关控制台打印出来结果和题目描述结果相同。

# LazyMan

``` 
 LazyMan('Hank')
 // Hi! This is Hank!
 
 
 LazyMan('Hank').eat(“dinner”)
 // Hi! This is Hank!
 // Eat dinner~
 
 
 LazyMan('Hank').sleep(10).eat(“dinner”)
 // Hi! This is Hank!
 // 等待1秒..
 // sleep 1s
 // Eat dinner~
```
分析思路: 先用`LazyMan('Hank')`返回一个对象，然后添加方法、链式调用、sleep函数则可以用setTimeout()模拟。好像很简单么、试一下
``` 
 ``` javascript
   // 以下代码不正确哦~
   function LazyMan(name) {
       return new __lazyman(name);
   }
  
   function __lazyman(name) {
       this.name = name;
       console.log('Hi '+ name);
       return this;
   }
   __lazyman.prototype = {
       eat: function(val){
           console.log('Eat ' + val);
           return this;
       },
       sleep: function (delay) {
           setTimeout(function () {
               console.log('Sleep ' + delay + 's')
           },delay*1000)
           return this;
       }
   }
 
   LazyMan('Nolan').sleep(2).eat('dinner');
   
   // Hi nolan
   // Eat dinner
   // Sleep 2s
 ```
我们可以看到虽然sleep()在eat()前被调用，但是Eat dinner还是在Sleep前被打印出来。这就是因为setTimeout是异步函数的原因了。

# 异步机制

对异步机制不了解的同学，建议看一下这篇文章 [阮一峰: JavaScript 运行机制详解：再谈Event Loop ](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

![event-loop](http://image.beekka.com/blog/2014/bg2014100802.png)

setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。
```
    sleep: function (delay) {
           setTimeout(function () {
               console.log('Sleep ' + delay + 's');           
           },delay*1000)           
           return this;
    }
    
```
上面代码setTimeOut中的回调函数其实是在执行完下面的 return this 以及后面的.eat方法之后再执行的。因为我们写的这些代码都在执行栈中。执行完，主线程才会去执行任务队列中回调函数`console.log('Sleep ' + delay + 's');`。                
               
# 解决办法
我们可以建立一个tasks数组，将回调函数存起来(tasks[0] = callback)。执行栈中的代码都执行结束，栈清空。但是这个tasks[]是不会消失的。等setTimeout发起回调的时候，再执行tasks[0]也就是之前存进去的回调函数就可以了
***`这个思想很重要！！！Promise和这个原理一毛一样！！`***

``` script

 function LazyMan(name) {
      return new __lazyman(name);
  }
 
  function __lazyman(name) {
      this.name = name;
      this.tasks = [];
      console.log('Hi '+ name + '~');
      let that = this;
      // 当执行栈所有代码执行结束后，启动tasks队列第一个回调任务。
      setTimeout(function () {
          that.next();
      },0);
      return this;
  }
  __lazyman.prototype = {
      eat: function(data){
          let callback2 =  function () {
              console.log('eat ' + data);
          };
          this.tasks.push(callback2);
          return this;
      },
      sleep: function (delay) {
          let that = this;
          console.log('sleep.... ');
          let callback1 = function () {
              setTimeout(function () {
                  console.log('weak up after ' + delay + 's')
                  that.next()
              },delay*1000)
          };
          this.tasks.push(callback1);
          return this;
      },
      next: function () {
         let fn = this.tasks.shift();
         fn && fn()
      }
  };
  LazyMan('Nolan').sleep(2).eat('dinner')
  
// Hi Nolan~
// sleep.... 等待 2s 后
// weak up after 2s
// eat dinner

```
完整代码如上图所示，代码的工作流程如下

- 1.利用工厂模式生成一个LazyMan对象。该对象具有name、tasks等属性，具有sleep、eat,next等方法。并在生成过程中控制台输出'Hi Nolan~'，返回该对象。
- 2.第一步返回的对象调用sleep方法，将callback1函数压入tasks队列中。现在该对象的tasks队列中已经有了一个callback1函数，该函数的功能是延时n秒后打印'weak up'，但是这个函数并不执行，返回该对象。
- 3.第二步返回的对象调用eat方法。将callback2函数压入tasks队列末尾中。该函数的功能是延时n秒后打印'eat dinner';现在该对象的tasks队列中保存有[callback1,callback2]两个回调函数。返回该函数。
- 4.执行栈中所有代码执行完毕。执行任务队列队首的函数（不是我们的tasks数组哦）也就是我们写在setTimeOut(fn,0)中的LazyMan.next();LazyMan.next()的功能是将tasks队列中队首的callback1函数弹出并执行。控制台输出'sleep.... 等待 2s 后'
- 5.延时2秒后，控制台输出'weak up after 2s
',此时再次运行LazyMan.next();将tasks队列中队首的callback2函数弹出并执行。控制台输出'Eat Dinner'


