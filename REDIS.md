# Redis 核心技术文档

> 由浅入深，环环相扣 | 每个知识点配有精简代码示例与注释

---

## 目录

- [1. Redis 概述](#1-redis-概述)
  - [1.1 为什么要使用 NoSQL](#11-为什么要使用-nosql)
  - [1.2 什么是 NoSQL](#12-什么是-nosql)
  - [1.3 当下 NoSQL 经典应用](#13-当下-nosql-经典应用)
  - [1.4 什么是 Redis](#14-什么是-redis)
- [2. Redis 安装与基础](#2-redis-安装与基础)
  - [2.1 Linux 下安装 Redis](#21-linux-下安装-redis)
  - [2.2 Docker 安装 Redis](#22-docker-安装-redis)
  - [2.3 基础知识](#23-基础知识)
- [3. Redis 数据类型](#3-redis-数据类型)
  - [3.1 Key 键操作](#31-key-键操作)
  - [3.2 String 字符串](#32-string-字符串)
  - [3.3 List 列表](#33-list-列表)
  - [3.4 Set 集合](#34-set-集合)
  - [3.5 Hash 哈希](#35-hash-哈希)
  - [3.6 ZSet 有序集合](#36-zset-有序集合)
  - [3.7 Bitmaps 位图](#37-bitmaps-位图)
  - [3.8 Geospatial 地理空间](#38-geospatial-地理空间)
  - [3.9 HyperLogLog 基数统计](#39-hyperloglog-基数统计)
- [4. Redis 配置文件与可视化](#4-redis-配置文件与可视化)
  - [4.1 Redis 配置文件](#41-redis-配置文件)
  - [4.2 可视化工具](#42-可视化工具)
- [5. Redis 核心功能](#5-redis-核心功能)
  - [5.1 发布订阅](#51-发布订阅)
  - [5.2 慢查询日志](#52-慢查询日志)
  - [5.3 流水线 Pipeline](#53-流水线-pipeline)
- [6. Redis 数据持久化](#6-redis-数据持久化)
  - [6.1 持久化机制概述](#61-持久化机制概述)
  - [6.2 RDB 持久化机制](#62-rdb-持久化机制)
  - [6.3 AOF 持久化机制](#63-aof-持久化机制)
  - [6.4 持久化策略选择](#64-持久化策略选择)
- [7. Redis 事务](#7-redis-事务)
  - [7.1 事务概念与 ACID 特性](#71-事务概念与-acid-特性)
  - [7.2 事务基本操作](#72-事务基本操作)
- [8. Redis 集群](#8-redis-集群)
  - [8.1 主从复制](#81-主从复制)
  - [8.2 哨兵监控](#82-哨兵监控)
  - [8.3 Cluster 集群模式](#83-cluster-集群模式)
  - [8.4 Java 操作 Redis 集群](#84-java-操作-redis-集群)
- [9. Java 整合 Redis](#9-java-整合-redis)
  - [9.1 Jedis 操作](#91-jedis-操作)
  - [9.2 Spring Data Redis](#92-spring-data-redis)
- [10. Redis Web 实践](#10-redis-web-实践)
  - [10.1 网页缓存实战](#101-网页缓存实战)
- [11. 企业级解决方案](#11-企业级解决方案)
  - [11.1 Redis 脑裂](#111-redis-脑裂)
  - [11.2 缓存预热](#112-缓存预热)
  - [11.3 缓存穿透](#113-缓存穿透)
  - [11.4 缓存击穿](#114-缓存击穿)
  - [11.5 缓存雪崩](#115-缓存雪崩)
  - [11.6 Redis 开发规范](#116-redis-开发规范)
  - [11.7 数据一致性](#117-数据一致性)

---

## 1. Redis 概述

### 1.1 为什么要使用 NoSQL

传统关系型数据库在面对海量数据、高并发场景时遇到瓶颈：

- **性能瓶颈**：单表数据量过千万后查询变慢
- **扩展性差**：垂直扩展成本高，水平扩展复杂
- **结构僵化**：表结构修改困难，不灵活

**NoSQL 优势**：轻量、高效、易扩展，适合互联网高并发场景。

### 1.2 什么是 NoSQL

NoSQL（Not Only SQL）泛指非关系型数据库，按数据模型分为四大类：

| 类型 | 代表产品 | 适用场景 |
|------|----------|----------|
| 键值存储 | Redis, Memcached | 高速缓存、会话存储 |
| 文档存储 | MongoDB, CouchDB | 内容管理、文档存储 |
| 列族存储 | HBase, Cassandra | 大数据分析、日志处理 |
| 图数据库 | Neo4j, OrientDB | 社交关系、推荐引擎 |

### 1.3 当下 NoSQL 经典应用

```java
// 电商场景：Redis 缓存商品信息
public Product getProduct(Long id) {
    String key = "product:" + id;
    // 1. 先查缓存
    String json = redis.get(key);
    if (json != null) {
        return JSON.parseObject(json, Product.class);
    }
    // 2. 缓存未命中，查数据库
    Product product = productMapper.selectById(id);
    if (product != null) {
        // 3. 写入缓存
        redis.setex(key, 3600, JSON.toJSONString(product));
    }
    return product;
}
```

### 1.4 什么是 Redis

Redis（Remote Dictionary Server）是一款开源的、基于内存的键值存储系统。

**核心特性**：
- 基于内存，读写速度极高（10万+ QPS）
- 支持丰富的数据类型
- 支持持久化、主从、集群
- 单线程模型，无竞争问题

```bash
# 启动 Redis 客户端测试
redis-cli ping    # 返回 PONG 表示连接成功
redis-cli info   # 查看服务器信息
```

---

## 2. Redis 安装与基础

### 2.1 Linux 下安装 Redis

```bash
# 下载并编译
wget https://download.redis.io/releases/redis-7.0.12.tar.gz
tar xzf redis-7.0.12.tar.gz
cd redis-7.0.12
make

# 安装到指定目录
make install PREFIX=/usr/local/redis

# 启动服务
/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf

# 客户端连接
/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 -a password
```

### 2.2 Docker 安装 Redis

```bash
# 拉取镜像
docker pull redis:7-alpine

# 启动容器（带密码、持久化）
docker run -d \
  --name redis \
  -p 6379:6379 \
  -e REDIS_PASSWORD=your_password \
  -v /data/redis:/data \
  redis:7-alpine \
  redis-server --requirepass your_password --appendonly yes

# 进入容器测试
docker exec -it redis redis-cli -a your_password
```

### 2.3 基础知识

**Redis 常用全局命令**：

```bash
# Key 操作
SET key value           # 设置键值对
GET key                 # 获取值
DEL key                 # 删除键
EXISTS key              # 判断是否存在
EXPIRE key seconds      # 设置过期时间
TTL key                 # 查看剩余生存时间
KEYS pattern            # 查找所有符合模式的 key（慎用）
SCAN cursor MATCH pattern COUNT count  # 渐进式扫描（推荐）

# 数据库操作
SELECT 0                # 切换数据库（0-15）
DBSIZE                  # 查看当前库 key 数量
FLUSHDB                 # 清空当前库
FLUSHALL                # 清空所有库

# 服务器信息
INFO server             # 服务器信息
INFO memory             # 内存使用
INFO replication        # 复制信息
INFO persistence        # 持久化信息
```

**Redis 单线程模型**：

Redis 使用单线程处理命令，避免了上下文切换和锁竞争，核心原理：
1. 纯内存操作，速度极快
2. IO 多路复用，处理大量并发连接
3. 避免锁竞争，无死锁问题

---

## 3. Redis 数据类型

### 3.1 Key 键操作

```bash
# 基础操作
SET user:1 "张三"          # 设置 key
SETNX user:1 "张三"        # 仅当 key 不存在时设置
SETEX session:abc 3600 "data"  # 设置带过期时间的 key

# 批量操作
MSET name "李四" age 25 city "北京"   # 批量设置
MGET name age city                     # 批量获取

# 过期管理
EXPIRE token:xyz 1800     # 30 分钟后过期
PEXPIRE token:xyz 1800000 # 毫秒级过期
TTL token:xyz             # 查看剩余秒数
PERSIST token:xyz         # 取消过期时间

# 键管理
RENAME old_key new_key    # 重命名
TYPE user:1               # 查看数据类型
RANDOMKEY                 # 随机返回一个 key
```

**Java 操作示例**：

```java
// 设置带过期时间的 key
jedis.setex("session:user1", 1800, "user_data");

// 原子性检查并设置（分布式锁基础）
String result = jedis.set("lock:order", "1", "NX", "EX", 10);
if ("OK".equals(result)) {
    // 获取锁成功
}
```

### 3.2 String 字符串

String 是 Redis 最基本的类型，可以存字符串、整数、二进制（最大 512MB）。

```bash
# 基本操作
SET str:name "Hello Redis"    # 设置值
GET str:name                   # 获取值
APPEND str:name " World"      # 追加内容
STRLEN str:name                # 获取长度
GETRANGE str:name 0 4          # 获取子串 [0, 4]
SETRANGE str:name 6 "Redis"    # 从指定位置覆盖

# 数值操作
SET counter 100               # 设置整数
INCR counter                  # 自增 1 -> 101
DECR counter                  # 自减 1 -> 100
INCRBY counter 10             # 增加指定值 -> 110
DECRBY counter 5              # 减少指定值 -> 95
```

**Java 应用示例 - 计数器**：

```java
// 文章阅读量统计
public Long incrementViewCount(Long articleId) {
    String key = "article:view:" + articleId;
    return jedis.incr(key);  // 原子性自增
}

// 限流计数器
public boolean isAllowed(String userId) {
    String key = "rate:" + userId;
    Long count = jedis.incr(key);
    if (count == 1) {
        jedis.expire(key, 60);  // 60秒窗口
    }
    return count <= 100;  // 每分钟最多100次
}
```

### 3.3 List 列表

List 是有序的字符串数组，支持从两端推入和弹出，最多 2^32 - 1 个元素。

```bash
# 插入操作（从两端）
LPUSH queue "task1" "task2"   # 从左侧推入
RPUSH queue "task3"            # 从右侧推入
RPOP queue                     # 从右侧弹出
LPOP queue                     # 从左侧弹出

# 查看操作
LRANGE queue 0 -1              # 查看全部元素
LINDEX queue 0                 # 获取指定索引元素
LLEN queue                     # 获取列表长度

# 高级操作
BRPOP queue timeout            # 阻塞式弹出（超时等待）
LINSERT queue BEFORE "task1" "new_task"  # 在指定元素前插入
LSET queue 0 "updated"         # 更新指定位置的元素
LTRIM queue 0 2                # 裁剪列表，只保留指定范围
```

**Java 应用示例 - 消息队列**：

```java
// 生产者：发送消息到队列
public void sendMessage(String channel, String message) {
    jedis.rpush("message:queue",
        JSON.toJSONString(new Message(channel, message, System.currentTimeMillis())));
}

// 消费者：阻塞式接收消息
public Message receiveMessage() {
    List<byte[]> result = jedis.brpop(0, "message:queue");  // 0 表示无限等待
    if (result != null && result.size() == 2) {
        return JSON.parseObject(result.get(1), Message.class);
    }
    return null;
}
```

### 3.4 Set 集合

Set 是无序不重复的字符串集合，支持交集、并集、差集运算。

```bash
# 基本操作
SADD tags "Java" "Redis" "MySQL"   # 添加元素
SMEMBERS tags                       # 查看所有元素
SCARD tags                          # 获取元素数量
SISMEMBER tags "Java"               # 判断元素是否存在

# 删除操作
SREM tags "MySQL"                   # 删除指定元素
SPOP tags                           # 随机删除并返回一个元素
SMISMEMBER tags "Java" "Redis"     # 批量判断存在性

# 集合运算
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"
SINTER set1 set2                    # 交集 -> b, c
SUNION set1 set2                    # 并集 -> a, b, c, d
SDIFF set1 set2                     # 差集 -> a
SDIFFSTORE dest set1 set2           # 存储差集到新集合
```

**Java 应用示例 - 标签管理**：

```java
// 用户标签
public void addUserTags(Long userId, Set<String> tags) {
    String key = "user:tags:" + userId;
    jedis.sadd(key, tags.toArray(new String[0]));
}

// 查找共同好友
public Set<String> getCommonFriends(Long userId1, Long userId2) {
    String key1 = "user:friends:" + userId1;
    String key2 = "user:friends:" + userId2;
    return jedis.sinter(key1, key2);
}
```

### 3.5 Hash 哈希

Hash 是一个键值对集合，适合存储对象，比 String + JSON 更节省内存。

```bash
# 基本操作
HSET user:1 name "张三" age 25 city "北京"   # 设置字段
HGET user:1 name                              # 获取单个字段
HMGET user:1 name age city                    # 获取多个字段
HGETALL user:1                                # 获取所有字段

# 删除与判断
HDEL user:1 city                              # 删除字段
HEXISTS user:1 name                           # 判断字段是否存在

# 遍历操作
HKEYS user:1                                  # 获取所有字段名
HVALS user:1                                  # 获取所有值
HLEN user:1                                   # 获取字段数量
HSCAN user:1 0 MATCH * COUNT 100              # 渐进式遍历

# 数值操作
HINCRBY counter:user1 login_count 1           # 字段值自增
```

**Java 应用示例 - 用户信息缓存**：

```java
// 保存用户信息到 Redis Hash
public void cacheUser(User user) {
    Map<String, String> map = new HashMap<>();
    map.put("name", user.getName());
    map.put("age", String.valueOf(user.getAge()));
    map.put("email", user.getEmail());
    jedis.hset("user:" + user.getId(), map);
    jedis.expire("user:" + user.getId(), 3600);
}

// 获取用户信息
public User getUser(Long userId) {
    Map<String, String> data = jedis.hgetAll("user:" + userId);
    if (data.isEmpty()) {
        return null;
    }
    User user = new User();
    user.setId(userId);
    user.setName(data.get("name"));
    user.setAge(Integer.parseInt(data.get("age")));
    user.setEmail(data.get("email"));
    return user;
}
```

### 3.6 ZSet 有序集合

ZSet 是有序的 Set，每个元素关联一个 score，按 score 排序。

```bash
# 基本操作
ZADD rank 100 "张三" 200 "李四" 150 "王五"   # 添加元素（score 越小越靠前）
ZRANGE rank 0 -1                             # 升序查看全部
ZREVRANGE rank 0 -1                           # 降序查看全部
ZSCORE rank "张三"                            # 获取分数
ZRANK rank "张三"                             # 获取排名（升序）
ZREVRANK rank "张三"                          # 获取排名（降序）

# 范围查询
ZRANGEBYSCORE rank 100 200                   # 按分数范围查询
ZRANGEBYSCORE rank 100 200 LIMIT 0 10        # 分页查询

# 删除操作
ZREM rank "王五"                              # 删除元素
ZREMRANGEBYRANK rank 0 2                     # 按排名范围删除
ZREMRANGEBYSCORE rank 100 150                # 按分数范围删除

# 数量统计
ZCARD rank                                    # 元素总数
ZCOUNT rank 100 200                           # 分数范围内的数量
```

**Java 应用示例 - 排行榜**：

```java
// 游戏排行榜
public void updateRank(Long userId, int score) {
    jedis.zadd("game:rank", score, String.valueOf(userId));
}

// 获取 Top 10
public List<RankItem> getTop10() {
    Set<String> top10 = jedis.zrevrange("game:rank", 0, 9);
    List<RankItem> result = new ArrayList<>();
    for (String userId : top10) {
        Double score = jedis.zscore("game:rank", userId);
        long rank = jedis.zrevrank("game:rank", userId) + 1;
        result.add(new RankItem(Long.parseLong(userId), score, rank));
    }
    return result;
}

// 查询用户排名
public long getUserRank(Long userId) {
    return jedis.zrevrank("game:rank", String.valueOf(userId)) + 1;
}
```

### 3.7 Bitmaps 位图

Bitmaps 用位操作存储数据，极省空间，适合记录布尔状态。

```bash
# 基本操作
SETBIT user:sign:20240101 1 1     # 第1位设置为1（签到）
SETBIT user:sign:20240101 7 1     # 第7位设置为1
GETBIT user:sign:20240101 1       # 获取第1位的值

# 统计操作
BITCOUNT user:sign:20240101       # 统计1的个数（签到天数）
BITCOUNT user:sign:20240101 0 30  # 统计指定范围

# 位运算（多个 bitmap 做运算）
BITOP OR result sign:1 sign:2     # 合并签到记录
BITFIELD user:sign:20240101 GET u16 0  # 获取位字段
```

**Java 应用示例 - 用户签到**：

```java
// 用户签到
public void checkIn(Long userId, String dateStr) {
    String key = "user:sign:" + dateStr;
    jedis.setbit(key, userId, true);  // userId 作为位索引
}

// 统计当月签到天数
public int countSignDays(String dateStr) {
    return jedis.bitcount("user:sign:" + dateStr);
}

// 判断某用户是否签到
public boolean hasSigned(Long userId, String dateStr) {
    return jedis.getbit("user:sign:" + dateStr, userId);
}
```

### 3.8 Geospatial 地理空间

Redis 3.2+ 支持地理坐标存储和距离计算。

```bash
# 添加地理位置（经度、纬度、成员名）
GEOADD restaurants 116.397428 39.90923 "天安门"
GEOADD restaurants 116.407526 39.904030 "北京站"
GEOADD restaurants 116.391280 39.907590 "北京饭店"

# 查询
GEOPOS restaurants "天安门"                # 获取坐标
GEODIST restaurants "天安门" "北京站"       # 计算距离（默认米）
GEODIST restaurants "天安门" "北京站" km    # 以公里为单位

# 范围查询
GEOSEARCH restaurants FROMLONLAT 116.397428 39.90923 BYRADIUS 1000 m  # 1公里内

# 获取坐标的 hash 值（用于排序）
GEOHASH restaurants "天安门"
```

**Java 应用示例 - 附近的人**：

```java
// 添加用户位置
public void addUserLocation(Long userId, double lng, double lat) {
    jedis.geoadd("user:location", lng, lat, String.valueOf(userId));
}

// 查询附近的用户
public List<Long> findNearbyUsers(double lng, double lat, double radiusInKm) {
    GeoRadiusParam param = GeoRadiusParam.geoRadiusParam()
        .withCoord()
        .withDist()
        .sortAscending();
    List<GeoRadiusResponse> results = jedis.georadius("user:location",
        lng, lat, radiusInKm, GeoUnit.KM, param);
    return results.stream()
        .map(r -> Long.parseLong(r.getMemberByString()))
        .collect(Collectors.toList());
}
```

### 3.9 HyperLogLog 基数统计

HyperLogLog 是概率性数据结构，用于统计基数（唯一元素数量），误差约 0.81%。

```bash
# 添加元素
PFADD visitor:20240101 "user1" "user2" "user3"    # 添加访问记录
PFADD visitor:20240101 "user1" "user2"            # 重复添加不影响结果

# 统计基数（近似值）
PFCOUNT visitor:20240101                           # 统计独立访客数

# 合并统计
PFADD visitor:20240102 "user1" "user4"
PFMERGE visitor:week visitor:20240101 visitor:20240102  # 合并多天数据
PFCOUNT visitor:week                               # 统计合并后的基数
```

**Java 应用示例 - UV 统计**：

```java
// 记录访客
public void recordVisitor(String date, String userId) {
    String key = "uv:" + date;
    jedis.pfadd(key, userId);
}

// 获取日 UV
public long getDailyUV(String date) {
    return jedis.pfcount("uv:" + date);
}

// 获取周 UV（合并 7 天数据）
public long getWeekUV(String startDate, String endDate) {
    List<String> keys = new ArrayList<>();
    // 生成日期范围内的 key
    for (String date : dateUtil.getDateRange(startDate, endDate)) {
        keys.add("uv:" + date);
    }
    jedis.pfmerge("uv:week", keys.toArray(new String[0]));
    return jedis.pfcount("uv:week");
}
```

**数据类型对比总结**：

| 类型 | 存储结构 | 常用场景 | 内存效率 |
|------|----------|----------|----------|
| String | 字节数组 | 计数器、缓存、分布式锁 | 高 |
| List | 双向链表 | 消息队列、最新动态 | 中 |
| Set | 哈希表/整数数组 | 标签、去重、共同好友 | 高 |
| Hash | 哈希表 | 对象存储、用户信息 | 高 |
| ZSet | 跳表 | 排行榜、延迟队列 | 中 |
| Bitmaps | 位数组 | 签到、统计 | 极高 |
| HyperLogLog | 概率结构 | UV统计、去重计数 | 极高 |

---

## 4. Redis 配置文件与可视化

### 4.1 Redis 配置文件

Redis 配置文件（redis.conf）是核心配置，掌握关键参数至关重要。

```bash
# ===== 基础配置 =====
bind 0.0.0.0              # 绑定地址，允许所有 IP 访问
port 6379                  # 端口号
protected-mode yes         # 保护模式（安全配置）
daemonize yes              # 后台运行
pidfile /var/run/redis_6379.pid  # PID 文件路径
loglevel notice            # 日志级别：debug/verbose/notice/warning
logfile ""                 # 日志文件路径，空则输出到 stdout
databases 16               # 数据库数量（0-15）

# ===== 网络配置 =====
timeout 300                # 客户端空闲超时时间（秒）
tcp-keepalive 300          # TCP 保活时间
tcp-backlog 511            # TCP 积压队列长度

# ===== 内存管理 =====
maxmemory 256mb            # 最大内存限制
maxmemory-policy allkeys-lru  # 内存淘汰策略（见下）
maxmemory-samples 5        # LRU 采样数量

# ===== 持久化配置 =====
save 900 1                 # RDB：900秒内1次修改触发保存
save 300 10                # RDB：300秒内10次修改触发保存
save 60 10000              # RDB：60秒内10000次修改触发保存
stop-writes-on-bgsave-error yes  # 后台保存出错时停止写入
rdbcompression yes          # RDB 文件压缩
dbfilename dump.rdb        # RDB 文件名
dir /var/lib/redis         # 工作目录

# ===== AOF 配置 =====
appendonly yes             # 开启 AOF
appendfilename "appendonly.aof"  # AOF 文件名
appendfsync everysec       # 同步策略：always/everysec/no
no-appendfsync-on-rewrite no  # 重写时不同步
auto-aof-rewrite-percentage 100  # AOF 重写触发百分比
auto-aof-rewrite-min-size 64mb  # AOF 重写最小尺寸

# ===== 主从配置 =====
# replicaof <masterip> <masterport>  # 从节点配置主节点
# masterauth <master-password>        # 主节点密码
replica-read-only yes       # 从节点只读
replica-serve-stale-data yes  # 从节点服务过期数据

# ===== 安全配置 =====
requirepass your_password   # 设置密码
# rename-command FLUSHDB ""  # 禁用危险命令
# rename-command FLUSHALL ""
# rename-command CONFIG ""

# ===== 慢查询日志 =====
slowlog-log-slower-than 10000  # 超过10ms记录（微秒）
slowlog-max-len 128            # 最多保存128条

# ===== 客户端配置 =====
maxclients 10000           # 最大客户端连接数
```

**内存淘汰策略详解**：

| 策略 | 说明 |
|------|------|
| noeviction | 不淘汰，内存满时写入报错 |
| allkeys-lru | 所有 key 使用 LRU 淘汰 |
| volatile-lru | 有过期时间的 key 使用 LRU 淘汰 |
| allkeys-lfu | 所有 key 使用 LFU 淘汰（4.0+） |
| volatile-lfu | 有过期时间的 key 使用 LFU 淘汰 |
| allkeys-random | 所有 key 随机淘汰 |
| volatile-random | 有过期时间的 key 随机淘汰 |
| volatile-ttl | 淘汰剩余存活时间最短的 key |

**在线修改配置**：

```bash
# 在线修改配置（无需重启）
CONFIG SET maxmemory 512mb
CONFIG SET maxmemory-policy allkeys-lru

# 查看当前配置
CONFIG GET maxmemory
CONFIG GET maxmemory-policy

# 重置统计
CONFIG RESETSTAT
```

### 4.2 可视化工具

**Redis Desktop Manager（推荐）**：

跨平台 GUI 工具，支持 Windows/Mac/Linux，功能完善。

```
连接配置：
- Host: Redis 服务器地址
- Port: 6379
- Password: 密码
- Database: 选择数据库

主要功能：
- Key 浏览器
- 数据编辑器
- 命令行终端
- 性能监控
- 导入/导出
```

**其他工具**：

| 工具 | 特点 |
|------|------|
| Redis Desktop Manager | 功能最全，跨平台 |
| RedisInsight | Redis 官方出品，可视化好 |
| Medis | 轻量级，Mac 专用 |
| Redis Commander | Web 版，Node.js 开发 |
| phpredisadmin | PHP 开发，适合 LAMP 环境 |

**使用 RedisInsight 监控示例**：

```bash
# Docker 启动 RedisInsight
docker run -d \
  --name redisinsight \
  -p 5540:5540 \
  redis/redisinsight:latest

# 访问
# http://localhost:5540
```

---

## 5. Redis 核心功能

### 5.1 发布订阅

Redis 的发布订阅（Pub/Sub）是消息广播模式，发送者不关心接收者。

```bash
# 订阅频道
SUBSCRIBE news             # 订阅 news 频道
PSUBSCRIBE news.*          # 模糊匹配订阅

# 发布消息
PUBLISH news "Hello"       # 发布到 news 频道

# 取消订阅
UNSUBSCRIBE news
PUNSUBSCRIBE news.*

# 查看订阅信息
PUBSUB CHANNELS            # 查看所有频道
PUBSUB NUMSUB news         # 查看频道订阅数
```

**Java 实现示例**：

```java
// 1. 消息监听器
public class MessageListener extends JedisPubSub {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("频道: " + channel + ", 消息: " + message);
        // 处理业务逻辑
        handleMessage(channel, message);
    }
    
    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
        System.out.println("订阅成功: " + channel);
    }
}

// 2. 订阅者
public class Subscriber {
    public void subscribe(String channel) {
        Jedis jedis = jedisPool.getResource();
        MessageListener listener = new MessageListener();
        // 阻塞式订阅
        jedis.subscribe(listener, channel);
    }
}

// 3. 发布者
public class Publisher {
    public void publish(String channel, String message) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.publish(channel, message);
        }
    }
}

// 4. 使用示例
// 订阅（新线程中执行）
new Thread(() -> new Subscriber().subscribe("news")).start();

// 发布消息
new Publisher().publish("news", "系统通知：服务即将升级");
```

**Pub/Sub vs 消息队列**：

| 特性 | Pub/Sub | 消息队列（List） |
|------|---------|-----------------|
| 消息持久化 | 不支持 | 支持 |
| 离线消费 | 不支持 | 支持 |
| 多消费者 | 广播模式 | 竞争模式 |
| 可靠性 | 低 | 高 |

### 5.2 慢查询日志

慢查询日志记录执行时间超过阈值的命令，帮助定位性能问题。

```bash
# 设置慢查询阈值（微秒）
CONFIG SET slowlog-log-slower-than 10000  # 10ms

# 设置最多保存条数
CONFIG SET slowlog-max-len 128

# 查看慢查询日志
SLOWLOG GET                  # 查看所有慢查询
SLOWLOG GET 10               # 查看最近10条
SLOWLOG GET 10 20            # 分页查看

# 获取日志条数
SLOWLOG LEN

# 清空日志
SLOWLOG RESET

# 日志字段说明
# 1) 日志 ID
# 2) 执行时间戳
# 3) 执行耗时（微秒）
# 4) 命令及参数
# 5) 客户端信息（4.0+）
```

**Java 封装慢查询分析**：

```java
// 获取慢查询统计
public List<SlowLogEntry> getSlowLogs(int count) {
    List<SlowLogEntry> entries = new ArrayList<>();
    List<List<Object>> slowLogs = jedis.slowlogGet(count);
    
    for (List<Object> log : slowLogs) {
        SlowLogEntry entry = new SlowLogEntry();
        entry.setId((Long) log.get(0));
        entry.setTimestamp((Long) log.get(1));
        entry.setDuration((Long) log.get(2));  // 微秒
        entry.setCommand((String) log.get(3));
        entries.add(entry);
    }
    return entries;
}

// 慢查询告警
public void checkAndAlert() {
    List<SlowLogEntry> slowLogs = getSlowLogs(10);
    for (SlowLogEntry entry : slowLogs) {
        if (entry.getDuration() > 50000) {  // 超过50ms
            // 发送告警
            alertService.alert("Redis慢查询: " + entry.getCommand() 
                + ", 耗时: " + entry.getDuration() + "μs");
        }
    }
}
```

### 5.3 流水线 Pipeline

Pipeline 允许批量执行命令，减少网络往返，提升吞吐量。

```bash
# 原理：客户端批量发送命令，服务器批量返回结果
# 普通模式：N 个命令 = N 次网络往返
# Pipeline：N 个命令 = 1 次网络往返

# 示例：批量设置 1000 个 key
# 普通模式：1000 次往返
for i in $(seq 1 1000); do
    SET key:$i value:$i
done

# Pipeline 模式：1 次往返
redis-cli <<EOF
$(for i in $(seq 1 1000); do echo "SET key:$i value:$i"; done)
EOF
```

**Java Pipeline 使用示例**：

```java
// 批量操作示例
public void batchUpdate(List<User> users) {
    try (Jedis jedis = jedisPool.getResource()) {
        Pipeline pipeline = jedis.pipelined();
        
        // 批量添加命令
        for (User user : users) {
            pipeline.setex(
                "user:" + user.getId(), 
                3600, 
                JSON.toJSONString(user)
            );
        }
        
        // 一次性执行
        List<Object> results = pipeline.sync();
        
        // 处理结果
        for (Object result : results) {
            if (!"OK".equals(result)) {
                log.error("批量更新失败");
            }
        }
    }
}

// Pipeline + 事务
public void transferWithPipeline(String from, String to, double amount) {
    try (Jedis jedis = jedisPool.getResource()) {
        // 开启事务
        jedis.watch(from, to);
        Transaction transaction = jedis.multi();
        
        // 批量命令
        transaction.decrByFloat(from, amount);
        transaction.incrByFloat(to, amount);
        
        // 执行事务
        List<Object> results = transaction.exec();
        if (results == null) {
            // 事务失败，重试
            retry(from, to, amount);
        }
    }
}

// 性能对比测试
public void performanceTest() {
    int count = 10000;
    long start, end;
    
    // 普通模式
    start = System.currentTimeMillis();
    for (int i = 0; i < count; i++) {
        jedis.set("perf:" + i, "value");
    }
    end = System.currentTimeMillis();
    System.out.println("普通模式: " + (end - start) + "ms");
    
    // Pipeline 模式
    start = System.currentTimeMillis();
    Pipeline pipeline = jedis.pipelined();
    for (int i = 0; i < count; i++) {
        pipeline.set("perf:" + i, "value");
    }
    pipeline.sync();
    end = System.currentTimeMillis();
    System.out.println("Pipeline模式: " + (end - start) + "ms");
    // 性能提升约 10-50 倍
}
```

**Pipeline 使用注意事项**：

1. Pipeline 命令没有原子性保证（事务需使用 MULTI/EXEC）
2. 避免在 Pipeline 中执行过多命令（建议 < 10000）
3. 长 Pipeline 可能阻塞服务器
4. Pipeline 适合批量读/写操作场景

---

## 6. Redis 数据持久化

### 6.1 持久化机制概述

Redis 是内存数据库，为防止数据丢失，需要将内存数据保存到磁盘。Redis 提供两种持久化机制：

- **RDB（Redis DataBase）**：快照方式，定时保存
- **AOF（Append Only File）**：日志方式，记录每个写命令

| 特性 | RDB | AOF |
|------|-----|-----|
| 数据安全性 | 可能丢失数分钟数据 | 最多丢失1秒数据 |
| 文件体积 | 小（压缩二进制） | 大（命令日志） |
| 恢复速度 | 快 | 慢 |
| 性能影响 | 几乎无 | 有轻微影响 |
| 适用场景 | 允许少量数据丢失 | 数据不能丢失 |

### 6.2 RDB 持久化机制

RDB 通过快照方式保存数据，在指定时间间隔内有指定数量的写操作时触发。

```bash
# ===== 触发方式 =====

# 1. 满足保存规则自动触发
save 900 1     # 900秒内至少1次修改
save 300 10    # 300秒内至少10次修改
save 60 10000  # 60秒内至少10000次修改

# 2. 手动触发
SAVE           # 同步保存（阻塞，不推荐）
BGSAVE         # 异步保存（fork 子进程，推荐）

# ===== 配置参数 =====
stop-writes-on-bgsave-error yes  # 保存出错时停止写入
rdbcompression yes               # 压缩 RDB 文件
rdbchecksum yes                  # 校验和检查
dbfilename dump.rdb              # RDB 文件名
dir /var/lib/redis               # 保存目录

# ===== 查看持久化状态 =====
LASTSAVE                       # 上次保存时间戳
INFO persistence               # 查看持久化信息
```

**RDB 工作流程**：

1. 调用 `BGSAVE` 或满足自动触发条件
2. Redis fork 子进程
3. 子进程遍历内存数据，写入临时 RDB 文件
4. 写入完成，替换旧的 RDB 文件
5. 子进程退出，通知父进程

**RDB 文件结构**：

```
| Magic Number | Version | DATBASES |
|--------------|---------|----------|
| 5 bytes     | 4 bytes | ...      |

数据库部分：
| SELECT DB | DBSIZE | EXPIRY | KEY-VALUE pairs |
|-----------|--------|--------|-----------------|
| 1 byte    | 4 bytes| ...    | ...             |

| EOF | CHECKSUM |
|-----|----------|
| 1 byte | 8 bytes |
```

**RDB 恢复数据**：

```bash
# 1. 备份 RDB 文件
cp /var/lib/redis/dump.rdb /backup/dump_20240101.rdb

# 2. 恢复数据
# 将 dump.rdb 放到 Redis 工作目录
cp /backup/dump_20240101.rdb /var/lib/redis/dump.rdb

# 3. 启动 Redis 自动加载
redis-server /path/to/redis.conf
# Redis 启动时检测到 dump.rdb 会自动加载

# 4. 验证数据
redis-cli dbsize
redis-cli keys '*'
```

### 6.3 AOF 持久化机制

AOF 以追加方式记录每个写命令（类似 MySQL binlog），数据安全性更高。

```bash
# ===== 开启 AOF =====
appendonly yes                    # 开启 AOF 持久化
appendfilename "appendonly.aof"   # AOF 文件名

# ===== 同步策略 =====
appendfsync always    # 每条命令同步，最安全，最慢
appendfsync everysec  # 每秒同步，平衡安全与性能（推荐）
appendfsync no        # 由 OS 决定同步时机，最快，最不安全

# ===== AOF 重写 =====
auto-aof-rewrite-percentage 100  # AOF 增长百分比触发重写
auto-aof-rewrite-min-size 64mb   # 最小重写尺寸
no-appendfsync-on-rewrite no     # 重写时是否暂停同步

# ===== 手动重写 =====
BGREWRITEAOF                     # 后台重写（压缩 AOF 文件）

# ===== 其他配置 =====
aof-load-truncated yes           # 加载时允许截断（兼容旧版本）
aof-use-rdb-preamble yes         # AOF 重写时使用 RDB 格式（4.0+）
```

**AOF 文件格式**：

```
*3\r\n           # 命令参数数量
$3\r\nSET\r\n    # 命令
$4\r\nname\r\n   # 参数1
$5\r\nredis\r\n  # 参数2
```

**AOF 重写机制**：

AOF 文件会不断增大，通过重写压缩：

```bash
# 重写前（冗余命令）
SET counter 1
INCR counter
INCR counter
INCR counter
# 实际 counter = 4

# 重写后（压缩结果）
SET counter 4
```

**AOF 工作流程**：

1. 客户端写命令到达
2. 命令追加到 AOF 缓冲区
3. 根据 `appendfsync` 策略刷到磁盘
4. 定期检测是否需要重写
5. 重写时 fork 子进程，生成压缩的新 AOF
6. 新 AOF 替换旧 AOF

**AOF 数据恢复**：

```bash
# 1. 检查 AOF 文件完整性
redis-server --appendonly yes --dir /backup

# 2. 恢复数据
# AOF 文件包含完整的写命令历史
# Redis 启动时会重放所有命令

# 3. 验证数据
redis-cli dbsize
redis-cli get counter
```

### 6.4 持久化策略选择

**企业级推荐方案**：

```
方案一：纯缓存场景
┌─────────────────────────────────┐
│ Redis 作为缓存，不做持久化        │
│ - 数据存储在数据库               │
│ - Redis 宕机后从数据库重建缓存    │
│ - 适合可接受数据丢失的场景       │
└─────────────────────────────────┘

方案二：RDB + AOF 混合（推荐）
┌─────────────────────────────────┐
│ 同时开启 RDB 和 AOF              │
│ - RDB 做冷备份（每天）           │
│ - AOF 做热备份（每秒）           │
│ - 数据恢复优先用 AOF，失败用 RDB │
│ - Redis 4.0+ 混合持久化更快      │
└─────────────────────────────────┘

方案三：仅 AOF（强一致场景）
┌─────────────────────────────────┐
│ 只开启 AOF，使用 everysec 策略   │
│ - 数据最多丢失 1 秒              │
│ - AOF 文件较大                   │
│ - 适合支付、订单等场景           │
└─────────────────────────────────┘
```

**混合持久化示例（Redis 4.0+）**：

```bash
# Redis 4.0+ 支持混合持久化
# AOF 重写时，前半部分用 RDB 格式，后半部分用 AOF 格式
# 兼顾恢复速度与数据安全

# 配置
appendonly yes
appendfsync everysec
aof-use-rdb-preamble yes  # 开启混合持久化

# 效果
# - 恢复速度接近 RDB（快）
# - 数据安全性接近 AOF（高）
# - 文件体积介于两者之间
```

**备份脚本示例**：

```bash
#!/bin/bash
# Redis 备份脚本

BACKUP_DIR="/data/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)
REDIS_CLI="/usr/local/redis/bin/redis-cli"
REDIS_PASS="your_password"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 触发 RDB 保存
$REDIS_CLI -a $REDIS_PASS BGSAVE

# 等待保存完成
while true; do
    LAST_SAVE=$($REDIS_CLI -a $REDIS_PASS LASTSAVE)
    sleep 1
    CURRENT_SAVE=$($REDIS_CLI -a $REDIS_PASS LASTSAVE)
    if [ "$LAST_SAVE" = "$CURRENT_SAVE" ]; then
        break
    fi
done

# 备份 RDB 文件
cp /var/lib/redis/dump.rdb $BACKUP_DIR/dump_$DATE.rdb

# 备份 AOF 文件
cp /var/lib/redis/appendonly.aof $BACKUP_DIR/appendonly_$DATE.aof

# 删除30天前的备份
find $BACKUP_DIR -name "*.rdb" -mtime +30 -delete
find $BACKUP_DIR -name "*.aof" -mtime +30 -delete

echo "Backup completed: $DATE"
```

---

## 7. Redis 事务

### 7.1 事务概念与 ACID 特性

#### 什么是事务

事务是一组命令的集合，要么全部执行成功，要么全部失败，保证数据一致性。

#### ACID 特性

| 特性 | 说明 | Redis 支持 |
|------|------|-----------|
| 原子性（Atomicity） | 事务中操作不可分割 | ✅ 支持 |
| 一致性（Consistency） | 事务前后数据一致 | ✅ 支持 |
| 隔离性（Isolation） | 并发事务互不干扰 | ✅ 单线程保证 |
| 持久性（Durability） | 事务提交后永久保存 | ⚠️ 依赖持久化配置 |

#### Redis 事务 vs 关系型数据库

| 特性 | Redis 事务 | MySQL 事务 |
|------|----------|-----------|
| 原子性 | 全部成功或失败 | 全部成功或回滚 |
| 回滚支持 | ❌ 不支持回滚 | ✅ 支持回滚 |
| 隔离级别 | 单线程天然隔离 | 多级别（读未提交等） |
| 锁机制 | WATCH 乐观锁 | 悲观锁/乐观锁 |
| 性能 | 高 | 中 |

#### Redis 事务的不足

1. **不支持回滚**：命令执行失败不会回滚
2. **不支持事务中加新操作**：必须提前定义所有命令
3. **不支持跨库**：事务只能在同一个连接上
4. **不支持 Lua 脚本与事务混用**

### 7.2 事务基本操作

```bash
# ===== 基本事务 =====
MULTI                    # 开启事务
SET account:user1 1000   # 命令1
SET account:user2 2000   # 命令2
EXEC                     # 执行事务

# 事务过程中命令返回 QUEUED
# 执行时统一返回结果

# ===== 取消事务 =====
MULTI
SET counter 100
DISCARD                  # 取消事务，所有命令被丢弃

# ===== 乐观锁（WATCH）=====
# 实现 CAS（Compare And Swap）
WATCH account:user1
GET account:user1
# 假设读到 1000
MULTI
SET account:user1 900
SET account:user2 2100
EXEC                     # 如果期间 account:user1 被修改，EXEC 返回 nil

# ===== 实现转账功能 =====
# 1. 加乐观锁
WATCH account:user1 account:user2
# 2. 检查余额
balance1 = GET account:user1
# 3. 开启事务
MULTI
DECRBY account:user1 100
INCRBY account:user2 100
EXEC  # 成功返回 [900, 2100]，失败返回 nil
```

**Java 事务实现**：

```java
// 基本事务操作
public void basicTransaction() {
    try (Jedis jedis = jedisPool.getResource()) {
        // 开启事务
        Transaction transaction = jedis.multi();
        
        // 添加命令
        transaction.setex("temp:key", 60, "value");
        transaction.incr("counter");
        
        // 执行事务
        List<Object> results = transaction.exec();
        
        // 处理结果
        for (Object result : results) {
            System.out.println("结果: " + result);
        }
    }
}

// 带乐观锁的转账
public boolean transfer(String from, String to, double amount) {
    try (Jedis jedis = jedisPool.getResource()) {
        // 1. 加乐观锁
        jedis.watch(from, to);
        
        // 2. 读取余额
        double fromBalance = Double.parseDouble(jedis.get(from));
        double toBalance = Double.parseDouble(jedis.get(to));
        
        // 3. 检查余额
        if (fromBalance < amount) {
            jedis.unwatch();
            return false;
        }
        
        // 4. 开启事务
        Transaction transaction = jedis.multi();
        transaction.decrByFloat(from, amount);
        transaction.incrByFloat(to, amount);
        
        // 5. 执行事务
        List<Object> results = transaction.exec();
        
        // 6. 判断结果
        if (results == null) {
            // 事务失败（期间数据被修改）
            return false;
        }
        return true;
    }
}

// 带重试的转账（推荐）
public boolean transferWithRetry(String from, String to, double amount, int maxRetries) {
    for (int i = 0; i < maxRetries; i++) {
        if (transfer(from, to, amount)) {
            return true;
        }
        // 短暂重试
        Thread.sleep(100);
    }
    return false;
}
```

**Lua 脚本实现原子操作**：

Lua 脚本可以在 Redis 中原子性执行，是事务的最佳替代方案。

```java
// Lua 脚本：原子性转账
public void transferWithLua(String from, String to, double amount) {
    String luaScript = 
        "local from = KEYS[1] " +
        "local to = KEYS[2] " +
        "local amount = tonumber(ARGV[1]) " +
        "local fromBalance = tonumber(redis.call('GET', from)) " +
        "if fromBalance < amount then " +
        "    return -1  -- 余额不足 " +
        "end " +
        "redis.call('DECRBYFLOAT', from, amount) " +
        "redis.call('INCRBYFLOAT', to, amount) " +
        "return 1";
    
    List<String> keys = Arrays.asList(from, to);
    List<String> args = Arrays.asList(String.valueOf(amount));
    
    Long result = jedis.eval(luaScript, keys, args);
    if (result == 1L) {
        System.out.println("转账成功");
    } else {
        System.out.println("转账失败，余额不足");
    }
}

// 使用已加载的脚本（EVALSHA）
public void transferWithCachedScript(String from, String to, double amount) {
    // 首次加载脚本
    String scriptSha = jedis.scriptLoad(luaScript);
    
    // 后续使用缓存的脚本（更高效）
    Long result = jedis.evalsha(scriptSha, 2, from, to, String.valueOf(amount));
}
```

**事务最佳实践**：

1. **短事务优先**：减少锁持有时间
2. **合理使用 WATCH**：避免过度竞争
3. **使用 Lua 脚本**：复杂逻辑用 Lua 实现原子操作
4. **重试机制**：乐观锁失败时自动重试
5. **避免长命令**：事务内不要执行耗时操作
6. **合理设置超时**：监控事务执行时间

---

## 8. Redis 集群

### 8.1 主从复制

主从复制是 Redis 集群的基础，实现数据冗余和读写分离。

#### 主从复制概念

```
┌─────────┐  写操作  ┌─────────┐
│  主节点  │ ───────► │  从节点1 │
│ (Master) │  数据同步  │(Replica)│
└─────────┘          └─────────┘
     │                     
     │ 数据同步             
     ▼                     
┌─────────┐                
│  从节点2 │                
│(Replica)│                
└─────────┘                

特点：
- 主节点负责写操作
- 从节点负责读操作（减轻主节点压力）
- 主节点故障时，需要手动切换主从
```

#### 主从复制搭建

```bash
# ===== 主节点配置（6379）=====
bind 0.0.0.0
port 6379
requirepass master_password
# 主节点ID（自动生成，也可手动指定）

# ===== 从节点配置（6380）=====
bind 0.0.0.0
port 6380
replicaof 127.0.0.1 6379  # 主节点地址和端口
masterauth master_password  # 主节点密码
requirepass replica_password

# ===== 查看复制状态 =====
# 在主节点执行
redis-cli -a master_password INFO replication

# 在从节点执行
redis-cli -a replica_password INFO replication
```

**Java 操作示例**：

```java
// 主从配置
public class RedisMasterSlave {
    private JedisPool masterPool;
    private List<JedisPool> slavePools;
    
    // 初始化连接池
    public void init() {
        // 主节点
        masterPool = new JedisPool(new JedisPoolConfig(), "127.0.0.1", 6379, 2000, "master_pwd");
        
        // 从节点列表
        slavePools = Arrays.asList(
            new JedisPool(new JedisPoolConfig(), "127.0.0.1", 6380, 2000, "slave_pwd"),
            new JedisPool(new JedisPoolConfig(), "127.0.0.1", 6381, 2000, "slave_pwd")
        );
    }
    
    // 写操作走主节点
    public void set(String key, String value) {
        try (Jedis jedis = masterPool.getResource()) {
            jedis.set(key, value);
        }
    }
    
    // 读操作走从节点（负载均衡）
    public String get(String key) {
        // 随机选择从节点
        int index = ThreadLocalRandom.current().nextInt(slavePools.size());
        try (Jedis jedis = slavePools.get(index).getResource()) {
            return jedis.get(key);
        }
    }
}
```

#### 主从复制原理

```
1. 从节点连接主节点
   └─ 发送 PSYNC 命令（2.8+版本）

2. 全量同步（首次连接或断点续传失败）
   └─ 主节点执行 BGSAVE 生成 RDB 文件
   └─ 发送 RDB 文件到从节点
   └─ 从节点加载 RDB 文件
   └─ 发送 RDB 期间的写命令

3. 增量同步（断点续传成功）
   └─ 主节点发送从节点缺失的写命令
   └─ 从节点接收并执行

4. 数据持续同步
   └─ 主节点将写命令写入 replication backlog
   └─ 从节点通过 PSYNC 命令拉取增量更新
```

**复制相关配置**：

```bash
# 主节点配置
# repl-backlog-size 1mb        # 复制积压缓冲区大小
# repl-backlog-ttl 3600        # 积压缓冲区生存时间
# repl-timeout 60              # 复制超时时间

# 从节点配置
# replica-read-only yes         # 从节点只读
# replica-serve-stale-data yes  # 服务过期数据
# replica-priority 100          # 优先级（哨兵选举时使用，越小越优先）
```

### 8.2 哨兵监控

哨兵（Sentinel）是 Redis 官方推荐的高可用方案，自动故障转移。

#### 哨兵概述

```
┌──────────┐
│  哨兵进程  │
│ (Sentinel)│
└─────┬─────┘
      │ 监控
      ▼
┌─────────┐  写操作  ┌─────────┐
│  主节点  │ ───────► │  从节点1 │
│         │  数据同步  │         │
└────┬────┘          └─────────┘
     │ 
     │ 故障时自动提升
     ▼
┌─────────┐
│  从节点2 │
│  (新主) │
└─────────┘
```

**哨兵功能**：

1. **监控（Monitoring）**：持续监控主从节点状态
2. **通知（Notification）**：故障时发送通知
3. **自动故障转移（Automatic Failover）**：自动切换主从
4. **配置提供者（Configuration Provider）**：客户端获取主节点地址

#### 哨兵搭建

```bash
# ===== 创建哨兵配置文件 =====
# sentinel.conf

# 监控的主节点（名称自定义）
sentinel monitor mymaster 127.0.0.1 6379 2
# 参数解释：
# mymaster: 主节点名称
# 127.0.0.1:6379: 主节点地址
# 2: 判定主节点失效的哨兵数（超过此数才故障转移）

# 主节点密码
sentinel auth-pass mymaster master_password

# 判定主观下线时间（毫秒）
sentinel down-after-milliseconds mymaster 5000

# 故障转移超时时间
sentinel failover-timeout mymaster 60000

# 并行同步数量
sentinel parallel-syncs mymaster 1

# ===== 启动哨兵 =====
redis-sentinel sentinel.conf

# ===== 查看哨兵状态 =====
redis-cli -p 26379 SENTINEL MASTER mymaster
redis-cli -p 26379 SENTINEL SLAVES mymaster
redis-cli -p 26379 SENTINEL SENTINELS mymaster
```

**推荐哨兵集群**：

```
┌─────────────┐
│  哨兵节点1    │ :26379
└─────────────┘
      │
      │ 三者互相监控
      ▼
┌─────────────┐     ┌─────────────┐
│  哨兵节点2    │     │  哨兵节点3    │
└─────────────┘     └─────────────┘

原因：
- 哨兵也可能故障
- 奇数个哨兵（3个）防止脑裂
- 多数派（2/3）同意才执行故障转移
```

**Java 哨兵模式使用**：

```java
// Sentinel 配置
public class RedisSentinelConfig {
    
    public JedisPool getSentinelPool() {
        Set<String> sentinels = new HashSet<>(Arrays.asList(
            "127.0.0.1:26379",
            "127.0.0.1:26380",
            "127.0.0.1:26381"
        ));
        
        // 通过哨兵获取主节点连接池
        return new JedisSentinelPool("mymaster", sentinels, 
            new JedisPoolConfig(), 2000, 2000, "master_password");
    }
}

// 业务使用
public class RedisService {
    private JedisSentinelPool pool;
    
    public RedisService() {
        Set<String> sentinels = new HashSet<>(Arrays.asList(
            "127.0.0.1:26379", "127.0.0.1:26380", "127.0.0.1:26381"
        ));
        pool = new JedisSentinelPool("mymaster", sentinels);
    }
    
    public void set(String key, String value) {
        try (Jedis jedis = pool.getResource()) {
            jedis.set(key, value);
        }
    }
    
    public String get(String key) {
        try (Jedis jedis = pool.getResource()) {
            return jedis.get(key);
        }
    }
}
```

#### 哨兵故障转移原理

```
1. 主观下线（Subjectively Down）
   └─ 单个哨兵在 down-after-milliseconds 内无法连接主节点

2. 客观下线（Objectively Down）
   └─ 超过 quorum 个哨兵判定主节点下线

3. 选举领头哨兵（Leader Election）
   └─ 所有哨兵通过投票选出领头
   └─ 领头负责执行故障转移

4. 故障转移执行
   └─ 从从节点中选择最优节点
      - 排除已下线或断线的从节点
      - 选择复制偏移量最大的（数据最新）
      - 按优先级排序
   └─ 将选中的从节点提升为新主节点
   └─ 配置其他从节点指向新主节点
   └─ 通知客户端新主节点地址
```

### 8.3 Cluster 集群模式

Cluster 是 Redis 3.0+ 官方推荐的分布式集群方案，支持横向扩展。

#### Cluster 模式概述

```
┌─────────┐  Hash Slot 0-5460  ┌─────────┐
│ 节点1   │ ──────────────────►│ 节点2   │
│ (Master)│  Hash Slot 5461-10922 (Master)│
└────┬────┘                    └────┬────┘
     │                              │
     │ 从节点                         │ 从节点
     ▼                              ▼
┌─────────┐                    ┌─────────┐
│ 节点4   │                    │ 节点5   │
│(Replica)│                    │(Replica)│
└─────────┘                    └─────────┘
     │                              │
     └──────────────────────────────┘
     │                              │
     ▼                              ▼
┌─────────┐  Hash Slot 10923-16383
│ 节点3   │
│ (Master)│
└────┬────┘
     │ 从节点
     ▼
┌─────────┐
│ 节点6   │
│(Replica)│
└─────────┘

核心概念：
- 16384 个 Hash Slot
- 每个 Key 分配到某个 Slot
- 每个节点负责一段 Slot
- 支持横向扩展
```

#### Cluster 搭建

```bash
# ===== 创建集群配置 =====
# 节点1 (端口 6379)
# node-1.conf
port 6379
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
cluster-announce-ip 127.0.0.1
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
requirepass cluster_password
masterauth cluster_password

# 其他节点类似配置，修改端口即可

# ===== 启动所有节点 =====
redis-server node-1.conf
redis-server node-2.conf
redis-server node-3.conf

# ===== 创建集群 =====
# 3 主 3 从（推荐配置）
redis-cli -a cluster_password \
  --cluster create \
  127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 \
  127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 \
  --cluster-replicas 1

# 参数解释：
# 前3个主节点
# 后3个从节点
# --cluster-replicas 1: 每个主节点1个从节点

# ===== 验证集群 =====
redis-cli -c -a cluster_password CLUSTER INFO
redis-cli -c -a cluster_password CLUSTER NODES
redis-cli -c -a cluster_password CLUSTER SLOTS
```

**Cluster 常用命令**：

```bash
# 集群管理
CLUSTER INFO                    # 集群信息
CLUSTER NODES                   # 所有节点信息
CLUSTER SLOTS                   # Slot 分配情况
CLUSTER KEYSLOT key             # Key 所在 Slot
CLUSTER COUNTKEYSINSLOT slot    # Slot 内 Key 数量
CLUSTER GETKEYSINSLOT slot count # 获取 Slot 内 Key

# 节点操作
CLUSTER MEET host port          # 节点间建立连接
CLUSTER ADDSLOTS slot [slot...] # 分配 Slot
CLUSTER DELSLOTS slot [slot...] # 删除 Slot
CLUSTER FAILOVER                # 手动故障转移

# 数据迁移
CLUSTER SETSLOT slot MIGRATING new_node_id
CLUSTER SETSLOT slot IMPORTING old_node_id
CLUSTER SETSLOT slot STABLE
CLUSTER SETSLOT slot NODE node_id
```

**Hash Slot 计算规则**：

```python
# CRC16 算法计算 Slot
def key_to_slot(key):
    """计算 Key 所在的 Hash Slot"""
    # 如果有 hash tag，使用 tag 内内容
    if '{' in key and '}' in key:
        tag_start = key.index('{') + 1
        tag_end = key.index('}')
        if tag_start < tag_end:
            key = key[tag_start:tag_end]
    
    # CRC16 计算
    crc = crc16(key.encode())
    return crc % 16384

# 示例
key_to_slot("user:123")  # => 5461
key_to_slot("{user}:123")  # 使用 {user} 作为 hash key
key_to_slot("{user}:456")  # 同样使用 {user}，保证落在同一 Slot
```

### 8.4 Java 操作 Redis 集群

```java
// Cluster 配置
public class RedisClusterConfig {
    
    public JedisCluster getJedisCluster() {
        Set<HostAndPort> nodes = new HashSet<>();
        nodes.add(new HostAndPort("127.0.0.1", 6379));
        nodes.add(new HostAndPort("127.0.0.1", 6380));
        nodes.add(new HostAndPort("127.0.0.1", 6381));
        
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(100);
        poolConfig.setMaxIdle(20);
        
        return new JedisCluster(
            nodes,
            2000,     // 连接超时
            2000,     // 读取超时
            3,        // 重试次数
            "cluster_password",
            poolConfig
        );
    }
}

// 业务层使用
public class RedisClusterService {
    private JedisCluster jedisCluster;
    
    public RedisClusterService() {
        Set<HostAndPort> nodes = new HashSet<>();
        nodes.add(new HostAndPort("127.0.0.1", 6379));
        nodes.add(new HostAndPort("127.0.0.1", 6380));
        nodes.add(new HostAndPort("127.0.0.1", 6381));
        jedisCluster = new JedisCluster(nodes, 2000, 2000, 3, "password");
    }
    
    // 基础操作
    public void set(String key, String value) {
        jedisCluster.set(key, value);
    }
    
    public String get(String key) {
        return jedisCluster.get(key);
    }
    
    // 批量操作（Pipeline）
    public List<Object> batchSet(Map<String, String> data) {
        Pipeline pipeline = jedisCluster.pipelined();
        for (Map.Entry<String, String> entry : data.entrySet()) {
            pipeline.set(entry.getKey(), entry.getValue());
        }
        return pipeline.sync();
    }
    
    // 事务操作
    public List<Object> transaction() {
        return jedisCluster.transaction(new TransactionCallback() {
            @Override
            public List<Object> execute(Transaction transaction) {
                transaction.setex("key1", 60, "value1");
                transaction.setex("key2", 60, "value2");
                return transaction.exec();
            }
        });
    }
    
    // Lua 脚本
    public Long evalLua(String script, List<String> keys, List<String> args) {
        return (Long) jedisCluster.eval(script, keys, args);
    }
}
```

**Spring Cloud Redis Cluster 配置**：

```yaml
# application.yml
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:6379
        - 127.0.0.1:6380
        - 127.0.0.1:6381
      max-redirects: 3
    lettuce:
      pool:
        max-active: 100
        max-idle: 20
        min-idle: 10
    password: cluster_password
    timeout: 2000
```

**Cluster vs 哨兵 vs 主从**：

| 特性 | 主从 | 哨兵 | Cluster |
|------|------|------|---------|
| 数据分片 | ❌ | ❌ | ✅ |
| 横向扩展 | ❌ | ❌ | ✅ |
| 高可用 | ❌ | ✅ | ✅ |
| 读写分离 | ✅ | ✅ | ✅ |
| 自动故障转移 | ❌ | ✅ | ✅ |
| 适用规模 | 小 | 中 | 大 |
| 复杂度 | 低 | 中 | 高 |

---

## 9. Java 整合 Redis

### 9.1 Jedis 操作

Jedis 是 Redis 的 Java 客户端库，简单易用，适合快速开发。

#### 环境准备

```xml
<!-- pom.xml 添加依赖 -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.4.6</version>
</dependency>
```

#### Jedis 基础操作（上）

```java
// 1. 创建连接
Jedis jedis = new Jedis("127.0.0.1", 6379);
jedis.auth("password");  // 设置密码

// 2. String 操作
jedis.set("key", "value");                    // 设置值
String value = jedis.get("key");              // 获取值
jedis.setex("token", 3600, "user123");        // 设置过期时间
jedis.incr("counter");                        // 自增

// 3. List 操作
jedis.rpush("queue", "task1", "task2");       // 右侧推入
List<String> list = jedis.lrange("queue", 0, -1);  // 获取列表
String task = jedis.rpop("queue");             // 右侧弹出

// 4. Set 操作
jedis.sadd("tags", "Java", "Redis");          // 添加元素
Set<String> tags = jedis.smembers("tags");    // 获取所有元素
boolean exists = jedis.sismember("tags", "Java");  // 判断存在

// 5. Hash 操作
Map<String, String> user = new HashMap<>();
user.put("name", "张三");
user.put("age", "25");
jedis.hset("user:1", user);                   // 设置 Hash
String name = jedis.hget("user:1", "name");   // 获取字段
Map<String, String> all = jedis.hgetAll("user:1");  // 获取所有

// 6. ZSet 操作
jedis.zadd("rank", 100, "张三");              // 添加元素
Set<String> top10 = jedis.zrevrange("rank", 0, 9);  // Top 10
long rank = jedis.zrevrank("rank", "张三");   // 获取排名

// 7. 关闭连接
jedis.close();
```

#### Jedis 高级操作（下）

```java
// ===== 连接池配置 =====
public class JedisPoolFactory {
    private static JedisPool pool;
    
    static {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(200);        // 最大连接数
        config.setMaxIdle(50);          // 最大空闲连接
        config.setMinIdle(10);          // 最小空闲连接
        config.setMaxWaitMillis(3000);  // 最大等待时间
        config.setTestOnBorrow(true);   // 借出时检测
        config.setTestOnReturn(true);   // 归还时检测
        config.setTestWhileIdle(true);  // 空闲时检测
        
        pool = new JedisPool(config, "127.0.0.1", 6379, 2000, "password");
    }
    
    public static Jedis getJedis() {
        return pool.getResource();
    }
    
    public static void closeJedis(Jedis jedis) {
        if (jedis != null) {
            jedis.close();
        }
    }
}

// ===== Pipeline 批量操作 =====
public void batchOperation() {
    try (Jedis jedis = JedisPoolFactory.getJedis()) {
        Pipeline pipeline = jedis.pipelined();
        
        // 批量写入
        for (int i = 1; i <= 1000; i++) {
            pipeline.setex("key:" + i, 3600, "value:" + i);
        }
        
        // 批量读取
        for (int i = 1; i <= 1000; i++) {
            pipeline.get("key:" + i);
        }
        
        // 执行并获取结果
        List<Object> results = pipeline.sync();
        System.out.println("批量操作完成，共 " + results.size() + " 个结果");
    }
}

// ===== 事务操作 =====
public void transactionOperation() {
    try (Jedis jedis = JedisPoolFactory.getJedis()) {
        // 开启事务
        Transaction transaction = jedis.multi();
        
        // 添加命令
        transaction.setex("temp:key", 60, "value");
        transaction.incr("counter");
        
        // 执行事务
        List<Object> results = transaction.exec();
        
        // 处理结果
        results.forEach(System.out::println);
    }
}

// ===== Lua 脚本 =====
public void luaScriptOperation() {
    try (Jedis jedis = JedisPoolFactory.getJedis()) {
        String script = 
            "local key = KEYS[1] " +
            "local value = ARGV[1] " +
            "redis.call('SET', key, value) " +
            "return redis.call('GET', key)";
        
        List<String> keys = Arrays.asList("test:key");
        List<String> args = Arrays.asList("hello");
        
        Object result = jedis.eval(script, keys, args);
        System.out.println("Lua 执行结果: " + result);
    }
}

// ===== 发布订阅 =====
public void pubSubOperation() {
    // 发布者
    try (Jedis jedis = JedisPoolFactory.getJedis()) {
        jedis.publish("news", "新消息发布");
    }
    
    // 订阅者（新线程）
    new Thread(() -> {
        try (Jedis jedis = JedisPoolFactory.getJedis()) {
            jedis.subscribe(new JedisPubSub() {
                @Override
                public void onMessage(String channel, String message) {
                    System.out.println("收到消息: " + message);
                }
            }, "news");
        }
    }).start();
}
```

### 9.2 Spring Data Redis

Spring Data Redis 是 Spring 官方推荐的 Redis 集成方案，封装了 Jedis 和 Lettuce。

#### 环境准备

```xml
<!-- pom.xml 添加依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- 使用 Lettuce 客户端（默认） -->
</dependencies>
```

#### Spring Boot 配置

```yaml
# application.yml
spring:
  data:
    redis:
      # 单机模式
      host: 127.0.0.1
      port: 6379
      password: your_password
      database: 0
      
      # 连接池配置
      lettuce:
        pool:
          max-active: 100      # 最大连接数
          max-idle: 50         # 最大空闲
          min-idle: 10         # 最小空闲
          max-wait: 3000ms     # 最大等待
        shutdown-timeout: 100ms
      
      # 超时配置
      timeout: 3s
      
      # 哨兵模式（可选）
      sentinel:
        master: mymaster
        nodes:
          - 127.0.0.1:26379
          - 127.0.0.1:26380
          - 127.0.0.1:26381
      
      # 集群模式（可选）
      cluster:
        nodes:
          - 127.0.0.1:6379
          - 127.0.0.1:6380
          - 127.0.0.1:6381
        max-redirects: 3
```

#### RedisTemplate 使用（上）

```java
// 配置 RedisTemplate
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // Key 使用 String 序列化
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        
        // Value 使用 JSON 序列化
        Jackson2JsonRedisSerializer<Object> jsonSerializer = 
            new Jackson2JsonRedisSerializer<>(Object.class);
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
    
    // 操作封装
    @Bean
    public RedisService redisService(RedisTemplate<String, Object> template) {
        return new RedisService(template);
    }
}

// 基础服务类
public class RedisService {
    private final RedisTemplate<String, Object> template;
    
    public RedisService(RedisTemplate<String, Object> template) {
        this.template = template;
    }
    
    // String 操作
    public void set(String key, Object value) {
        template.opsForValue().set(key, value);
    }
    
    public void set(String key, Object value, long timeout, TimeUnit unit) {
        template.opsForValue().set(key, value, timeout, unit);
    }
    
    public Object get(String key) {
        return template.opsForValue().get(key);
    }
    
    public Boolean delete(String key) {
        return template.delete(key);
    }
    
    public Boolean expire(String key, long timeout, TimeUnit unit) {
        return template.expire(key, timeout, unit);
    }
    
    // Hash 操作
    public void hSet(String key, String hashKey, Object value) {
        template.opsForHash().put(key, hashKey, value);
    }
    
    public Object hGet(String key, String hashKey) {
        return template.opsForHash().get(key, hashKey);
    }
    
    // List 操作
    public Long lPush(String key, Object value) {
        return template.opsForList().leftPush(key, value);
    }
    
    public List<Object> lRange(String key, long start, long end) {
        return template.opsForList().range(key, start, end);
    }
    
    // Set 操作
    public Long sAdd(String key, Object... values) {
        return template.opsForSet().add(key, values);
    }
    
    // ZSet 操作
    public Boolean zAdd(String key, Object value, double score) {
        return template.opsForZSet().add(key, value, score);
    }
    
    // 过期时间
    public Long getExpire(String key) {
        return template.getExpire(key, TimeUnit.SECONDS);
    }
}
```

#### RedisTemplate 使用（下）

```java
// ===== 缓存注解使用 =====
@Service
public class ProductService {
    
    // 查询时缓存
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productMapper.selectById(id);
    }
    
    // 更新时刷新缓存
    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        productMapper.updateById(product);
        return product;
    }
    
    // 删除时清除缓存
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productMapper.deleteById(id);
    }
    
    // 清除所有缓存
    @CacheEvict(value = "products", allEntries = true)
    public void clearAllCache() {
    }
}

// ===== 配置文件启用缓存 =====
@Configuration
@EnableCaching
public class CacheConfig {
    // 缓存配置
}

// ===== 分布式锁实现 =====
public class DistributedLock {
    private final RedisTemplate<String, Object> redisTemplate;
    
    public DistributedLock(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    // 获取锁
    public boolean tryLock(String key, String value, long timeout) {
        Boolean result = redisTemplate.opsForValue()
            .setIfAbsent(key, value, timeout, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(result);
    }
    
    // 释放锁（Lua 保证原子性）
    public boolean releaseLock(String key, String value) {
        String script = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(key),
            value
        );
        return Long.valueOf(1L).equals(result);
    }
}

// ===== 发布订阅 =====
@Service
public class MessageService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    public MessageService(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    // 发布消息
    public void publish(String channel, Object message) {
        redisTemplate.convertAndSend(channel, message);
    }
}

// 消息监听器
@Component
public class RedisMessageListener {
    
    @Autowired
    private MessageService messageService;
    
    @Bean
    public RedisMessageListenerContainer container(
            RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = 
            new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        
        // 监听频道
        container.addMessageListener(
            new MessageListener(), 
            new ChannelTopic("news")
        );
        
        return container;
    }
    
    private class MessageListener implements RedisMessageListener {
        @Override
        public void onMessage(Message message, byte[] pattern) {
            String channel = new String(message.getChannel());
            Object body = message.getBody();
            System.out.println("频道: " + channel + ", 消息: " + body);
        }
    }
}
```

---

## 10. Redis Web 实践

### 10.1 网页缓存实战

#### 缓存穿透、击穿、雪崩解决方案

```java
// ===== 缓存工具类 =====
@Component
public class CacheManager {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 本地缓存（Caffeine，可选）
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build();
    
    public CacheManager(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    // 三级缓存查询
    public <T> T get(String key, Class<T> clazz, Callable<T> loader) {
        // 1. 本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return (T) value;
        }
        
        // 2. Redis 缓存
        value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            localCache.put(key, value);
            return (T) value;
        }
        
        // 3. 数据库查询
        try {
            value = loader.call();
            if (value != null) {
                // 写入缓存（防穿透：缓存空值）
                redisTemplate.opsForValue().set(key, value, 
                    30, TimeUnit.MINUTES);
                localCache.put(key, value);
            } else {
                // 防穿透：缓存空值
                redisTemplate.opsForValue().set(key, "NULL", 
                    60, TimeUnit.SECONDS);
            }
            return (T) value;
        } catch (Exception e) {
            throw new RuntimeException("加载缓存失败", e);
        }
    }
    
    // 带随机过期时间（防雪崩）
    public void setWithRandomExpire(String key, Object value, 
            long baseTime, TimeUnit unit) {
        // 随机增加 0-30% 过期时间
        long randomTime = (long) (baseTime * (1 + Math.random() * 0.3));
        redisTemplate.opsForValue().set(key, value, randomTime, unit);
    }
    
    // 删除缓存
    public void delete(String key) {
        localCache.invalidate(key);
        redisTemplate.delete(key);
    }
    
    // 批量删除
    public void deleteByPattern(String pattern) {
        Set<String> keys = redisTemplate.keys(pattern);
        if (keys != null && !keys.isEmpty()) {
            redisTemplate.delete(keys);
        }
    }
}

// ===== 业务层使用 =====
@Service
public class ProductService {
    
    private final CacheManager cacheManager;
    private final ProductMapper productMapper;
    
    public ProductService(CacheManager cacheManager, 
                          ProductMapper productMapper) {
        this.cacheManager = cacheManager;
        this.productMapper = productMapper;
    }
    
    // 查询商品（带缓存保护）
    public Product getProductById(Long id) {
        String key = "product:" + id;
        return cacheManager.get(key, Product.class, () -> {
            Product product = productMapper.selectById(id);
            if (product != null) {
                // 随机过期时间防雪崩
                cacheManager.setWithRandomExpire(key, product, 
                    30, TimeUnit.MINUTES);
            }
            return product;
        });
    }
    
    // 更新商品
    @Transactional
    public void updateProduct(Product product) {
        productMapper.updateById(product);
        // 删除缓存
        cacheManager.delete("product:" + product.getId());
    }
    
    // 删除商品
    @Transactional
    public void deleteProduct(Long id) {
        productMapper.deleteById(id);
        cacheManager.delete("product:" + id);
    }
    
    // 分页查询（缓存列表）
    public PageResult<Product> getProductList(int page, int size) {
        String key = "product:list:" + page + ":" + size;
        return cacheManager.get(key, PageResult.class, () -> {
            // 查询数据库
            List<Product> products = productMapper.selectPage(page, size);
            long total = productMapper.selectCount();
            return new PageResult<>(products, total);
        });
    }
}
```

#### 分布式锁实现

```java
// ===== 分布式锁工具 =====
@Component
public class RedisLock {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 线程本地变量存储锁标识
    private final ThreadLocal<String> lockOwners = new ThreadLocal<>();
    
    public RedisLock(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    // 获取锁（阻塞方式）
    public boolean lock(String key, long timeout, long waitTimeout) {
        String requestId = UUID.randomUUID().toString();
        long startTime = System.currentTimeMillis();
        
        while (true) {
            // 尝试获取锁
            Boolean locked = redisTemplate.opsForValue()
                .setIfAbsent(key, requestId, timeout, TimeUnit.MILLISECONDS);
            
            if (Boolean.TRUE.equals(locked)) {
                lockOwners.set(requestId);
                return true;
            }
            
            // 检查是否超时
            if (System.currentTimeMillis() - startTime > waitTimeout) {
                return false;
            }
            
            // 短暂等待
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
    }
    
    // 释放锁（Lua 保证原子性）
    public boolean unlock(String key) {
        String requestId = lockOwners.get();
        if (requestId == null) {
            return false;
        }
        
        String script = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(key),
            requestId
        );
        
        if (Long.valueOf(1L).equals(result)) {
            lockOwners.remove();
            return true;
        }
        return false;
    }
    
    // 重试获取锁
    public <T> T executeWithLock(String lockKey, long timeout, 
            long waitTimeout, Supplier<T> action) {
        if (!lock(lockKey, timeout, waitTimeout)) {
            throw new RuntimeException("获取锁超时");
        }
        try {
            return action.get();
        } finally {
            unlock(lockKey);
        }
    }
}

// ===== 秒杀场景使用 =====
@Service
public class SeckillService {
    
    private final RedisLock redisLock;
    private final RedisTemplate<String, Object> redisTemplate;
    private final OrderMapper orderMapper;
    
    public SeckillService(RedisLock redisLock, 
                          RedisTemplate<String, Object> redisTemplate,
                          OrderMapper orderMapper) {
        this.redisLock = redisLock;
        this.redisTemplate = redisTemplate;
        this.orderMapper = orderMapper;
    }
    
    // 秒杀下单
    public Order seckill(Long userId, Long productId) {
        String lockKey = "seckill:lock:" + productId;
        
        return redisLock.executeWithLock(lockKey, 5000, 3000, () -> {
            // 1. 检查库存
            String stockKey = "stock:" + productId;
            Long stock = redisTemplate.opsForValue().decrement(stockKey);
            
            if (stock < 0) {
                // 库存不足，恢复库存
                redisTemplate.opsForValue().increment(stockKey);
                throw new RuntimeException("商品已售罄");
            }
            
            // 2. 检查是否重复下单
            String orderKey = "order:" + productId + ":" + userId;
            Boolean isNew = redisTemplate.opsForValue()
                .setIfAbsent(orderKey, "1", 30, TimeUnit.MINUTES);
            if (!Boolean.TRUE.equals(isNew)) {
                redisTemplate.opsForValue().increment(stockKey);
                throw new RuntimeException("您已下单，请勿重复操作");
            }
            
            // 3. 创建订单
            Order order = new Order();
            order.setUserId(userId);
            order.setProductId(productId);
            order.setCreateTime(new Date());
            orderMapper.insert(order);
            
            return order;
        });
    }
    
    // 初始化库存
    public void initStock(Long productId, int stock) {
        String stockKey = "stock:" + productId;
        redisTemplate.opsForValue().set(stockKey, stock);
    }
}
```

---

## 11. 企业级解决方案

### 11.1 Redis 脑裂

#### 什么是脑裂

脑裂（Split-Brain）是指集群中不同节点对"谁是主节点"产生分歧，导致多个主节点同时服务。

```
正常状态：
┌─────────┐
│ 主节点 A │ ─── 同步 ───► 从节点
└─────────┘

网络分区（脑裂）：
┌─────────┐     ┌─────────┐
│ 主节点 A │     │ 从节点 B │
│ (失联)   │     │(被提升为主)│
└─────────┘     └─────────┘
     │                │
     ▼                ▼
  继续接受写      也接受写
  （数据不一致）  （数据不一致）
```

#### 脑裂解决方案

**方案一：Quorum 机制（推荐）**

使用奇数个哨兵节点（3个或5个），必须超过半数同意才能选举新主。

```bash
# sentinel.conf
# 至少需要 3 个哨兵节点
sentinel monitor mymaster 127.0.0.1 6379 2
# 参数 2 表示：至少 2 个哨兵同意才切换主节点
# 3 个哨兵时：2/3 同意
# 5 个哨兵时：3/5 同意
```

**方案二：设置告警阈值**

```bash
# 配置告警
sentinel notification-script mymaster /path/to/notify.sh
# 当发生 failover 时执行通知脚本

# 通知脚本 notify.sh
#!/bin/bash
CHANNEL=$1
EVENT=$2
# 发送告警邮件/短信
echo "Redis Sentinel Alert: $EVENT" | mail admin@example.com
```

**方案三： ZooKeeper/etcd 协调**

```java
// 使用 ZooKeeper 做分布式锁
public class ZkRedisLock {
    private ZooKeeper zk;
    
    // 创建节点表示主节点
    public void createMasterNode(String masterId) {
        String path = "/redis/master";
        try {
            zk.create(path, masterId.getBytes(), 
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL);  // 临时节点，断开自动删除
        } catch (NodeExistsException e) {
            // 主节点已存在，获取锁失败
            throw new RuntimeException("主节点已存在");
        }
    }
    
    // 监听主节点变化
    public void listenMasterChange() {
        zk.getData("/redis/master", true, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getType() == Event.EventType.NodeDeleted) {
                    // 主节点下线，尝试成为新主
                    tryBecomeMaster();
                }
            }
        }, null);
    }
}
```

### 11.2 缓存预热

#### 什么是缓存预热

缓存预热是系统启动时，提前将热点数据加载到缓存，避免首次访问穿透到数据库。

#### 缓存预热方案

**方案一：启动时预热**

```java
// 应用启动时预热缓存
@Component
public class CachePreloader implements CommandLineRunner {
    
    private final ProductService productService;
    private final RedisTemplate<String, Object> redisTemplate;
    
    public CachePreloader(ProductService productService, 
                          RedisTemplate<String, Object> redisTemplate) {
        this.productService = productService;
        this.redisTemplate = redisTemplate;
    }
    
    @Override
    public void run(String... args) {
        // 预热热点商品
        preloadHotProducts();
        // 预热系统配置
        preloadSystemConfig();
        // 预热用户权限
        preloadUserPermissions();
        
        System.out.println("缓存预热完成");
    }
    
    private void preloadHotProducts() {
        List<Product> hotProducts = productService.getHotProducts();
        for (Product product : hotProducts) {
            String key = "product:" + product.getId();
            redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
        }
    }
    
    private void preloadSystemConfig() {
        List<SystemConfig> configs = configMapper.selectAll();
        for (SystemConfig config : configs) {
            String key = "config:" + config.getKey();
            redisTemplate.opsForValue().set(key, config.getValue());
        }
    }
    
    private void preloadUserPermissions() {
        List<Role> roles = roleMapper.selectAll();
        for (Role role : roles) {
            Set<String> permissions = permissionMapper.selectByRoleId(role.getId());
            String key = "perm:" + role.getName();
            redisTemplate.opsForSet().add(key, permissions.toArray());
        }
    }
}
```

**方案二：定时预热**

```java
// 定时刷新热点数据
@Component
public class CacheRefresher {
    
    private final ProductService productService;
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 每 5 分钟刷新一次热点数据
    @Scheduled(fixedRate = 300000)
    public void refreshHotData() {
        // 1. 获取访问量最高的商品
        List<Long> hotProductIds = getHotProductIds();
        
        // 2. 批量刷新缓存
        Pipeline pipeline = redisTemplate.pipelined();
        for (Long id : hotProductIds) {
            Product product = productService.getProductFromDB(id);
            if (product != null) {
                String key = "product:" + id;
                pipeline.setex(key, 30 * 60, JSON.toJSONString(product));
            }
        }
        pipeline.sync();
        
        // 3. 清理过期热点
        cleanExpiredHotKeys();
    }
    
    private List<Long> getHotProductIds() {
        // 从 ZSet 获取访问量最高的商品
        Set<String> hotIds = redisTemplate.opsForZSet()
            .reverseRange("product:visit:count", 0, 99);
        return hotIds.stream()
            .map(Long::parseLong)
            .collect(Collectors.toList());
    }
}
```

**方案三：消息驱动预热**

```java
// 监听数据库变更，自动刷新缓存
@Component
public class CacheRefreshListener {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    @EventListener
    public void onProductUpdated(ProductUpdatedEvent event) {
        // 收到商品更新事件，刷新缓存
        String key = "product:" + event.getProductId();
        Product product = productMapper.selectById(event.getProductId());
        redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
    }
    
    @EventListener
    public void onCategoryChanged(CategoryChangedEvent event) {
        // 分类变更，清除相关商品缓存
        String pattern = "product:cat:" + event.getCategoryId() + ":*";
        Set<String> keys = redisTemplate.keys(pattern);
        if (keys != null) {
            redisTemplate.delete(keys);
        }
    }
}
```

### 11.3 缓存穿透

#### 什么是缓存穿透

查询一个不存在的数据，缓存和数据库都查不到，每次请求都穿透到数据库。

```
请求 -> Redis(未命中) -> MySQL(未命中) -> 返回空
请求 -> Redis(未命中) -> MySQL(未命中) -> 返回空  (循环穿透)
```

#### 穿透解决方案

**方案一：缓存空值**

```java
@Service
public class AntiPenetrationService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 查询时缓存空值
    public Product getProduct(Long id) {
        String key = "product:" + id;
        
        // 1. 查缓存
        Object cached = redisTemplate.opsForValue().get(key);
        if (cached != null) {
            // 缓存的是空值标记
            if ("NULL".equals(cached)) {
                return null;
            }
            return (Product) cached;
        }
        
        // 2. 查数据库
        Product product = productMapper.selectById(id);
        if (product != null) {
            // 缓存真实数据
            redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
        } else {
            // 缓存空值标记（短期）
            redisTemplate.opsForValue().set(key, "NULL", 60, TimeUnit.SECONDS);
        }
        
        return product;
    }
}
```

**方案二：布隆过滤器（推荐）**

```java
// 使用布隆过滤器过滤不存在的数据
@Component
public class BloomFilterService {
    
    private BloomFilter<Long> bloomFilter;
    
    public BloomFilterService() {
        // 初始化布隆过滤器
        // 预计存放 100 万商品，误判率 0.1%
        this.bloomFilter = BloomFilter.create(
            Funnels.longFunnel(), 
            1000000, 
            0.001
        );
        
        // 初始化时加载所有商品 ID
        initProductIds();
    }
    
    private void initProductIds() {
        List<Long> allIds = productMapper.selectAllIds();
        for (Long id : allIds) {
            bloomFilter.put(id);
        }
    }
    
    // 判断商品是否存在
    public boolean mightContain(Long productId) {
        return bloomFilter.mightContain(productId);
    }
    
    // 新增商品时更新布隆过滤器
    public void addProduct(Long productId) {
        bloomFilter.put(productId);
    }
}

// 业务层使用
public Product getProductWithBloom(Long id) {
    // 1. 布隆过滤器检查
    if (!bloomFilterService.mightContain(id)) {
        return null;  // 肯定不存在，直接返回
    }
    
    // 2. 查缓存
    String key = "product:" + id;
    Product product = (Product) redisTemplate.opsForValue().get(key);
    if (product != null) {
        return product;
    }
    
    // 3. 查数据库
    product = productMapper.selectById(id);
    if (product != null) {
        redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
    }
    
    return product;
}
```

**方案三：参数校验**

```java
// 接口层做参数校验
@RestController
public class ProductController {
    
    // 参数校验防止非法 ID 查询
    @GetMapping("/product/{id}")
    public Product getProduct(
            @PathVariable @Min(1) @Max(Long.MAX_VALUE) Long id) {
        // id 必须是正整数，否则直接返回错误
        return productService.getProduct(id);
    }
    
    // 对于特殊格式的 ID（如 UUID），用正则校验
    @GetMapping("/order/{orderNo}")
    public Order getOrder(
            @PathVariable @Pattern(regexp = "^ORD\\d{16}$") String orderNo) {
        return orderService.getByOrderNo(orderNo);
    }
}
```

### 11.4 缓存击穿

#### 什么是缓存击穿

某个热点 key 过期的瞬间，大量并发请求穿透到数据库。

```
时间线：
T0: 热点 Key 过期
T1: 请求1 -> 缓存(未命中) -> 数据库
T2: 请求2 -> 缓存(未命中) -> 数据库
T3: 请求3 -> 缓存(未命中) -> 数据库
... 
Tn: 大量请求同时打到数据库（雪崩效应）
```

#### 击穿解决方案

**方案一：互斥锁（推荐）**

```java
@Service
public class HotKeyService {
    
    private final RedisLock redisLock;
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 获取热点数据（带互斥锁）
    public Product getHotProduct(Long id) {
        String key = "hot:product:" + id;
        
        // 1. 先查缓存
        Product product = (Product) redisTemplate.opsForValue().get(key);
        if (product != null) {
            return product;
        }
        
        // 2. 缓存未命中，获取分布式锁
        String lockKey = "lock:hot:product:" + id;
        boolean locked = redisLock.lock(lockKey, 5000, 3000);
        
        if (!locked) {
            // 3. 获取锁失败，短暂等待后重试
            sleep(100);
            return getHotProduct(id);  // 递归重试
        }
        
        try {
            // 4. 获取锁成功，再查缓存（可能已被其他线程刷新）
            product = (Product) redisTemplate.opsForValue().get(key);
            if (product != null) {
                return product;
            }
            
            // 5. 查数据库
            product = productMapper.selectById(id);
            if (product != null) {
                // 6. 写入缓存，设置随机过期时间
                long expireTime = 30 + (long)(Math.random() * 10);  // 30-40分钟
                redisTemplate.opsForValue().set(key, product, expireTime, TimeUnit.MINUTES);
            }
            
            return product;
        } finally {
            // 7. 释放锁
            redisLock.unlock(lockKey);
        }
    }
}
```

**方案二：永不过期（逻辑过期）**

```java
// 逻辑过期方案
public class LogicalExpireService {
    
    // 缓存结构：value + 过期时间戳
    public void setWithLogicalExpire(String key, Object value, long expireTime) {
        CacheObject cache = new CacheObject(value, expireTime + System.currentTimeMillis());
        redisTemplate.opsForValue().set(key, cache);
    }
    
    public Product getWithLogicalExpire(Long id) {
        String key = "product:" + id;
        
        CacheObject cache = (CacheObject) redisTemplate.opsForValue().get(key);
        if (cache == null) {
            return null;
        }
        
        // 判断是否逻辑过期
        if (cache.isExpired()) {
            // 过期了，启动新线程更新缓存
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    String lockKey = "lock:product:" + id;
                    if (tryLock(lockKey)) {
                        // 查数据库，更新缓存
                        Product product = productMapper.selectById(id);
                        setWithLogicalExpire(key, product, 30 * 60);
                    }
                } catch (Exception e) {
                    log.error("缓存重建失败", e);
                }
            });
        }
        
        // 直接返回旧数据（保证可用性）
        return (Product) cache.getValue();
    }
    
    private boolean tryLock(String key) {
        Boolean result = redisTemplate.opsForValue()
            .setIfAbsent(key, "1", 3, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(result);
    }
}

// 缓存对象封装
@Data
public class CacheObject {
    private Object value;
    private long expireTime;
    
    public boolean isExpired() {
        return System.currentTimeMillis() > expireTime;
    }
}
```

**方案三：热点Key永不过期 + 后台刷新**

```java
// 预热热点 Key，永不过期
@Component
public class HotKeyManager {
    
    private final ScheduledExecutorService executor;
    
    public void start() {
        // 启动时加载热点数据
        loadHotKeys();
        
        // 后台定时刷新（每 5 分钟）
        executor.scheduleAtFixedRate(() -> {
            refreshHotKeys();
        }, 5, 5, TimeUnit.MINUTES);
    }
    
    private void loadHotKeys() {
        // 加载热点商品、热门搜索等
        List<Product> hotProducts = productMapper.selectHotProducts();
        for (Product product : hotProducts) {
            String key = "hot:product:" + product.getId();
            // 不设置过期时间
            redisTemplate.opsForValue().set(key, product);
        }
    }
    
    private void refreshHotKeys() {
        // 重新加载最新热点数据
        List<Product> latestHot = productMapper.selectHotProducts();
        for (Product product : latestHot) {
            String key = "hot:product:" + product.getId();
            redisTemplate.opsForValue().set(key, product);
        }
    }
}
```

### 11.5 缓存雪崩

#### 什么是缓存雪崩

大量 Key 在同一时间过期，或 Redis 实例宕机，导致请求全部打到数据库。

```
时间线：
T0: 大量 Key 同时过期（比如午夜批量过期）
T1: 请求 -> 缓存(未命中) -> 数据库
T2: 请求 -> 缓存(未命中) -> 数据库
...
Tn: 数据库被压垮，雪崩发生

另一种场景：
Redis 集群故障 -> 所有请求直接访问数据库 -> 数据库过载
```

#### 雪崩解决方案

**方案一：随机过期时间**

```java
// 基础时间 + 随机时间，避免集中过期
public void setWithRandomExpire(String key, Object value, long baseMinutes) {
    // 随机增加 0-20% 时间
    long randomExtra = (long) (baseMinutes * 0.2 * Math.random());
    long totalMinutes = baseMinutes + randomExtra;
    redisTemplate.opsForValue().set(key, value, totalMinutes, TimeUnit.MINUTES);
}
```

**方案二：多级缓存**

```java
// 本地缓存 + Redis + 数据库
@Service
public class MultiLevelCacheService {
    
    // 本地缓存（Caffeine）
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(5000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build();
    
    // Redis 缓存
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 获取数据（三级缓存）
    public <T> T get(String key, Class<T> type, Callable<T> loader) {
        // 1. 本地缓存（毫秒级）
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return (T) value;
        }
        
        // 2. Redis 缓存（亚毫秒级）
        try {
            value = redisTemplate.opsForValue().get(key);
            if (value != null) {
                localCache.put(key, value);
                return (T) value;
            }
        } catch (Exception e) {
            // Redis 不可用，降级到数据库
            log.warn("Redis 不可用，降级到数据库");
        }
        
        // 3. 数据库
        try {
            value = loader.call();
            if (value != null) {
                // 写入两级缓存
                localCache.put(key, value);
                try {
                    redisTemplate.opsForValue().set(key, value, 30, TimeUnit.MINUTES);
                } catch (Exception e) {
                    log.warn("Redis 写入失败");
                }
            }
            return (T) value;
        } catch (Exception e) {
            throw new RuntimeException("加载数据失败", e);
        }
    }
}
```

**方案三：熔断降级**

```java
// 使用 Sentinel 做熔断降级
@Configuration
@EnableSentinel
public class SentinelConfig {
    
    @Bean
    public SentinelResource sentinelResource() {
        return new SentinelResource("redis-fallback", 
            new FallbackProvider() {
                @Override
                public Throwable provide(BlockException ex) {
                    // Redis 不可用时的降级逻辑
                    return new RuntimeException("服务暂不可用，请稍后重试");
                }
            });
    }
}

// 使用 Sentinel 保护 Redis 调用
@Service
public class SentinelRedisService {
    
    public Product getProduct(Long id) {
        String key = "product:" + id;
        
        return SentinelUtil.doFallback("redis-get", 
            // 正常逻辑（访问 Redis）
            () -> {
                Object value = redisTemplate.opsForValue().get(key);
                if (value != null) {
                    return (Product) value;
                }
                // Redis 未命中，查数据库
                Product product = productMapper.selectById(id);
                if (product != null) {
                    redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
                }
                return product;
            },
            // 降级逻辑（Redis 不可用时）
            throwable -> {
                // 直接查数据库（可能较慢但保证可用）
                return productMapper.selectById(id);
            }
        );
    }
}
```

**方案四：Redis 高可用**

```
推荐架构：
┌─────────────┐
│  客户端应用  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  Sentinel 1 │────►│  Sentinel 2 │
└──────┬──────┘     └──────┬──────┘
       │                    │
       ▼                    ▼
┌─────────────┐     ┌─────────────┐
│  主节点      │◄────┤  从节点      │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  从节点      │
└─────────────┘

关键点：
- 至少 3 个 Sentinel 节点
- 每个主节点至少 1 个从节点
- 主从分别部署在不同物理机
- 监控告警（Prometheus + Grafana）
```

### 11.6 Redis 开发规范

#### Key 命名规范

```java
// 推荐的 Key 命名规范
// 格式：业务前缀:对象类型:唯一标识[:扩展标识]

// 示例
String productKey = "product:info:" + productId;          // 商品信息
String userKey = "user:profile:" + userId;                 // 用户资料
String orderKey = "order:detail:" + orderNo;               // 订单详情
String tokenKey = "auth:token:" + userId + ":" + token;    // 认证令牌
String rankKey = "rank:game:" + seasonId;                  // 排行榜
String configKey = "config:system:" + configKey;           // 系统配置
String lockKey = "lock:order:" + orderNo;                   // 分布式锁
String rateLimitKey = "rate:limit:" + userId + ":" + api;  // 限流

// 禁止使用的 Key
// ❌ "user"  - 太泛
// ❌ "12345" - 无意义
// ❌ "user_info_123" - 不规范
```

#### 使用规范

```java
// 1. 强制设置过期时间
public void setWithExpire(String key, Object value) {
    // 必须指定过期时间
    redisTemplate.opsForValue().set(key, value, 30, TimeUnit.MINUTES);
}

// 2. 避免使用 KEYS 命令
// ❌ 禁止在生产环境使用 KEYS *
// ✅ 使用 SCAN 渐进式遍历
public void scanKeys(String pattern) {
    Set<String> keys = new HashSet<>();
    ScanOptions options = ScanOptions.scanOptions()
        .match(pattern)
        .count(100)
        .build();
    
    try (Cursor<String> cursor = redisTemplate.scan(options)) {
        while (cursor.hasNext()) {
            keys.add(cursor.next());
        }
    }
}

// 3. 使用 Pipeline 批量操作
public void batchSet(Map<String, Object> data) {
    redisTemplate.executePipelined((RedisCallback<Object>) connection -> {
        for (Map.Entry<String, Object> entry : data.entrySet()) {
            connection.set(
                entry.getKey().getBytes(),
                SerializationUtils.serialize(entry.getValue()),
                Expiration.from(30, TimeUnit.MINUTES.toSeconds()),
                RedisStringCommands.SetOption.UPSERT
            );
        }
        return null;
    });
}

// 4. 大 Key 拆分
// ❌ 存储超大 Hash（如百万级用户信息）
// ✅ 按业务拆分，每个用户独立存储
public void saveUserInfo(User user) {
    String key = "user:info:" + user.getId();
    Map<String, String> map = convertToMap(user);
    redisTemplate.opsForHash().putAll(key, map);
    redisTemplate.expire(key, 30, TimeUnit.DAYS);
}

// 5. 避免热 Key 问题
// 热 Key 分片存储
public void saveHotData(String baseKey, String data) {
    // 使用随机后缀分片
    int shard = Math.abs(data.hashCode()) % 10;
    String key = baseKey + ":" + shard;
    redisTemplate.opsForValue().set(key, data, 1, TimeUnit.HOURS);
}
```

#### 监控与告警

```yaml
# Prometheus Redis 监控配置
# prometheus.yml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['redis:9121']
    metrics_path: /metrics
    params:
      auth: ['your_password']
```

```bash
# Redis 监控指标
# 内存使用
INFO memory
used_memory_human  # 已用内存
maxmemory_human    # 最大内存限制
mem_fragmentation_ratio  # 内存碎片率

# 连接信息
INFO clients
connected_clients  # 已连接客户端
blocked_clients    # 阻塞客户端

# 性能指标
INFO stats
total_commands_processed  # 总命令数
instantaneous_ops_per_sec  # 每秒操作数
keyspace_hits     # 缓存命中
keyspace_misses   # 缓存未命中
hit_rate = hits / (hits + misses)  # 命中率

# 慢查询
SLOWLOG GET 10     # 最近 10 条慢查询
SLOWLOG LEN        # 慢查询数量
```

### 11.7 数据一致性

#### 一致性场景

```
场景一：缓存与数据库一致性
┌─────────┐  读  ┌──────┐  读  ┌──────┐
│  客户端  │────►│ Redis │────►│ MySQL │
└─────────┘     └──────┘     └──────┘
  写入时：先写数据库，再删缓存（Cache-Aside 模式）

场景二：分布式事务一致性
┌─────────┐  写  ┌──────┐  写  ┌──────┐
│ 订单服务 │────►│ Redis │────►│ MySQL │
└─────────┘     └──────┘     └──────┘
  需要保证两边数据一致
```

#### 一致性解决方案

**方案一：Cache-Aside 模式（推荐）**

```java
// 最常用的缓存模式
// 读：先查缓存，缓存未命中再查数据库，回填缓存
// 写：先写数据库，再删除缓存（不是更新缓存）

@Service
public class CacheAsideService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    private final ProductMapper productMapper;
    
    // 读操作
    public Product getProduct(Long id) {
        String key = "product:" + id;
        
        // 1. 查缓存
        Product product = (Product) redisTemplate.opsForValue().get(key);
        if (product != null) {
            return product;
        }
        
        // 2. 查数据库
        product = productMapper.selectById(id);
        if (product != null) {
            // 3. 回填缓存
            redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
        }
        return product;
    }
    
    // 写操作
    @Transactional
    public void updateProduct(Product product) {
        // 1. 先写数据库
        productMapper.updateById(product);
        
        // 2. 再删缓存（而非更新缓存）
        String key = "product:" + product.getId();
        redisTemplate.delete(key);
    }
}
```

**方案二：Canal + MySQL Binlog**

```java
// 使用 Canal 监听 MySQL binlog，自动刷新缓存
@Component
public class CanalCacheUpdater {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // Canal 监听商品表变更
    @CanalListener(table = "product")
    public void onProductChange(CanalMessage message) {
        switch (message.getType()) {
            case INSERT:
            case UPDATE:
                // 插入或更新时，刷新缓存
                Long id = message.getData().get("id");
                Product product = productMapper.selectById(id);
                String key = "product:" + id;
                redisTemplate.opsForValue().set(key, product, 30, TimeUnit.MINUTES);
                break;
                
            case DELETE:
                // 删除时，清除缓存
                Long id = message.getData().get("id");
                String key = "product:" + id;
                redisTemplate.delete(key);
                break;
        }
    }
}
```

**方案三：基于消息队列的最终一致性**

```java
// 使用 RocketMQ/Kafka 发送消息保证最终一致性
@Service
public class MqCacheService {
    
    private final RocketMQTemplate mqTemplate;
    private final RedisTemplate<String, Object> redisTemplate;
    
    // 写操作：先写数据库，再发消息
    @Transactional
    public void updateProductWithMq(Product product) {
        // 1. 写数据库
        productMapper.updateById(product);
        
        // 2. 发送缓存更新消息
        String msg = JSON.toJSONString(new CacheUpdateMessage(
            "product", product.getId(), "UPDATE"
        ));
        mqTemplate.convertAndSend("CACHE_UPDATE_TOPIC", msg);
    }
    
    // 消息消费者
    @RocketMQMessageListener(topic = "CACHE_UPDATE_TOPIC", consumerGroup = "cache-group")
    @Component
    public class CacheUpdateConsumer implements RocketMQListener<String> {
        
        @Override
        public void onMessage(String message) {
            CacheUpdateMessage msg = JSON.parseObject(message, CacheUpdateMessage.class);
            
            // 处理缓存更新
            switch (msg.getType()) {
                case "UPDATE":
                case "INSERT":
                    Object data = getDataFromDB(msg.getTableName(), msg.getDataId());
                    String key = msg.getTableName() + ":" + msg.getDataId();
                    redisTemplate.opsForValue().set(key, data, 30, TimeUnit.MINUTES);
                    break;
                    
                case "DELETE":
                    String deleteKey = msg.getTableName() + ":" + msg.getDataId();
                    redisTemplate.delete(deleteKey);
                    break;
            }
        }
    }
}
```

#### 一致性方案对比

| 方案 | 一致性 | 性能 | 复杂度 | 适用场景 |
|------|--------|------|--------|----------|
| Cache-Aside | 最终一致 | 高 | 低 | 绝大多数场景 |
| Canal + Binlog | 准实时一致 | 高 | 中 | 对一致性要求高 |
| MQ 消息队列 | 最终一致 | 中 | 高 | 复杂分布式系统 |
| 双写 | 强一致 | 低 | 高 | 不推荐 |
| 先更新缓存 | 强一致 | 高 | 低 | 不推荐（易脏读） |

**最佳实践总结**：

1. **Cache-Aside 模式**：先写库后删缓存，最常用方案
2. **设置过期时间**：保证缓存最终会被刷新
3. **异步刷新**：使用 MQ 或 Canal 实现准实时更新
4. **重试机制**：缓存操作失败时自动重试
5. **监控告警**：监控缓存命中率和一致性
6. **兜底方案**：缓存全部未命中时直接查数据库

---

## 附录：Redis 面试高频问题

### 基础篇

1. Redis 为什么这么快？
   - 纯内存操作、IO 多路复用、单线程模型

2. Redis 支持哪些数据类型？
   - String、List、Set、Hash、ZSet、Bitmaps、HyperLogLog、Geospatial、Stream

3. Redis 过期键删除策略？
   - 惰性删除 + 定期删除 + 内存淘汰策略

4. 什么是缓存穿透、击穿、雪崩？如何解决？

### 进阶篇

5. Redis 持久化方式？RDB 和 AOF 区别？

6. Redis 事务实现原理？为什么不支持回滚？

7. 主从复制原理？全量同步和增量同步区别？

8. 哨兵工作原理？如何判断主节点故障？

9. Cluster 模式如何分片？Hash Slot 计算方式？

10. 如何实现分布式锁？Redisson 原理？

### 高级篇

11. Redis 内存满了怎么办？内存淘汰策略？

12. 如何保证 Redis 与数据库的数据一致性？

13. Redis 集群如何扩容？如何处理数据迁移？

14. 如何优化慢查询？常见性能问题？

15. Redis 脑裂是什么？如何解决？
