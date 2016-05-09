---
layout: post
title: "Set up Jenkins with Docker"
date: 2016-04-29 12:00:00
comments: true
categories:
---

用Docker搭建Jenkins，可以快速方便的复制CI环境。比如可以在本地尝试搭建，搭建完成后将数据和Dockerfile迁移至云端等。

### 想法：
1. Jenkins自己就跑在docker container中
2. 所有的测试也跑在自己的container中
3. 逻辑上，测试的docker image在Jenkins中构建，并在Jenkins中运行
4. 实际上，所有的docker都在host上运行
5. 不管是Jenkins还是测试的image都保存在host上
6. Jenkins本身的数据信息也保存到host上

### Dockerfile
``` dockerfile
FROM jenkins:latest
MAINTAINER bfeng@thoughtworks.com
ENV REFRESHED_AT 2015_4_27

ENV JENKINS_OPTS --httpPort=8080 --prefix=/jenkins

USER root

RUN apt-get update -qq && apt-get install -qqy curl apt-transport-https
RUN apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
RUN echo deb https://apt.dockerproject.org/repo ubuntu-trusty main > /etc/apt/sources.list.d/docker.list
RUN apt-get update -qq && apt-get install -qqy iptables ca-certificates openjdk-7-jdk git-core docker-engine

RUN usermod -aG docker jenkins

EXPOSE 8080
```
>*基于最新的官方Jenkins镜像，在Jenkins中再安装docker。Jenkins运行在**8080**端口，前缀是**jenkins*，反向代理的时候会用到*

### 构建镜像
`sudo docker build -t bfeng/jenkins .`

### 运行Container
`sudo docker run --name jenkins -p 8080:8080 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root -d bfeng/jenkins`
>*将8080端口映射到host上的8080*
>*把jenkins_home目录挂载到host上，这样即使重新构建jenkins镜像，jenkins的数据也不会丢失*
>*需要把docker.sock挂载出来，在Jenkins中运行的docker才能实际跑在host上*

这样的话，Jenkins就运行起来了，输入http://your-website.com:8080/jenkins就可以看到Jenkins的界面了。

如果想设置反向代理，可以参考下一篇。
