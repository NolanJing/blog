---
layout:     post
title:      "理解异步(二)——等待者模式"
subtitle:   "Keynote: waiter pattern"
date:       2017-10-3
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - JavaScript
    - waiter pattern
---
# 前言
前面我们用到的都是一个异步操作,即在执行完该操作后执行回调函数，如下面代码所示，在异步ajax请求回数据后

``` 
 var ajax = $.ajax({
     url: '/getStudents',
     success: function () {
         console.log('正在处理获得的学生数据')
     },
     error: function () {
         console.log('获得的学生数据失败')
     }
 })
 
 console.log(ajax) // 返回一个 XHR 对象
```
上面是一段最常见的$.ajax的代码，使用万恶的callback方式，缺点就是容易引起回调地域。
 
# 等待者模式
```
   var Waiter = function() {
       var dfd = [], //等待对象容器
           doneArr = [], //成功回调容器
           failArr = [], //失败回调容器
           slice = Array.prototype.slice,
           that = this;
       //监控对象类
       var Promise = function() {
           //监控对象是否解决成功状态
           this.resolved = false;
           //监控对象是否解决失败状态
           this.rejected = false;
       }
       Promise.prototype = {
           //解决成功
           resolve: function() {
               //设置当前监控状态是成功
               this.resolved = true;
               if(!dfd.length) return;
               console.log("进入 resolve");
               //对象监控对象遍历如果任一个对象没有解决或者失败就返回
               for(var i = dfd.length - 1; i >= 0; i--) {
                   if(dfd[i] && !dfd[i].resolved || dfd[i].rejected) {
                       return;
                   }
                   dfd.splice(i, 1);
               }
               _exec(doneArr)
           },
           //解决失败
           reject: function() {
               //设置当前监控状态是失败
               this.rejected = true;
               //没有监控对象取消
               if(!dfd.length) return;
               //清楚监控对象
               dfd.splice(0)
               _exec(failArr)
           }
       }
       that.Deferred = function() {
           return new Promise();
       };
       //回调执行方法
       function _exec(arr) {
           var i = 0,
               len = arr.length;
           for(; i < len; i++) {
               try {
                   arr[i] && arr[i]();
               } catch(e) {}
           }
       };
       //监控异步方法参数：监控对象
       that.when = function() {
           //设置监控对象
           console.dir(arguments)
           dfd = slice.call(arguments);
           var i = dfd.length;
           //向前遍历监控对象，最后一个监控对象索引值length-1
           var i = i -1
           for(i; i >= 0; i--) {
               //不存在监控对象，监控对象已经解决，监控对象失败
               if(!dfd[i] || dfd[i].resolved || dfd[i].rejected || !dfd[i] instanceof Promise) {
                   dfd.splice(i, 1)
               }
           }
           //返回等待者对象
           return that;
       };
       //解决成功回调函数添加方法
       that.done = function() {
           //向成功毁掉函数容器中添加回调方法
           doneArr = doneArr.concat(slice.call(arguments));
           return that;
       };
       //解决失败回调函数添加方法
       that.fail = function() {
           //向失败回调函数中添加方法
           failArr = failArr.concat(slice.call(arguments));
           return that;
       };
   }
   
   
   
   var waiter = new Waiter();//创建一个等待者实例


   var first = function() {
       var dtd = waiter.Deferred();
       setTimeout(function() {
           dtd.resolve();
       }, 3000)
       console.log(dtd)
       return dtd;//返回监听这对象
   }()

   console.log(first)
   var second = function() {//第二个对象
       var dtd = waiter.Deferred();
       setTimeout(function() {
           dtd.resolve();
       }, 5000)
       return dtd;
   }()
   waiter.when(first, second)
   .done(function() {
       console.log("success")
   }, function() {
       console.log("success again")
   })
   .fail(function() {
       console.log("fail")
   })
 
```
 

