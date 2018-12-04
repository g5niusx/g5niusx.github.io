---
layout: post
title: 'Docker 基础使用'
tags: [docker]
---

## Docker中的镜像和容器

docker的核心就是镜像(image)和容器(container),简单理解就是**容器是由镜像+文件系统组成的**，容器好比是一辆汽车，镜像就是汽车的发动机，文件就是汽车的
外部构成。这样结合在一起就是一个可以运行的汽车，也就是程序。

**Docker并不是一个虚拟机,Docker比虚拟机更加的方便和轻巧**

## Docker常用命令

`docker images`  显示所有的镜像
`docker image rm [imageid]` 删除本地的镜像,如果这个镜像被使用将无法删除
`docker system df` 查看docker占用的磁盘大小
`docker ps` 显示正在运行的容器
`docker exec -it [containerid] run /bin/bash` 进入容器的终端

**容器进入以后就是一个系统，如果容器启动失败，配置文件无法加载都可以进入容器去查看配置文件有没有加载，以及启动失败的原因**

## DockerFile

DockerFile是用来构建镜像的基础，一个镜像可以通过DockerFile完美的构建出来

- 编写DockerFile

下面的这个dockerfile表示基于nginx构建了一个镜像，将`<h1>test docker file</h1>`写入到了`/usr/share/nginx/html/index.html`文件里面。
这样我们在启动这个镜像的时候，nginx的首页就会变成`test docker file`

```docker
from nginx
# 修改首页内容
run echo '<h1>test docker file</h1>' > /usr/share/nginx/html/index.html
```

- 生成镜像
在写完DockerFile之后还需要使用过 `docker build `将DockerFile构建成一个真正的镜像

```
docker build -t my-nginx .
```

构建结果如下

![docker build]({{"/public/images/docker/docker-build.png"}} "docker build")

运行 `docker run -p 90:80 -d  my-nginx` 访问对应服务器的90端口就可以看到效果

![docker result]({{"/public/images/docker/docker-build-result.png"}} "docker result")