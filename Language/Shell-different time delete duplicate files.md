Title: Shell脚本 不同时间重复文件删除
Date: 2016-05-04 11:20:57 
Modified: 2016-05-04 11:20:57 
Category: language
Tags: Script
Slug: 
Author: allposs


##适用环境##

操作系统：Redhat\* CentOS\*

##版本号##
  
V1.0

##作用##

根据不同时间删除相同文件

##代码##

	#!/bin/bash
	systemdate=`date | cut -d' ' -f 3`
	for path in `(find / -name "Monitor1*.txt")`
	do
        data=`echo "$path" | xargs ls -al  | cut -d " " -f 7 `
        value=`let ""$systemdate"-"$data""`
        if [[ $value -le 3 ]]; then
                rm -rf "$data"
        else
                echo "ok!"
	fi
	done
