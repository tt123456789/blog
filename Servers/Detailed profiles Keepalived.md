Title: Keepalived 配置文件详解
Date: 2016/5/19 16:47:25    
Modified: 2016/5/19 16:47:27    
Category: Servers
Tags: keepalived
Slug: 
Author: allposs

##简介##
 &#160; &#160; &#160; &#160;Keepalived的作用是检测web服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的web服务器从系统中剔除，当web服务器工作正常后Keepalived自动将web服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的web服务器。

##版本##
V1.0


##正文##

####1. keepalived.confg文件解析####


	! Configuration File for keepalived     #!表示注释
	global_defs {
		notification_email {        root@example.com		 #接收警报的邮箱,可以添加多个
							}
		notification_email_from root@localhost   
		smtp_server 127.0.0.1                #使用本机转发 email
        router_id LVS_BACKUP				 #设置lvs主机的id，在一个网络内每一台都应该是唯一的
	}
        vrrp_instance VIP1 {				#定义VRRP热备实例为VIP1
                state BACKUP				#热备状态，MASTER(主)；SLAVE(从)；BACKUP(备份)
                interface eno33554968		#承载VIP地址的物理接口
				#lvs_sync_daemon_inteface eth2   #负载均衡主备间的同步
                virtual_router_id 50		#虚拟路由器的ID号，组内保存一致
                priority 80					#优先级,数值越大优先级越高。
                advert_int 1				#主备之间的通告间隔秒数（心跳频率）
                authentication {			#认证信息，每个热备组保持一致
				nopreempt					#设置为不抢占 
                auth_type PASS				#认证类型，主备切换时的验证
                        auth_pass 1111		#密码字串
                }
        virtual_ipaddress {					#指定漂移地址（VIP），可以有多个
                        172.16.0.10/24		#HA 虚拟 ip,可加多个
                }
        }

		virtual_server 172.16.0.10 80 {				#虚拟服务器地址（VIP）、端口
        		delay_loop 6						#健康检查的间隔时间（秒）
        		lb_algo wrr							#lvs 调度算法,这里使用加权轮询
        		lb_kind DR							#lvs 负载均衡机制,这里使用直连路由
       			nat_mask 255.255.255.0				
        		persistence_timeout 50				#同一IP连接50秒内被分配到同一台服务器
        		protocol TCP						#用 TCP 协议检查 realserver 状态

        		real_server 172.16.0.203 80 {		#第一个WEB节点的地址、端口
						eight 3						#节点的权重
                		TCP_CHECK {						#健康检查方式 TCP
                        		connect_timeout 5		#故障重试秒数（即连接超时）
                        		nb_get_retry 3			#重试延迟（即重试次数）
                        		delay_before_retry 3	#重试间隔（秒）
                        		connect_port 80			#检查的目标端口
               			 }       
        			}       
        
        real_server 172.16.0.204 80 {		#第二个WEB节点的地址、端口
                		weight 3
                		TCP_CHECK {
                        		connect_timeout 10
                        		nb_get_retry 3
                        		delay_before_retry 3
                        		connect_port 80
                			}       
        			}       
        


	}



####2. 参数vrrp_script解析
#####基本运用

 &#160; &#160; &#160; &#160;通常情况下，利用keepalived做热备，其中一台设置为master，一台设置为backup。当master出现异常后，backup自动切换为master。当backup成为master后，master恢复正常后会再次抢占成为master，导致不必要的主备切换。因此可以将两台keepalived初始状态均配置为backup，设置不同的优先级，优先级高的设置nopreempt解决异常恢复后再次抢占的问题。

&#160; &#160; &#160; &#160;然而keepalived只能做到对网络故障和keepalived本身的监控，即当出现网络故障或者keepalived本身出现问题时，进行切换。但是这些还不够，我们还需要监控keepalived所在服务器上的其他业务进程，根据业务进程的运行状态决定是否需要进行主备切换。这个时候，我们可以通过编写脚本对业务进程进行检测监控。

例如编写个简单脚本查看haproxy进程是否存活

	
	#!/bin/bash
		count = `ps aux | grep -v grep | grep haproxy | wc -l`
		if [ $count > 0 ]; then
    	exit 0
		else
    	exit 1
		fi

在keepalived的配置文件中增加相应配置项

	vrrp_script checkhaproxy
	{
    	script "/home/check.sh"
    	interval 3
    	weight -20
	}
 
	vrrp_instance test
	{
    	...
     
    	track_script
    	{
        	checkhaproxy
    	}
     
    	...
	}


keepalived会定时执行脚本并对脚本执行的结果进行分析，动态调整vrrp_instance的优先级。

如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加

如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少

其他情况，维持原本配置的优先级，即配置文件中priority对应的值。

这里需要注意的是：

1） 优先级不会不断的提高或者降低

2） 可以编写多个检测脚本并为每个检测脚本设置不同的weight

3） 不管提高优先级还是降低优先级，最终优先级的范围是在[1,254]，不会出现优先级小于等于0或者优先级大于等于255的情况

这样可以做到利用脚本检测业务进程的状态，并动态调整优先级从而实现主备切换。
但是利用该方式会存在一个问题，例如：A,B两台keepalived

A的配置大概为：

	vrrp_script checkhaproxy
	{
    	script "/etc/check.sh"
    	interval 3
    	weight -20
 
	}
 
	vrrp_instance test
	{
    	....
     
    	state backup
    	priority 80
    	nopreempt
 
    	track_script
    	{
        	checkhaproxy
    	}
 
    	....
	}

B的配置大概为：


	vrrp_script checkhaproxy
	{
    	script "/etc/check.sh"
    	interval 3
    	weight -20
	}
 
	vrrp_instance test
	{
    	....
     
    	state backup
    	priority 70
 
    	track_script
    	{
        	checkhaproxy
    	}
 
    	....
	}

&#160; &#160; &#160; &#160;A,B同时启动后，由于A的优先级较高，因此通过选举会成为master。当A上的业务进程出现问题时，优先级会降低到60。此时B收到优先级比自己低的vrrp广播包时，将切换为master状态。那么当B上的业务出现问题时，优先级降低到50，尽管A的优先级比B的要高，但是由于设置了nopreempt，A不会再抢占成为master状态。

&#160; &#160; &#160; &#160;所以，可以在检测脚本中增加杀掉keepalived进程（或者停用keepalived服务）的方式，做到业务进程出现问题时完成主备切换。 

####其它####
注意：
在keepalived搭配别的软件使用的时候（nginx，haproxy），需要对他们进程进行监控与管理，这个时候就需要在vrrp里面引用相关脚本来保证keepalived的正常迁移。以下是引用方法：

	notify_master /etc/keepalived/scripts/start_haproxy.sh		#表示当切换到master状态时,要执行的脚本
	notify_fault  /etc/keepalived/scripts/stop_keepalived.sh		#故障时执行的脚本
	notify_stop   /etc/keepalived/scripts/stop_haproxy.sh		# keepalived停止运行前运行notify_stop指定的脚本

VRRP配置包括三个类:VRRP同步组(synchroization group)，VRRP实例(VRRP Instance)，VRRP脚本，VRRP脚本(vrrp_script)和VRRP实例(vrrp_instance)属于同一个级别。