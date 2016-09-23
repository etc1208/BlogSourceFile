---
title: 记录React Native的踩坑之旅
date: 2016-09-11 21:35:59
tags: [ReactNative]
---

刚刚结束了在有道第一个月的实习生活。从今天起，要在姐的光辉照耀下好好踩RN的坑...

从今天搭环境开始慢慢记录吧....(世界上本没有坑，RN做多了便有了坑)

![React Native](http://static.open-open.com/lib/uploadImg/20160412/20160412193341_428.png)

<!--more-->

搭建ios和android的开发环境主要参考了官方指南：http://reactnative.cn/docs/0.31/getting-started.html#content

记一些重点的地方：

- React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。

		npm install -g react-native-cli
		
安装时可能遇到权限问题，记得加sudo...

- ios的搭建到还好，按着文档的步骤和注意事项基本可以搞定；搭好后执行命令：

		react-native run-ios 
即可启动ios项目

- ### android就处处是坑了。。。。

按照官方文档的指示，注意提到的需要安装的一些列东东，
- Android Studio2.0以上
- JDK1.8以上
- 安装完成后，在Android Studio的启动欢迎界面中选择Configure | SDK Manager。在SDK Platforms窗口中，选择Show Package Details，然后在Android 6.0 (Marshmallow)中勾选Google APIs、Intel x86 Atom System Image、Intel x86 Atom_64 System Image以及Google APIs Intel x86 Atom_64 System Image。
- 在SDK Tools窗口中，选择Show Package Details，然后在Android SDK Build Tools中勾选Android SDK Build-Tools 23.0.1。（必须是这个版本）

- 安卓需要配置环境变量如下：向~/.bash_profile文件中添加：

		export ANDROID_HOME=~/Library/Android/sdk
	
		export PATH=${PATH}:~/Library/Android/sdk/platform-tools
	
		export PATH=${PATH}:~/Library/Android/sdk/tools
 
- 步骤一：打开一个终端：输入android avd: 创建并start一个安卓模拟器
- 步骤二：打开另一个终端：进入项目目录后输入react-native run-android即可启动安卓项目

#### （以上两步顺序不能倒，必须先开启一个且只能一个模拟器）
 
#### 注意：若提示zsh: command not found: adb ，执行命令 source ~/.bash_profile(再执行adb:即可看到输出结果)

搭好环境后执行：

	react-native init xxxxx
即可初始化生成一个RN项目结构，项目名为xxxxx。

-------

## RN----导入组件，import from 'xxxx'的用法详解
就挑几种在RN中，特别是初学期常用的方式来说说：

	1.import React ,{ Component } from 'react';

这是RN 0.26后导入React的方式，这意思是，导入‘react’文件里export的一个默认的组件，将其命名为React以及Component这个非默认组件


	2.import Home from './compoments/Home';

这是导入‘compoments/Home’文件里export的带default关键字的组件，即默认组件,将其命名为Home(可以自定义命名)

	3.import { Home } from './compoments/Home';

导入‘compoments/Home’文件里export的叫Home的非默认组件，注意，非默认，以及命名Home

	4.import { Home , Discover } from './compoments/Home';
跟3的差不多，不过是{  },可以导入多个组件，用，隔开就可以

	5.import * as Home from'./compoments/Home';

意思是将./compoments/Home'文件里的所有非默认组件，全部集结成一个Home模型组件，命名可以自定义，然后可以通过点语法，来使用组件里面的所有export的组件

--------

## 高度与宽度

#### 指定宽高

最简单的给组件设定尺寸的方式就是在样式中指定固定的width和height。React Native中的尺寸都是无单位的，表示的是与设备像素密度无关的逻辑像素点。

#### 弹性（Flex）宽高

在组件样式中使用flex可以使其在可利用的空间中动态地扩张或收缩。一般而言我们会使用flex:1来指定某个组件扩张以撑满所有剩余的空间。如果有多个并列的子组件使用了flex:1，则这些子组件会平分父容器中剩余的空间。如果这些并列的子组件的flex值不一样，则谁的值更大，谁占据剩余空间的比例就更大（即占据剩余空间的比等于并列组件间flex值的比）。

#### ！！！组件能够撑满剩余空间的前提是其父容器的尺寸不为零。如果父容器既没有固定的width和height，也没有设定flex，则父容器的尺寸为零。其子组件如果使用了flex，也是无法显示的。

	  render() {
	    return (
	      // 试试去掉父View中的`flex: 1`。
	      // 则父View不再具有尺寸，因此子组件也无法再撑开。
	      // 然后再用`height: 300`来代替父View的`flex: 1`试试看？
	      <View style={{flex: 1}}>
	        <View style={{flex: 1, backgroundColor: 'powderblue'}} />
	        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
	        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
	      </View>
	    );
	  }

--------

## 使用Flexbox布局
我们在React Native中使用flexbox规则来指定某个组件的子元素的布局。Flexbox可以在不同屏幕尺寸上提供一致的布局结构。

- 一般来说，使用flexDirection、alignItems和 justifyContent三个样式属性就已经能满足大多数布局需求。

React Native中的Flexbox的工作原理和web上的CSS基本一致，当然也存在少许差异。

- 首先是默认值不同：flexDirection的默认值是column而不是row，alignItems的默认值是stretch而不是flex-start，以及flex只能指定一个数字值。

#### Align Items

在组件的style中指定alignItems可以决定其子元素沿着次轴（与主轴垂直的轴，比如若主轴方向为row，则次轴方向为column）的排列方式。对应的这些可选项有：flex-start、center、flex-end以及stretch。

#### 注意：要使stretch选项生效的话，子元素在次轴方向上不能有固定的尺寸。

--------

- ListView更适于长列表数据，且元素个数可以增删。和ScrollView不同的是，ListView并不立即渲染所有元素，而是优先渲染屏幕上可见的元素。

#### 很多要在App中显示的图片并不能在编译的时候获得，又或者有时候需要动态载入来减少打包后的二进制文件的大小。这些时候，与静态资源不同的是，你需要手动指定图片的尺寸。

	// 正确
	<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}
	       style={{width: 400, height: 400}} />
	
	// 错误
	<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}} />


