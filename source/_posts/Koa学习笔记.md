---
title: Koa学习笔记
date: 2016-06-21 10:36:28
tags: [Koa, Node, 前端, 笔记]
---
转载自：[http://book.apebook.org/minghe/koa-action/index.html](http://book.apebook.org/minghe/koa-action/index.html)

koa.js 是下一代的node.js框架，由Express团队开发，通过生成器（generators JavaScript 1.7新引入的，用于解决回调嵌套的方案），减少异步回调，提高代码的可读性和可维护性，同时改进了错误处理.

koa 的先天优势在于 generator，带来的主要好处如下：

- 更优雅、简单、安全的中间件机制，后面章节会详细说明
- 更优雅、简单的异常处理
- 更优雅、简单的异步编程方式

<!--more-->
# 安装 generator-k #

`generator-k` 是koa项目工程生成器，带有经过筛选的优秀中间件，生成即用。

    npm install -g yo generator-k
generator-k 依赖于 yoeman，所以必须安装 yo。

安装成功后，创建一个 book 目录（作为demo工程），并进入，然后运行：

    yo k

运行应用

	node --harmony app.js

app.js 的核心代码是：

	var koa = require('koa');
	var app = koa();
	app.listen(3000);


# koa应用初见 #

	var koa = require('koa');
	var app = koa();
	
	app.use(function *(){
	    var path = this.path;
	    this.body = path;
	});
	
	app.listen(3000);


初始化 koa ：

	var koa = require('koa');
	var app = koa();

使用`app.use()`注入中间件，这是与 express 最大不同的地方。

所有的koa中间件，必须是` generator function `，即 `function *(){}` 语法，不然会报错。

**中间件 的上下文 this，指向用户当前的请求**，中间件只有在请求时才会触发逻辑，比如获取当前请求的路径，可以使用：

	app.use(function *(){
	    var path = this.path;
	});

`this.body` 用于控制输出到页面的内容，比如：

	app.use(function *(){
	    this.body = '<p>我是个html片段</p>';
	});

可以注入多个中间件，koa 会从上到下执行，然后从下到上执行。

	app.use(function *(){
	    this.demo = 'test text';
	});
	
	app.use(function *(){
	    this.body = this.demo;
	});

其实 app.use() 就干了一件事，就是将中间件放入一个数组，真正执行逻辑的是：

	app.listen(3000);
koa 的 `listen()` 除了指定了 http 服务的端口号外，还会启动 `http server`，等价于：

	var http = require('http');
	http.createServer(app.callback()).listen(3000);
后面这种繁琐的形式有什么用呢？

一个典型的场景是启动 https 服务，默认 app.listen(); 是启动 http 服务，启动 https 服务就需要：
	
	var https = require('https');
	https.createServer(app.callback()).listen(3000);


# koa路由 #
koa是个极简的web框架，简单到连路由模块都没有配备，多个页面在koa中要这么写：

	app.use(function *(){
	    //我是首页
	    if(this.path==='/'){
	
	    }
	});

	app.use(function *(){
    	//我是详情页
    	if(this.path==='/detail'){

    	}
	});


我们需要引入个路由中间件` koa-router` ，有了 koa-router 我们就可以像写 express 路由一样，写 koa 应用页面：

注入 koa-router 中间件：

	var router = require('koa-router');
	app.use(router(app));
简单的路由写法：

	app.get('/', function *(next) {
	    //我是首页
	    //this 指向请求
	});
	app.get('/detail/:id', function *(next) {
	    //我是详情页面
	    //:id 是路由通配规则，示例请求 /detail/123 就会进入该 generator function 逻辑
	    var id = this.params.id; //123
	});
除了最常用的`get()`外，还有` post() `、`put()` 、`patch()` 、`delete()`。

koa-router 拥有丰富的 api 细节，用好这些 api ，可以让页面代码更为优雅与可维护。

比如 `param() `方法，用于路由参数的处理：

	app.param('id',function *(id,next){
	    this.id = Number(id);
	    if ( typeof this.id != 'number') return this.status = 404;
	    yield next;
	}).get('/detail/:id', function *(next) {
	    //我是详情页面
	    var id = this.id; //123
	    this.body = id;
	});
param() 用于封装参数处理中间件，当访问 /detail/:id 路由时，会先执行 param() 定义的 generator function 逻辑。

**函数的第一个是路由参数的值，next 是中间件流程关键标识变量。**

    yield next;
表示执行下一个中间件。

**通过 param() ，我们可以把参数的处理给抽象出来。**

# koa的错误处理 #

koa 有 `error` 事件，当发生错误时，可以通过该事件，对错误进行统一的处理。

我们创建个 error.js：

	var koa = require('koa');
	var app = koa();
	
	app.on('error', function(err,ctx){
	    if (process.env.NODE_ENV != 'test') {
	        console.log(err.message);
	        console.log(err);
	    }
	});   

	app.listen(3000);
监听 error 事件：

`err 参数 `： 错误对象，留意抛异常时，请务必抛出 Error 对象，不能只是简单的字符串，这是良好的编码习惯.

`ctx` ：为发生请求的上下文对象

`process.env.NODE_ENV` 是环境变量配置。

process.env.NODE_ENV != 'test' 即不是单元测试环境时。

我们注入个中间件，并抛出个异常：

	app.use(function *(){
	    throw new Error('demo error');
	});
再次强调，**抛出异常必须是 Error 实例**。

运行：node --harmony error.js，打开 http://localhost:3000，页面会打印出 “Internal Server Error” （这是koa对错误的默认处理）。

同时会触发 error 事件，打印出 err.message。

“Internal Server Error” 的页面信息过于简陋，我们**引入 koa-onerror 中间件，优化错误信息**。

koa-onerror 会在 `process.env.NODE_ENV === 'development'`（不存在环境变量时，也会输出错误信息） 时抛出错误堆栈信息。

	var onerror = require('koa-onerror');
	onerror(app);
再次运行试试，页面打印的信息就会变成：

	Error
	Looks like something broke!
	
	
	Error: demo error
	    at Object.<anonymous> (/dev/node/k-demo/error.js:5:11)
	    at GeneratorFunctionPrototype.next (native)
	    at Object.respond (/dev/node/k-demo/node_modules/koa/lib/application.js:172:10)
	    at GeneratorFunctionPrototype.next (native)
	    at Object.<anonymous> (/dev/node/k-demo/node_modules/koa/node_modules/koa-compose/index.js:37:12)
	    at GeneratorFunctionPrototype.next (native)
	    at next (/dev/node/k-demo/node_modules/koa/node_modules/co/index.js:83:21)
	    at Object.<anonymous> (/dev/node/k-demo/node_modules/koa/node_modules/co/index.js:56:5)
	    at Server.<anonymous> (/dev/node/k-demo/node_modules/koa/lib/application.js:123:8)
	    at Server.EventEmitter.emit (events.js:110:17)

# 应用环境配置 #
我们通常通过环境变量，来区分各个应用的部署环境，根据不同的环境使用不同的配置，比如在开发环境中，我们将错误堆栈信息直接抛到页面中，而在生产环境中，我们会跳转到一个友好的错误页面。

**启动应用时，可以带上定义的环境变量**：

	NODE_ENV=development node --harmony app.js
我们配置了个 NODE_ENV 环境变量，定义为开发环境（development）。

**（PS: WIN下配置环境变量的方式不同，需要使用 set NODE_ENV=development。）**

打开 config/config.js，可以找到针对开发环境做的配置：

	var _ = require('underscore');
	//本地/开发环境配置
	var local = require('./local');
	if(process.env.NODE_ENV === 'local' || process.env.NODE_ENV === 'development'){
	    //使用local.js中的配置覆盖config.js中的配置
	    config = _.extend(config,local);
	}
**process.env** 包含了应用所有配置的环境变量。

local.js 的配置内容如下：

	module.exports = {
	    "env":"local",
	    "debug": true
	};
为了方便中间件获取应用配置，我们将配置信息，注入到中间件上下文：

	var config = require('./config/config');
	app.use(function *(next){
	    //config 注入中间件，方便调用配置信息
	    if(!this.config){
	        this.config = config;
	    }
	    yield next;
	});
路由中获取配置：

	app.get('/config',function *(){
	    var config = this.config;
	    this.body = config.env;
	});
页面输出是 local ，说明生产环境中 env 配置项已经被 local.js 中的配置所覆盖。

# debug模块的使用 #

推荐使用 debug 模块，debug 模块可以对log进行分组，而且只有设置了环境变量` DEBUG `的情况下才会输出 log 。

	var debug = require('debug')('book');
	app.get('/config',function *(){
	    var config = this.config;
	    debug('env: %s',config.env);
	    this.body = config.env;
	});
配置环境变量：`DEBUG=book` ，这样只会输出 book 分组的 log 信息。

    DEBUG=book NODE_ENV=local node --harmony app.js
多个环境变量，使用空格隔开。

访问页面后，输出的log是：

book env: local +0ms

**如果去掉环境变量 DEBUG=book 将不输出任何信息，这样当我们需要调试某个模块文件时，可以保证只有该模块的 log ，不会被其他模块所干扰。**

如果我们想输出所有的调试信息呢？

可以使用 `DEBUG=*` 。

# 日志记录 #

日志是应用不可缺少的部分，但 koa 没有日志记录中间件，我们使用 `mini-logger` 。

mini-logger 会创建日志记录文件，自动使用日期归档，而且还可以自定义日志分类。

	var Logger = require('mini-logger');
	var logger = Logger({
	    dir: config.logDir,
	    categories: [ 'router','model','controller'],
	    format: 'YYYY-MM-DD-[{category}][.log]'
	});
**dir** ：指定日志放在哪里

**categorie**s ：自定义日志分类

**format** ：日志文件名格式

记录 Error() 实例：

	logger.error(new Error('test'));
**使用 error() ，会将出错堆栈信息输出到 xxxx-xx-xx-error.log 文件中。**

**当定义自定义分类后，会自动注册分类日志方法**，比如定义了 `categories: ['router'] `：

	logger.router('test router');
**就有了 logger.router() 方法，并将日志记录到 xxxx-xx-xx-router.log 文件中。**

----------

# ES6的generator #
ES6的generator 是 koa 的基础，想要用好 koa 离不开对 generator 的理解。

定义一个 generator 函数：

    var generator_func = function* () { };
接下来我们要用到 `yield` 关键字，**用于停止执行和保存当前的堆栈**。

我们通过一个数字的例子来演示其用法：

	var r = 3;
	
	function* infinite_ap(a) {
	    for( var i = 0; i < 3 ; i++) {
	        a = a + r ;
	        yield a;
	    }
	}
	
	var sum = infinite_ap(5);
	
	console.log(sum.next()); // returns { value : 8, done : false }
	console.log(sum.next()); // returns { value : 11, done: false }
	console.log(sum.next()); // returns { value : 14, done: false }
	console.log(sum.next()); //return { value: undefined, done: true }

**yield a 会暂停执行并保存当前堆栈，返回当前的a。**

generator 函数与普通函数不同，只定义遍历器，而不会执行，每次调用这个遍历器的**next**方法，就从函数体的头部或者上一次停下来的地方开始执行，直到遇到下一个yield语句为止。

**yield语句就是暂停标志，next方法遇到yield，就会暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回对象的value属性的值。**当下一次调用next方法时，再继续往下执行，直到遇到下一个yield语句。如果没有再遇到新的yield语句，就一直运行到函数结束，将return语句后面的表达式的值，作为value属性的值，如果该函数没有return语句，则value属性的值为undefined。

当第一次调用 sum.next() 时 返回的a变量值是5 + 3，同理第二次调用 sum.next() ，a变量值是8 +3，知道循环执行结束，返回done:true标识。

大家有没有发现个问题，koa 中 generator 的用法与上述 demo 演示的用法有非常大得差异，那是因为**koa 中的 generator 使用了 co 进行了封装**。

----------

# co 的简单使用 #

	var co = require('co');
	var fs = require('fs');
	
	function read(file) {
	  return function(fn){
	    fs.readFile(file, 'utf8', fn);
	  }
	}
	co(function *(){
	
	  var a = yield read('.gitignore');
	  console.log(a.length);
	
	  var b = yield read('package.json');
	  console.log(b.length);
	});
co 要求所有的异步函数必须是偏函数，称之为 thunk :

	function read(file) {
	  return function(fn){
	    fs.readFile(file, 'utf8', fn);
	  }
	}
上述代码封装了 fs.readFile() 异步读取文件的方法。

node 异步方法的回调函数规范是 function(err,result){} ,co 会处理该形式的函数，result或error会在异步函数执行完毕后获取到。

如果需要对 thunk 的返回数据做下处理，可以这么写：

	function read(file) {
	  return function(fn){
	    fs.readFile(file, 'utf8', function(err,result){
	        if (err) return fn(err);
	        fn(null, result);
	    });
	  }
	}
如果嫌写thunk函数别扭，可以使用 thunkify：

	var thunkify = require('thunkify');
	var fs = require('fs');

	var read = thunkify(fs.readFile);
获取 thunk 函数的结果，只要使用 yield 关键字：

	var a = yield read('.gitignore');
	console.log(a.length);
**在 co 中，我们无需使用控制流程的 next 方法，co 已经封装了 generator function 的流转。**

----------


> 中间件从上到下执行完后，可以从下到上执行 yield next 后的逻辑
	
	var app = new SimpleKoa();
	
	app.use(function *(next){
	    this.body = '1';
	    yield next;
	    this.body += '5';
	    console.log(this.body);
	});
	app.use(function *(next){
	    this.body += '2';
	    yield next;
	    this.body += '4';
	});
	app.use(function *(next){
	    this.body += '3';
	});
	app.listen();
**执行后控制台输出：123456**



----------
# ES6的Promise #

	var promise = new Promise(function(resolve, reject){
	    FS.readFile("foo.txt", "utf-8", function (error, text) {
	        if (error) {
	            reject(new Error(error));
	        } else {
	            resolve(text);
	        }
	    });
	}
	return promise;
使用 promise：

	promise.then(function(){
	
	}).fail(function(){
	
	})

----------
# node单元测试 #

**mocha** 是最好的 node 单元测试框架，使用简单且灵活，是进行 node 单测的首选。
    
    webstorm 已经集成 mocha 。

安装 mocha：

	npm install -g mocha    
案例：

	var assert = require("assert");
	describe('Array', function() {
	    describe('#indexOf()', function() {
	        it('should return -1 when the value is not present', function() {
	            assert.equal(-1, [1,2,3].indexOf(5));
	            assert.equal(-1, [1,2,3].indexOf(0));
	        });
	    });
	});


- assert 模块是 node 内置的断言库。

- describe() 用于定义测试用例组，是可以嵌套的。

- it() 是定义具体的测试用例。

- assert.equal() 用于判断值是否符合预期。

命令行运行用例 ：

	mocha
输出结果：

	Array
	#indexOf()
	  ✓ should return -1 when the value is not present


----------
# should断言库 #
node 内置的断言库 assert ，功能比较弱，不太好用，推荐使用 `should` 。

should 的断言方法注入到 Object.prototype 中，所以断言的风格更符合用户思维习惯，也支持链式调用，跟 jQuery 有点像：

	var should = require("should");
	describe('Should test', function() {
	    it('number', function() {
	        (123).should.be.a.Number;
	    });
	    it('object property', function() {
	        var obj = {name:'minghe',email:"minghe36@gmail.com"};
	        obj.should.have.property('name','minghe');
	        obj.should.have.property('email');
	    });
	});
> (123).should.be.a.Number 判断 123 是否是一个数字，适用于其他类型的判断。
> 
> obj.should.have.property('name','minghe') obj 对象是否包含属性 name ，且 name = 'minghe' 。

## 常用的 api ##

使用 ok 判断值是否为 true：

	it('ok',function(){
	    (true).should.be.ok;
	})
使用 equal 判断一个值是否符合预期：

	it('equal',function(){
	    'abc'.should.equal('abc');
	});
使用 not 取反：

	it('not equal',function(){
	    'abc'.should.not.equal('ddd');
	});
判断值是否存在：

	it('exist',function(){
	    var result = {};
	    should.exist(result);
	})


----------

# mocha异步测试 #

	var fs = require('fs');
	var should = require('should');
	describe('fs', function() {
	    describe('#readFile()', function() {
	        it('should not be null', function(done) {
	            fs.readFile('./package.json', 'utf8', function(err,res){
	                if (err) throw err;
	                res.should.not.equal(null);
	                done();
	            });
	        });
	    });
	});
**关键在于 done 实参，必须在执行完异步后（在异步回调中）执行下 done()，就能捕获到用例。
回调函数 done() 支持接收一个错误：done(err)，用于简化错误处理。**

----------

# supertest请求测试 #
在 node 业务应用中，我们经常需要`测试路由`的可用性，如何处理呢？

可以使用` supertest` 模块，supertest 专门`用于 http 断言，支持 koa 的 http 请求测试`。

**为了保证应用的可测试性，我们需要把应用脚本比如 app.js 中的 koa 实例暴露出来：**

	var koa = require('koa');
	var app = koa();
	//一堆中间件...
	module.exports = app;
用例写法：

	var superagent = require('supertest');
	var app = require('../app');
	
	function request() {
	    return superagent(app.listen());
	}
	
	describe('Routes', function () {
	    describe('GET /', function () {
	        it('should return 200', function (done) {
	            request()
	                .get('/')
	                .expect(200, done);
	        });
	    });
	});
`superagent(app.listen())` 会截获 koa 的 http 请求，可以使用 get 、 post 等方法，对请求进行测试。

	request()
	    .get('/')
	    .expect(200, done);
**get('/') 即测试首页 get 请求，.expect(200, done) 测试 请求状态码是否为 200 （请求成功），done 是必须传入的，这样请求测试结束后，才能把测试信息推送给mocha处理。**

上述测试代码等价于：
	
	request()
	    .get('/')
	    .expect(200)
	    .end(function(err, res){
	        if (err) return done(err);
	        done();
	    });
.end() 回调会在请求完成后触发，可以在回调中对错误进行处理，res 包含完整的请求信息，可以对这些信息进行测试，比如页面输出的内容等。

运行命令 ：

	mocha --harmony
**留意：测试 koa 的请求必须加--harmony，否者会抛异常。**

我们经常需要对 **json 接口的数据结构合法性进行测试**，如何借助 supertest 实现测试呢？

我们新建个 /api/user/:id 的路由，返回一个用户信息：

	app.get('/api/user/:id',function *(){
	    var user = {name:'minghe',email:'minghe36@gmail.com'};
	    user = JSON.stringify(user);
	    this.body = user;
	})
测试此路由是否返回正确的数据：

	it('should be json',function(done){
	    request()
	        .get('/api/user/1')
	        .expect(200)
	        .end(function(err, res){
	            if (err) return done(err);
	            var text = res.text;
	            var json = JSON.parse(text);
	            json.should.have.property('email');
	            json.should.have.property('minghe');
	            done();
	        });
	})
supertest 很强大，**可以设置请求的头信息**，使用 set() ：

	request()
	    .get('/')
	    .set('Accept', 'application/json')
	    .expect('Content-Type', /json/)    

**而 expect() 除了支持状态码测试外，还支持头信息测试：.expect('Content-Type', /json/)**
