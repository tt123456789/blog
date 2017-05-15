Title: MySql 数据自动备份脚本
Date: 2016/5/10 21:25:39 
Modified: 2016/5/28 11:41:58   
Category: language
Tags: Script
Slug: 
Author: allposs
##适用环境

操作系统：Redhat\* CentOS\*

##版本号
V1.0

##作用


用于MySQL数据库Database备份

##代码


	#!/bin/bash 
	PATH=/bin:/sbin:/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/local/mysql/bin
	export PATH
	dbuser=root
	dbpassword=ane56!
	dbname=ane_gis
	backtime=`date +%Y%m%d`
	mkdir -p /root/backup/mysql/$(date +%Y)/$(date +%m)/
	logpath=/root/backup/mysql
	datapath=/root/backup/mysql
	echo "${backtime},dbname:${dbname},backup path:${datapath} >> ${logpath}/mysqllog.log"

	for table in $dbname ;do
	filename="$table"_"`date +%Y_%m_%d`"
	source=`mysqldump -u ${dbuser} -p${dbpassword} ${table} > ${datapath}/${filename}.sql`  2>> ${logpath}/mysqllog.log

    	if [ "$?" -eq 0 ] ; then
            cd $datapath
            tar -jcf ${filename}.tar.bz2 ${filename}.sql > /dev/null
            rm -f ${filename}.sql
            echo "${dbname} backup success!" >> ${logpath}/mysqllog.log
    	else
           echo "${dbname} backup fails !" >> ${logpath}/mysqllog.log
    	fi
	done
