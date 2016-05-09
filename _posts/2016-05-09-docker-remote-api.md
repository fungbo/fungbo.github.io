---
layout: post
title: "Docker remote api"
date: 2016-05-09 12:00:00
comments: true
categories:
---

通常操作docker时，docker的client和daemon都在同一台机器上。其实docker还提供了remote api，
remote api就是一些Restful api，通过这些api，就可以达到远程操作docker的目的，虽然remote api还不完善。

### 如何打开remote api
编辑远程机器的*/etc/init/docker.conf*的script中的DOCKER_OPTS   

`DOCKER_OPTS='-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'`

### 下载docker镜像到远程机器
`curl -s -XPOST http://remote-server:4243/images/create?fromImage=bfeng/jenkins`
>fromImage后面的镜像默认会从DockerHub上下载，如果想从自己的仓库下载需要加上仓库的地址， 例如：   
>`curl -s -XPOST http://remote-server:4243/images/create?fromImage=10.230.230.10:5000/bfeng/jenkins`   
>因为仓库默认使用https，所以如果你的仓库是http的话，要在修改DOCKER_OPTS   

>`DOCKER_OPTS='-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock --insecure-registry 10.230.230.10:5000'`

### 创建container
通过remote api没办法直接运行container，container的创建和运行是分开的

`curl -X POST -H "Content-Type: application/json" http://remote-server:4243/containers/create -d '{"Image":"bfeng/android-test"}'`   
>在远程机器上根据*bfeng/android-test*的image创建container

详细的可以json参数信息:   

``` json
curl -X POST -H "Content-Type: application/json" http://remote-server:4243/containers/create -d '{
     "Hostname":"",
     "User":"",
     "Memory":0,
     "MemorySwap":0,
     "AttachStdin":false,
     "AttachStdout":true,
     "AttachStderr":true,
     "PortSpecs":null,
     "Privileged": false,
     "Tty":false,
     "OpenStdin":false,
     "StdinOnce":false,
     "Env":null,
     "Dns":null,
     "Image":"vieux/elasticsearch",
     "Volumes":{},
     "VolumesFrom":"",
     "WorkingDir":""
}'
```
>悲剧的是没有发现Name，这就意味着只能捕获id，之后通过id来启动container了

### 启动container
`curl -XPOST http://remote-server:4243/containers/container_id`

Remote api还有其他的接口，比如获得远程机器的image和container信息等，这里就不一一介绍了。
