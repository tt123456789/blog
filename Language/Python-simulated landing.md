Title: python 模拟登陆
Date: 15:43 2016/9/10
Modified: 13:48 2016/9/21
Category: language
Tags: Python
Slug: 
Author: allposs
##适用环境

操作系统：python3*

##版本号
V1.0

##说明（README.md）

###1.基本概述

&#160; &#160; &#160; &#160;模拟模拟登陆是由两个文件组成，一个为模拟登陆主程序，一个为帐号密码存储文件。

###2.技巧运用

&#160; &#160; &#160; &#160;其中运用了字典的读取与转化，并利用格式转化功能对读取到的文件进行格式化，并运用了for循环与if循环

###3.程序运行概述

&#160; &#160; &#160; &#160;当运行模拟登陆主程序时，会提示用户输入帐号和密码，当帐号与密码和帐号密码存储文件里数据相匹配就会打印Welcome logon in！，如果配置则进行三次尝试！

##流程图

![](http://image.allposs.cn/20160921133545.png)

##代码
	python代码：
	
	#!/usr/bin/env python
	#_*_coding:utf-8_*_
	#Version:1.0
	#data:

	Counter = 0
	PasswdDictionaries={}
	Input={}
	for i in range (10) :
		if Counter < 3 :
			Counter += 1
			InputUser = input("please enter user name:")
			InputPassword = input("please enter password:")
			InputPassword = InputPassword.strip()
			InputUser = InputUser.strip()
			for line in open("passwd"):
				key, value = line.strip().split(',')
				PasswdDictionaries[key] = value
	#对passwd文件的数据进行格式化并存储到PasswdDictionaries字典里
			if PasswdDictionaries.get(InputUser) and PasswdDictionaries[InputUser] == InputPassword:
	#注：PasswdDictionaries[InputUser]使用，如果InputUser在字典里并没有就会会报keyerror,PasswdDictionaries.get(InputUser)使用并不会报
				print ( "Welcome logon in !")
				exit(1)
			else :
				print ('''User name or password is incorrect , please try again''')

		else :
			exit(2)
	#尝试三次后直接退出			
	passwd文件：
		user,password
		admin,123
		123,admin