# BlogSourceFile
基于Hexo构建博客所需的源文件

----------

> **BlogSourceFile**库存储的是构建博客所需的原文件，用于在不同机器上进行备份克隆，随时更新博客；

> **etc1208.github.io**库存储的是基于Hexo生成的博客文件；

> 假如要在一台新机器上更新博客，首先将此库`clone`到本地；

> 之后在本地执行`npm install`安装所需依赖后便可新建博文`（hexo new "xxx"）`或修改原来的博文（在目录source\_posts下的.md文件）

> 新建/修改博文后，执行 ` hexo g, hexo d` 两条命令即可生成目标文件并部署到etc1208.github.io上；

> 最后不要忘记同步本地修改过的文件到BlogSourceFile远程仓库；