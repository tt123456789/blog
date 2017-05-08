Title: Nginx 后端健康检查详解
Date: 2016/5/14 10:38:49 
Modified: 2016/5/14 10:38:52 
Category: Servers
Tags: nginx
Slug: 
Author: allposs

##简介##
&#160; &#160; &#160; &#160;nginx目前，nginx对后端节点健康检查的方式主要有3种，一种是nginx自带的ngx_http_proxy_module 模块和ngx_http_upstream_module模块;一种是nginx_upstream_check_module模块，还有一个就是ngx_http_healthcheck_module模块。反向代理包含内置的或第三方扩展来实现服务器健康检测的。如果后端某台服务器响应失败，nginx会标记该台服务器失效，在特定时间内，请求不分发到该台上。

##版本##

V1.0

##正文##


目前，nginx对后端节点健康检查的方式主要有3种，如下：

+ 1、ngx_http_proxy_module 模块和ngx_http_upstream_module模块（自带）
+     官网地址：http://nginx.org/cn/docs/http/ng ... proxy_next_upstream
+ 2、nginx_upstream_check_module模块
+     官网网址：https://github.com/yaoweibin/nginx_upstream_check_module
+ 3、ngx_http_healthcheck_module模块
+     官网网址：http://wiki.nginx.org/NginxHttpHealthcheckModule
&#160; &#160; &#160; &#160;严格来说，nginx是没有自带针对负载均衡后端节点的健康检查的模块，但是可以通过默认自带的ngx_http_proxy_module 模块和ngx_http_upstream_module模块中的相关指令来完成当后端节点出现故障时，自动切换到健康节点来提供访问的要求。
###1. ngx_http_proxy_module 模块
&#160; &#160; &#160; &#160;ngx_http_proxy_module 模块的作用是把这台服务器的请求发送到其它服务器上，也就现在所说的代理，ngx_http_proxy_module 模块也就是代理模块。
配置示例

    location / {
        proxy_pass       http://localhost:8000;
        proxy_set_header Host      $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
