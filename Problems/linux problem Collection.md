Title: linux系统问题合集
Date: 15:20 2016/11/5 
Modified: 15:20 2016/11/5 
Category: Problems
Tags: 问题合集
Slug: 
Author: allposs
##问题一##
###环境###
+ 操作系统：RedHat6.5 X86_64
+ Yum源：本地源
+ IP地址：172.17.117.24
+ DNS：无
+ 主机名：tianlog_pda03
+ URL：

###现象###

	发现主机df -h 显示根目录为80%，但是使用du -sh 却发现根目录用量少得可怜，根本达不到80%

###错误日志###

	无

###分析###

	刚开始检查tomcat的日志目录，发现其中有64G大小的日志文件，使用rm -rf删除不行，多删除几次并用file Transfer查看文件，发现文件消息，而现象依旧，这个时候就可以用lsof列出当前系统打开文件，发现删除的文件是被打开状态，问题显而意见

###解决###
	使用lsof工具查看当前系统打开的文件并过滤相关文件
	[root@tianlong_pda03 tomcat_pda03]# lsof | grep catalina.out
	杀死相关进程（这里是tomcat）
	[root@tianlong_pda03 tomcat_pda03]# kill -9 9199
	使用df查看根目录，发现恢复
