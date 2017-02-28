---
layout: post
title: "快速了解JavaScript中的函数"
date: 2017-01-23
comments: true
categories: JavaScript
tags: [随笔]
keywords: 快速了解JavaScript中的函数
publish: true
description: 快速了解JavaScript中的函数
---
最近在学习前端知识，顺手把一些东西记录下来，加深记忆，忘记的时候翻阅一下，温故而知新。

## 快速了解JavaScript中的函数
### 1.普通函数

#### 1.1 普通函数示例
```
function sayHello() {
	console.log("hello");
}
console.log("------------------------------------------------");
```

#### 1.2 同名函数覆盖,
JS中没有函数重载，定义相同的函数名，不同参数的函数。 调用时，后面的函数会覆盖前面的函数。

```
function add(value1) {
	return value1 + 1;
}
	
function add(value1, value2) {
	return value1 + 2;
}
console.log(add(2)); //4
console.log("------------------------------------------------");
```

#### 1.3 arguments对象 
 传入参数大于定义参数时，arguments对象存放了函数调用时的所有参数
```
function showNames(name) {
	console.log(name);
	for (var i = 0; i < arguments.length; i++) {
		console.log(arguments[i]);
	} 
}
showNames("🍎", "🍐", "🍑"); //🍎🍎🍐🍑
console.log("------------------------------------------------");
```

#### 1.4 “undefined”
若函数没有指明返回值，默认返回的是“undefined”
```
function showUndefined() {
	
}
console.log(showUndefined());
console.log("------------------------------------------------");
```

### 2.匿名函数
#### 2.1 变量匿名函数 
可以把函数赋值给变量，事件。
```
var add = function (num1, num2) {
	console.log(num1 + num2);
}
	
add(1, 1);
console.log("------------------------------------------------");
```

#### 2.2 无名称匿名函数 
即在函数声明时，在后面紧跟参数。JS解析语法时立即执行该函数。
```
(function(arg) {
	console.log("无名称匿名函数" + arg);
})(33);
console.log("------------------------------------------------");
```
适用场景，只需执行一次的。如浏览器加载完，只需要执行一次且后面不执行的功能。

### 3.闭包函数
 * 函数A内部声明了个函数B，函数B引用了函数B之外的变量，并且函数A的返回值是函数B的引用，那么函数B就是闭包函数。

#### 3.1 全局引用与局部引用
```
function funA() {
	var i = 0;
	function funB() { // 闭包函数funB
		i++;
		console.log(i);
	}
	return funB();
}
// allShowA是全局变量，引用了函数funA,重复运行allShowA(),输出1,2,3,4等累加值。
var allShowA = funA(); 

// 执行函数partShowA(),因为内部只声明了局部变量showA来引用函数funA,函数执行完由于作用域，showA变量就被释放了。
function partShowA() {
	var showA = funA();
	showA();
}
// 闭包的关键就在于作用域：全局变量占有的资源只有当页面变换或浏览器关闭后才会释放。var allShowA = funA()时，相当于allShowA引用了funB()，从而使funB()里的资源不被GC回收，因此funA()里的资源也不会。
console.log("------------------------------------------------");
```

#### 3.2 有参数的闭包函数
```
 function funA(arg1, arg2) {
 	var i = 0;
 	function funB(num) {
 		i = i + num;
 		console.log(i);
 	}
 	return funB;
 }
	
 var allShowA = funA(3, 4);
 allShowA(1); // 1
 allShowA(2); // 3
 allShowA(3); // 6
	
 function partShowA(arg) {
	var showA = funA(3, 4);
	showA(arg);
}
partShowA(1); // 1
partShowA(2); // 2
partShowA(3); // 3
console.log("------------------------------------------------");
```
#### 3.3 函数内变量共享
```
function funA1() {
	var i = 0;
	function funB() {
		i++;
		console.log(i);
	}
	allShowC = function(){
		i++;
		console.log(i);
	}
	return funB();
}
var allShow1 = funA1();
var allShow2 = funA1();
console.log("------------------------------------------------");
```

#### 参考
[JavaScript function函数种类](http://www.cnblogs.com/polk6/p/3284839.html)


