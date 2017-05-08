Title: Nginx 问题合集
Date: 2016/5/14 10:42:04 
Modified: 2016/5/14 10:42:06 
Category: problems
Tags: nginx
Slug: 
Author: allposs

##问题一##
###环境###
+ 操作系统：CentOS7.1 X86_64
+ Yum源：自定义源
+ IP地址：10.199.255.201,192.168.200.201 
+ DNS：10.199.255.15
+ 主机名：node1
+ URL：
###现象###

	[root@node1 ~]# /usr/local/nginx/sbin/nginx 
	nginx: [emerg] getpwnam("nginx") failed


###错误日志###

	2016/01/01 04:00:23 [emerg] 2714#0: getpwnam("nginx") failed



###分析###

nginx启动的时候没有相关权限。

###解决###

	[root@node1 ~]# groupadd -r nginx
	[root@node1 ~]# useradd -s /sbin/nologin -g nginx -r nginx

