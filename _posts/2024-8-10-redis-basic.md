---
title: Redis 数据库
author: shane
date: 2024-08-10
description: 基于内存的 key-value 结构数据库
categories: [SQL]
tags: [Redis]
comments: true
toc: true
---

## Redis 简介

基于**内存**的 `key-value` 结构数据库。

- 基于内存，读写性能高。
- 适合存储热点数据。

## 安装

```shell
sudo pacman -S redis
```

## 启动服务

```shell
systemctl start redis
```

## 配置

默认配置文件位于 `/etc/redis/redis.conf`。可以调整其中的配置项如：

- 默认端口：默认端口为 6379。
- 启用持久化：通过修改 `save` 和 `appendonly` 参数。
- 设置最大内存：使用 `maxmemory` 配置限制 Redis 使用的内存量。

配置好之后需要重启 Redis：

```shell
systemctl restart redis
```

## Redis 设置用户和 ACL（访问控制列表）

多用户或多应用场景中需要对 Redis 进行多用户设置以提高安全性。默认配置文件中的 `requirepass` 是 Redis 早期版本中设置密码的简单方式。它为所有连接到 Redis 的客户端设置了单一密码。

Redis 6.0 引入了 ACL，允许更灵活地管理用户和权限，限制用户可以执行的命令、访问的键空间，以及其他与安全相关的配置。

开启 ACL 需要在 `/etc/redis/` 目录中创建 `user.acl` 文件，在该文件中配置用户及权限：

```
user default off
user admin on >adminpassword ~* +@all
user readonly on >readonlypassword ~keys +get +mget
user writer on >writerpassword ~* +set +hset -@admin
```

上面是一个配置示例，比如设置默认登录的用户没有任何权限；设置 `admin` 用户密码为 `adminpassword`，权限为 `~* +@all` 所有；定义了一个名为 `readonly` 的用户，密码为 `readonlypassword`，只允许执行 keys、get 和 mget 命令，适合只读操作...

配置好 `users.acl` 文件后，在 `/etc/redis/redis.conf` 中取消注释 `aclfile /etc/redis/users.acl` 这一行。然后重启服务：

```bash
systemctl restart redis
```

再次使用 `redis-cli` 以默认用户进入 `redis` 后，默认没有任何权限。如果要以设置好的用户身份进入 redis，可以在 redis 中输入：

```
127.0.0.1:6379> AUTH username password
```
如果返回 `OK` 这表明登入成功。

当然，ACL 配置也可以在 Redis 中通过命令交互式配置，这里不多赘述，可以自行 Google。

## 关于 Redis 的持久化

持久化是指将数据保存到持久存储设备（如硬盘或SSD）上，以确保即使系统重启或发生故障，数据也不会丢失。对于数据库系统和缓存系统，持久化是一个重要的功能，因为它保障了数据的可靠性和持久性。

Redis 默认情况下所有数据都存储在内存中。为了避免意外导致的数据丢失，Redis 也提供了**三种**持久化机制。

### RDB（Radis DataBase）快照持久化

Redis 在指定时间间隔内或是满足一定条件时，将内存中的数据生成快照，并保存为一个二进制文件 `dump.rdb`。RDB 文件是一个非常紧凑的文件，方便进行备份，可以快速恢复到内存中。但是如果 Redis 发生意外，可能会丢失最后一次快照后的数据，并且 RDB 快照过程可能会占用比较多的 CPU 和 I/O 资源。

### AOF（Append Only File）日志持久化

Redis 将每个写操作（如SET、DEL）追加到 AOF 日志文件 `appendonly.aof` 中。在服务器重启时，Redis 可以通过重放 AOF 日志中的命令来重建数据集。

虽然比 RDB 更安全，但是 AOF 文件比 RDB 更大，恢复速度更慢。

### 混合持久化

Redis 4.0 引入了混合持久化，将 RDB 和 AOF 结合在一起，Redis 可以在 RDB 快照之后记录 AOF 日志，确保在恢复数据时能够快速加载快照，并重放日志中的最新操作，最大限度减少数据丢失。

## 常用操作

Redis 中的数据类型：

- 字符串
- 哈希
- 列表
- 集合
- 有序集合

`redis-cli` 进入 Redis

### 字符串操作

- 设置键值对：`SET key value`
- 获取键值：`GET key`
- 只有在 key 不存在时设置 key 的值：`SETNX key value`
- 设置带有过期时间的键：`SET key value EX 10`

### 哈希操作

Redis hash 是一个 string 类型的 field 和 value 的映射表，适合存储对象。简单理解就是 key 对应的 value 由一系列 filed-value 组成。

- 将哈希表 key 中的字段 field 的值设置为 value：`HSET key field value`
- 获取存储在哈希表中指定字段的值：`HGET key field`
- 删除存储在哈希表中的指定字段：`HDEL key field`
- 获取哈希表中所有字段：`HKEYS key`
- 获取哈希表中所有值：`HVALS key`

### 列表操作

- 将一个或多个值插入到列表头部：`LPUSH key value1 [value2]`
- 获取列表指定范围内的元素：`LRANGE key start stop`
- 移除并获取列表最后一个元素：`RPOP key`
- 获取列表长度：`LLEN key`

### 集合操作

集合无序且不重复

- 向集合中添加一个或多个成员：`SADD key member1 [member2]`
- 返回集合中的所有成员：`SMEMBERS key`
- 获取集合的成员数：`SCARD key`
- 返回给定所有集合的交集：`SINTER key1 [key2]`
- 返回所有给定集合的并集：`SUNION key1 [key2]`
- 删除集合中的一个或多个元素：`SREM key member1 [member2]`

### 有序集合

有序集合有顺序，但是不重复。**每个元素都会关联一个 double 类型的分数进行排序**。

- 向集合中添加一个或多个成员：`ZADD key score1 member1 [score2 member2]`
- 通过索引区间返回有序集合中指定区间内的成员：`ZRANGE key start stop [WITHSCORES]`
- 有序集合中对指定成员的分数加上增量 increment：`ZINCRBY key increment member`
- 移除有序集合中的一个或多个成员：`ZREM key member [member ...]`

### 通用命令

- 查找所有符合给定模式（patten）的 key：`KEYS pattern`
- 检查键是否存在：`EXISTS key`
- 检查键的剩余生存时间：`TTL key`
- 返回键所存储的值的类型：`TYPE key`
- 在键存在时删除键：`DEL key`

## Redis 与 Java

Redis 的 java 客户端：

- Jedis
- Lettuce：性能好
- Spring Data Redis：Spring 的一部分，对 Redis 底层开发包进行了高度封装。

