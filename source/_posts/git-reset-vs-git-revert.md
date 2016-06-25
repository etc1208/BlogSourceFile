---
title: git reset vs. git revert
date: 2016-06-25 13:32:09
tags: [git, github]
---

在使用git和远程仓库时经常因为误操作或后来想要回退/回滚/取消提交/返回上一版本，我们需要用到git reset 和 git revert两个命令。
![git](http://s3.51cto.com/wyfs02/M01/74/4A/wKioL1YYooWw4lxSAADhaps5s7s012.jpg)

<!--more-->

----------

当你遇到下面的问题.

- (1)改完代码匆忙提交,上线发现有问题,怎么办? 赶紧回滚.

- (2)改完代码测试也没有问题,但是上线发现你的修改影响了之前运行正常的代码报错,必须回滚.


大致分为下面2种情况:



## 1.没有push到远程仓库 ##

这种情况发生在你的本地代码仓库,可能你`add` ,`commit` 以后发现代码有点问题,准备取消提交,用到下面命令

	git reset [--soft | --mixed | --hard


上面常见三种类型



**--mixed**

会保留源码,只是将git commit和index 信息回退到了某个版本.

> git reset 默认是 --mixed 模式 

> git reset --mixed  等价于  git reset

**--soft**

保留源码,只回退到commit 信息到某个版本.不涉及index的回退,如果还需要提交,直接commit即可.

**--hard**

源码也会回退到某个版本,commit和index 都回回退到某个版本.(**注意,这种方式是改变本地代码仓库源码**)

    当然有人在push代码以后,也使用 reset --hard <commit...> 回退代码到某个版本之前,但是这样会有一个问题,你线上的代码没有变,线上commit,index都没有变,当你把本地代码修改完提交的时候你会发现全是冲突.....

所以,这种情况你要使用下面的方式


## 2.已经push到远程仓 ##

对于已经把代码push到线上仓库,你**回退本地代码其实也想同时回退线上代码**,回滚到某个指定的版本,线上,线下代码保持一致.你要用到下面的命令


	git revert     用于反转提交,执行revert命令时要求工作树必须是干净的.

git revert**用一个新提交来消除一个历史提交所做的任何修改**.

####revert 之后你的本地代码会回滚到指定的历史版本,这时你再 git push 既可以把线上的代码更新.(这里不会像reset造成冲突的问题)



###revert 使用:
需要先找到你想回滚版本唯一的**commit标识代码**,可以用 `git log` 查看.

举例：
	
	git revert c011eb33ubv5ybv54b895gb893b4
	
	通常,前几位即可
	git revert c011eb3

---------

- git revert是用一次新的commit来回滚之前的commit
- git reset是直接删除指定的commit

## 区别： ##


1. 如果你已经push到远程仓库, reset 删除指定commit以后,你git push可能导致一大堆冲突.但是revert 并不会.



2. 如果现有分支和历史分支需要合并的时候,reset 的代码依然会出现在历史分支里.但是revert 提交的commit 并不会出现在历史分支里.



3. reset 是在正常的commit历史中删除了指定的commit,这时 HEAD 是向后移动了,而 revert 是在正常的commit历史中再commit一次,只不过是反向提交,他的 HEAD 是一直向前的.


## 补充：github远程仓库上如何删除一个文件夹？ ##

如果直接`git rm`：则本地的文件夹也被删除，应该删缓冲。

这样合理：

	git rm -r --cached some-directory
	git commit -m "xxxxxx"
	git push ....
