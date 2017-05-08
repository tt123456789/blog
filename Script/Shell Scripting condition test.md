Title: Shell脚本 条件测试
Date: 2016/5/13 20:00:40  
Modified: 2016/5/13 20:00:44  
Category: Shell
Tags: Script
Slug: 
Author: allposs


##简介##
&#160; &#160; &#160; &#160;Linux 的Shell中存在一组测试命令，该组命令用于测试某种条件或某几种条件是否真实存在。测试命令是判断语句和循环语句中条件测试工具，所以，其对于编写Shell非常重要

##版本##

任何linux版本

##正文##



###1. 测试方法：

 	1、_[_expression_]_；expression是一个表达式，该表达式可为数字、字符串、文本和文件属性的比较，同时可同时加入各种算术、字符串、文本等运算符。

                _[_expression_]_中“[”是启动测试命令，_表示空格。使用该命令要特别注意“[”后和“]”前的空格必不可少。这种方法比较常用。

	2、_[[_expression_]]_

                _[[_expression_]]_与第一个方法差不多，具体方法以后补充。

	3、test expresssion

               test比较简单，test空格后面接表达式即可。

###2. 整数测试运算符：

    	-gt 格式：test num1 -gt num2 或 [ num1 -gt num2 ] (注意空格)  含义：测试num1是否大于num2
    	-eq 格式：test num1 -eq num2 或 [ num1 -eq num2 ] (注意空格)  含义：测试num1是否等于num2
    	-ge 格式：test num1 -ge num2 或 [ num1 -ge num2 ] (注意空格)  含义：测试num1是否大于或等于num2
    	-le 格式：test num1 -le num2 或 [ num1 -le num2 ] (注意空格)  含义：测试num1是否小于或等于num2
    	-lt  格式：test num1 -lt num2 或 [ num1 -lt num2 ] (注意空格)  含义：测试num1是否小于num2
    	-ne  格式：test num1 -ne num2 或 [ num1 -ne num2 ] (注意空格)  含义：测试num1是否不等于num2

注：整数比较运算符不适用于浮点型数值比较。

###3.文件测试运算符

    -e  格式：test -e filename 或 [ -e filename ] (注意空格)  含义：测试filename文件是否存在
    -f  格式：test -f filename 或 [ -f filename ] (注意空格)  含义：测试filename文件是否是普通文件
    -d  格式：test -d filename 或 [ -d filename ] (注意空格)  含义：测试filename文件是否是目录
    -r  格式：test -r filename 或 [ -r filename ] (注意空格)  含义：测试filename文件对于当前用户是否有可读权限
    -w  格式：test -w filename 或 [ -w filename ] (注意空格)  含义：测试filename文件对于当前用户是否有可写权限
    -x  格式：test -x filename 或 [ -x filename ] (注意空格)  含义：测试filename文件对于当前用户是否有可执行权限













