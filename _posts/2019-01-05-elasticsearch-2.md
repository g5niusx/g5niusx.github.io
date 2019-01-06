---
layout: post
title: 'Elasticsearch 系列 ———— (二) Elasticsearch 的索引'
tags: [elasticsearch]
---

## Elasticsearch 的增删改查

作为一个搜索引擎的同时，Elasticsearch 也是一个数据库，所以它也需要支持增删改查。Elasticsearch 的增删改查语句是基于`json`的`dsl`语句


### 索引

Elasticsearch 对索引的定义是存储一系列文档的集合，索引的名称必须为小写，后续操作文档的时候需要带上索引名称。

> 这个索引并不是关系型数据库里面的索引，它类似关系型数据库里面的 `database`,仅仅是类似，并不能当它当 `database` 来对待

#### 查询索引

通过使用 **get** 方法访问 `http://localhost:9200/_cat/indices?v` 这个地址，就可以在查看 Elasticsearch 里面所有的索引

![查询索引]({{"/public/images/elasticsearch/es-02.png"}} "查询索引")

> 上图中的 `green` 代表这个索引状态一切正常，`yellow` 代表这个索引数据状态正常，但是缺少了备份, `red` 代表这个索引不可用

#### 增加索引

通过使用 **put** 方法访问 `http://localhost:9200/app?pretty` 这个地址，就可以在 Elasticsearch 里面创建一个名称为`app`的索引，后续指定的数据都会存放到app这个索引里面。

#### 删除索引

通过使用 **delete** 方法访问 `http://127.0.0.1:9200/app` 这个地址，就可以在 Elasticsearch 删除名称为 `app` 的索引

### 总结

Elasticsearch 里面通过 `PUT` , `DELETE` , `POST` , `GET` 4种http的请求方法来对应关系型数据里面的增删改查,所以在通过http接口来操作数据的时候需要注意请求方法，不同的请求方法会造成不同的结果。












