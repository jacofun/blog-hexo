---
title: 编译安装Nginx
date: 2023-02-27 21:04:00
---
系统为Debian10，直接放步骤：
<!--more-->

### 1.更新源及安装依赖
	$ apt-get update

	$ apt install gcc make libpcre3 libpcre3-dev openssl libssl-dev zlib1g-dev

### 2.添加Nginx用户同时禁止登录shell
	$ adduser --system --home /nonexistent --shell /bin/false --no-create-home --gecos "nginx user" --group --disabled-login --disabled-password nginx

### 3.下载Nginx源码
版本选择的是2021年11月左右发布的1.21.4

	$ wget https://nginx.org/download/nginx-1.21.4.tar.gz && tar zxvf nginx-1.21.4.tar.gz
	$ cd nginx-1.21.4

### 4.配置&编译
	./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-pcre-jit --with-openssl-opt=no-nextprotoneg --with-debug
	$ make install

### 5.做软连接
	$ ln -s /usr/lib/nginx/modules /etc/nginx/modules
	$ mkdir /var/cache/nginx -p
	$ mkdir /etc/nginx/vhost -p

### 6.创建Nginx服务
	$ vi /lib/systemd/system/nginx.service
写入以下内容：

	[Unit]
	Description=nginx - high performance web server
	Documentation=http://nginx.org/en/docs/
	After=network-online.target remote-fs.target nss-lookup.target
	Wants=network-online.target


	[Service]
	Type=forking
	PIDFile=/var/run/nginx.pid
	ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
	ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
	ExecReload=/bin/kill -s HUP $MAINPID
	ExecStop=/bin/kill -s TERM $MAINPID

	[Install]
	WantedBy=multi-user.target

启动Nginx服务：

	$ systemctl daemon-reload
	$ systemctl enable nginx.service
	$ systemctl start nginx.service

### 7.设置Nginx开机启动
	$ systemctl is-enabled nginx.service


