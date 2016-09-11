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
#### 如果此时已经安装有其他版本的Android SDK Build-Tools，记得反选下去！！！只保留23.0.1
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

