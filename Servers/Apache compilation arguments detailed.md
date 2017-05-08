Title: Apache 编译参数详解
Date: 2016/5/5 8:48:36  
Modified: 2016/5/5 8:48:31 
Category: Servers
Tags: Apache
Slug: 
Author: allposs

##简介##
&#160; &#160; &#160; &#160;Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩充，将Perl/Python等解释器编译到服务器中。

##版本##

apache 2.2.31

##正文##


	[root@Monitor httpd-2.2.31]# ./configure --help
	`configure' configures this package to adapt to many kinds of systems.

	Usage: ./configure [OPTION]... [VAR=VALUE]...

	To assign environment variables (e.g., CC, CFLAGS...), specify them as
	VAR=VALUE.  See below for descriptions of some of the useful variables.

	Defaults for the options are specified in brackets.

	Configuration:
	  -h, --help              display this help and exit                                
      	--help=short        display options specific to this package
      	--help=recursive    display the short help of all the included packages
	  -V, --version           display version information and exit
	  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
	  -C, --config-cache      alias for `--cache-file=config.cache'
	  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

	Installation directories:                                                         
  		--prefix=PREFIX         install architecture-independent files in PREFIX         
		#在PREFIX指定的目录下安装独立体系的文件
                          [/usr/local/apache2]
	  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX          
	#在EPREFIX指定的目录下安装非独立体系的文件,默认与prefix一样
                          [PREFIX]

	By default, `make install' will install all the files in
	`/usr/local/apache2/bin', `/usr/local/apache2/lib' etc.  You can specify
	an installation prefix other than `/usr/local/apache2' using `--prefix',
	for instance `--prefix=$HOME'.                                                    
	#默认安装地与指定安装使用方法，如'--prefix=$HOME'

	For better control, use the options below.

	Fine tuning of the installation directories:                                       
	#细调参数
  		--bindir=DIR            user executables [EPREFIX/bin]                          
	#bin目录，用于存放对网站管理员很有帮助的htpasswd, dbmmanage之类的支持程序.
  		--sbindir=DIR           system admin executables [EPREFIX/sbin]                  
	#sbin目录，系统管理员可执行文件目录，用于存放运行HTTP服务器所必须的httpd, apachectl, suexec之类的服务程序
  		--libexecdir=DIR        program executables [EPREFIX/libexec]                    
	#libexec目录，存放程序可执行文件目录，也就是动态加载模块目录.
  		--sysconfdir=DIR        read-only single-machine data [PREFIX/etc]               
	#etc目录，配置文件目录，用于存放httpd.conf和mime.types之类的服务器配置文件
  		--sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]    
	#com目录，可改写的体系无关数据目录DIR.
  		--localstatedir=DIR     modifiable single-machine data [PREFIX/var]              
	#var目录，可改写的单一机器数据目录DIR.
  		--libdir=DIR            object code libraries [EPREFIX/lib]                     
	#lib目录，对象代码库目录DIR.
  		--includedir=DIR        C header files [PREFIX/include]                          
	#Include目录，C头文件目录DIR.
  		--oldincludedir=DIR     C header files for non-gcc [/usr/include]                
	#include目录，非gcc的C头文件目录DIR.
  		--datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]     
	#share目录，Web服务器只读的体系无关数据根目录.
  		--datadir=DIR           read-only architecture-independent data [DATAROOTDIR]    
	#Web服务器只读的体系无关数据目录DIR.
  		--infodir=DIR           info documentation [DATAROOTDIR/info]                    
	#info目录，信息文档目录DIR.
  		--localedir=DIR         locale-dependent data [DATAROOTDIR/locale]               
	#locale目录，地区相关数据DIR.
  		--mandir=DIR            man documentation [DATAROOTDIR/man]                      
	#man目录，手册文档目录.
  		--docdir=DIR            documentation root [DATAROOTDIR/doc/PACKAGE]             
	#手册目录.
  		--htmldir=DIR           html documentation [DOCDIR]                              
	#HTML格式帮助文档目录.
  		--dvidir=DIR            dvi documentation [DOCDIR]                               
	#dvi格式帮助文档目录.
  		--pdfdir=DIR            pdf documentation [DOCDIR]                               
	#PDF格式帮助文档目录.
  		--psdir=DIR             ps documentation [DOCDIR]                                
	#PS格式帮助文档目录.

	System types:                                                                      
	#交叉编译选项：这些选项用于交叉编译在其他平台上运行的Apache HTTP服务器。
  		--build=BUILD     configure for building on BUILD [guessed]                      
	#指定编译工具所在系统的系统类型BUILD.
  		--host=HOST       cross-compile to build programs to run on HOST [BUILD]         
	#指定<b>Apache</b> HTTP服务器将要进行交叉编译时运行的目标系统类型HOST.
  		--target=TARGET   configure for building compilers for TARGET [HOST]             
	#指定交叉编译所产生的目标代码类型.

	Optional Features:                                                                 
	#编译可选功能：
  		--disable-option-checking  ignore unrecognized --enable/--with options           
	#忽略无法识别的enable或with选项.
  		--disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)     
	#不使用任何软件特性.
  		--enable-FEATURE[=ARG]  include FEATURE [ARG=yes]                                
	#使用软件特性.
  		--enable-layout=LAYOUT                                                           
	#预定义的安装路径布局。选项使用config.layout文件中的配置。只使用"--enable-layout", 而不指定LAYOUT, 相当于"--enable-layout=Apache".
  		--enable-v4-mapped      Allow IPv6 sockets to handle IPv4 connections            
	#允许IPv6套接字处理IPv4连接，使用相同的套接字同时处理IPv4和IPv6的连接，也就是启用地址映射。在FreeBSD、NetBSD、OpenBSD以外的平台上是默认值。
  		--enable-exception-hook Enable fatal exception hook                              
	#启用致命异常钩子，允许在子进程崩溃以后启用一个钩子来运行异常处理程序。
  		--enable-maintainer-mode                                                        
	#打开调试和编译时警告，使用所有警告和调试符号编译源代码，会影响服务器性能。
                          Turn on debugging and compile time warnings               
  		--enable-pie            Build httpd as a Position Independent Executable         
	#编译http作为一个独立的可执行文件。
  		--enable-modules=MODULE-LIST
                          Space-separated list of modules to enable | "all" |
                          "most"                                                   
	#启用的模块，用空格分别列出，或使用all，most列出所有或常用的模块。
  		--enable-mods-shared=MODULE-LIST
                          Space-separated list of shared modules to enable |
                          "all" | "most"                                           
	#启用的共享DSO模块，用空格分别列出，或使用all，most列出所有或常用的共享DOS模块。(注1)
  		--disable-authn-file    file-based authentication control                        
	#禁用基于文件的身份验证控制。
  		--enable-authn-dbm      DBM-based authentication control                         
	#启用基于DBM的验证机制。
  		--enable-authn-anon     anonymous user authentication control                    
	#启用匿名的身份验证机制。
  		--enable-authn-dbd      SQL-based authentication control                         
	#启用基于SQL的验证机制。
  		--disable-authn-default authentication backstopper                               
	#禁止默认的验证机制方式backstopper。（问1）
  		--enable-authn-alias    auth provider alias                                      
	#启用别名验证。
  		--disable-authz-host    host-based authorization control                         
	#禁用基于主机的访问控制。
  		--disable-authz-groupfile
                          'require group' authorization control                    
	#禁用基于组的访问控制。
  		--disable-authz-user    'require user' authorization control                     
	#禁用基于用户的访问控制。
  		--enable-authz-dbm      DBM-based authorization control                          
	#启用基于DBM数据库的认证机制。
  		--enable-authz-owner    'require file-owner' authorization control              
	#启用基于文件所有者的认证机制。
  		--enable-authnz-ldap    LDAP based authentication                                
	#启用基于LDAP的认证。
  		--disable-authz-default authorization control backstopper                        
	#禁用默认的backstopper授权方式。
  		--disable-auth-basic    basic authentication                                     
	#禁用基本认证。
  		--enable-auth-digest    RFC2617 Digest authentication                            
	#启用RFC2617摘要式身份验证。（注2）
  		--enable-isapi          isapi extension support                                  
	#打开ISAPI扩展支持。（注3）
  		--enable-file-cache     File cache                                               
	#启用文件缓存。
  		--enable-cache          dynamic file caching                                     
	#启用动态文件缓存。
  		--enable-disk-cache     disk caching module                                      
	#启用磁盘缓存模块。
  		--enable-mem-cache      memory caching module                                    
	#启用内存缓存模块。
  		--enable-dbd            Apache DBD Framework                                     
	#启用apache的DBD框架。
  		--enable-bucketeer      buckets manipulation filter                              
	#（（问1））
  		--enable-dumpio         I/O dump filter                                          
	#启用I/O转储过滤器。
  		--enable-echo           ECHO server                                              
	#启用回显服务。
  		--enable-example        example and demo module                                  
	#启用实例和演示模块。
  		--enable-case-filter    example uppercase conversion filter                      
	#启用大写字母转换过滤器示例。
  		--enable-case-filter-in example uppercase conversion input filter                
	#启用大写字母输入过滤器示例。
  		--enable-reqtimeout     Limit time waiting for request from client               
	#启用等待客户端响应超时时间。
  		--enable-ext-filter     external filter module                                   
	#启用外部过滤器模块。
  		--disable-include       Server Side Includes                                     
	#禁用禁用服务器端嵌入(SSI)（注4）。
  		--disable-filter        Smart Filtering                                          
	#禁用智能过滤。
  		--enable-substitute     response content rewrite-like filtering                  
	#启用类似响应内容重写过滤。
  		--disable-charset-lite  character set translation                                
	#禁用字符集转换。
  		--enable-charset-lite   character set translation                                
	#启用字符集转换。
  		--enable-deflate        Deflate transfer encoding support                        
	#启用压缩传输编码支持。
  		--enable-ldap           LDAP caching and connection pooling services             
	#启用LDAP的高速缓存和连接池服务。
  		--disable-log-config    logging configuration                                    
	#禁用日志配置。
  		--enable-log-forensic   forensic logging                                         
	#启用forensic日志记录。
  		--enable-logio          input and output logging                                 
	#启用输入输出日志。
  		--disable-env           clearing/setting of ENV vars                             
	#禁用清除/设置环境变量。
  		--enable-mime-magic     automagically determining MIME type                      
	#启用自动确定MIME类型。
  		--enable-cern-meta      CERN-type meta files                                     
	#启用CERN类型元文件。
  		--enable-expires        Expires header control                                   
	#启用头部有效期控制。
  		--enable-headers        HTTP header control                                      
	#启用HTTP头控制。
  		--enable-ident          RFC 1413 identity check                                  
	#启用RFC 1413身份检查。
  		--enable-usertrack      user-session tracking                                    
	#启用用户会话跟踪。
  		--enable-unique-id      per-request unique ids                                   
	#启用对每一个请求做唯一标识ID。
  		--disable-setenvif      basing ENV vars on headers                               
	#禁用基于标题的环境变量。
  		--disable-version       determining httpd version in config files                
	#禁用基本配置文件确定httpd服务器版本。
  		--enable-proxy          Apache proxy module                                      
	#启用Apache的proxy模块，代理模块。
  		--enable-proxy-connect  Apache proxy CONNECT module                              
	#启用Apache的proxy CONNECT模块，代理连接模块。
  		--enable-proxy-ftp      Apache proxy FTP module                                  
	#启用apache代理ftp模块。
  		--enable-proxy-http     Apache proxy HTTP module                                 
	#启用apache代理http模块。
  		--enable-proxy-scgi     Apache proxy SCGI module                                 
	#启用apache代理SCGI模块。
  		--enable-proxy-ajp      Apache proxy AJP module                                  
	#启用apache代理AJP模块。
  		--enable-proxy-balancer Apache proxy BALANCER module                             
	#启用apache代理balancer模块。
  		--enable-ssl            SSL/TLS support (mod_ssl)                                
	#启用SSL/TLS支持
  		--enable-distcache      Select distcache support in mod_ssl                      
	#在mod_ssl模块中启用Distcache。磁盘缓存(Distcache)用于分布式的会话缓存。主要用在 SSL/TLS 服务器。它可以被Apache使用。
  		--enable-optional-hook-export
                          example optional hook exporter                           
	#启用可选钩子输出者示例。
  		--enable-optional-hook-import
                          example optional hook importer                           
	#启用可选钩子输入者示例。
  		--enable-optional-fn-import
                          example optional function importer                       
	#启用可选函数输入者示例。
  		--enable-optional-fn-export
                          example optional function exporter                       
	#启用可选函数输出者示例。
  		--enable-static-support Build a statically linked version of the support
                          binaries                                                 
	#为所支持的二进制文件建立一个静态链接的版本。
  		--enable-static-htpasswd
                          Build a statically linked version of htpasswd            
	#建立静态版本的htpasswd。
  		--enable-static-htdigest
                          Build a statically linked version of htdigest            
	#建立htdigest的静态版本。
  		--enable-static-rotatelogs
                          Build a statically linked version of rotatelogs          
	#建立rotatelogs的静态版本。
  		--enable-static-logresolve
                          Build a statically linked version of logresolve          
	#建立logresolve的静态版本。
  		--enable-static-htdbm   Build a statically linked version of htdbm               
	#建立htdbm的静态版本。
  		--enable-static-ab      Build a statically linked version of ab                  
	#建立ab的静态版本。
  		--enable-static-checkgid
                          Build a statically linked version of checkgid            
	#建立checkgid的静态版本。
  		--enable-static-htcacheclean
                          Build a statically linked version of htcacheclean        
	#建立htcacheclean的静态版本。
  		--enable-static-httxt2dbm
                          Build a statically linked version of httxt2dbm           
	#建立httxt2dbm的静态版本。
  		--enable-http           HTTP protocol handling                                   
	#启用http协议处理。
  		--disable-mime          mapping of file-extension to MIME                        
	#禁用映射文件扩展名到mime类型。
  		--enable-dav            WebDAV protocol handling                                 
	#启用webdav协议处理。
  		--disable-status        process/thread monitoring                                
	#禁用进程或线程的监控。
  		--disable-autoindex     directory listing                                        
	#禁用自动目录索引。
  		--disable-asis          as-is filetypes                                          
	#禁用as-is文件类型
  		--enable-info           server information                                       
	#启用服务信息。
  		--enable-suexec         set uid and gid for spawned processes                    
	#启用suexec，为产生的进程设置uid和gid。
  		--disable-cgid          CGI scripts                                              
	#禁用CGID。
  		--enable-cgi            CGI scripts                                              
	#启用CGI。
  		--disable-cgi           CGI scripts                                              
	#禁用CGI。
  		--enable-cgid           CGI scripts                                              
	#启用CGID。
 	 	--enable-dav-fs         DAV provider for the filesystem                          
	#启用DAV文件系统提供者。
  		--enable-dav-lock       DAV provider for generic locking                         
	#启用DAV提供者的一般锁定。
  		--enable-vhost-alias    mass virtual hosting module                              
	#启用大规模的虚拟主机模块。
  		--disable-negotiation   content negotiation                                      
	#禁用内容协商。
  		--disable-dir           directory request handling                               
	#禁用目录请求处理。
  		--enable-imagemap       server-side imagemaps                                    
	#启用服务器端图片映射图。
  		--disable-actions       Action triggering on requests                            
	#禁用请求上的行为触发器。
  		--enable-speling        correct common URL misspellings                          
	#启用常见的URL拼写错误纠正。
  		--disable-userdir       mapping of requests to user-specific directories         
	#禁用特定用户目录的请求的映射。
  		--disable-alias         mapping of requests to different filesystem parts        
	#禁用不同文件系统部分的请求的映射。
 		 --enable-rewrite        rule based URL manipulation                              
	#启用mod_rewrite(允许URL重写)。
  		--enable-so             DSO capability                                           
	#启用DSO性能。

	Optional Packages:                                                                 
	#可选包
  		--with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]                                    
	#使用PACKAGE[ARG=yes]。
  		--without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)           
	#不使用PACKAGE(等同与--with-PACKAGE=no)。
  		--with-included-apr     Use bundled copies of APR/APR-Util                       
	#使用arp绑定工具。
  		--with-apr=PATH         prefix for installed APR or the full path to
                             apr-config                                            
	#APR的安装路径。
  		--with-apr-util=PATH    prefix for installed APU or the full path to
                             apu-config                                            
	#APU的安装路径。
  		--with-pcre=PATH        Use external PCRE library                                
	#使用使用扩展的PCRE正则表达式库。
  		--with-port=PORT        Port on which to listen (default is 80)                  
	#设置Apache监听的端口，（默认是80）。
  		--with-sslport=SSLPORT  Port on which to securelisten (default is 443)           
	#设置SSL端口（默认是443）。
  		--with-z=DIR            use a specific zlib library                              
	#使用一个特定的zlib库。
  		--with-sslc=DIR         RSA SSL-C SSL/TLS toolkit                                
	#使用RSA ssl-c SSL/TLS的工具包。）
  		--with-ssl=DIR          OpenSSL SSL/TLS toolkit                                 
	#使用OpenSSL工具集。
  		--with-mpm=MPM          Choose the process model for Apache to use.              
	#选择Apache使用的进程模块。
                          MPM={beos|event|worker|prefork|mpmt_os2|winnt}
  		--with-module=module-type:module-file
                          Enable module-file in the modules/<module-type>
                          directory.                                               
	#激活在目录modules中的模块文件,使用第三方模块。
  		--with-program-name     alternate executable name                                
	#设置可选的可执行文件的名字。
  		--with-suexec-bin       Path to suexec binary                                    
	#设置suexec可执行文件的路径。
  		--with-suexec-caller    User allowed to call SuExec                              
	#允许用户调用SuExec。
  		--with-suexec-userdir   User subdirectory                                        
	#设置用户子目录。
  		--with-suexec-docroot   SuExec root directory                                    
	#设置SuExec根目录。
  		--with-suexec-uidmin    Minimal allowed UID                                      
	#所允许的最小UID。
  		--with-suexec-gidmin    Minimal allowed GID                                      
	#所允许的最小GID。
  		--with-suexec-logfile   Set the logfile                                          
	#设置日志文件。
  		--with-suexec-safepath  Set the safepath                                         
	#设置安全路径。
  		--with-suexec-umask     umask for suexec'd process                               
	#设置suexec进程的用户缺省许可。

	Some influential environment variables:
  		CC          C compiler command
  		CFLAGS      C compiler flags
  		LDFLAGS     linker flags, e.g. -L<lib dir> if you have 	libraries in a
              nonstandard directory <lib dir>
  		LIBS        libraries to pass to the linker, e.g. -l<library>
  		CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  		CPP         C preprocessor

	Use these variables to override the choices made by `configure' or to help
	it to find libraries and programs with nonstandard names/locations.

	Report bugs to the package provider.

注：

1、模块列表：

&#160; &#160; &#160; &#160;基本(B)模块默认包含，必须明确禁用；扩展(E)/实验(X)模块默认不包含，必须明确启用

模块名称	状态	简要描述

	mod_actions	(B)	基于媒体类型或请求方法，为执行CGI脚本而提供
	mod_alias	(B)	提供从文件系统的不同部分到文档树的映射和URL重定向
	mod_asis	(B)	发送自己包含HTTP头内容的文件
	mod_auth_basic	(B)	使用基本认证
	mod_authn_default	(B)	在未正确配置认证模块的情况下简单拒绝一切认证信息
	mod_authn_file	(B)	使用纯文本文件为认证提供支持
	mod_authz_default	(B)	在未正确配置授权支持模块的情况下简单拒绝一切授权请求
	mod_authz_groupfile	(B)	使用纯文本文件为组提供授权支持
	mod_authz_host	(B)	供基于主机名、IP地址、请求特征的访问控制
	mod_authz_user	(B)	基于每个用户提供授权支持
	mod_autoindex	(B)	自动对目录中的内容生成列表，类似于"ls"或"dir"命令
	mod_cgi	(B)	在非线程型MPM(prefork)上提供对CGI脚本执行的支持
	mod_cgid	(B)	在线程型MPM(worker)上用一个外部CGI守护进程执行CGI脚本
	mod_dir	(B)	指定目录索引文件以及为目录提供"尾斜杠"重定向
	mod_env	(B)	允许Apache修改或清除传送到CGI脚本和SSI页面的环境变量
	mod_filter	(B)	根据上下文实际情况对输出过滤器进行动态配置
	mod_imagemap	(B)	处理服务器端图像映射
	mod_include	(B)	实现服务端包含文档(SSI)处理
	mod_isapi	(B)	仅限于在Windows平台上实现ISAPI扩展
	mod_log_config	(B)	允许记录日志和定制日志文件格式
	mod_mime	(B)	根据文件扩展名决定应答的行为(处理器/过滤器)和内容(MIME类型/语言/字符集/编码)
	mod_negotiation	(B)	提供内容协商支持
	mod_nw_ssl	(B)	仅限于在NetWare平台上实现SSL加密支持
	mod_setenvif	(B)	根据客户端请求头字段设置环境变量
	mod_status	(B)	生成描述服务器状态的Web页面
	mod_userdir	(B)	允许用户从自己的主目录中提供页面(使用"/~username")
	mod_auth_digest	(X)	使用MD5摘要认证(更安全，但是只有最新的浏览器才支持)
	mod_authn_alias	(E)	基于实际认证支持者创建扩展的认证支持者，并为它起一个别名以便于引用
	mod_authn_anon	(E)	提供匿名用户认证支持
	mod_authn_dbd	(E)	使用SQL数据库为认证提供支持
	mod_authn_dbm	(E)	使用DBM数据库为认证提供支持
	mod_authnz_ldap	(E)	允许使用一个LDAP目录存储用户名和密码数据库来执行基本认证和授权
	mod_authz_dbm	(E)	使用DBM数据库文件为组提供授权支持
	mod_authz_owner	(E)	基于文件的所有者进行授权
	mod_cache	(E)	基于URI键的内容动态缓冲(内存或磁盘)
	mod_cern_meta	(E)	允许Apache使用CERN httpd元文件，从而可以在发送文件时对头进行修改
	mod_charset_lite	(X)	允许对页面进行字符集转换
	mod_dav	(E)	允许Apache提供DAV协议支持
	mod_dav_fs	(E)	为mod_dav访问服务器上的文件系统提供支持
	mod_dav_lock	(E)	为mod_dav锁定服务器上的文件提供支持
	mod_dbd	(E)	管理SQL数据库连接，为需要数据库功能的模块提供支持
	mod_deflate	(E)	压缩发送给客户端的内容
	mod_disk_cache	(E)	基于磁盘的缓冲管理器
	mod_dumpio	(E)	将所有I/O操作转储到错误日志中
	mod_echo	(X)	一个很简单的协议演示模块
	mod_example	(X)	一个很简单的Apache模块API演示模块
	mod_expires	(E)	允许通过配置文件控制HTTP的"Expires:"和"Cache-Control:"头内容
	mod_ext_filter	(E)	使用外部程序作为过滤器
	mod_file_cache	(X)	提供文件描述符缓存支持，从而提高Apache性能
	mod_headers	(E)	允许通过配置文件控制任意的HTTP请求和应答头信息
	mod_ident	(E)	实现RFC1413规定的ident查找
	mod_info	(E)	生成Apache配置情况的Web页面
	mod_ldap	(E)	为其它LDAP模块提供LDAP连接池和结果缓冲服务
	mod_log_forensic	(E)	实现"对比日志"，即在请求被处理之前和处理完成之后进行两次记录
	mod_logio	(E)	对每个请求的输入/输出字节数以及HTTP头进行日志记录
	mod_mem_cache	(E)	基于内存的缓冲管理器
	mod_mime_magic	(E)	通过读取部分文件内容自动猜测文件的MIME类型
	mod_proxy	(E)	提供HTTP/1.1的代理/网关功能支持
	mod_proxy_ajp	(E)	mod_proxy的扩展，提供Apache JServ Protocol支持
	mod_proxy_balancer	(E)	mod_proxy的扩展，提供负载平衡支持
	mod_proxy_connect	(E)	mod_proxy的扩展，提供对处理HTTP CONNECT方法的支持
	mod_proxy_ftp	(E)	mod_proxy的FTP支持模块
	mod_proxy_http	(E)	mod_proxy的HTTP支持模块
	mod_rewrite	(E)	一个基于一定规则的实时重写URL请求的引擎
	mod_so	(E)	允许运行时加载DSO模块
	mod_speling	(E)	自动纠正URL中的拼写错误
	mod_ssl	(E)	使用安全套接字层(SSL)和传输层安全(TLS)协议实现高强度加密传输
	mod_suexec	(E)	使用与调用web服务器的用户不同的用户身份来运行CGI和SSI程序
	mod_unique_id	(E)	为每个请求生成唯一的标识以便跟踪
	mod_usertrack	(E)	使用Session跟踪用户(会发送很多Cookie)，以记录用户的点击流
	mod_version	(E)	提供基于版本的配置段支持
	mod_vhost_alias	(E)	提供大批量虚拟主机的动态配置支持
2、  摘要认证（Digest authentication）用来提供比基础认证更高级别的安全。在RFC2617中有关于它的描述，摘要认证是一种基于挑战-应答模式的认证模型。这 是一种常用的技术，用于证明某人知道某个秘密，而不要求他以容易被窃听的明文形式发送该秘密。

&#160; &#160; &#160; &#160;摘要认证与基础认证的工作原理很相似，用户先 发出一个没有认证证书的请求，Web服务器回复一个带有WWW-Authenticate头的响应，指明访问 所请求的资源需要证书。但是和基础认证发送以Base 64编码的用户名和密码不同，在摘要认证中服务器让客户选一个随机数（称作”nonce“），然后浏览器使用一个单向的加密函数生成一个消息摘要 （message digest），该摘要是关于用户名、密码、给定的nonce值、HTTP方法，以及所请求的URL。消息摘要函数也被称为散列算法，是一种在一个方向上 很容易计算，反方向却不可行的加密算法。与基础认证对比，解码基础认证中的Base 64是很容易办到的。在服务器口令中，可以指定任意的散列算法。在RFC 2617中，描述了以MD5散列函数作为默认算法。

3、 ISAPI,全称是Internet Server Application Programming Interface(Internet服务器应用编程接口)它为开发人员提供了更加强大的对于IIS功能的扩展,这也就是我们常说的ISAPI扩展。 ISAPI本身并没有受限于IIS,相反它为IIS提供了更高的灵活性与扩展。

4、  服务器端嵌入：Server Side Include，是一种类似于ASP的基于服务器的网页制作技术。大多数（尤其是基于Unix平台）的WEB服务器如Netscape Enterprise Server等均支持SSI命令。另外，在计算机硬件领域SSI是同步串行接口（Synchronous Serial Interface）的英文缩写。

&#160; &#160; &#160; &#160;将内容发送到浏览器之前，可以使用“服务器端包含 (SSI）”指令将文本、图形或应用程序信息包含到网页中。例如，可以使用 SSI 包含时间/日期戳、版权声明或供客户填写并返回的表单。 对于在多个文件中重复出现的文本或图形，使用包含文件是一种简便的方法。将内容存入一个包含文件中即可，而不必将内容输入所有文件。通过一个非常简单的语 句即可调用包含文件，此语句指示 Web 服务器将内容插入适当网页。而且，使用包含文件时，对内容的所有更改只需在一个地方就能完成。

&#160; &#160; &#160; &#160;因为包含 SSI 指令的文件要求特殊处理，所以必须为所有 SSI 文件赋予 SSI文件扩展名。默认扩展名是 .stm、.shtm 和 .shtml


```
