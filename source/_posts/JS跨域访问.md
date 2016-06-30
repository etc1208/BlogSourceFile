---
title: JS跨域访问
date: 2016-06-30 17:28:26
tags: [跨域, javaScript, 前端]
---
同协议，domain（或ip），同端口视为同一个域，一个域内的脚本仅仅具有本域内的权限，可以理解为本域脚本只能读写本域内的资源，而无法访问其它域的资源。这种安全限制称为同源策略。

![JS](http://i1.51cto.com/images/201507/3739fcd798f1b32a155886400d40cd297c4c12.jpg)

<!--more-->


----------

## jsonp ##

**在javascript中，我们不能直接用ajax请求不同域上的数据。但是，在页面上引入不同域上的js脚本文件却是可以的，jsonp正是利用这个特性来实现的.**


> 我们启动两个服务器，分别监听9000端口和9001端口，这样两个地之间互相访问数据就构成了跨域。

在9001下jsonp_test.html中


	<!DOCTYPE html>
	<html>
	<head>
	    <title>jsonp-test</title>
	</head>
	<body>
	    <script type="text/javascript">
	        function callback_data (data) {
	            console.log(data);
	        }
	    </script>
	    <script type="text/javascript" src="http://127.0.0.1:9000/jsonp.php?callback=callback_data"></script>
	</body>
	</html>

可以看到我们在向9000目录下的jsonp.php文件获取数据时，地址后面跟了一个callback参数。

在9000目录下jsonp.php中

	<?php
	    $callback = $_GET['callback'];  // 获取回调函数名
	    $arr = array("name" => "alsy", "age" => "20"); // 要请求的数据
	    echo $callback."(". json_encode($arr) .");"; // 输出
	?>

这样我们就获取到不同域中返回的数据了，同时jsonp的原理也就清楚了：

    通过script标签引入一个js文件，这个js文件载入成功后会执行我们在url参数中指定的函数，同时把我们需要的json数据作为参数传入。所以jsonp是需要服务器端和客户端相互配合的。

知道jsonp跨域的原理后我们就可以用js动态生成script标签来进行跨域操作了，而不用特意的手动的书写那些script标签。比如jQuery封装的方法就能很方便的来进行jsonp操作了。

9001目录下的html中：

	//$.getJSON()方法跨域请求
	$.getJSON("http://127.0.0.1:9000/jsonp.php?callback=?", function(data){
	    console.log(data);
	});

原理是一样的，只不过我们不需要手动的插入script标签以及定义回掉函数。jQuery会自动生成一个全局函数来替换callback=?中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。


	$.getJSON方法会自动判断是否跨域，不跨域的话，就调用普通的ajax方法；跨域的话，则会以异步加载js文件的形式来调用jsonp的回调函数。

**另外jsonp是无法post数据的，尽管jQuery.getJSON(url,[data],[callback]); 提供data参数让你可以携带数据发起请求，但这样是以get方式传递的。或者是调用$.ajax()方法指定type为post，它还是会转成get方式请求。**

*以get形式的话是可以携带少量数据，但是数据量一大就不行了。*

网上找到一种方法，直接调用ajax方法，设置crossDomain(跨域): true，服务器响应头设置

    header('Access-Control-Allow-Origin: *');
    header('Access-Control-Allow-Methods: POST');
    header('Access-Control-Max-Age: 1000');
比如：

9001目录下的一个html文件：

	$.ajax({
	    type: 'post',
	    url: "http://127.0.0.1:9000/jsonp_post.php",
	    crossDomain: true,
	    data: {u: 'alsy', age: 20},
	    dataType: "json",
	    success: function(r){
	        console.log(r);
	    }
	});

9000目录下的php文件：

	<?php
	    header('Access-Control-Allow-Origin: *');
	    header('Access-Control-Allow-Methods: POST');
	    header('Access-Control-Max-Age: 1000');
	    if($_POST){
	        $arr = array('name' => $_POST['u'], 'age' => $_POST['age']);
	        echo json_encode($arr);
	    } else {
	        echo json_encode([]);
	    }
	?>

----------

## iframe+window.domain 实现跨子域 ##

在页面中存在一个不同域的框架（iframe），而iframe载入的页面是和目标域在同一个域的，是能够向目标域发起ajax请求获取数据的。

那么就想能不能控制这个iframe让它去发起ajax请求，但在同源策略下，不同域的框架之间也是不能够进行js的交互。

在这个时候，document.domain就派上用场了，我们只要两个域的页面的document.domain设置成一样了，这个例子中由于端口不同，两边的document.domain也要重新设置成"127.0.0.1"，才能正常通信。

要注意的是，document.domain的设置是有限制的，我们只能把document.domain设置成自身或更高一级的父域，主域必须相同。

另举例：a.b.example.com 中某个文档的document.domain 可以设成a.b.example.com、b.example.com 、example.com中的任意一个；

但是不可以设成c.a.b.example.com，因为这是当前域的子域，也不可以设成baidu.com，因为主域已经不相同了。

通过设置document.domain为相同，现在已经能够控制iframe载入的页面进行ajax操作了。


----------

## iframe+window.name实现跨域 ##

window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内，窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限。（window.name的值只能是字符串的形式，这个字符串的大小最大能允许2M左右甚至更大的一个容量，具体取决于不同的浏览器，一般够用了。）

----------


## Access Control ##

Access Control是比较超越的跨域方式，目前只在很少的浏览器中得以支持，这些浏览器可以发送一个跨域的HTTP请求（Firefox, Google Chrome等通过XMLHTTPRequest实现，IE8下通过XDomainRequest实现），请求的响应必须包含一个Access- Control-Allow-Origin的HTTP响应头，该响应头声明了请求域的可访问权限。例如www.a.com对www.b.com下的 asset.php发送了一个跨域的HTTP请求，那么asset.php必须加入如下的响应头：

	header("Access-Control-Allow-Origin: http://www.a.com");












