title: mac遇到的一个svn的问题
date: 2015-10-29 13:34:37
tags: svn
categories: 开发工具
---
今天在mac使用svn pe svn:ignore 来设置忽略文件夹时出现了下面的错误
```
svn: E205007: None of the environment variables SVN_EDITOR, VISUAL or EDITOR are set, and no 'editor-cmd' run-time configuration option was found
```
解决办法，在.bash_profile中增加下面一行，然后source生效即可
```
export SVN_EDITOR=vim
```
注意使用
```
svn pe svn:global-ignores .
```
进行全局设置，避免给每个目录执行
```
svn pe svn:ignore .
```
来设置
