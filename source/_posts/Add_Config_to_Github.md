---
title: 连接Github出现"closed by 192.30.255.112 port 22"
date: 2023-11-26 21:43:00
tags: Programming
---

今天在测试与GitHub的连接时出现了远程连接被关闭的错误：

    closed by 192.30.255.112 port 22
<!--more-->

## 在~/.ssh下新建config文件 ##

打开~/.ssh，新建一个名为config的文件，写入以下内容：

    Host github.com
    User YourGitHubEmail@example.com
    Hostname ssh.github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
    Port 443

保存后，再次执行测试连接即可。

排除网络以及代理问题后，可尝试删除~/.ssh下所有的文件后重试以上步骤。