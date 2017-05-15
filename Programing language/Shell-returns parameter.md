Title: Shell脚本 返回参数
Date: 2016-05-04 11:20:57 
Modified: 2016-05-04 11:20:57 
Category: Programing language
Tags: Script
Slug: 
Author: allposs


##简介##
&#160; &#160; &#160; &#160;返回值参数主要用于脚本之间数据的传递。

##版本##

任何版本

##正文##

	$$
	Shell本身的PID（ProcessID）
	$!
	Shell最后运行的后台Process的PID
	$?
	最后运行的命令的结束代码（返回值）
	$-
	使用Set命令设定的Flag一览
	$*
	所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
	$@
	所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
	$#
	添加到Shell的参数个数
	$0
	Shell本身的文件名
	$1～$n
	添加到Shell的各参数值。$1是第1参数、$2是第2参数…。


