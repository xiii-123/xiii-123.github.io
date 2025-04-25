---
title: Redis入门
published: 2025-04-22
description: 'Redis入门'
image: ''
tags: [Redis, database]
category: 'redis'
draft: false 
---

## Redis简介

Redis是一个开源的内存数据结构存储，用作数据库、缓存和消息代理。它支持多种类型的数据结构，如字符串、哈希、列表、集合、有序集合等。Redis的数据存储在内存中，但也可以持久化到磁盘，以防止数据丢失。

Redis的性能非常高，读写速度非常快，因此常用于需要快速访问数据的应用程序中。

## Redis安装

在此处只介绍基于Docker的安装方式。

1. 拉取redis镜像
   ```bash
   # 下载镜像
    docker pull redis

    # 检查当前所有Docker下载的镜像
    docker images
    ```
2. 创建redis配置文件
   ```bash
   mkdir -p /usr/local/redis/conf
   cd /usr/local/redis/conf
   touch redis.conf
   ```
3. 修改配置文件
   ```bash
   # 修改 /opt/redis/conf/redis.conf
    protected-mode no
    bind 0.0.0.0
    ```
4. 启动redis容器
   ```bash
   # Docker 创建 Redis 容器命令
    docker run \
    --restart=always \
    --log-opt max-size=100m \
    --log-opt max-file=2 \
    -p 6379:6379 \
    --name redis \
    -v /opt/redis/conf/redis.conf:/etc/redis/redis.conf  \
    -v /opt/redis/data:/data \
    -d redis redis-server /etc/redis/redis.conf \
    --appendonly yes \
    --requirepass 123456 
    ```
   1. `docker run`：这是Docker的基本命令，用于启动一个新的容器实例。
   2. `--restart=always`：这个参数告诉Docker，无论容器因何种原因停止，都应始终重新启动它。这意味着如果你的服务器重启或者容器崩溃，Docker会自动重启这个容器。
   3. `--log-opt max-size=100m`：这个参数设置了容器日志文件的最大大小为100MB。当日志文件达到这个大小时，Docker会自动轮换日志文件。
   4. `--log-opt max-file=2`：这个参数设置了日志文件的最大数量为2。结合上一个参数，这意味着Docker会保留最多两个日志文件，每个文件最大100MB。
   5. `-p 6379:6379`：这个参数将主机的6379端口映射到容器的6379端口。Redis默认使用6379端口，所以这样设置后，你可以在主机上通过6379端口访问容器内的Redis服务。
   6.  `--name redis`：这个参数为容器指定了一个名称，这里命名为"redis"。这样你可以在后续的Docker命令中通过这个名称来引用这个容器。
   7.  `-v /opt/redis/conf/redis.conf:/etc/redis/redis.conf`：这个参数将主机上的`/opt/redis/conf/redis.conf`文件映射到容器内的`/etc/redis/redis.conf`。这样，容器内的Redis服务器会使用主机上的配置文件。
   8.  `-v /opt/redis/data:/data`：这个参数将主机上的`/opt/redis/data`目录映射到容器内的`/data`目录。Redis使用这个目录来存储数据，所以这样设置后，Redis的数据会保存在主机上的`/opt/redis/data`目录中。
   9.  `-d`：这个参数告诉Docker以守护进程（后台）模式运行容器。
   10. `redis`：这个参数指定了要使用的镜像名称，这里使用的是"redis"镜像。
   11. `redis-server /etc/redis/redis.conf`：这个参数是容器启动后要执行的命令，这里告诉Redis服务器使用`/etc/redis/redis.conf`作为配置文件。
   12. `--appendonly yes`：这个参数开启了Redis的AOF（Append Only File）持久化模式。在这种模式下，Redis会将每个写操作记录到日志文件中，以便在重启时恢复数据。
   13. `--requirepass 123456`：这个参数为Redis服务器设置了一个密码，这里设置的密码是"123456"。客户端连接到Redis服务器时需要提供这个密码。
   
## 常用指令

以下是 Redis 中常见数据类型及其常用命令的详细介绍：
---
### 1. 字符串（String）
字符串是 Redis 最简单的数据类型，用于存储键值对形式的文本或二进制数据。
**常用命令：**
- `SET key value`：设置键的值。
- `GET key`：获取键的值。
- `GETRANGE key start end`：获取字符串的子字符串。
- `INCR key`：将键的整数值自增1。
- `DECR key`：将键的整数值自减1。
- `INCRBY key increment`：将键的整数值增加指定的增量。
- `DECRBY key decrement`：将键的整数值减少指定的减量。
**示例：**
```bash
SET username "Alice"
GET username
INCRBY user_count 5
```
---
### 2. 列表（List）
列表是简单的字符串列表，按照插入顺序排序，支持从头部或尾部插入和删除元素。
**常用命令：**
- `LPUSH key value1 [value2]`：从列表头部插入一个或多个值。
- `RPUSH key value1 [value2]`：从列表尾部插入一个或多个值。
- `LPOP key`：移出并获取列表的第一个元素。
- `RPOP key`：移出并获取列表的最后一个元素。
- `LRANGE key start stop`：获取列表指定范围内的元素。
- `LLEN key`：获取列表的长度。
**示例：**
```bash
LPUSH mylist "item1" "item2"
RPOP mylist
LRANGE mylist 0 -1
```
---
### 3. 哈希（Hash）
哈希类型用于存储键值对集合，适合存储对象数据。
**常用命令：**
- `HSET key field value`：设置哈希表中的字段值。
- `HGET key field`：获取哈希表中的字段值。
- `HGETALL key`：获取哈希表中的所有字段和值。
- `HEXISTS key field`：检查哈希表中是否存在指定字段。
- `HINCRBY key field increment`：将哈希表中字段的整数值增加指定的增量。
**示例：**
```bash
HSET user:1000 username "Alice" age 30
HGET user:1000 username
HINCRBY user:1000 age 1
```
---
### 4. 集合（Set）
集合类型存储唯一、无序的字符串集合，适合实现去重和交集、并集等操作。
**常用命令：**
- `SADD key member [member1, member2]`：向集合中添加一个或多个成员。
- `SREM key member [member1, member2]`：移除集合中的一个或多个成员。
- `SMEMBERS key`：获取集合中的所有成员。
- `SISMEMBER key member`：检查成员是否存在于集合中。
- `SINTER key1 key2`：计算两个集合的交集。
- `SUNION key1 key2`：计算两个集合的并集。
**示例：**
```bash
SADD myset "item1" "item2" "item3"
SREM myset "item2"
SINTER myset myset2
```
---
### 5. 有序集合（Sorted Set）
有序集合结合了集合和排序的特性，每个成员都有一个分数，用于排序。
**常用命令：**
- `ZADD key score member [score1 member1]`：向有序集合中添加一个或多个成员及其分数。
- `ZREM key member [member1, member2]`：移除有序集合中的一个或多个成员。
- `ZRANGE key start stop [WITHSCORES]`：获取有序集合中指定范围内的成员及其分数。
- `ZCARD key`：获取有序集合中的成员数量。
- `ZSCORE key member`：获取成员的分数。
**示例：**
```bash
ZADD myscores 100 "Alice" 200 "Bob"
ZRANGE myscores 0 -1 WITHSCORES
ZREM myscores "Alice"
```
---

### 通用命令
通用命令部分数据类型。常用命令：
- `KEY pattern`：查找符合模式的键。
- `DEL key`：删除指定的键。
- `EXISTS key`：检查指定的键是否存在。
- `TYPE key`：获取键的类型。

```bash
KEYS *
DEL key1 key2
EXISTS key1
TYPE key1
```
---

## Spring Boot 集成 Redis

### 一、添加依赖
在Spring Boot项目中，首先需要在`pom.xml`或`build.gradle`文件中添加`spring-boot-starter-data-redis`依赖。这将包含所有必要的库。
#### Maven依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
#### Gradle依赖：
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```
---
### 二、配置Redis连接信息
在`application.yml`或`application.properties`文件中配置Redis服务器的连接信息，包括地址、端口和密码（如有）。
#### 示例（`application.yml`）：
```yaml
spring:
  redis:
    host: localhost  # Redis服务器地址
    port: 6379       # Redis服务器端口
    password: ''     # 密码（如果没有则为空）
    lettuce:
      pool:
        max-active: 8    # 连接池最大连接数
        max-idle: 8     # 连接池最大空闲连接数
        min-idle: 0     # 连接池最小空闲连接数
        max-wait: -1ms  # 连接池最大等待时间
```
---
### 三、创建RedisTemplate Bean
Spring Data Redis提供了`RedisTemplate`类，用于执行Redis操作。您可以通过自定义配置创建一个`RedisTemplate` Bean。
#### 示例代码：
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 设置key的序列化方式
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        // 设置value的序列化方式
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }
}
```
---
### 四、使用RedisTemplate操作Redis
通过`RedisTemplate`，您可以执行常见的Redis操作，如存取数据、设置过期时间等。
#### 示例代码：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
@Service
public class RedisService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    // 设置值
    public void setValue(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }
    // 获取值
    public Object getValue(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    // 设置过期时间
    public void setExpire(String key, long timeout, TimeUnit unit) {
        redisTemplate.expire(key, timeout, unit);
    }
}
```
---
### 五、使用Spring Cache简化缓存操作
Spring Cache是一个声明式缓存抽象框架，可以与Spring Data Redis结合使用，简化缓存操作。
#### 示例步骤：
1. 在配置类中启用缓存：
   ```java
   @EnableCaching
   @Configuration
   public class CacheConfig {
   }
   ```
2. 在需要缓存的方法上添加注解：
   ```java
   @Service
   public class UserService {
       @Cacheable(value = "users", key = "#id")
       public User getUserById(String id) {
           // 查询数据库或其他操作
           return user;
       }
   }
   ```
