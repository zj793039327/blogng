---
title: redis知识点
date: 2016-11-21 17:57:18
modified: 2016-11-21 17:57:18
author: jojoster
postid: 192
slug: 192
nicename: redis_point
attachments: $ATTACHMENTS
layout: post
poststatus: publish
tags: [redis']
categories: tech
---

主要整理一些redis的知识点，方便自己回顾。
<!--more-->

# redis

## 概述、适合场景

### summary

* 支持发布订阅、主从复制、磁盘持久化
* 多种数据结构
* 短事务
* 高吞吐量、低延迟
* geo支持（地理位置支持，geo作为一个单独的数据结构，有自己的增上改查命令）
  * [geoadd](http://www.redis.io/commands/geoadd)

| Name     | Type                                    | Data Storage options                     | Query types                              | Additional features                      | Ops        | Latency  |
| -------- | --------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------- | -------- |
| Redis    | **In- memory non- relational database** | **Strings, lists, sets, hashes, sorted sets, geo** | Commands for each data type for common access patterns, and partial transaction support | **Publish/ Subscribe, master/ slave replication, disk persistence, scripting (stored procedures)** | **120000** | **<1ms** |
| MongoDB  | On-disk non- relational document store  | Databases of tables of schema-less BSON documents | Commands for create, read, update, delete, conditional queries, and more | Supports map- reduce operations, master/ slave replication, sharding, spatial indexes | 3500       | <100ms   |
| Memcache | In-memory key-value cache               | Mapping of keys to values                | Commands for create, read, update, delete, and a few others | Multithreaded server for additional performance | 80000      | <1ms     |
| MySQL    | Relational database                     | Databases of tables of rows, views over tables, spatial and third-party extensions | Select, insert, update, delete, functions, stored procedures | ACID compliant (with InnoDB), master/ slave and master/ master replication | 900        | \>100ms  |

## Redis介绍

### 存储实现

#### 内存数据结构

* redisServer
  * redisDb
    * dict
      * dictht
        * dictEntry（链表）
          * redisObject
            * type
            * encoding
            * ptr
    * expires(过期属性)

#### redisObject分类

每一种类型的object都有两种实现，分别是压缩和非压缩

* String
  * int（压缩）
  * sds
* List
  * linkedlist
  * ziplist（压缩）
* Hash
  * ht
  * ziplist（压缩）
* Set
  * ht
  * intset（压缩）
* Zset
  * skiplist
  * ziplist（压缩）

#### 数据存储类型

* INT：压缩存储string

  * 常量数字对象共享（0~9999）

* SDS：存储string

  * > int len;
    >
    > int free;
    >
    > char buf[]

  * 变长字符数组

  * 常用字符串共享

  * 优化长度计算

* LinkedList 双向列表，存储list

  * 优化长度计算
  * 支持双端遍历

* HT（hashtable）：存储set和hash

  * 扩展、收缩，根据填充绿 used/size
  * 渐进式hash：hash操作、事件触发

* INTSET：压缩存储set（整数）

* SKIPLIST：跳表，存储有序集

* ZIPLIST：压缩存储hash、list、zset

#### 内存管理、淘汰

##### 内存管理

* Zmalloc（匹配系统，根据系统使用不同的分配算法）
  * Tcmalloc、Jemalloc、Malloc（MAC）
    * 根据不同系统的内存分配长度不同，提前计算，分配指定长度
  * 内存统计

##### 内存淘汰

* 被动淘汰
  * 达到maxmemory，采用淘汰策略
    * lru
      * volatile-lru/allkeys-lru/
    * random
      * vloatile-random/allkeys-random/
    * ttl
      * vloatile-ttl/
    * Noeviction(simple)
    * 访问时淘汰
  * 主动淘汰（random）

#### 事件模型

##### 网络模型-libae

* Epoll
* Select
* Kqueue
* Evport(Solaris)

##### 事件类型

* TimeEvent
* FileEvent

#### 线程模型

* MainThread
* BioThread

### 重要feature

#### 主从同步

* 常规同步：异步复制
  * 修改不会直接到从库
  * 主库收到cmd以后，同时修改master_repl_off(repl状态表)
  * 使用real_buf存储最后修改的数据
  * 主库针对 每一个slave有一个output buf
  * 定时 分别将每一个output buf传输到指定的 slave
  * slave返回 real_off响应
  * 主库 根据返回值 修改 master_repl_off表
* 增量同步
* Redis状态机

#### 持久化

* RDB（snapshot）
  * Save（阻塞）/bgsave(非阻塞)
  * 文件格式
    * Redis
    * rdb-version
    * select-db
    * key-value-pairs
    * eof
    * check-sum
    * db-data
      * optional-expire-time
      * type-of-value
      * key
      * value


* rdbload，全量数据load到内存
  * 每一条数据 checksum检查
* 通信协议：RESP（redis serialization protocol）
  * 便于实现、解析、理解、二进制安全
* AOF（oplog）
  * 三种配置， 进行fsync
    * AOF_FSYNC_NO
    * AOF_FSYNC_EVERYSEC（每一秒写入）
    * AOF_FSYNC_ALWAYS（每一条cmd写入磁盘）
  * AOF load（fake client）
  * AOF rewrite
    * merge 不同的cmd，聚合cmd，最后只有一个cmd

### 集群方案

#### Twemproxy

​	高效的路由转发

* 根据key哈希，数据分片
* 实现
  * client_in 
  * server_in
  * server_out

#### redis-cluster

​	P2P

* Gossip协议
* Auto failover
* replicas migration
* online reshard，多个实例会成为一个分片

## Mysql&Redis增量同步方案

* 通过proxy分发请求
  * 写请求发送mysql
  * 读请求发送redis
* Mysql-binLog 获取数据库增量更新
* Databus、Broker进行消息队列写入
* Redis根据队列内容增量更新
