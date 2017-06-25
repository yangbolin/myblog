title: 谈谈几个svn命令
date: 2014-05-10 16:20:24
tags: svn
categories: 开发工具
---

####一.概述
很多公司目前都在使用svn来管理代码，当然要是你的开发环境不是Linux或许你不知到这几个命令你一样可以开发，但是要是你的开发环境是Linux，那你就需要了解这几个开发命令。关于代码的签出，提交我们就暂时不用考虑了，这些命令太简单。

<!-- more -->

####二.命令

* svn pg svn:ignore 目录
这个命令可以获取到目录下面那些文件不会被提交到svn服务器上

* svn pe svn:ignore 目录
这个命令可以设置目录下面的那些文件不会被提交到svn服务器上

* svn pg svn:externals 目录
这个命令专门用来解决分支A依赖分支B的场景，同时被依赖的分支会部署这里指定的目录下面

* svn pe svn:externals 目录
这个目录用来设置当前分支需要依赖的分支

* svn revert file
要是我们add了一个文件，但是还么有ci，此时如果我们不想ci这个文件了，我们就可以使用这个命令把这个文件的add取消掉。

* svn revert --depth=infinity 文件夹
功能同上，不过带上--depth参数就可以取消对整个文件夹的add

* svn merge
这个命令也经常会用到，我们在处理代码冲突的时候，经常需要把自己分支上的代码和主干合并
svn merge -rM:N svn-branch
M必须小于N,M和N都是svn-branch的变更版本，通常我们都会写成下面这样
svn merge -rVERSION:HEAD svn-branch
这里的VERSION我们可以选择svn-branch变更记录中的任何一个版本

* svn cleanup
这个命令从名称上来看是用作清理的，比如当使用svn st查看文件变更时，发现文件前面的标识为L标识当前文件被锁定了，要想解除这个锁定，就得使用svn cleanup这个命令。

* svn propdel svn:externals .
删除svn分支的externals属性

* svn propedit -r N --revprop svn:log URL 
修改提交日志，N表示提交的版本

* svn propset -r N --revprop svn:log "new log message" URL
修改提交日志，N表示提交的版本