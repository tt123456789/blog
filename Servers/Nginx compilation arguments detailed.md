Title: Nginx 编译参数详解
Date: 2016/5/14 10:00:00   
Modified: 2016/5/14 10:00:05  
Category: Servers
Tags: nginx
Slug: 
Author: allposs


##简介##
&#160; &#160; &#160; &#160;Nginx ("engine x") 是一个高性能的HTTP和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 服务器。 Nginx 是由 Igor Sysoev 为俄罗斯访问量第二的 Rambler.ru 站点开发的，第一个公开版本0.1.0发布于2004年10月4日。其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011年6月1日，nginx 1.0.4发布。

&#160; &#160; &#160; &#160;Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler（俄文：Рамблер）使用。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、新浪、网易、腾讯等。
##版本##

nginx_1.8.0

##正文##

ginx编译安装参数：

	/configure --help
		--help 
				#显示本提示信息
		--prefix=PATH 
				#设定安装目录
		--sbin-path=PATH 
				#设定程序文件目录
		--conf-path=PATH 
				#设定配置文件(nginx.conf)目录
		--error-log-path=PATH 
				#设定错误日志目录
		--pid-path=PATH 
				#设定pid文件(nginx.pid)目录
		--lock-path=PATH 
				#设定lock文件(nginx.lock)目录
		--user=USER 
				#设定程序运行的用户环境(www)
		--group=GROUP 
				#设定程序运行的组环境(www)
		--builddir=DIR 
				#设定程序编译目录
		--with-rtsig_module 
				#允许rtsig模块
				#rtsig模块是一种实时信号，在Linux 2.2.19 默认情况下，实时信号连接数不超过1024，但是对于高负载是肯定不够的。因此通过调整内核参数/proc/sys/kernel/rtsig-max达 到效果。但是Linux 2.6.6-mm2开始，这个参数不再可用，并为每个进程有一个独立的信号队列，数字是由RLIMIT_SIGPENDING确定。当队列变得满载 时，nginx开始抛弃连接并使用poll方法，直到负载恢复正常。
		--with-select_module 
				#允许select模块(一种轮询模式,不推荐用在高载环境)
		--without-select_module 
				#不使用select模块
				#标准连接模式。默认情况下自动编译方式。您可以启用或禁用通过使用-select_module和不带- select_module配置参数这个模块。
		--with-poll_module 
				#允许poll模块(一种轮询模式,不推荐用在高载环境)
				#标准连接模式。默认情况下自动编译方式。
		--without-poll_module 
				#不使用poll模块
		--with-http_ssl_module 
				#允许ngx_http_ssl_module模块(Apache对应:mod_ssl)
		--with-http_realip_module 
				#允许ngx_http_realip_module模块(mod_rpaf)
				#此模块支持显示真实来源IP地址，主要用于NGINX做前端负载均衡服务器使用。
		--with-http_addition_module 
				#允许ngx_http_addition_module模块(mod_layout)
				#游戏服务器不必安装，门户网站可以安装，有利于被搜索引擎收录页面信息。
		--with-http_xslt_module 
				#允许ngx_http_xslt_module模块
				#这个模块是一个过滤器，它可以通过XSLT模板转换XML应答。0.7.8后面版本才可以使用。
		--with-http_sub_module 
				#允许ngx_http_sub_module模块
				#这个模块可以能够在nginx的应答中搜索并替换文本。
		--with-http_dav_module 
				#允许ngx_http_dav_module模块(mod_dav)
				#为文件和目录指定权限，限制不同类型的用户对于页面有不同的操作权限
		--with-http_flv_module 
				#允许ngx_http_flv_module模块(mod_flvx)
				#这个模块支持对FLV（flash）文件的拖动播放。
		--with-http_gzip_static_module 
				#允许ngx_http_gzip_static_module模块(mod_dflate)。
				#这个模块在一个预压缩文件传送到开启Gzip压缩的客户端之前检查是否已经存在以“.gz”结尾的压缩文件，这样可以防止文件被重复压缩。
		--with-http_random_index_module 
				#允许ngx_http_random_index_module模块(mod_autoindex)，从目录中选择一个随机主页
		--with-http_stub_status_module 
				#允许ngx_http_stub_status_module模块(mod_status)
				#这个模块可以取得一些nginx的运行状态，如果是工业状况，可以直接取消。
		--without-http_charset_module 
				#不使用ngx_http_charset_module模块
				#这个模块将在应答头中为"Content-Type"字段添加字符编码
		--without-http_gzip_module 
				#不使用ngx_http_gzip_module模块，文件压缩模式。
		--without-http_ssi_module 
				#不使用ngx_http_ssi_module模块，此模块处理服务器端包含文件(ssi)的处理.
		--without-http_userid_module 
				#不使用ngx_http_userid_module模块
		The module ngx_http_userid_module gives out cookies for identification of clients
		--without-http_access_module 
				#不使用ngx_http_access_module模块
		--without-http_auth_basic_module 
				#不使用ngx_http_auth_basic_module模块
		--without-http_autoindex_module 
				#不使用ngx_http_autoindex_module模块
		--without-http_geo_module 
				#不使用ngx_http_geo_module模块
				#这个模块基于客户端的IP地址创建一些ngx_http_geoip_module变量，并与MaxMindGeoIP文件进行匹配，该模块仅用于 0.7.63和0.8.6版本之后。但效果不太理想，对于城市的IP记录并不是特别准确，不过对于网站的来源访问区域的分析大致有一定参考性。
		--without-http_map_module 
				#不使用ngx_http_map_module模块
				#这个模块允许你分类或者同时映射多个值到多个不同值并储存到一个变量中，可用于页面跳转其他域名使用。
		--without-http_referer_module 
				#不使用ngx_http_referer_module模块
				#当一个请求头的Referer字段中包含一些非正确的字段，这个模块可以禁止这个请求访问站点。可以禁止盗链的情况发生。
		--without-http_rewrite_module 
				#不使用ngx_http_rewrite_module模块，跳转模块
		--without-http_proxy_module 
				#不使用ngx_http_proxy_module模块，代理模块
		--without-http_fastcgi_module 
				#不使用ngx_http_fastcgi_module模块
		--without-http_memcached_module 
				#不使用ngx_http_memcached_module模块
		--without-http_limit_zone_module 
				#不使用ngx_http_limit_zone_module模块
				#此模块可以限制并发连接，达到减少攻击的效果
		--without-http_empty_gif_module 
				#不使用ngx_http_empty_gif_module模块
				#这个模块在内存中保存一个能够很快传递的1×1透明GIF
		--without-http_browser_module 
				#不使用ngx_http_browser_module模块
				#识别客户端浏览器版本来制定计划。可以实现更加来源浏览器的版本访问不同页面的效果。
		--without-http_upstream_ip_hash_module 
				#不使用ngx_http_upstream_ip_hash_module模块
		--with-http_perl_module 
				#允许ngx_http_perl_module模块，这个模块允许nginx使用SSI调用perl或直接执行perl 
		--with-perl_modules_path=PATH 
				#设置perl模块路径
		--with-perl=PATH 
				#设置perl库文件路径
		--http-log-path=PATH 
				#设置access log文件路径
		--http-client-body-temp-path=PATH 
				#设置客户端请求临时文件路径
		--http-proxy-temp-path=PATH 
				#设置http proxy临时文件路径
		--http-fastcgi-temp-path=PATH 
				#设置http fastcgi临时文件路径
		--without-http 
				#不使用HTTP server功能，如果只是做代理服务器，可以不提供http服务
		--with-mail 
				#允许POP3/IMAP4/SMTP代理模块
		--with-mail_ssl_module 
				#允许ngx_mail_ssl_module模块
				#这个模块使得POP3／IMAP／SMTP可以使用SSL／TLS.配置已经定义了HTTP SSL模块，但是不支持客户端证书检测
		--without-mail_pop3_module 
				#不允许ngx_mail_pop3_module模块
		--without-mail_imap_module 
				#不允许ngx_mail_imap_module模块
		--without-mail_smtp_module 
				#不允许ngx_mail_smtp_module模块
		--with-google_perftools_module 
				#允许ngx_google_perftools_module模块(调试用)
				#这个模块可以启用google性能分析工具套件，模块在版本0.6.29增加。
		--with-cpp_test_module 
				#允许ngx_cpp_test_module模块
		--add-module=PATH 
				#允许使用外部模块,以及路径
		--with-cc=PATH 
				#设置C编译器路径
		--with-cpp=PATH 
				#设置C预处理路径
		--with-cc-opt=OPTIONS 
				#设置C编译器参数
		--with-ld-opt=OPTIONS 
				#设置连接文件参数
		--with-cpu-opt=CPU 
				#为指定CPU优化,可选参数有:
	pentium, pentiumpro, pentium3, pentium4,
	athlon, opteron, sparc32, sparc64, ppc64
		--without-pcre 
				#不使用pcre库文件，pcre是包括 perl 兼容的正规表达式库
		--with-pcre=DIR 
				#设定PCRE库路径
		--with-pcre-opt=OPTIONS 
				#设置PCRE运行参数
		--with-md5=DIR 
				#设定md5库文件路径
		--with-md5-opt=OPTIONS 
				#设置md5运行参数
		--with-md5-asm 
				#使用md5源文件编译
		--with-sha1=DIR 
				#设定sha1库文件路径
				#sha1：安全哈希算法（Secure Hash Algorithm）主要适用于数字签名标准（Digital Signature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA）。对于长度小于2^64位的消息，SHA1会产生一个160位的消息摘要。当接收到消息的时候，这个消息摘要可以用来验证数据的完整性。在传输的 过程中，数据很可能会发生变化，那么这时候就会产生不同的消息摘要。
		--with-sha1-opt=OPTIONS 
				#设置sha1运行参数
		--with-sha1-asm 
				#使用sha1源文件编译
		--with-zlib=DIR 
				#设定zlib库文件路径
				#很多程序中的压缩或者解压缩函数都会用到这个库
		--with-zlib-opt=OPTIONS 
				#设置zlib运行参数
		--with-zlib-asm=CPU 
				#使zlib对特定的CPU进行优化,可选参数:
	pentium, pentiumpro
		--with-openssl=DIR 
				#设定OpenSSL库文件路径
				#Openssl作为一个基于密码学的安全开发包，OpenSSL提供的功能相当强大和全面，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。
		--with-openssl-opt=OPTIONS 
				#设置OpenSSL运行参数
		--with-debug 
				#允许调试日志

