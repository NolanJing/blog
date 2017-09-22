---
layout:     post
title:      "图片懒加载、去抖、节流"
subtitle:   "Keynote: Lazyload Debounce Throttle"
date:       2017-09-15
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 算法
    - JavaScript
    - 页面优化
---
# 懒加载
 
对于图片较多的页面，使用懒加载可以大幅提高页面加载速度，提高用户体验。

对页面加载速度影响最大的就是图片，一张普通的图片可以达到几M的大小，而代码也许就只有几十KB。当页面图片很多时，页面的加载速度缓慢，几S钟内页面没有加载完成，也许会失去很多的用户。

所以，对于图片过多的页面，为了加速页面加载速度，所以很多时候我们需要将页面内未出现在可视区域内的图片先不做加载， 等到滚动到可视区域后再去加载。这样子对于页面加载性能上会有很大的提升，也提高了用户体验。

# 原理
将页面中的`img`标签`src`指向一张小图片或者`src`为空，然后定义`data-src`（这个属性可以自定义命名，我才用`data-src`）属性指向真实的图片。`src`指向一张默认的图片，否则当`src`为空时也会向服务器发送一次请求。可以指向loading的地址。

当载入页面时，先把可视区域内的`img`标签的`data-src`属性值负给`src`，然后监听滚动事件，把用户即将看到的图片加载。这样便实现了懒加载。


``` html
 <head>
     <meta charset="UTF-8">
     <title>Document</title>
     <style>
         img {
             display: block;
             margin-bottom: 50px;
             width: 400px;
             height: 400px;
         }
     </style>
 </head>
 <body>
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg" alt="">
     <img src="default.jpg" data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
     <img src="default.jpg" data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg" alt="">
 </body>

```


``` javascript
 <script>
     var num = document.getElementsByTagName('img').length;
     var img = document.getElementsByTagName("img");
     var n = 0; //存储图片加载到的位置，避免每次都从第一张图片开始遍历
     lazyload(); //页面载入完毕加载可是区域内的图片
     window.onscroll = lazyload;
     function lazyload() { //监听页面滚动事件
         var seeHeight = document.documentElement.clientHeight; //可见区域高度
         var scrollTop = document.documentElement.scrollTop || document.body.scrollTop; //滚动条距离顶部高度
         for (var i = n; i < num; i++) {
             if (img[i].offsetTop < seeHeight + scrollTop) {
                 if (img[i].getAttribute("src") == "default.jpg") {
                     img[i].src = img[i].getAttribute("data-src");
                 }
                 n = i + 1;
             }
         }
     }
 </script>

```

# 使用节流函数进行性能优化

如果直接将函数绑定在`scroll`事件上，当页面滚动时，函数会被高频触发，这非常影响浏览器的性能。

我想实现限制触发频率，来优化性能。

节流函数：只允许一个函数在N秒内执行一次。下面是一个简单的节流函数：


``` javascript
 // 简单的节流函数
 //fun 要执行的函数
 //delay 延迟
 //time  在time时间内必须执行一次
 function throttle(fun, delay, time) {
     var timeout,
         startTime = new Date();
     return function() {
         var context = this,
             args = arguments,
             curTime = new Date();
         clearTimeout(timeout);
         // 如果达到了规定的触发时间间隔，触发 handler
         if (curTime - startTime >= time) {
             fun.apply(context, args);
             startTime = curTime;
             // 没达到触发间隔，重新设定定时器
         } else {
             timeout = setTimeout(function(){
 	            fun.apply(context, args);
             }, delay);
         }
     };
 };
 // 实际想绑定在 scroll 事件上的 handler
 function lazyload(event) {}
 // 采用了节流函数
 window.addEventListener('scroll',throttle(lazyload,500,1000));

```
# 使用去抖函数进行性能优化

去抖相比较节流函数要稍微简单一点，去抖是让函数延迟执行，而节流比去抖多了一个在一定时间内必须要执行一次。

``` script

// debounce函数用来包裹我们的事件
function debounce(fn, delay) {
  // 持久化一个定时器 timer
  let timer = null;
  // 闭包函数可以访问 timer
  return function() {
    // 通过 'this' 和 'arguments'
    // 获得函数的作用域和参数
    let context = this;
    let args = arguments;
    // 如果事件被触发，清除 timer 并重新开始计时
    clearTimeout(timer);
    timer = setTimeout(function() {
      fn.apply(context, args);
    }, delay);
  }
}
// 实际想绑定在 scroll 事件上的 handler
function lazyload(event) {}
// 采用了去抖函数
window.addEventListener('scroll',throttle(lazyload,500));

```


