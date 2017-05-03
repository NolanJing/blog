---
layout:     post
title:      "JS手撸数据结构系列 (二) —— 树的遍历"
subtitle:   ""
date:       2017-04-23
author:     "Nolan"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Javascript
    - 数据结构
    - 算法
---

## 构造二叉树
 
![二叉树](http://images.cnitblog.com/blog/441348/201301/07113506-aa51aed49a8849be9f751f37647b0714.jpg)

  
``` javascript
let tree = {
 value: 'G',
 left: {
  value: 'D',
  left: {
   value: 'A'
  },
  right: {
  	value: 'F',
  	left:{
  		value:'E'
  	}
  }
 },
 right: {
  value: 'M',
  left: {
   value: 'H',
  },
  right: {
   value: 'Z'
  }
 }
}

```
##前序遍历

      1.访问根节点 
      2.前序遍历左子树 
      3.前序遍历右子树 

上图的二叉树遍历后 :  GDAFEMHZ



```
function prePrint(tree) {
// 输出字符串
	var str ='';
	function getChild(tree) {
		if(!tree){
		// 递归终止
            return;
		} else {
		   str =str + '->' + tree.value;
		   // 如果想改成后续遍历
		   // 只要把下面三行语句顺序换一下就可以
           getChild(tree.left) 
           getChild(tree.right)
           return str
		}	 		 	
	 } 
	 return getChild(tree);
}

function postPrint(tree){
	var str ='';
	function getChild(tree) {
		if(!tree){           
           //  return 
		} else {
			 getChild(tree.left); 
             getChild(tree.right);
			 str = str + '->' + tree.value;                    
           return str;
		}	 		 	
	 } 
	 return getChild(tree);

}

console.log('pre' + prePrint(tree));
console.log('post' + postPrint(tree));
```

## 运行结果

![这里写图片描述](http://img.blog.csdn.net/20170423223838620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2Nhcmd0dA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




>  传变量起名，域名占星
>  招聘码农相面，上线日期择吉
>  奇门遁甲推死线，辟邪镇宅驱爆栈
> 开光显示器键盘SSD，代写注释文档PPT
> 
> 修电脑、装系统、设置路由器、疏通下水道。上门服务 价格公道   
> 电话：23232333


