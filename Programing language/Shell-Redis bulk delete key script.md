Title: Redis 批量删除key脚本
Date: 10:56 2016/9/8
Modified: 10:56 2016/9/8
Category: Programing language
Tags: Script
Slug: 
Author: allposs
##适用环境

操作系统：Redhat\* CentOS\*

##版本号
V1.0

##作用


用于redis集群或redis单节点

##代码




	#!/bin/bash

	Key='ANE_EX_JIBAO.*'
	Port1=6383
	Port2=6384


	/opt/redis-cluster/redis-3.0.2/src/redis-cli -c -p $Port1 keys $Key > /tmp/$Port1

	for i in `(cat /tmp/$Port1)`
	do
			/opt/redis-cluster/redis-3.0.2/src/redis-cli -c -p $Port1 del $i
	done


	/opt/redis-cluster/redis-3.0.2/src/redis-cli -c -p $Port2 keys $Key > /tmp/$Port2

	for i in `(cat /tmp/$Port2)`
	do
			/opt/redis-cluster/redis-3.0.2/src/redis-cli -c -p $Port2 del $i
	done
   
  