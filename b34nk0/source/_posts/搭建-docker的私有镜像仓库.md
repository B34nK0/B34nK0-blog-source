---
title: 搭建 docker的私有镜像仓库
date: 2022-01-27 10:48:40
categories: 运维
tags: kebernetes
---

# 概要
Docker registry是专门用于存放docker镜像的，docker官方提供了docker hub，是全球最大的docker镜像存储中心。但是在中国既没有服务器也没有CDN，所以导致pull镜像特别的慢，而且很不稳定。解决这个问题的方式一般有两种：

1、搭建自己私有的docker registry，存储镜像，并定期同步官方常用的镜像。
2、搭建docker mirror。

搭建 mirror镜像相比搭建registry私有存储仓库来说，维护成本低，因为mirror主要通过缓存的方式解决速度慢不稳定的问题，不需要维护版本。
<!--more-->  

# morror部署

1、获取registry镜像的配置，这里采用2.5.0版本
```shell
docker run -it --rm --entrypoint cat registry:2.5.0  /etc/docker/registry/config.yml > config.yml
```

registry采用的搭建方式也是docker

2、修改配置config.yml，配置proxy
```shell
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
proxy:
  remoteurl: https://registry-1.docker.io
```

3、启动registry docker，并配置docker镜像的存储位置 /data
```
docker run  --restart=always -p 5000:5000 --name v2-mirror -v /data:/var/lib/registry -v  $PWD/config.yml:/etc/registry/config.yml registry:2.5.0 /etc/registry/config.yml
```
如果采用docker-compose启动
```shell
version: '2'
services:
  registry:
    image: library/registry:2.5.0
    container_name: registry_mirror
    restart: always
    volumes:
      - /data:/var/lib/registry
      - ./config.yml:/etc/registry/config.yml
    ports:
      - 5000:5000
    command:
      ["serve", "/etc/registry/config.yml"]
```

4、使用配置的mirror镜像仓库

修改 Docker 的配置文件 daemon.json
在 /etc/docker/daemon.json 文件中，增加镜像源
```shell
{ 
    "registry-mirrors": ["http://127.0.0.1:5000"] 
}
```
重新加载配置
```shell
systemctl reload docker
```
测试拉取镜像
```shell
docker pull centos
docker pull ubuntu
```

这个时候拉取docker镜像时，会先到docker hub去获取镜像的index，然后在mirror比对是否有符合的镜像缓存，如果没有的话会到hub去拉取并缓存到registry mirror中