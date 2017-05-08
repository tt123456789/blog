Title: zabbix 配置mutt+msmtp发送报警邮件
Date: 2016/5/4 11:30:27 
Modified: 2016/5/4 11:30:24 
Category: Monitor
Tags: zabbix
Slug: 
Author: allposs
## 简介
&#160; &#160; &#160; &#160;本文是继之前zabbix安装后做的一些相关工作。使用mutt+msmtp软件发送告警邮件。
&#160; &#160; &#160; &#160;Zabbix支持多种报警的方式，其中成本最低、最方便的就是邮件报警的方式了。但是因为它不支持邮件的用户认证，这种方式现在也非常少见，同时安全性也差，如果在本机建邮件服务器的话也容易被误入垃圾邮件。所以一推荐使用公司邮箱或163和QQ邮箱，但是现在国内免费邮箱基本都封锁了pop和smtp.所以这里看个人自己的需要。
## 环境

+ 操作系统：CenOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：Moniter

## 软件包
###1. Yum源

	[root@Moniter ~]# yum  -y install mutt

###2. 源码包
	[root@Moniter ~]#  wget http://downloads.sourceforge.net/project/msmtp/msmtp/1.6.2/msmtp-1.6.2.tar.xz?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fmsmtp%2Ffiles%2Fmsmtp%2F1.6.2%2F&ts=1439358641&use_mirror=nchc


## 正文

###1. 安装配置msmtp
######1.1安装msmtp

	[root@Moniter ~]#  wget http://downloads.sourceforge.net/project/msmtp/msmtp/1.6.2/msmtp-1.6.2.tar.xz?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fmsmtp%2Ffiles%2Fmsmtp%2F1.6.2%2F&ts=1439358641&use_mirror=nchc
	[root@Moniter ~]# mv msmtp-1.6.2.tar.xz\?r\=http\:%2F%2Fsourceforge.net%2Fprojects%2Fmsmtp%2Ffiles%2Fmsmtp%2F1.6.2%2F msmtp-1.6.2.tar.xz
	[root@Moniter ~]# tar -xvf msmtp-1.6.2.tar.xz
	[root@Moniter ~]# cd msmtp-1.6.2
	[root@Moniter ~]# ./configure --prefix=/usr/local/msmtp
	[root@Moniter ~]# make && make install

######1.2 配置msmtp

	[root@Moniter ~]# mkdir -p /usr/local/msmtp/etc
	[root@Moniter ~]# vim /usr/local/msmtp/etc/msmtprc

写入

	defaults
	tls on
	logfile /var/log/zext_msmtp.log

	account acc1
	host smtp.163.com
	port 25
	from hello@163.com
	auth login
	tls off
	user hello@163.com
	password mypasswd

	account acc2
	host smtp.qq.com
	port 465
	from 123456@qq.com
	auth login
	tls_starttls off
	tls on
	tls_certcheck off
	user 123456@qq.com
	password mypasswd 
	account default : acc2

注意：msmtp文件不能有任何中文注解或配置，也不能粘贴中文注释，否则必报错

QQ邮箱不支持tls，使用QQ邮箱需要关闭tls_starttls，
网易免费邮箱的ssl证书通不过验证，所以使用163邮箱时，只能关闭tls证书验证。

测试
![](http://image.allposs.cn/20150924001.png)


	[root@Moniter ~]# /usr/local/msmtp/bin/msmtp test@163.com
		hello word!
		this is a test mail!

按Ctrl+d结束输入，然后到邮箱里检查是否收到相关邮件。否则根据日志和报错进行排错。


###2. 安装mutt
######2.1 安装mutt

	[root@Moniter ~]# yum  -y install mutt

######2.2 配置mutt

	[root@Moniter ~]# vim  /etc/Muttrc.local
		set sendmail="/usr/local/msmtp/bin/msmtp"
		set use_from=yes
		set realname="邮箱地址"
		set editor="vim"

注意：

如果不设置use_from项，发向gmail的邮件会被退回；
设置了use_from项，收件人会看到发件人在系统里的用户名（例如root）；如果是在web邮箱发的邮件，收件人那里会看到QQ邮箱的昵称。

测试

	[root@Moniter ~]# echo "<a href="http://so.21ops.com/cse/search?s=9181936462520079739&entry=1&q=helloworld" class="bdcs-inlinelink" target="_blank">helloworld</a>" | mutt -s "test" test@163.com

![](http://image.allposs.cn/20150924003.png)

###3. 配置zabbix
######3.1配置脚本文件

	[root@Moniter ~]# mkdir /etc/zabbix/AlertScripts/
	[root@Moniter ~]# vim /etc/zabbix/AlertScripts/mail.sh
 		#!/bin/bash
 		# $1 sendmail address
 		# $2 sendmail subject
 			echo "$3" | mutt -s"$2" $1

	[root@Moniter ~]# chmod 777 /etc/zabbix/AlertScripts/mail.sh

######3.2 配置zabbix配置文件

	[root@Moniter ~]# vim /etc/zabbix/zabbix_server.conf

修改引用脚本路径

	AlertScriptsPath=/etc/zabbix/AlertScripts

###4. 配置zabbix网页
######4.1 配置示警媒介
管理—>示警媒介—>创建媒介类型：

![](http://image.allposs.cn/20150924004.png)

######4.2 配置报警邮箱

管理—>用户—>Admin (Zabbix Administrator)—>示警媒介—>添加：

![](http://image.allposs.cn/20150924006.png)

![](http://image.allposs.cn/20150924007.png)
######4.3 配置示警动作

组态—>动作—>创建动作：

![](http://image.allposs.cn/20150924008.png)


写入：


	名称                        Action-Email
	默认接收人                  故障{TRIGGER.STATUS},服务器:{HOSTNAME1}发生: {TRIGGER.NAME}故障!
	默认信息                    
                            告警主机:{HOSTNAME1}
                            告警时间:{EVENT.DATE} {EVENT.TIME}
                            告警等级:{TRIGGER.SEVERITY}
                            告警信息: {TRIGGER.NAME}
                            告警项目:{TRIGGER.KEY1}
                            问题详情:{ITEM.NAME}:{ITEM.VALUE}
                            当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
                            事件ID:{EVENT.ID}

	恢复主旨                    恢复{TRIGGER.STATUS}, 服务器:{HOSTNAME1}: {TRIGGER.NAME}已恢复!

                            恢复信息
                            告警主机:{HOSTNAME1}
                            告警时间:{EVENT.DATE} {EVENT.TIME}
                            告警等级:{TRIGGER.SEVERITY}
                            告警信息: {TRIGGER.NAME}
                            告警项目:{TRIGGER.KEY1}
                            问题详情:{ITEM.NAME}:{ITEM.VALUE}
                            当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
                            事件ID:{EVENT.ID}

并把“以启用”打勾。当然你也可以根据自己需要修改。

![](http://image.allposs.cn/20150924009.png)

![](http://image.allposs.cn/20150924010.png)

以上算事整个个邮件告警安装完成，最后自己可以测试一下。



## 结束##