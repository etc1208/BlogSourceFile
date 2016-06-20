---
title: Hexo使用笔记
date: 2016-06-20 15:53:46
tags: [随笔,Hexo,笔记]
---
#### HEXO+Github搭建个人博客

<!--more-->

### 配置环境
1. 安装Node（必须）
2. 安装Git（必须）

### 正式安装Hexo

执行如下命令安装Hexo：

    sudo npm install-g hexo

初始化然后，执行init命令初始化hexo,命令：

    hexo init

若安装以来出现错误，手动执行安装：	
    
	npm install

### 配置Github

####建立Repository

建立与你用户名对应的仓库，仓库名必须为【your_user_name.github.io】，固定写法

然后建立关联，需要配置_config.yml文件：

翻到最下面，改成这样子：

    deploy:
    
         type: git
    
         repo:https://github.com/your_user_name/your_user_name.github.io.git
    
         branch:master

然后执行命令：

npm install hexo-deployer-git --save

然后，执行配置命令：

    hexo deploy

然后再浏览器中输入http://your_user_name.github.io/就行了

### 新建文章

    hexo new "xxx"
默认会在source\_posts\目录下生成xxx.md，打开进行编辑即可。

### 部署步骤

每次部署的步骤，可按以下三步来进行。
		
	hexo clean 
    
        hexo generate 或：hexo g
    
        hexo deploy   或：hexo d



