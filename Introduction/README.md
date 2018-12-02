
# 介绍

## 目录
- [文档数据库](#document-database)
- [核心特性](#key-features)
    - [高性能](#high-performance)
    - [丰富的查询语言](#rich-query-language)
    - [高可用](#high-availability)
    - [水平扩展](#horizontal-scalability)
    - [支持多种存储引擎](#support-for-multiple-storage-engines)

MongoDB 是一个高性能，高可用，自动伸缩的开源数据库。


## Document Database
MongoDB的一行记录叫做一个文档（相当于MySQL的一行）， 这个文档是一种键值结构，类似于JSON类型。这个键对应的值既可以是字符串，数组或者是其它文档(即文档的嵌套)。

使用文档类型（document）的优点如下：
- 文档这种类型在众多编程语言中比较贴近于原始数据
- 文档的嵌套可以减少合并（join）这种操作带来的昂贵开销
- Dynamic schema supports fluent polymorphism
  

## Key Features
### High Performance
MongoDB 提供高性能的数据存储，尤其是：
- 支持嵌套的数据模型，减少IO操作带来的损耗
- Indexes support faster queries and can include keys from embedded documents and arrays.


### Rich Query Language
MongoDB 支持丰富的读写操作（[CRUD](https://docs.mongodb.com/manual/crud/)），比如：
- [数据聚合](https://docs.mongodb.com/manual/core/aggregation-pipeline/)
- [文本搜索](https://docs.mongodb.com/manual/text-search/)和[位图查询](https://docs.mongodb.com/manual/tutorial/geospatial-tutorial/)（ Geospatial Queries）

### High Availability
高可用的代表是MongoDB的副本集（replica set），提供：
- 自动故障转移
- 数据冗余（因为备份了数据，所以肯定会产生冗余）

副本集是一组维护者相同数据集的mongod进程，从而增加了数据的可用性（丢失概率小）

### Horizontal Scalability
MongoDB把水平扩展（即分片）作为它的核心功能之一：
- [Sharding](https://docs.mongodb.com/manual/sharding/#sharding-introduction) distributes data across a cluster of machines
- Starting in 3.4, MongoDB supports creating [zones](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding) of data based on the shard key. In a balanced cluster, MongoDB directs reads and writes covered by a zone only to those shards inside the zone. See the Zones manual page for more information

### Support for Multiple Storage Engines
MongoDB支持以下的存储引擎：
- [WiredTiger Storage Engine](https://docs.mongodb.com/manual/core/wiredtiger/) (including support for [Encryption at Rest](https://docs.mongodb.com/manual/core/security-encryption-at-rest/))
- [In-Memory Storage Engine](https://docs.mongodb.com/manual/core/inmemory/)
- [MMAPv1 Storage Engine](https://docs.mongodb.com/manual/core/mmapv1/) (MongoDB 4.0被移除)

另外，MongoDB还提供了可插拔的存储引擎API，允许第三方自己开发存储引擎






