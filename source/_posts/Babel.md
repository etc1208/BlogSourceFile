---
title: Babel
date: 2016-07-06 10:37:55
tags: [Babel, ES6, 前端]
---

Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。

参考：[http://www.ruanyifeng.com/blog/2016/01/babel.html](http://www.ruanyifeng.com/blog/2016/01/babel.html "Babel")

![Babel](http://i.imgur.com/DIkO1Ox.png)

<!--more-->

## 一、配置文件.babelrc ##
Babel的配置文件是`.babelrc`，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。

该文件用来设置`转码规则`和`插件`，基本格式如下。

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
**注意，以下所有Babel工具和模块的使用，都必须先写好.babelrc。**

----------

## 二、命令行转码babel-cli ##
Babel提供`babel-cli`工具，用于命令行转码。

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

安装到当前项目：
	$ npm install --save-dev babel-cli
然后，改写package.json。

	{
	  // ...
	  "devDependencies": {
	    "babel-cli": "^6.0.0"
	  },
	  "scripts": {
	    "build": "babel src -d lib"
	  },
	}
转码的时候，就执行下面的命令。

	$ npm run build

----------

## 三、babel-polyfill ##

**Babel默认只转换新的JavaScript句法，而不转换新的API**，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。

举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用`babel-polyfill`，为当前环境提供一个垫片。
安装命令如下。

	$ npm install --save babel-polyfill
然后，在脚本头部，加入如下一行代码。
	
	import 'babel-polyfill';
	// 或者
	require('babel-polyfill');


----------

## 四、浏览器环境 ##

Babel也可以用于浏览器环境。但是，从Babel 6.0开始，不再直接提供浏览器版本，而是要用构建工具构建出来。如果你没有或不想使用构建工具，可以通过安装5.x版本的babel-core模块获取。

	$ npm install babel-core@5

运行上面的命令以后，就可以在当前目录的`node_modules/babel-core/`子目录里面，找到babel的浏览器版本`browser.js`（未精简）和`browser.min.js`（已精简）。
然后，将下面的代码插入网页。

	<script src="node_modules/babel-core/browser.js"></script>
	<script type="text/babel">
	// Your ES6 code
	</script>
上面代码中，**browser.js是Babel提供的转换器脚本**，可以在浏览器运行。用户的ES6脚本放在script标签之中，但是要注明**type="text/babel"**。

另一种方法是使用`babel-standalone`模块提供的浏览器版本，将其插入网页。

	<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.4.4/babel.min.js"></script>
	<script type="text/babel">
	// Your ES6 code
	</script>
**注意，网页中实时将ES6代码转为ES5，对性能会有影响。生产环境需要加载已经转码完成的脚本。**

----------
## 五、与其他工具的配合 ##

许多工具需要Babel进行**前置转码**，这里举两个例子：`ESLint和Mocha`。

ESLint 用于静态检查代码的语法和风格，安装命令如下。

	$ npm install --save-dev eslint babel-eslint
然后，在项目根目录下，新建一个配置文件.eslint，在其中加入parser字段。

	{
	  "parser": "babel-eslint",
	  "rules": {
	    ...
	  }
	}
再在`package.json`之中，加入相应的scripts脚本。

	  {
	    "name": "my-module",
	    "scripts": {
	      "lint": "eslint my-files.js"
	    },
	    "devDependencies": {
	      "babel-eslint": "...",
	      "eslint": "..."
	    }
	  }

Mocha 则是一个测试框架，如果需要执行使用ES6语法的测试脚本，可以修改package.json的scripts.test。

	"scripts": {
	  "test": "mocha --ui qunit --compilers js:babel-core/register"
	}
上面命令中，--compilers参数指定脚本的转码器，规定后缀名为js的文件，都需要使用babel-core/register先转码。




