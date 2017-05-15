Title: Shell脚本 MAC地址扫描
Date: 2016-05-04 10:55:08 
Modified: 2016-05-04 10:55:08 
Category: Programing language
Tags: script
Slug: 
Author: allposs
##适用环境##

Redhat* CentOS* linux*

##版本号##

v1.0

##作用##

根据IP地址扫描出MAC地址


##代码##

	#!/bin/bash
	PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
	export PATH
	ip="192.168.0."#扫描的网段
	ip2=1 #扫描的起始IP
	ip3=254 #扫描的终止IP
	echo "this time scanning in ">>MAC.txt
	date >> MAC.txt
	while [ "$ip2" != "$(($ip3+1))" ];do #当起始IP不等于255时
	MAC=`arping  -f $ip$ip2  -w 1 |awk  '{print $5}'|sed  -n  '2p'`      #arping  -f $ip$ip2  -w 1 从F口发送一个包，awk  '{print $5}'截取打印值的第五行，sed  -n  '2p'截取第二列。
	if [ "$MAC" != "broadcast(s)" ]; #当$MAC不等于broadcast(s)时
		then
		MACADD=`echo $MAC|cut -c 2-18`
		echo "$ip$ip2   $MACADD" >> MAC.txt
	else
		echo "This host (IP:$ip$ip2) is nonentity."
	fi
		ip2=$(($ip2+1))
	done
