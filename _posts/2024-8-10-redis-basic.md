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

总的来说是单线程的，但是性能极高：

- 基于内存（重要原因）
- IO多路复用
- 良好的源代码

Redis 也支持**数据的持久化、主从集群、分片集群**，默认有16个库，不能修改名字，只能调整数量。

## 安装及启动

```shell
sudo pacman -S redis

systemctl start redis.service
```

## 使用方法

```bash
Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
                     You can also use the REDISCLI_AUTH environment
                     variable to pass this password more safely
```

## 配置

默认配置文件位于 `/etc/redis/redis.conf`。可以调整其中的配置项如：

- 默认端口：默认端口为 6379。
- 启用持久化：通过修改 `save` 和 `appendonly` 参数。
- 设置最大内存：使用 `maxmemory` 配置限制 Redis 使用的内存量。
- 设置访问密码：开启 `requirepass` 选项并设置密码

修改配置后需要重启 Redis：

```shell
systemctl restart redis.service
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
systemctl restart redis.service
```

再次使用 `redis-cli` 以默认用户进入 `redis` 后，默认没有任何权限。如果要以设置好的用户身份进入 redis，可以在 redis 中输入：

```
127.0.0.1:6379> AUTH username password
```

如果返回 `OK` 这表明登入成功。

当然，ACL 配置也可以在 Redis 中通过命令交互式配置，这里不多赘述，可以自行 Google。

## Redis 数据结构

key 一般为 `String` 类型，value 可以有：

- 基本类型
    - `String`
    - `Hash`
    - `List`
    - `Set`
    - `SortedSet`

- 特殊类型
    - `GEO`
    - `BitMap`
    - `HyprLog`

数据类型。

## java 客户端

- `jedis`：线程不安全，多线程环境下需要基于连接池使用。
- `lettuce`：线程安全，支持redis哨兵模式，集群模式和管道模式。
- `Redission`：分布式、可伸缩的java数据结构合集。

#### 使用方法

- 引入依赖
- 建立连接
- 使用redis操作
- 释放资源

```java
import redis.clients.jedis.Jedis;

@SpringBootTest
class DemoApplicationTests {

  private Jedis jedis;

  @BeforeEach
  void setUp() {
    // 建立连接
    jedis = new Jedis("127.0.0.1", 6379);
    // 设置密码
    jedis.auth("123456");
    // 选择库
    jedis.select(0);
    // 操作redis
  }

  @Test
  void testString() {
  String result = jedis.set("name", "google");
  System.out.println(result);

  // 获取数据
  String name = jedis.get("name");
  System.out.println("Name: " + name);
  }

  // 释放资源
  @AfterEach
  void tearDown() {
    if (jedis != null) {
      jedis.close();
    }
  }
}
```

Jedis 本身是**线程不安全**的，并且**频繁创建和销毁连接会有性能损耗**，因此推荐使用**Jedis连接池**代替Jedis直连方式。

使用 `JedisPoolConfig` 的配置方式：

```java
package com.shaneworld.jedisDemo.util;

import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * JedisConnectionsFactory
 */
public class JedisConnectionsFactory {

  private static final JedisPool jedisPool;

  static {
    // 配置连接池
    JedisPoolConfig poolConfig = new JedisPoolConfig();

    // 最大连接数
    poolConfig.setMaxTotal(8);

    // 即使没有连接，也会准备的个数
    poolConfig.setMaxIdle(8);

    poolConfig.setMinIdle(0);

    // 当连接池中没有连接可用的时候需不需要等待？等待多长时间？
    poolConfig.setMaxWaitMillis(1000);

    // 创建连接池对象
    jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379, 1000, "123456");

  }
}
```

`JedisPoolConfig` 早期是基于 `GenericObjectPoolConfig` 类的一个简单扩展，但后来随着 Jedis 的版本更新，它被弃用，转而推荐使用更灵活的 `GenericObjectPoolConfig`。

```java
package com.shaneworld.jedisDemo.util;

import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Jedis;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;

import java.time.Duration;

/**
 * JedisConnectionsFactory
 */
public class JedisConnectionsFactory {

  private static final JedisPool jedisPool;

  static {

    GenericObjectPoolConfig<Jedis> poolConfig = new GenericObjectPoolConfig<>();

    // 最大连接数
    poolConfig.setMaxTotal(8);

    // 即使没有连接，也会准备的个数
    poolConfig.setMaxIdle(8);

    poolConfig.setMinIdle(0);

    // 当连接池中没有连接可用的时候需不需要等待？等待多长时间？
    poolConfig.setMaxWait(Duration.ofSeconds(2));

    // 创建连接池对象
    jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379, 1000, "123456");

  }

  // 提供获取redis的方法
  public static Jedis getJedis() {
    return jedisPool.getResource();
  }
}
```

从这里我们不难看出，Jedis连接池其实采用的是工厂设计模式，实现了**解耦**。

这样，我们只需要在创建jedis的时候将原先创建单个jedis对象的语句改为使用 `getJedis()` 方法从池子中获取对象：

```Java
@BeforeEach
void setUp() {
    // 建立连接
    //jedis = new Jedis("127.0.0.1", 6379);
    jedis = JedisConnectionsFactory.getJedis();
    // 设置密码
    jedis.auth("123456");
    // 选择库
    jedis.select(0);
    // 操作redis
}
```

原先释放资源的代码也不用修改。如果深入 `close()` 方法就会发现当我们的jedis对象是从连接池中获取的时候，调用 `close()` 方法其实是将对象归还到池子中去。

#### SpringDataRedis

`SpringData` 是 Spring 中数据操作的模块，包含对各种数据库的集成，其中对 Redis 的集成模块就叫做 `SpringDataRedis`，整合了 `Lettuce` 和 `Jedis` 客户端。

`SpringDataRedis` 中提供了 `RedisTemplate` 工具类，其中封装了各种对 `Redis` 的操作。并且将不同数据类型的操作 API 封装到了不同的类型中。

使用步骤如下：

- 引入依赖
- 配置
- 注入
- 编写测试

引入依赖后，需要在 `application.yaml` 中配置信息：

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: "123456"
      lettuce:
        pool:
          max-active: 8 # 最大连接数
          max-idle: 8 # 最大空闲连接
          min-idle: 0 # 最小空闲连接
          max-wait: 1000ms # 连接等待时间
```

> 对于 Spring Boot 2.x，不需要 `data` 字段，Spring Boot 3.x 之后都需要添加 `data` 字段。
{: .prompt-info }

`RedisTemplate` 可以接收任意 Object 作为值写入 Redis，只不过写入前会把 Object 序列化为字节形式,默认是采用 JDK 序列化，这样存储可读性差，内存占用较大。因此需要修改 `RedisTemplate` 的序列化方式。

spring 默认使用 `lettuce` 的连接池，如果使用 `jedis` 的连接池，则需要额外引入依赖。此外，**必须要配置 pool 的相关信息才会生效。**

## Redis 序列化和反序列化

使用 `RedisTemplate` 存入键值对后，会发现存入的键值对都会被添加一些额外字符导致无法准确获取，这就是RedisTemplate序列化导致的，需要进行序列化配置。

```java
package com.shaneworld.springDataRedis.config;
/**
 * RedisConfig
 */
@Configuration
public class RedisConfig {

  @Bean
  @Primary
  public RedisTemplate<String, Object> template(RedisConnectionFactory connectionFactory) throws UnknownHostException {
    // 创建对象
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    // 设置连接工厂
    template.setConnectionFactory(connectionFactory);
    // 设置JSON序列化工具
    GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
    // 设置key的序列化
    template.setKeySerializer(RedisSerializer.string());
    template.setHashKeySerializer(RedisSerializer.string());
    // 设置value序列化
    template.setValueSerializer(jsonRedisSerializer);
    template.setHashValueSerializer(jsonRedisSerializer);
    // 返回
    return template;
  }
}
```

反序列化配置好后，我们发现对于Java对象如`User`类，在存入 Redis 的时候会同时存入如 `"@class":"com.shaneworld.springDataRedis.pojo.User"` 这样的字段，会占用大量的空间。这种情况下我们不能使用自动序列化反序列化配置，而要手动设置序列化。

```java
// 序列化工具
private static final ObjectMapper mapper = new ObjectMapper();

@Test
void testUserSaving() throws JsonProcessingException {
    // 创建对象
    User user = new User("孙沙", 21);
    // 手动序列化
    String json =  mapper.writeValueAsString(user);
    // 写入数据
    stringRedisTemplate.opsForValue().set("user:200", json);
    // 获取数据
    String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
    // 手动反序列化
    User user1 = mapper.readValue(jsonUser, User.class);
    System.out.println("User1 = " + user1);
}
```

手动进行序列化反序列化能够大大减少内存消耗。

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

