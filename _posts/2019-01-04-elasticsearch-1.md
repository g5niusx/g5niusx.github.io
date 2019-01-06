---
layout: post
title: 'Elasticsearch 系列 ———— (一)安装 Elasticsearch'
tags: [elasticsearch]
---

## Elasticsearch 是什么?
Elasticsearch(ES)是一个个高效的全文检索引擎，可以帮我们完成全文搜索功能。同时它还是一个文档型的分布式数据库，毕竟存储了数据才能进行搜索。

## 安装 Elasticsearch

去[官网下载页](https://www.elastic.co/downloads/elasticsearch)下载最新的安装包，如果是 windows 系统，直接启动 `bin\elasticsearch.bat`
如果是 linux 系统直接启动 `bin\elasticsearch` 然后通过登陆 `http://localhost:9200/` 来检查服务是否启动成功。

**Elasticsearch 默认至少需要1GB的内存，否则无法启动，我的阿里云服务器最低配没法启动 - -!**

![安装页面]({{"/public/images/elasticsearch/es-01.png"}} "安装页面")

## 和 Elasticsearch 交互

Elasticsearch 支持 RESTful API 的交互，通过传输json数据可以完成对数据的搜索,后续也是通过 api 来充当客户端对 Elasticsearch 做增删改查。

比如 `http://localhost:9200/_count?pretty` 这个http请求是获取了本地 Elasticsearch 的文档数量，后面的参数`pretty`代表返回的json数据是否格式化




