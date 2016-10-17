---
title: chrome disable-web-security 解决跨域
date: 2016-10-17 09:55:19
tags: [chrome, 跨域]
---

参考：*[chrome disable-web-security 关闭安全策略 解决跨域](http://www.cnblogs.com/zhongxia/p/5416024.html)*

前后端分离之后，联调的时候就会出现问题，那就是Ajax跨域问题。 跨域问题的解决方案有很多种：

比如常规的 后端使用CORS，设置允许访问接口的地址 或者 使用 JSONP等等。
这里就不说前端常规的跨域解决方案，而是提供一个奇葩的方案，简单到哭!

![chrome](http://www.uimaker.com/uploads/allimg/130610/1_130610215821_1.jpg)

<!--more-->

### 经测试，发现 chrome53 使用在 chrome快捷方式 的属性里面 添加 --disable-web-security --user-data-dir  不起作用。需要使用在命令行下打开

-----

## 一、命令行打开方式

	"C:\Users\UserName\AppData\Local\Google\Chrome\Application\chrome.exe" --disable-web-security --user-data-dir
//不知道chrome.exe 地址的话
 
	通过命令行启动chrome：
	
	open -a "Google Chrome" --args --disable-web-security  --user-data-dir

	chrome 48 命令行启动不支持设置跨域了,想要跨域,还需要需要在加上 —user-data-dir

 
## 前后端跨域解决方案
可以通过使用chrome命令行启动参数来改变chrome浏览器的设置，具体的启动参数说明参考这篇介绍。
https://code.google.com/p/xiaody/wiki/ChromiumCommandLineSwitches

这里介绍的是--disable-web-security参数。这个参数可以降低chrome浏览器的安全性，禁用同源策略，利于开发人员本地调试。

这里提供一个更简单的跨域解决方案， 设置Chrome浏览器的 disable-web-security, 实现跨域访问后端的接口。

具体操作步骤如下：
#### windows系统（Win7)下的Chrome
1、关闭所有打开的Chrome。（重要）。否则，将没有效果！

2、创建Chrome的快捷方式，修改快捷方式的目标为：
"C:\Program Files\Google\Chrome\Application\chrome.exe" --args --disable-web-security

3、双击我们创建的Chrome快捷方式，打开Chrome，如图出现“您使用的是不受支持的命令行标记：--disable-web-security。稳定性和安全性会有所下降”，表示你取消了跨域限制了，可以随意跨域调用数据了。
如图：您使用的是不受支持的命令行标记：--disable-web-security。稳定性和安全性会有所下降

#### Mac os 下面用
 
	/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --disable-web-security
	或者
	open -a "Google Chrome" --args --disable-web-security
	Ubuntu?Linux:
	 
	chromium-browser --disable-web-security
	用命令行打开 Apple Safafi 方法是：（Mac OS 下）
	 
	open -a '/Applications/Safari.app' --args --disable-web-security	
	 