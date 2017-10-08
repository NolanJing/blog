---
layout:     post
title:      "图片加载中占位符与失败重连"
subtitle:   "Keynote: Javaascript img-loading-alt"
date:       2017-06-23
author:     "Nolan"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 算法
    - JavaScript
    - 数据结构
---
# 图片加载中占位符与失败重连
 
项目中遇到了一个需求，要求在图片加载时候显示loading图片，让用户看到加载中的状态。并且在图片载入失败时候完成点击重传。

* 思路是这样的，写一个公用函数，输入是每个图片的dom元素，然后对其进行设置占位符。



``` javascript
module.exports = {
    /**
     * 图片加载时，显示占位符
     * @param url 图片原始 url
     * @param imgElement 图片DOM元素
     */
    hasImgComplete: function(url, imgElement) {
        var val = url;
        var img = new Image();

        // 开始加载
        img.onload = function() {
            if (img.complete === true) {
                // 加载完毕
                imgElement.setAttribute('src', img.src);
            }
        };
        // 加载出错
        img.onerror = function() {
            // 点击重传
            imgElement.setAttribute(
                'src',
                'reload.gif'
            );
            imgElement.onclick = function() {
                this.setAttribute('src', val);
            };
        };

        img.src = val;
    },
    init: function(el) {
        if (el) {
            var imgList = el.querySelectorAll('img');
            for (var i = 0, len = imgList.length; i < len; i++) {
                var url = imgList[i].getAttribute('src');

                if (!imgList[i].getAttribute('data-setAlt')) {
                    // 设置loading
                    imgList[i].setAttribute('src', 'loading.gif');
                    // 打上setAlt标记
                    imgList[i].setAttribute('data-setAlt', true);
                } else {
                    return;
                }

                this.hasImgComplete(url, imgList[i]);
            }
        }
    },
};

```

*  然后在需要用到的组件中直接调用就好了。

``` javascript
 imgLoadingAlt.init(this.$el);
```