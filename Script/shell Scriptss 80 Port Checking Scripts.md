Title: Shell端口检查脚本
Date: 2016-05-04 10:49
Modified: 2016-05-04 10:49
Category: Shell
Tags: script
Slug: 
Author: allposs

##适用环境##
linux*

##版本号##
V1.0

##作用##

每10秒检一次网页端口是否正常


##代码##

	#! /bin/bash
	while : ;do
	/home/somedir/scripts.sh 2>/dev/null &
	sleep 10
	time=`date`
	echo $time >> /root/1.txt
	curl -o /dev/null -s -w %{http_code} "http://www.baidu.com/" >> 1.txt
	echo "" >> /root/1.txt
	done
