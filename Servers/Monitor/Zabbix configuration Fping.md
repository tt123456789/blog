Title: Zabbix 配置Fping
Date: 2016/5/4 11:30:27 
Modified: 2016/5/4 11:30:24 
Category: Servers
Tags: zabbix
Slug: 
Author: allposs

## 简介##
&#160; &#160; &#160; &#160;本文是继之前安装后做的一些相关工作，所有前提在前文提到。这次是安装Fping利用ICMP协议进行判断网络主机是否存活。

## 环境##


+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：Server:10.199.255.200
+ DNS：10.199.255.15
+ 主机名：moniter

## 软件包##

###1. Yum源###

###2. 源码包###

	[root@Moniter ~]# wget http://www.fping.org/dist/fping-3.10.tar.gz



## 正文##

###1.下载Fping###

	[root@Moniter ~]# wget http://www.fping.org/dist/fping-3.10.tar.gz

###2.安装Fping###

	[root@Moniter ~]# tar -zxvf fping-3.10.tar.gz 
	[root@Moniter ~]# cd fping-3.10
	[root@Moniter fping-3.10]# ./configure --prefix=/usr/local/Fping
	[root@Moniter fping-3.10]# make&&make install

###3.修改fping的权限###

	[root@Moniter ~]# chown root:root /usr/local/Fping/sbin/fping
	[root@Moniter ~]# chmod u+s /usr/local/Fping/sbin/fping

###4.zabbix配置###

	[root@Moniter fping-3.10]# vim /etc/zabbix/zabbix_server.conf
修改
	FpingLocation=/usr/local/Fping/sbin/fping

![](http://image.allposs.cn/20151231001.png)

###5.添加zabbix模板###

配置监控项ICMP_Loss

![](http://image.allposs.cn/20151231002.png)

配置监控项ICMP_Ping

![](http://image.allposs.cn/20151231003.png)

配置监控项ICMP_Response_Time

![](http://image.allposs.cn/20151231004.png)

配置触发器ping失效

![](http://image.allposs.cn/20151231005.png)

{Ping:icmpping.max(#3)}=0的意思是：在ping子集下的icmpping的最大值等于0


配置触发器ping失效后平均值小于0.15

![](http://image.allposs.cn/20151231006.png)

配置触发器Ping失效后最大时间为20

![](http://image.allposs.cn/20151231007.png)

三者之间的依赖关系


## 结束##