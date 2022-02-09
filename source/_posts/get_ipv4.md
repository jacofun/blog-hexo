---
title: Nginx搭建获取访问主机公网ip的网页
---

只需在新的虚拟主机配置中添加如下内容即可：
<!--more-->



	server {
    	server_name ipv4.jaco.fun;
    	listen 80;

    	location / {
        	default_type text/plain;
        	return 200 "$remote_addr" ;
    }

别忘记添加A记录指向ipv4.jaco.fun
访问此页面时反显访问者的公网ip地址，需要做ddns的人应该会用到吧。
