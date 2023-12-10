---
title: 在服务器部署Hexo博客（Debian 11）
---
### 写在前面
网上教程更多的是部署到Github，因为大部分操作在本地完成，所以部署起来很快，出错率低。
<!--more-->


但是由于GitHub的访问速度众人皆知，所以我更倾向于将网站部署到自己的服务器上。然而一旦涉及到服务器，就躲不开在Linux里折腾，对初学者来说安装并配置好一个软件就已经十分不易了，更何况独立部署一个项目。

我不敢说自己对Linux系统有着多么高深的理解与掌握，因为自己也是从照搬一键脚本的初学者一步一步走过来的。一键脚本可以用，但是最终还是要自己去研究，没有人会一直为你写脚本，更别说很多的一键脚本还藏着后门和漏洞...

### 为什么用Hexo
我在[第一篇文章](https://yanxiao.me/2023/02/28/hi_jaco/)里有写，主要原因还是可玩性强，效率高，速度快。在写第一篇文章的时候就在酝酿怎么将hexo部署到自己的服务器上，今天才算是弄好。

根据我的理解，Hexo是一个生成静态博客页面的框架，hexo的最终产品是页面，因此写作工作在本地进行，接着由Hexo渲染成静态页面，最终部署到Web服务器上即可。相比于Wordpress通过访问Web后台进行编辑及操作要更简单。


但是每次登录Shell上传网页源代码再service nginx reload未免太过麻烦，况且有时只添加一篇post就上传整个网页的源代码未免有些小题大作，于是我们需要配合使用git以及git的hook（钩子）功能。


git的作用是版本控制，钩子的作用是侦测到仓库有提交时自动部署，也就是当有代码push的时候，自动更新至网站root文件夹（部署为GitHub Pages上则自动更新）。这样一来，一旦服务器环境搭建完毕，除非必要维护，可以再不用登录服务器操作了。

对于本地端来说，Hexo的工作需要nodejs做支撑，同时也需要git做版本控制。我建议将Hexo框架的代码（内含配置文件、模板、插件等，即hexo根目录）托管至Github上。

## 服务器配置（Debian 11）

### 1.安装git

	$ sudo apt-get update && install git -y

### 2.配置git用户
#### 添加git用户

	$ sudo useradd git
	$ sudo passwd git

为git用户配置sudo权限：

	$ sudo chmod 740 /etc/sudoers #sudoers默认不可修改
	$ sudo vi /etc/sudoers

找到
	
	root ALL=(ALL) ALL
	...

在它下方加入一行

	git ALL=(ALL) ALL

<kbd>:wq</kbd>后将sudoers改为不可写

	$ sudo chmod 400 /etc/sudoers

注意：如果<kbd>/etc</kbd>中没有sudoers，请先安装sodo

	$ sudo apt-get install sudo

#### 为git用户添加ssh密钥

	$ su git #切换至git用户
	$ mkdir -p ~/.ssh #在家目录创建.ssh文件夹
	$ ssh-keygen -t rsa 
	$ cat ./id_rsa.pub >> ./authorized_keys 

#### 修改git用户默认shell

	$ sudo vi /etc/passwd

将git的shell改为<kbd>/usr/bin/git-shell</kbd>， 目的是禁止git用户登录到shell。

### 4.配置git仓库

	$ sudo mkdir -p /var/repo #网页源代码仓库
	$ sudo mkdir -p /var/www/hexo #网页根目录
	
在<kbd>/var/repo</kbd>下

	$ sudo git init --bare hexo-blog.git #新建一个叫hexo-blog的裸仓库
	$ sudo vi /var/repo/hexo-blog.git/hooks/post-update

写如下内容：

	#!/bin/bash
	git --work-tree=/var/www/hexo --git-dir=/var/repo/blog.git checkout -f

给post-update可执行权限：

	$ sudo chmod +x post-update 

变更仓库和根目录所有权：

	$ sudo chown -R git:git /var/repo 
	$ sudo chown -R git:git /var/www/hexo


### 5.安装Nginx
这块没什么好说的，<kbd>sudo apt-get install nginx -y</kbd>就可以，也可以[编译安装Nginx](https://yanxiao.me/2023/02/28/Nginx_install/)

在配置nginx时，将网页根目录(root)修改为/var/www/hexo

## 本地配置（Windows 10）
这一块就简单许多，基本上以gui操作为主

### 1.安装Git和Nodejs
在[https://git-scm.com/downloads](https://git-scm.com/downloads)下载Windows版本的git
在[https://nodejs.org/en/](https://nodejs.org)下载Windows版本的Node.js

安装时使用默认配置，一直下一步直到安装结束。
### 2.安装hexo

新建一个文件夹作为本地代码仓库，右键<kbd>Git Bash Here</kbd>

	$ npm install hexo-cli -g
	$ hexo init blog
	$ cd blog
	$ npm install

Hexo框架安装完成。
	
### 3.推荐安装NexT主题（可选）
[Github项目地址](https://github.com/next-theme/hexo-theme-next)



在<kbd>blog/theme/</kbd>下

	$ git clone git@github.com:next-theme/hexo-theme-next.git

clone项目需要注册Github，或着

	$ wget https://github.com/next-theme/hexo-theme-next.git

在<kbd>blog/_config.yml</kbd>中修改

	theme: next

在<kbd>deploy</kbd>下修改

	deploy：
	  type: git
	  repo: ssh://git@ip:port/var/repo/blog-hexo.git	
	  branch: master

ip和port是服务器的公网ip和ssh端口
将git用户的私钥下载到<kbd>C:\Users\\{你的用户名}\\.ssh</kbd>

	$ ssh -T ip地址 -p 端口

来测试是否连接成功 
	
### 4.推荐使用MarkdownPad 2 编辑md文件（可选）
[http://markdownpad.com/](http://markdownpad.com/)

### 5.部署
#### 如何写博文？
在<kbd>/blog/source/_posts</kbd>下新建md文件即可

	$ git add -A
	$ git commit -m "Update a new post" #引号内容可自拟
	$ git push origin remote #将Hexo源码push至Github
	$ hexo clean
	$ hexo g
	$ hexo d
