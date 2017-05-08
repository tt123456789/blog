Title: Shell的tomcat日志清除脚本
Date: 11:07 2016/10/21
Modified: 11:06 2016/10/21
Category: Shell
Tags: script
Slug: 
Author: allposs

##适用环境##
linux*

##版本号##
V1.0

##作用##

清除4天以外的日志文件与catalina.out


##代码##

	#!/bin/bash
	PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
	export PATH

	Path=/ane/tomcat/logs/


	echo " " > /ane/tomcat/logs/catalina.out
	find $Path  -mtime +3 -name 'catalina.*' -exec rm -rf {} \;
	find $Path  -mtime +3 -name 'host-manager.*' -exec rm -rf {} \;
	find $Path  -mtime +3 -name 'localhost.*' -exec rm -rf {} \;
	find $Path  -mtime +3 -name 'localhost_access_log.*' -exec rm -rf {} \;
	find $Path  -mtime +3 -name 'manager.*' -exec rm -rf {} \;
	find /ane/bak/  -mtime +7 -name '*' -exec rm -rf {} \;