---
title: npm scripts入门
date: 2016-10-14 15:23:12
tags: [npm scripts]
---

npm 不仅可以用于 模块管理 ，还可以用于 执行脚本 。

参考：[http://www.tuicool.com/articles/Y3Mvmiz](http://www.tuicool.com/articles/Y3Mvmiz)

![npm scripts](http://image.codes51.com/Article/image/20160823/20160823121748_0170.png)

<!--more-->

package.json 文件有一个 scripts 字段，可以用于指定脚本命令，供 npm 直接调用。

	{
	 "scripts": {
	 "commit": "sh commit.sh"
	 }
	}
其中 commit.sh 文件内容为：

	git add--all
	git commit-m"common commit"
	git push
接下来我们运行：

	$ npm run commit
	# 或
	$ npm run-script commit
便相当于执行 sh commit.sh 任务

npm run 会创建一个新的 shell ，执行指定的命令，并将 node_modules/.bin 加入 PATH 变量。当脚本内容结束，则子 shell 关闭，回到父 shell 中。

### 通配符
由于 npm 脚本就是 Shell 脚本，因为可以使用 Shell 通配符。

	"lint": "jshint *.js"
	"lint": "jshint **/*.js"
上面代码中，* 表示任意文件名，**表示任意一层子目录。
如果要将通配符传入原始命令，防止被 Shell 转义，要将星号转义。

	"test": "tap test/\*.js"
	
### 传参
向 npm 脚本传入参数，要使用--标明。

	"lint": "jshint **.js"
向上面的npm run lint命令传入参数，必须写成下面这样。

	$ npm run lint --  --reporter checkstyle > checkstyle.xml
也可以在package.json里面再封装一个命令。

	"lint": "jshint **.js",
	"lint:checkstyle": "npm run lint -- --reporter checkstyle > checkstyle.xml"
### 执行顺序
如果 npm 脚本里面需要执行多个任务，那么需要明确它们的执行顺序。
如果是并行执行（即同时的平行执行），可以使用&符号。

	$ npm run script1.js & npm run script2.js
如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用&&符号。

	$ npm run script1.js && npm run script2.js
这两个符号是 Bash 的功能。此外，还可以使用 node 的任务管理模块：script-runner、npm-run-all、redrun。

### bash 脚本

写在 scripts 属性中的命令，也可以在 node_modules/.bin 目录中直接写成 bash 脚本。下面是一个 bash 脚本（命名为： build-js.sh ）。

	#!/bin/bash
	
	cd site/main
	browserify browser/main.js|uglifyjs-mc>static/bundle.js
我们在 package.json 内定义执行该脚本的命令：

	"scripts": {
	 "build-js": "sh bin/build-js.sh"
	}
并且需要设置脚本文件权限为可执行。

之后我们只需要运行：

	npm run build-js
即可自动执行我们建立好的脚本！

### 参数

npm run 命令还可以接参数。

package.json ：在 newTxt 任务中未指定具体执行的脚本

	"script": {
	 "newTxt": "sh"
	}
我们在 node_module/.bin/ 下新建一个待会要传入 newTxt 任务的脚本文件 newTxt.sh ：

	touch newTxt.txt
然后我们运行：

	npm run newTxt--newTxt.sh
这样， newTxt.sh 便会作为参数传入我们的 newTxt 任务。

因为一个命令可以执行多个脚本：所以传入的参数并不会覆盖原来的参数，而是并行（或先后）执行。

	"scripts": {
	 "test": "mocha test/"
	}
	
	
	$ npm run test--anothertest.js
	# 等同于
	$ mocha test/anothertest.js
上面命令表示， mocha 要运行所有 test 子目录的测试脚本， 以及 另外一个测试脚本 anothertest.js 。

传入参数的格式： -- 

在参数之前加入 -- 并用空格使命令与参数隔开

其他参数： 

npm run 本身有一个参数-s，表示关闭 npm 工具本身的输出， 只输出脚本产生的结果 。

// 输出npm命令头信息
	$ npm run test

// 不输出npm命令头
	$ npm run-s test

## pre- 和 post- 脚本命令

npm run 为每条命令提供了 pre- 和 post- 两个钩子（ hook ）。以 npm run test 为例，如果我们的 scripts 字段规定了 pretest 和 posttest ：

	 "scripts": {
	 "test": "mocha test/",
	 "pretest": "echo test started!",
	 "posttest": "echo test end!"
	}
则会先执行 pretest 任务，再执行 test 任务，完成 test 任务后即执行 posttest 任务。

可以简单的将二者理解为： 预执行 、 后执行

### 内部变量

scripts 字段可以使用一些内部变量，主要是 package.json 的各种字段。内部变量的主要特征是 $npm_package_key 。

比如， package.json 的内容是 {"name":"foo", "version":"1.2.5"} ，那么变量 $npm_package_name 的值是 foo ，变量 npm_package_version 的值是 1.2.5 。

	{
	 "version": "1.2.5",
	 "scripts":{
	 "bundle": "mkdir -p build/$npm_package_version/"
	 }
	}
运行 npm run bundle 以后，将会生成 build/1.2.5/ 子目录。

config 字段也可以用于设置内部字段。

	 "name": "fooproject",
	 "config": {
	 "port": "3001"
	 },
	 "scripts": {
	 "serve": "http.createServer(...).listen(process.env.$npm_package_config_port)"
	 }
上面代码中，变量 npm_package_config_port 对应的就是 3001 。

### 通配规则

	* 匹配 0 个或多个字符
	? 匹配 1 个字符
	[...] 匹配某个范围的字符。如果该范围的第一个字符是 ! 或 ^ ，则匹配不在该范围的字符。
	!(pattern|pattern|pattern) 匹配任何不符合给定的模式
	?(pattern|pattern|pattern) 匹配 0 个或 1 个给定的模式
	+(pattern|pattern|pattern) 匹配 1 个或多个给定的模式
	*(a|b|c) 匹配 0 个或多个给定的模式
	@(pattern|pat*|pat?erN) 只匹配给定模式之一
	** 如果出现在路径部分，表示0个或多个子目录。