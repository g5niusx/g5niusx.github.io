---
layout: post
title: 'Elasticsearch 系列 ———— (一) 安装 Elasticsearch'
tags: [elasticsearch]
---

## Elasticsearch 是什么?
Elasticsearch(ES) 是一个高效的全文检索引擎，可以帮我们完成全文搜索功能。同时它还是一个文档型的分布式数据库，毕竟存储了数据才能进行搜索。

## 安装 Elasticsearch

去[官网下载页](https://www.elastic.co/downloads/elasticsearch)下载最新的安装包，如果是 windows 系统，直接启动 `bin\elasticsearch.bat`
如果是 linux 系统直接启动 `bin\elasticsearch` 然后通过登陆 `http://localhost:9200/` 来检查服务是否启动成功。

**Elasticsearch 默认至少需要1GB的内存，否则无法启动，我的阿里云服务器最低配没法启动 - -!**

![安装页面]({{"/public/images/elasticsearch/es-01.png"}} "安装页面")

## Elasticsearch 配置

- 应用配置
Elasticsearch 默认是不需要配置的，如果想对当前 Elasticsearch 的名称自定义或者日志输出路径自定义，可以修改 `config/elasticsearch.yml` 文件

```yaml
# 集群名称
cluster.name: cluster
# 当前节点名称
node.name: node-1
# 数据存储路径
path.data: /path/to/data
# 日志存储路径
path.logs: /path/to/logs
# 主机地址，默认为localhost,在集群的情况下需要修改
network.host: 192.168.0.1
# 发现指定的节点,如果没有指定端口，则会使用[transport.profiles.default.port]或者[transport.tcp.port]的端口
discovery.zen.ping.unicast.hosts:
    - 192.168.1.10:9300
    - 192.168.1.9:9301
# 设置最大节点数,计算公式为 (nodeCount/2)+1，所有的节点数量除以二再加1
discovery.zen.minimum_master_nodes: 2

```
- jvm 配置

可以修改 `config/jvm.options`，一般不需要配置





