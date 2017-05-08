Title: zabbix问题合集
Date: 2016/5/14 9:29:33 
Modified: 2016/5/14 9:29:36 
Category: Problems
Tags: 问题合集
Slug: 
Author: allposs
##问题一##
###环境###
+ 操作系统：RedHat7.1 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：Monitor
+ URL：

###现象###

	在安装zabbix中，会报“zabbix Error connecting to database: No such file or directory"

###错误日志###

	zabbix Error connecting to database: No such file or directory

###分析###



###解决###
注意mariadb的dbtabase的权限，然后操作如下：

cd /usr/local/MariaDB/lib/注意系统版本同，文件版本可能不一样，添加软链接：


	[root@moniter lib]# ln -s libmysqlclient.so.18  libmysqlclient.so
	[root@moniter lib]# ln -s libmysqlclient.so.18.0.0  libmysqlclient_r.so

如果是yum安装的只需要在/usr/local/lib64或/usr/local/lib下面链接





##问题二##
###环境###
+ 操作系统：RedHat6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.255.200
+ DNS：10.199.255.15
+ 主机名：Monitor
+ URL：

###现象###

	日志或提示报错：1904:20150922:085003.340 fping failed: (null): can't create socket (must run as root?) : Permission denied

###错误日志###

	1904:20150922:085003.340 fping failed: (null): can't create socket (must run as root?) : Permission denied

###分析###

因为fping权限不够，需要更改权限

###解决###
修改fping权限

	[root@Moniter ~]# chown root:root /usr/local/Fping/sbin/fping
	[root@Moniter ~]# chmod u+s /usr/local/Fping/sbin/fping
