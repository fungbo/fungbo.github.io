---
layout: post
title: "Set up your own ghost"
date: 2016-05-01 12:00:00
comments: true
categories:
---

#### Run Ghost container
**Ghost**是一个开源的blog平台，并且已经提供了Docker image，下面就基于Docker来搭建自己的blog。

`docker run —name ghost -d -p 2368:2368 -v /home/ubuntu/ghost/blog:/var/lib/ghost ghost`  

只需要运行*ghost*的image即可, Ghost默认运行在2368端口。
然后，你就可以通过http://your-website.com:2368 访问了

#### Visit Ghost by subdomain
按照http://your-website.com:2368 访问的话需要host开发2368端口，并且不好记忆。我们希望通过http://blog.your-website.com 的方式来访问。

1. 在你的DNS服务商处设置A记录*blog*， 地址指向你的ip

2. 进入*/etc/apache2/sites-available*，新增blog.conf

3. 写入如下内容，设置VirtualHost
	``` apache
	<VirtualHost *:80>
	    ServerName blog.your-website.com
	    ProxyPreserveHost on
	    ProxyPass / http://localhost:2368/
	</VirtualHost>
	```    

4. Reload apache2
`sudo service apache2 reload`
