Title: git gitlab配置安装
Date: 16:16 2017/3/18
Modified: 16:16 2017/3/18
Category: Servers
Tags: git
Slug: 
Author: allposs

## 简介
&#160; &#160; &#160; &#160;Git - 版本控制工具

&#160; &#160; &#160; &#160;Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。

&#160; &#160; &#160; &#160;Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。

&#160; &#160; &#160; &#160;Torvalds 开始着手开发 Git 是为了作为一种过渡方案来替代 BitKeeper，后者之前一直是 Linux 内核开发人员在全球使用的主要源代码工具。开放源码社区中的有些人觉得 BitKeeper 的许可证并不适合开放源码社区的工作，因此 Torvalds 决定着手研究许可证更为灵活的版本控制系统。尽管最初 Git 的开发是为了辅助 Linux 内核开发的过程，但是我们已经发现在很多其他自由软件项目中也使用了 Git。例如 最近就迁移到 Git 上来了，很多 Freedesktop 的项目也迁移到了 Git 上。

&#160; &#160; &#160; &#160;Github - 一个网站，提供给用户空间创建git仓储，保存用户的一些数据文档或者代码等,作为开源代码库以及版本控制系统，Github目前拥有140多万开发者用户。随着越来越多的应用程序转移到了云上，Github已经成为了管理软件开发以及发现已有代码的首选方法。如前所述，作为一个分布式的版本控制系统，在Git中并不存在主库这样的概念，每一份复制出的库都可以独立使用，任何两个库之间的不一致之处都可以进行合并。

&#160; &#160; &#160; &#160;GitHub可以托管各种git库，并提供一个web界面，但与其它像 SourceForge或Google Code这样的服务不同，GitHub的独特卖点在于从另外一个项目进行分支的简易性。为一个项目贡献代码非常简单：首先点击项目站点的“fork”的按钮，然后将代码检出并将修改加入到刚才分出的代码库中，最后通过内建的“pull request”机制向项目负责人申请代码合并。已经有人将GitHub称为代码玩家的MySpace。

&#160; &#160; &#160; &#160;GitLab - 基于Git的项目管理软件

&#160; &#160; &#160; &#160;GitLab 是一个用于仓库管理系统的开源项目。使用Git作为代码管理工具，并在此基础上搭建起来的web服务。

三者都是基于git的，可以说是git的衍生品。
## 环境
+ 操作系统：CentOS 7.2 x64
+ Yum源：163源
+ IP地址：
			7Node1 10.199.200.101
+ DNS：
+ 主机名：
			7Node1.example.com 10.199.200.101

## 软件包

###1. Yum源
	
	[root@7Node1 ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-8.8.9-ce.0.el7.x86_64.rpm
	[root@7Node1 ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-8.8.5-ce.1.el7.x86_64.rpm

###2. 源码包
	
	[root@7Node1 ~]# wget https://www.kernel.org/pub/software/scm/git/git-2.9.3.tar.gz
	
	
##拓扑图



## 正文

###1. git安装
	[root@7Node1 ~]# yum  groupinstall "Development Tools"
	[root@7Node1 ~]# yum install gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel expat-devel  libcurl-devel
	[root@7Node1 ~]# wget https://www.kernel.org/pub/software/scm/git/git-2.9.3.tar.gz
	[root@7Node1 ~]# tar xf git-2.9.3.tar.gz
	[root@7Node1 git-2.9.3]# cd git-2.9.3/
	[root@7Node1 git-2.9.3]# make prefix=/usr/local/git all
	[root@7Node1 git-2.9.3]# make prefix=/usr/local/git install
	[root@7Node1 git-2.9.3]# vi /etc/profile
		export GIT_HOME=/usr/local/git
		PATH=$GIT_HOME/bin:$PATH
		export PATH
	[root@7Node1 git-2.9.3]# source /etc/profile
	[root@7Node1 git-2.9.3]# git --version

###2. gitlab安装
###2.1 安装gitlab
	[root@7Node1 ~]# yum install curl policycoreutils openssh-server openssh-clients libsemanage-static libsemanage-devel
	[root@7Node1 ~]# systemctl enable sshd
	[root@7Node1 ~]# systemctl start sshd
	[root@7Node1 ~]# systemctl enable postfix
	[root@7Node1 ~]# systemctl start postfix
	[root@7Node1 ~]# firewall-cmd --permanent --add-service=http
	[root@7Node1 ~]# systemctl reload firewalld
	添加GitLab仓库,并安装到服务器上
	[root@7Node1 ~]# curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.rpm.sh | bash
	[root@7Node1 ~]# sudo yum install gitlab-ce
	如果需要手动下载可以用下面方法：
	[root@7Node1 ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-8.8.2-ce.0.el7.x86_64.rpm
	[root@7Node1 ~]# yum install gitlab-ce-8.8.2-ce.0.el7.x86_64.rpm
	[root@7Node1 ~]# yum install libsemanage-static libsemanage-devel
	[root@7Node1 ~]# gitlab-ctl reconfigure
###2.2 汉化gitlab

&#160; &#160; &#160; &#160;汉化之前要先确认当前汉化版本的 VERSION 是否相同。
&#160; &#160; &#160; &#160;如果安装版本小于当前汉化版本，请先升级。如果安装版本大于当前汉化版本，请在本项目中提交新的 issue。
&#160; &#160; &#160; &#160;如果版本相同，首先在本地 clone 仓库。
	# GitLab.com 仓库
	git clone https://gitlab.com/larryli/gitlab.git
	# 或 Coder.net 镜像
	git clone https://git.coding.net/larryli/gitlab.git
	升级gitlab
	[root@7Node1 ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-8.8.5-ce.1.el7.x86_64.rpm
	[root@7Node1 ~]#  yum update gitlab-ce-8.8.5-ce.1.el7.x86_64.rpm 
	[root@7Node1 ~]# gitlab-ctl reconfigure
	[root@7Node1 ~]# gitlab-ctl stop
	[root@7Node1 ~]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
	[root@7Node1 ~]# /bin/cp -rf gitlab-L-zh/* /opt/gitlab/embedded/service/gitlab-rails/
	[root@7Node1 ~]# gitlab-ctl start

