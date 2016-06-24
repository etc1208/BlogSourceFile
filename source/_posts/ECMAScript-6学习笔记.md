---
title: ECMAScript 6学习笔记
date: 2016-06-24 16:01:24
tags: [ES6, 前端, 笔记]
---

参考：[http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰老师ES6入门")

ES6是JavaScript语言的下一代标准，已经在2015年6月正式发布。
![ES6](http://img3x1.ddimg.cn/43/36/23840431-1_u_2.jpg)

<!--more-->


----------

# Babel转码器 #
Babel是一个广泛使用的ES6转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。

	// 转码前
	input.map(item => item + 1);
	
	// 转码后
	input.map(function (item) {
	  return item + 1;
	});


## 配置文件.babelrc ##
Babel的配置文件是`.babelrc`，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。

该文件用来设置转码规则和插件，基本格式如下。

	{
	  "presets": [],
	  "plugins": []
	}
presets字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。

	# ES2015转码规则
	$ npm install --save-dev babel-preset-es2015
	
	# react转码规则
	$ npm install --save-dev babel-preset-react
	
	# ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
	$ npm install --save-dev babel-preset-stage-0
	$ npm install --save-dev babel-preset-stage-1
	$ npm install --save-dev babel-preset-stage-2
	$ npm install --save-dev babel-preset-stage-3
然后，将这些规则加入.babelrc。

	  {
	    "presets": [
	      "es2015",
	      "react",
	      "stage-2"
	    ],
	    "plugins": []
	  }
注意，以下所有Babel工具和模块的使用，都必须先写好.babelrc。

## 命令行转码babel-cli ##
Babel提供`babel-cli`工具，用于命令行转码。

它的安装命令如下。

	$ npm install --global babel-cli
基本用法如下。

	# 转码结果输出到标准输出
	$ babel example.js

	# 转码结果写入一个文件
	# --out-file 或 -o 参数指定输出文件
	$ babel example.js --out-file compiled.js
	# 或者
	$ babel example.js -o compiled.js
	
	# 整个目录转码
	# --out-dir 或 -d 参数指定输出目录
	$ babel src --out-dir lib
	# 或者
	$ babel src -d lib
	
	# -s 参数生成source map文件
	$ babel src -d lib -s

----------

# let和const命令 #

## let命令 ##
let命令，用来声明变量。用法类似于var，但所声明的变量只在let命令所在的代码块内有效。

	{
	  let a = 10;
	  var b = 1;
	}

	a // ReferenceError: a is not defined.
	b // 1

for循环的计数器，就很合适使用let命令。

	for (let i = 0; i < arr.length; i++) {}
	
	console.log(i);
	//ReferenceError: i is not defined


**不存在变量提升**

let不像var那样会发生“变量提升”现象。所以，**变量一定要在声明后使用，否则报错**。

	console.log(foo); // 输出undefined
	console.log(bar); // 报错ReferenceError
	
	var foo = 2;
	let bar = 2;

暂时性死区
只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

	var tmp = 123;
	
	if (true) {
	  tmp = 'abc'; // ReferenceError
	  let tmp;
	}
上面代码中，存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

**不允许重复声明**

let不允许在相同作用域内，重复声明同一个变量。
	
	// 报错
	function () {
	  let a = 10;
	  var a = 1;
	}
	
	// 报错
	function () {
	  let a = 10;
	  let a = 1;
	}
因此，不能在函数内部重新声明参数。

	function func(arg) {
	  let arg; // 报错
	}
	
	function func(arg) {
	  {
	    let arg; // 不报错
	  }
	}

**ES6的块级作用域**


	function f1() {
	  let n = 5;
	  if (true) {
	    let n = 10;
	  }
	  console.log(n); // 5
	}
上面的函数有两个代码块，都声明了变量n，运行后输出5。这表示外层代码块不受内层代码块的影响。如果使用var定义变量n，最后输出的值就是10。

## const命令 ##

const声明一个只读的常量。一旦声明，常量的值就不能改变。

	const PI = 3.1415;
	PI // 3.1415
	
	PI = 3;
	// TypeError: Assignment to constant variable.
上面代码表明改变常量的值会报错。

**const一旦声明变量，就必须立即初始化，不能留到以后赋值。**

	const foo;
	// SyntaxError: Missing initializer in const declaration
上面代码表示，对于const来说，只声明不赋值，就会报错。

const的作用域与let命令相同：只在声明所在的块级作用域内有效。

	if (true) {
	  const MAX = 5;
	}
	
	MAX // Uncaught ReferenceError: MAX is not defined
const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

	if (true) {
	  console.log(MAX); // ReferenceError
	  const MAX = 5;
	}
上面代码在常量MAX声明之前就调用，结果报错。

const声明的常量，也与let一样不可重复声明。



下面是另一个例子。

	const a = [];
	a.push('Hello'); // 可执行
	a.length = 0;    // 可执行
	a = ['Dave'];    // 报错
上面代码中，常量a是一个数组，这个数组本身是可写的，但是如果将另一个数组赋值给a，就会报错。

如果真的想将对象冻结，应该使用Object.freeze方法。

	const foo = Object.freeze({});


ES6为了保持兼容性，var命令和function命令声明的全局变量，依旧是全局对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于全局对象的属性。也就是说，从ES6开始，全局变量将逐步与全局对象的属性脱钩。

	var a = 1;
	window.a // 1
	
	let b = 1;
	window.b // undefined
上面代码中，全局变量a由var命令声明，所以它是全局对象的属性；全局变量b由let命令声明，所以它不是全局对象的属性，返回undefined。

----------








