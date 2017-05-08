Title: Tomcat问题合集
Date: 2016/5/14 10:52:40 
Modified: 2016/5/14 10:52:42 
Category: problems
Tags: tomcat
Slug: 
Author: allposs


##问题一
###环境
+ 操作系统：CentOS7.1 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：node1
+ URL：

###现象
运行java -version出现如下错误：

	error :/usr/lib/libjvm.so: cannot restore segment prot after reloc: Permission denied .
	use the command:

###错误日志

	error :/usr/lib/libjvm.so: cannot restore segment prot after reloc: Permission denied

###分析

selinux的安全策略导致

###解决

	chcon -t textrel_shlib_t /usr/lib/libjvm.so


