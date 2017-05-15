Title: Shell脚本 NFS挂载自动备份
Date: 2016/5/13 19:55:48 
Modified: 2016/5/13 19:55:51 
Category: Programing language
Tags: script
Slug: 
Author: allposs


##适用环境##
linux*

##版本号##
V1.0

##作用##

NFS挂载备份app应用


##代码##

	#!/bin/bash
	#backup.sh
	#PATH
	PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
	export PATH
	
	#######################
	Path=/data/aneweb
	Appname=www.ane56.com
	Servername=tomcat
	######################

	BackupPath=/root/bak/
	LogFile=/root/.script/back.log
	time="`date +%Y%m%d`"
	local_ip=`ip addr | grep inet.*eth| awk -F ' ' '{print $2}' | awk -F '/' '{print $1}'|awk -F '.' '{print $1 "." pint $2 "." pint $3 "." pint $4 }'`
	name="$Appname"_"$local_ip"_"$time"
	showmount -e 172.17.99.13 2>/dev/null | grep bak >/dev/null
	net="$?"
	if [ $net -eq 0  ] ;then
		if [  ! -d $BackupPath  ] ; then
			mkdir $BackupPath -p
			mount -t nfs 172.17.99.13:/nas/bak $BackupPath
			mount -l 2>/dev/null | grep bak >/dev/null	
			m="$?"
	  		if [ -d $BackupPath$Appname  ] ; then  	 
				tar -czf "$BackupPath$Appname/$name".tar.gz "$Path" 2>/dev/null
				echo "-------------------------------------------" >> $LogFile
		 		echo $(date +"%y-%m-%d %H:%M:%S") >> $LogFile
		 		echo "backup is ok!" >> $LogFile
		 		echo "tar -czf "$BackupPath$Appname/$name".tar.gz "$Path"" >> $LogFile
		 		echo "--------------------------" >> $LogFile
		 		if [ -e $BackupPath$Appname/$name.tar.gz ] ; then
					find $BackupPath$Appname/ -mtime +7 -type f |xargs rm -f
					cd /tmp
					umount $BackupPath
				else
					echo "-------------------------------------------" >> $LogFile
					echo "error, Create file failed!" >> $LogFile
					echo "--------------------------" >> $LogFile
				fi		
			elif [ $m -eq 0 ] ; then
				mkdir "$BackupPath$Appname"
				tar -czf "$BackupPath$Appname/$name".tar.gz "$Path" 2>/dev/null
				echo "-------------------------------------------" >> $LogFile
				echo $(date +"%y-%m-%d %H:%M:%S") >> $LogFile
				echo "backup is ok!" >> $LogFile
				echo "tar -czf "$BackupPath$Appname"/"$name".tar.gz "$Path"" >> $LogFile
				echo "--------------------------" >> $LogFile 
				if [ -e $BackupPath$Appname/$name.tar.gz ] ; then
					find $BackupPath$Appname/ -mtime +7 -type f |xargs rm -f
					cd /tmp
					umount $BackupPath
				else
					echo "-------------------------------------------" >> $LogFile
					echo "error, Create file failed!" >> $LogFile
					echo "--------------------------" >> $LogFile
				fi
	 		else
				echo "-------------------------------------------" >> $LogFile
				echo "error, the reason is not linked to success!" >> $LogFile
		 		echo "--------------------------" >> $LogFile
	 		fi
		else
			mount -t nfs 172.17.99.13:/nas/bak $BackupPath
			mount -l 2>/dev/null | grep bak >/dev/null		
			m="$?"
  			if [ -d $BackupPath$Appname  ] ; then  	 
				tar -czf "$BackupPath$Appname/$name".tar.gz "$Path" 2>/dev/null
		 		echo "-------------------------------------------" >> $LogFile
		 		echo $(date +"%y-%m-%d %H:%M:%S") >> $LogFile
		 		echo "backup is ok!" >> $LogFile
		 		echo "tar -czf "$BackupPath$Appname/$name".tar.gz "$Path"" >> $LogFile
		 		echo "--------------------------" >> $LogFile
		 		if [ -e $BackupPath$Appname/$name.tar.gz ] ; then
					find $BackupPath$Appname/ -mtime +7 -type f |xargs rm -f
					cd /tmp
					umount $BackupPath
				else
					echo "-------------------------------------------" >> $LogFile
					echo "error, Create file failed!" >> $LogFile
					echo "--------------------------" >> $LogFile
				fi		
			elif [ $m -eq 0 ] ; then
				mkdir "$BackupPath$Appname"
				tar -czf "$BackupPath$Appname/$name".tar.gz "$Path" 2>/dev/null
				echo "-------------------------------------------" >> $LogFile
				echo $(date +"%y-%m-%d %H:%M:%S") >> $LogFile
				echo "backup is ok!" >> $LogFile
				echo "tar -czf "$BackupPath$Appname/$name".tar.gz "$Path"" >> $LogFile
				echo "--------------------------" >> $LogFile 
				if [ -e $BackupPath$Appname/$name.tar.gz ] ; then
					find $BackupPath$Appname/ -mtime +7 -type f |xargs rm -f
					cd /tmp
					umount $BackupPath
				else
					echo "-------------------------------------------" >> $LogFile
					echo "error, Create file failed!" >> $LogFile
					echo "--------------------------" >> $LogFile
				fi
	 		else
				echo "-------------------------------------------" >> $LogFile
				echo "error, the reason is not linked to success!" >> $LogFile
				echo "--------------------------" >> $LogFile
	 		fi
		fi
	else
		echo "-------------------------------------------" >> $LogFile
		echo "error, nfs is not">> $LogFile
		echo "-------------------------------------------" >> $LogFile
	fi
