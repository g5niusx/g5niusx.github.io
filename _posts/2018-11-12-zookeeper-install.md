---
layout: post
title: 'Zookeeper 安装'
tags: [code]
---

参加了<a href="https://blog.biezhi.me/" target="_blank">biezhi</a>大佬组织的`20DaysOfCode`,需要在第一个阶段写出一个简单的RPC轮子,因为之前没有接触过RPC项目，所以先安装一个zookeeper体验一下
### 下载
去[官网](https://www.apache.org/dyn/closer.cgi/zookeeper/)下载安装包我下载的是`http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz`
```
1. wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
2. tar -zxvf zookeeper-3.4.13.tar.gz
```
### 配置Zookeeper
下载完zookeeper以后，如果直接去bin目录下面运行`./zkServer.sh`会报错在conf下面没有zoo.cfg文件,需要我们手动创建出来。
进入到conf目录下面创建zoo.cfg文件,将zoo_sample.cfg的内容复制到zoo.cfg即可或者直接将zoo_sample.cfg重命名为zoo.cfg都可以。

```
# Client和Server通信心跳时间
tickTime=2000
# 集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）
initLimit=10
# 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数
syncLimit=5
# 数据文件目录
dataDir=/root/zookeeper/zookeeper-3.4.13/data
# 客户端连接端口
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

```
**注意事项**

**默认的数据目录是一个临时目录，需要修改成自己创建的目录或者指定已经存在的目录都可以**

### 常用的Zookeeper命令

`启动服务` ./zkServer.sh start   
`停止服务` ./zkServer.sh stop  
`显示状态` ./zkServer.sh status  
`重启服务` ./zkServer.sh restart  

后续如果要用到集群模式，再来更新博客





