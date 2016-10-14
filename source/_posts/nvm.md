---
title: nvm
date: 2016-10-14 17:17:46
tags: [node, nvm]
---

参考：[nvm和nodejs安装使用](http://www.kancloud.cn/summer/nodejs-install/71975), [使用 nvm 管理不同版本的 node 与 npm](http://www.tuicool.com/articles/Vzquy2)

如果你想长期做 node 开发, 或者想快速更新 node 版本, 或者想快速切换 node 版本,那么在非 Windows(如 osx, linux) 环境下, 请使用 nvm 来安装你的 node 开发环境, 保持系统的干净.
如果你使用 Windows 做开发, 那么你可以使用 nvmw 来替代 nvm

![nvm](http://img0.imgtn.bdimg.com/it/u=1107945347,1019779897&fm=21&gp=0.jpg)

<!--more-->

## git clone nvm

直接从 github clone nvm 到本地, 这里假设大家都使用 ~/git 目录存放 git 项目:

	$ cd ~/git
	$ git clone https://github.com/creationix/nvm.git
配置终端启动时自动执行 source ~/git/nvm/nvm.sh,

在 ~/.bashrc, ~/.bash_profile, ~/.profile, 或者 ~/.zshrc 文件添加以下命令:

	source ~/git/nvm/nvm.sh
重新打开你的终端, 输入 nvm

	$ nvm
	
	Node Version Manager
	
	Usage:
	    nvm help                    Show this message
	    nvm --version               Print out the latest released version of nvm
	    nvm install [-s] <version>  Download and install a <version>, [-s] from source
	    nvm uninstall <version>     Uninstall a version
	    nvm use <version>           Modify PATH to use <version>
	...
## 通过 nvm 安装任意版本的 node

nvm 默认是从 http://nodejs.org/dist/ 下载的, 国外服务器, 必然很慢,好在 nvm 以及支持从镜像服务器下载包, 于是我们可以方便地从七牛的 node dist 镜像下载:

	$ NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install 4

如果你不想每次都输入环境变量 NVM_NODEJS_ORG_MIRROR, 那么我建议你加入到 .bashrc 文件中:

	# nvm
	export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
	source ~/git/nvm/nvm.sh
然后你可以继续非常方便地安装各个版本的 node 了, 你可以查看一下你当前已经安装的版本:

	$ nvm ls
	         nvm
	     v0.8.26
	    v0.10.26
	    v0.11.11
	->  v4.3.2
	
## 安装切换各版本 node/npm

	nvm install stable #安装最新稳定版 node，现在是 5.0.0
	nvm install 4.2.2 #安装 4.2.2 版本
	nvm install 0.12.7 #安装 0.12.7 版本

#### 特别说明：以下模块安装仅供演示说明，并非必须安装模块
	nvm use 0 #切换至 0.12.7 版本
	npm install -g mz-fis #安装 mz-fis 模块至全局目录，安装完成的路径是 /Users/<你的用户名>/.nvm/versions/node/v0.12.7/lib/mz-fis
	nvm use 4 #切换至 4.2.2 版本
	npm install -g react-native-cli #安装 react-native-cli 模块至全局目录，安装完成的路径是 /Users/<你的用户名>/.nvm/versions/node/v4.2.2/lib/react-native-cli
	
	nvm alias default 0.12.7 #设置默认 node 版本为 0.12.7
## 使用 .nvmrc 文件配置项目所使用的 node 版本

如果你的默认 node 版本（通过 nvm alias 命令设置的）与项目所需的版本不同，则可在项目根目录或其任意父级目录中创建 .nvmrc 文件，在文件中指定使用的 node 版本号，例如：

	cd <项目根目录>  #进入项目根目录
	echo 4 > .nvmrc #添加 .nvmrc 文件
	nvm use #无需指定版本号，会自动使用 .nvmrc 文件中配置的版本
	node -v #查看 node 是否切换为对应版本
## nvm 与 n 的区别

node 版本管理工具还有一个是 TJ 大神的 n 命令，n 命令是作为一个 node 的模块而存在，而 nvm 是一个独立于 node/npm 的外部 shell 脚本，因此 n 命令相比 nvm 更加局限。

由于 npm 安装的模块路径均为 /usr/local/lib/node_modules ，当使用 n 切换不同的 node 版本时，实际上会共用全局的 node/npm 目录。 因此不能很好的满足『按不同 node 版本使用不同全局 node 模块』的需求。