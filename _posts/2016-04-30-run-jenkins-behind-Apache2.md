---
layout: post
title: "Run Jenkins behind Apache"
date: 2016-04-30 12:00:00
comments: true
categories:
---

有时，处于种种原因，我们想让Jenkins运行在Web server后面，比如Apache，这时就需要反向代理设置。

上一篇，使用Docker已经把Jenkins运行起来了, 输入http://your-website.com:8080/jenkins就可以登录Jenkins。
但是如果不想把8080端口暴露出去，或者通过更表意的url访问。比如想用http://your-website.com/jenkins来访问jenkins，将来使用http://your-website.com/blog来访问博客，这就需要Reverse proxy。Apache和Nginx都有反向代理的功能，设置也很简单。

#### 在Apache后面运行Jenkins
1. 给Jenkins加上prefix
要把Jenkins运行在Web server后面，需要给Jenkins加上Prefix，就是上一篇提到的
`JENKINS_OPTS --httpPort=8080 --prefix=/jenkins`
>需要在JENKINS_OPTS上指定--prefix=/jenkins

2. 设置Apache反向代理
需要用到Apache的两个模块
 ``` bash
 sudo a2enmod proxy
 sudo a2enmod proxy_http
 ```
进入*/etc/apache2/sites-available*，直接编辑默认的配置文件*000-default.conf*或者新建一个自己的配置文件都可以
``` apache
ProxyPass /jenkins http://localhost:8080/jenkins nocanon
ProxyPassReverse /jenkins http://localhost:8080/jenkins
ProxyRequests Off
AllowEncodedSlashes NoDecode

<Proxy http://localhost:8080/jenkins*>
	Order deny,allow
	Allow from all
</Proxy>
```
所有/jenkins后缀的都被映射到8080端口的jenkins。
`sudo service apache2 reload`
重新加载Apache

如果是创建自己的配置文件，还要激活配置文件，再重启Apache
`sudo a2ensite you.conf`
