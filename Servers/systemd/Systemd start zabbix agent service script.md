Title: Systemd启动zabbix agent服务脚本
Date: 2016/5/4 11:26:33 
Modified: 2016/5/4 11:26:35
Category: Servers
Tags: zabbix
Slug: 
Author: allposs


##适用环境##

Redhat7.*+Zabbix Agent*
##版本号##
V1.0

##作用##

Systemd启动Zabbix agent启动脚本


##代码##

	[Unit]
	Description=Zabbix Monitor agentd
	After=network.target

	[Service]
	Type=forking
	PIDFile=/tmp/zabbix_agentd.pid
	Environment="CONFIG=/etc/zabbix/zabbix_agentd.conf"
	ExecStart=/usr/local/zabbix/sbin/zabbix_agentd  -c ${CONFIG}
	ExecStop=/bin/kill `cat $PIDFile`
	RemainAfterExit=yes
	Restart=always

	[Install]
	WantedBy=multi-user.target
