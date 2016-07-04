---
title: Mocha测试框架
date: 2016-07-01 11:18:14
tags: [Mocha, 测试, 前端]
---

Mocha是现在最流行的JavaScript测试框架之一，在浏览器和Node环境都可以使用。

参考：[http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html "测试框架 Mocha 实例教程")

![Mocha](http://i.imgur.com/RaZHByh.jpg)

<!--more-->

为了操作的方便，请在全面环境也安装一下Mocha。

    $ npm install --global mocha

----------

## 测试脚本的写法 ##

下面是一个加法模块add.js的代码。

	// add.js
	function add(x, y) {
	  return x + y;
	}

	module.exports = add;

add.js的测试脚本名字就是add.test.js。

	// add.test.js
	var add = require('./add.js');
	var expect = require('chai').expect;
	
	describe('加法函数的测试', function() {
	  it('1 加 1 应该等于 2', function() {
	    expect(add(1, 1)).to.be.equal(2);
	  });
	});

上面这段代码，就是测试脚本，它可以独立执行。

测试脚本里面应该包括`一个或多个describe块`，每个describe块应该包括`一个或多个it块`。

- **describe**块称为"测试套件"，表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称，第二个参数是一个实际执行的函数。

- **it**块称为"测试用例"，表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称，第二个参数是一个实际执行的函数。


----------

## 断言库的用法 ##

所谓"断言"，就是判断源码的实际执行结果与预期结果是否一致，如果不一致就抛出一个错误。

所有的测试用例（it块）都应该含有一句或多句的断言。它是编写测试用例的关键。断言功能由断言库来实现，**Mocha本身不带断言库，所以必须先引入断言库**。

    var expect = require('chai').expect;

断言库有很多种，Mocha并不限制使用哪一种。上面代码引入的断言库是`chai`，并且指定使用它的`expect`断言风格。

**expect断言的优点是很接近自然语言**，下面是一些例子。

	// 相等或不相等
	expect(4 + 5).to.be.equal(9);
	expect(4 + 5).to.be.not.equal(10);
	expect(foo).to.be.deep.equal({ bar: 'baz' });
	
	// 布尔值为true
	expect('everthing').to.be.ok;
	expect(false).to.not.be.ok;
	
	// typeof
	expect('test').to.be.a('string');
	expect({ foo: 'bar' }).to.be.an('object');
	expect(foo).to.be.an.instanceof(Foo);
	
	// include
	expect([1,2,3]).to.include(2);
	expect('foobar').to.contain('foo');
	expect({ foo: 'bar', hello: 'universe' }).to.include.keys('foo');
	
	// empty
	expect([]).to.be.empty;
	expect('').to.be.empty;
	expect({}).to.be.empty;
	
	// match
	expect('foobar').to.match(/^foo/);

**基本上，expect断言的写法都是一样的。头部是expect方法，尾部是断言方法，比如equal、a/an、ok、match等。两者之间使用to或to.be连接。**

如果expect断言不成立，就会抛出一个错误。事实上，只要不抛出错误，测试用例就算通过。

---------

## Mocha的基本用法 ##


	$ mocha add.test.js
	
	  加法函数的测试
	    ✓ 1 加 1 应该等于 2
	
	  1 passing (8ms)

上面的运行结果表示，测试脚本通过了测试，一共只有1个测试用例，耗时是8毫秒。
mocha命令后面紧跟测试脚本的路径和文件名，可以指定多个测试脚本。

	$ mocha file1 file2 file3

**Mocha默认运行test子目录里面的测试脚本。所以，一般都会把测试脚本放在test目录里面，然后执行mocha就不需要参数了。**

	$ mocha
	
	  加法函数的测试
	    ✓ 1 加 1 应该等于 2
	    ✓ 任何数加0应该等于自身
	
	  2 passing (9ms)

**这时，test子目录里面的测试脚本执行了。但是，如果test子目录下面还有子目录，里面还有测试脚本，他们并不会得到执行。原来，Mocha默认只执行test子目录下面第一层的测试用例，不会执行更下层的用例。**

为了改变这种行为，就必须加上**--recursive**参数，这时test子目录下面所有的测试用例----不管在哪一层----都会执行。

	$ mocha --recursive
	
	  加法函数的测试
	    ✓ 1 加 1 应该等于 2
	    ✓ 任何数加0应该等于自身
	
	  乘法函数的测试
	    ✓ 1 乘 1 应该等于 1
	
	  3 passing (9ms)

## 命令行参数 ##

- --help或-h参数，用来查看Mocha的所有命令行参数。

- --reporters参数可以显示所有内置的报告格式。

- --growl参数，就会将测试结果在桌面显示。

- --watch参数用来监视指定的测试脚本。只要测试脚本有变化，就会自动运行Mocha。

- --bail参数指定只要有一个测试用例没有通过，就停止执行后面的测试用例。

- --grep参数用于搜索测试用例的名称（即it块的第一个参数），然后只执行匹配的测试用例

		$ mocha --grep "1 加 1"

- --invert参数表示只运行不符合条件的测试脚本，必须与--grep参数配合使用。

		$ mocha --grep "1 加 1" --invert


----------
## 配置文件mocha.opts ##

Mocha允许在test目录下面，放置配置文件`mocha.opts`，把命令行参数写在里面，运行下面的命令。

	$ mocha --recursive --reporter tap --growl

上面这个命令有三个参数--recursive、--reporter tap、--growl。
然后，把这三个参数写入test目录下的mocha.opts文件。

	--reporter tap
	--recursive
	--growl

然后，执行mocha就能取得与第一行命令一样的效果。

	$ mocha
**如果测试用例不是存放在test子目录**，可以在mocha.opts写入以下内容。

	server-tests
	--recursive
**上面代码指定运行server-tests目录及其子目录之中的测试脚本**。

----------

## ES6测试 ##

如果测试脚本是用`ES6`写的，那么运行测试之前，需要**先用Babel转码**。可以看到这个测试用例是用ES6写的。

	import add from '../src/add.js';
	import chai from 'chai';
	
	let expect = chai.expect;
	
	describe('加法函数的测试', function() {
	  it('1 加 1 应该等于 2', function() {
	    expect(add(1, 1)).to.be.equal(2);
	  });
	});
**ES6转码，需要安装Babel。**

	$ npm install babel-core babel-preset-es2015 --save-dev
然后，在项目目录下面，新建一个**.babelrc**配置文件。

	{
	  "presets": [ "es2015" ]
	}

最后，使用`--compilers`参数指定测试脚本的转码器。

	$ ../node_modules/mocha/bin/mocha --compilers js:babel-core/register

上面代码中，**--compilers**参数后面紧跟一个用**冒号**分隔的字符串，**冒号左边是文件的后缀名，右边是用来处理这一类文件的模块名**。**上面代码表示，运行测试之前，先用babel-core/register模块，处理一下.js文件**。由于这里的转码器安装在项目内，所以要使用项目内安装的Mocha；如果转码器安装在全局，就可以使用全局的Mocha。

下面是另外一个例子，使用Mocha测试CoffeeScript脚本。测试之前，先将.coffee文件转成.js文件。

	$ mocha --compilers coffee:coffee-script/register

**注意，Babel默认不会对Iterator、Generator、Promise、Map、Set等全局对象，以及一些全局对象的方法（比如Object.assign）转码。如果你想要对这些对象转码，就要安装babel-polyfill。**

	$ npm install babel-polyfill --save
然后，在你的脚本头部加上一行。

	import 'babel-polyfill'


----------

## 异步测试 ##

Mocha默认每个测试用例最多执行**2000毫秒**，如果到时没有得到结果，就报错。对于涉及异步操作的测试用例，这个时间往往是不够的，需要用`-t或--timeout`参数指定超时门槛。

	it('测试应该5000毫秒后结束', function(done) {
	  var x = true;
	  var f = function() {
	    x = false;
	    expect(x).to.be.not.ok;
	    done(); // 通知Mocha测试结束
	  };
	  setTimeout(f, 4000);
	});
上面的测试用例，需要4000毫秒之后，才有运行结果。所以，**需要用-t或--timeout参数，改变默认的超时设置。**

	$ mocha -t 5000 timeout.test.js

上面命令将测试的超时时限指定为5000毫秒。
另外，上面的测试用例里面，有一个`done`函数。`it`块执行的时候，传入一个`done`参数，**当测试结束的时候，必须显式调用这个函数**，告诉Mocha测试结束了。否则，Mocha就无法知道，测试是否结束，会一直等到超时报错。你可以把这行删除试试看。
Mocha默认会高亮显示超过75毫秒的测试用例，可以用`-s或--slow`调整这个参数。

	$ mocha -t 5000 -s 1000 timeout.test.js

***上面命令指定高亮显示耗时超过1000毫秒的测试用例。***

下面是另外一个异步测试的例子async.test.js。

	it('异步请求应该返回一个对象', function(done){
	  request
	    .get('https://api.github.com')
	    .end(function(err, res){
	      expect(res).to.be.an('object');
	      done();
	    });
	});
运行下面命令，可以看到这个测试会通过。

	$ mocha -t 10000 async.test.js
另外，**Mocha内置对Promise的支持，允许直接返回Promise，等到它的状态改变，再执行断言，而不用显式调用done方法**。请看promise.test.js。

	it('异步请求应该返回一个对象', function() {
	  return fetch('https://api.github.com')
	    .then(function(res) {
	      return res.json();
	    }).then(function(json) {
	      expect(json).to.be.an('object');
	    });
	});

----------

## 测试用例的钩子 ##

Mocha在`describe`块之中，提供测试用例的四个钩子：**before()、after()、beforeEach()和afterEach()**。它们会在指定时间执行。

	describe('hooks', function() {
	
	  before(function() {
	    // 在本区块的所有测试用例之前执行
	  });
	
	  after(function() {
	    // 在本区块的所有测试用例之后执行
	  });
	
	  beforeEach(function() {
	    // 在本区块的每个测试用例之前执行
	  });
	
	  afterEach(function() {
	    // 在本区块的每个测试用例之后执行
	  });
	
	  // test cases
	});

例一：

	// beforeEach.test.js
	describe('beforeEach示例', function() {
	  var foo = false;
	
	  beforeEach(function() {
	    foo = true;
	  });
	
	  it('修改全局变量应该成功', function() {
	    expect(foo).to.be.equal(true);
	  });
	});
上面代码中，beforeEach会在it之前执行，所以会修改全局变量。

另一个例子：

	// beforeEach-async.test.js
	describe('异步 beforeEach 示例', function() {
	  var foo = false;
	
	  beforeEach(function(done) {
	    setTimeout(function() {
	      foo = true;
	      done();
	    }, 50);
	  });
	
	  it('全局变量异步修改应该成功', function() {
	    expect(foo).to.be.equal(true);
	  });
	});

----------

## 浏览器测试 ##

除了在命令行运行，Mocha还可以在浏览器运行。

首先，使用**mocha init**命令在指定目录生成初始化文件。

	$ mocha init demo08
运行上面命令，就会在目录下生成**index.html**文件，**以及配套的脚本和样式表**。

	<!DOCTYPE html>
	<html>
	  <body>
	    <h1>Unit.js tests in the browser with Mocha</h1>
	    <div id="mocha"></div>
	    <script src="mocha.js"></script>
	    <script>
	      mocha.setup('bdd');
	    </script>
	    <script src="tests.js"></script>
	    <script>
	      mocha.run();
	    </script>
	  </body>
	</html>

然后，新建一个源码文件add.js。

	// add.js
	function add(x, y) {
	  return x + y;
	}
然后，把这个文件，以及断言库chai.js，加入index.html。

	<script>
	  mocha.setup('bdd');
	</script>
	<script src="add.js"></script>
	<script src="http://chaijs.com/chai.js"></script>
	<script src="tests.js"></script>
	<script>
	  mocha.run();
	</script>
最后，在tests.js里面写入测试脚本。

	var expect = chai.expect;
	
	describe('加法函数的测试', function() {
	  it('1 加 1 应该等于 2', function() {
	    expect(add(1, 1)).to.be.equal(2);
	  });
	
	  it('任何数加0等于自身', function() {
	    expect(add(1, 0)).to.be.equal(1);
	    expect(add(0, 0)).to.be.equal(0);
	  });
	});
现在，在浏览器里面打开**index.html**，就可以看到测试脚本的运行结果。


----------














