---
layout: post
title: 'Hello Docker'
tags: [docker]
---

## Docker安装
- 安装docker的依赖
1. `sudo yum install -y yum-utils  device-mapper-persistent-data  lvm2`
2. `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
3. `sudo yum-config-manager --enable docker-ce-edge`
4. `sudo yum-config-manager --enable docker-ce-test`
5. `sudo yum install docker-ce`

安装docker-ce是因为docker-ce是开源的项目，而docker-ee是一个商业产品，需要付费。

如果在不报错的情况下，docker就会安装完毕，运行docker --version就可以看到版本号

![docker安装完毕]({{"/public/images/docker/docker-installed.png"}} "docker安装完毕")

运行docker `systemctl start docker`

![docker运行]({{"/public/images/docker/docker-start.png"}} "docker运行")

停止docker `systemctl stop docker`


## Docker简单体验,在容器里面运行Tomcat

`docker run -p 8080:8080 tomcat`通过一行简单的命令，就可以在docker里面创建并启动一个tomcat。通过访问主机的 **8080** 端口就可以看到tomcat的界面

**一定要联网才可以安装成功tomcat,因为docker会从远程仓库下载tomcat的镜像**

![docker运行tomcat]({{"/public/images/docker/docker-tomcat.png"}} "docker运行tomcat")




