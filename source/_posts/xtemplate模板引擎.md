---
title: xtemplate模板引擎
date: 2016-06-21 16:07:34
tags: [模板引擎, xtemplate, 前端]
---
转自：[http://book.apebook.org/minghe/koa-action/xtemplate/index.html](http://book.apebook.org/minghe/koa-action/xtemplate/index.html)
####适用于 koa 的模板引擎选择非常多，比如 jade、ejs、nunjucks、xtemplate 等。

![](http://media02.hongkiat.com/wallpapers-coder-geek-designer/code-geek-designer-wallpapers.jpg)

<!--more-->
# 几个模板引擎的对比 #

渲染html到页面中，在koa中可以这么干：

	app.get('/',function*(){
	    this.body = "<p>'+title+'</p>";
	})
但问题很大，模板混入到js逻辑了，非常不利于维护，基于 mvc 模式，我们需要将html模板抽离到 view 中方便维护，这时候我们就需要一款模板渲染引擎。

ejs、nunjucks、xtemplate 的上手难度差不多，但ejs的扩展写法有些诡异，特别是 filter 的写法：

	<p><%=: users | first | capitalize %></p>
xtemplate 相对于 nunjucks 的优势是，模板的逻辑写法体验更接近js（ejs的逻辑表达也很接近js），上手更为简单，来看下同样一个 if 判断的写法差异。

xtemplate:

	{{#if(variable===0)}}
	  It is true    
	{{/if}}
nunjucks:

	{% if variable == 0 %}
	  It is true
	{% endif %}
**所以上手难度：xtemplate < nunjucks < ejs < jade**

nunjucks 无疑是功能最全面的模板引擎，xtemplate 拥有 nunjucks 大部分的特性，除了filter，但 xtemplate 拥有非常强悍的拓展性：

	xtpl.addCommand('money',function(scope, option){
	    var money = option.params[0];
	    if(typeof money !== 'number') return '';
	    //金额除以100（接口返回的金额都是以分为单位，转成元）
	    var s = Number(money)/100;
	    return s;
	})
模板中使用：

	{{money(10000)}}
nunjucks 与 xtemplate 都拥有实用的宏定义功能：

	{{#macro("test","param", default=1)}}
	    param is {{param}} {{default}}
	{{/macro}}
模板中使用：

	{{macro("test","2")}}
输出内容：

	param is 2 1
jade 中的 Mixins，ejs 中的 function 也可以实现类似的抽取公用html代码块的目的 。

如果真要分个高下：nunjucks > xtemplate > jade > ejs 。

是否支持前后端混用

- jade 直接淘汰，相信在前端js领域一般不会选择 jade 来渲染。

- nunjucks 也使用的比较少（ejs其实也少），更多人会选择使用 handlebars 。

- xtemplate 目前在阿里的系统中前后端混用中已经得到论证，节约了模板前后端转换的时间。

----------

# xtemplate的使用 #
xtemplate 模板引擎在前后端都可以使用，nodejs端使用需要使用` xtpl 模块`。

package.json定义依赖：

    "xtemplate": "4.2.4",
    "xtpl": "3.1.9"
在 koa 中使用

	//xtemplate模板引擎对koa的适配
	var xtpl = require('xtpl/lib/koa');
	//xtemplate模板渲染
	xtpl(app,{
	    //配置模板目录，指向工程的view目录
	    views: config.viewDir
	});
在 view 目录下放一个简单的 test.xtpl 模板（**xtemplate 模板统一使用.xtpl后缀**），内容如下：

    <h2>{{title}}</h2>
写一个简单的 router：

	app.get('/view-test',function *(){
	    yield this.render('test',{"title":"xtemplate demo"});
	})
特别注意，**this.render() 必须使用 yield**，刚开始写会经常忘记掉，需要多留心。

render() 第一个参数为模板路径（无需加.xtpl后缀），比如模板路径为 view/test/a.xtpl ，参数值为 test/a ；第二个参数为传递给引擎处理的数据，demo中 占位变量 会被字段 title 所替换。

-----------------

具体语法参见：[http://book.apebook.org/minghe/koa-action/xtemplate/base.html](http://book.apebook.org/minghe/koa-action/xtemplate/base.html)