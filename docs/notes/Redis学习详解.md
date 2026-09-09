# Redis 学习详解（从零开始，白话讲解）

> 本文档目标：让完全不懂的人也能看懂 Redis，学会安装、操作、持久化、主从复制、哨兵高可用。
> 写作风格：生活比喻 + 逐行解释 + 对比表格 + 记忆口诀，不怕长，就怕你看不懂。

---

## 目录

- [第一章：Redis 是什么？](#第一章redis-是什么)
- [第二章：安装 Redis](#第二章安装-redis)
- [第三章：基本操作命令](#第三章基本操作命令)
- [第四章：五种数据类型](#第四章五种数据类型)
- [第五章：持久化（把数据存到硬盘）](#第五章持久化把数据存到硬盘)
- [第六章：主从复制](#第六章主从复制)
- [第七章：哨兵 Sentinel 高可用](#第七章哨兵-sentinel-高可用)
- [第八章：知识总结与面试速查](#第八章知识总结与面试速查)

---

# 第一章：Redis 是什么？

## 1.1 一句话解释

Redis 是一个**存在内存里的数据库**，读写速度极快。

> **生活比喻**：
> - MySQL 就像一个**大仓库**，东西多、容量大，但每次找东西要去仓库翻，比较慢。
> - Redis 就像你**桌子上的口袋**，容量小，但东西放在手边，拿取飞快。
>
> 所以实际工作中，经常把 Redis 和 MySQL 配合使用：
> - 经常要用的数据 → 放 Redis（快）
> - 不常用但要永久保存的数据 → 放 MySQL（稳）

## 1.2 Redis 和 MySQL 的区别

| 对比项 | MySQL | Redis |
|--------|-------|-------|
| 数据存哪里 | 硬盘 | 内存（也可以存硬盘） |
| 速度 | 慢一些（毫秒级） | 极快（微秒级） |
| 容量 | 大（几百GB甚至TB） | 小（受内存限制，通常几十GB） |
| 数据会不会丢 | 不会（存硬盘了） | 有可能（断电就没了，除非做持久化） |
| 数据类型 | 表格（行和列） | 5种特殊类型 |
| 用途 | 存重要数据 | 缓存、排行榜、计数器、消息队列 |

## 1.3 Redis 能干什么？

| 场景 | 举个例子 |
|------|---------|
| **缓存** | 网页上的热门文章，先从Redis读，没有再去MySQL查 |
| **排行榜** | 游戏里的战力排行榜，用Redis的有序集合实现 |
| **计数器** | 抖音视频的点赞数、微博转发数，用Redis实时计数 |
| **会话管理** | 用户登录后的Session信息，存在Redis里 |
| **消息队列** | 两个系统之间传消息，用Redis的列表类型实现 |

## 1.4 Redis 的特点（面试常问）

1. **快** —— 数据在内存里，读写速度是MySQL的10~100倍
2. **支持多种数据类型** —— 5种，比MySQL灵活
3. **支持持久化** —— 虽然在内存里，但可以存到硬盘防止丢失
4. **支持主从复制** —— 一主多从，数据自动同步
5. **支持哨兵高可用** —— 主库挂了自动切换从库上位
6. **单线程** —— Redis 6.0之前是单线程的，但依然很快（因为内存操作本身极快）

> **为什么单线程还这么快？**
> 因为它在内存里操作，不需要像MySQL那样读写硬盘。
> 就像你在家厨房做饭（内存）和去超市买菜（硬盘），当然家里快。
> 而且单线程避免了来回切换的消耗，反而更高效。

---

# 第二章：安装 Redis

## 2.1 环境说明

我们在三台虚拟机上操作：
- **hadoop101**（IP: 192.168.121.101）—— 主库
- **hadoop102**（IP: 192.168.121.102）—— 从库
- **hadoop103**（IP: 192.168.121.103）—— 备用

操作系统：CentOS 7

## 2.2 安装步骤

Redis 不在 CentOS 默认的软件源里，需要先装一个叫 **EPEL** 的额外软件源。

> **生活比喻**：
> EPEL 就像一个"额外货架"，CentOS 自带的货架上没有 Redis，加了 EPEL 货架就有了。

### 第 1 步：安装 EPEL 源

```bash
yum install -y epel-release
```

逐行解释：
- `yum install` —— 安装软件的命令
- `-y` —— 自动回答"yes"，不用手动确认
- `epel-release` —— EPEL 软件源的包名

### 第 2 步：安装 Redis

```bash
yum install -y redis
```

### 第 3 步：启动 Redis 并设置开机自启

```bash
systemctl start redis
systemctl enable redis
```

逐行解释：
- `systemctl start redis` —— 现在立刻启动 Redis
- `systemctl enable redis` —— 设置开机自动启动（下次重启虚拟机不用手动启动）

### 第 4 步：检查 Redis 是否正常

```bash
redis-cli ping
```

如果返回 `PONG`，说明 Redis 正常运行了。

> **生活比喻**：
> 你喊一声"喂？"（ping），Redis 回一句"在呢！"（PONG），说明它活着。

## 2.3 连接 Redis

```bash
redis-cli
```

输入后你会看到提示符变成：

```
127.0.0.1:6379>
```

这个提示符的含义：
- `127.0.0.1` —— 本机地址（你连的是本机的 Redis）
- `6379` —— Redis 默认端口号

> **什么是端口？**
> 一台电脑上可以跑很多服务，每个服务用一个"门牌号"来区分。
> Redis 的门牌号是 6379，MySQL 的是 3306，Nginx 的是 80。
> 就像一栋楼里，Redis 住6379号房间，MySQL 住3306号房间。

## 2.4 Redis 基本概念

| 概念 | 解释 | 生活比喻 |
|------|------|---------|
| 端口 | Redis监听的"门牌号" | 6379号房间 |
| 数据库编号 | Redis默认有16个数据库（0~15号） | 一栋楼有16层 |
| 默认数据库 | 不选择的话默认用0号 | 默认在1楼 |
| 选择数据库 | `SELECT 1` 切换到1号数据库 | 坐电梯去2楼 |
| key-value | Redis里所有数据都是"键-值"对 | 抽屉上贴标签（key），里面放东西（value） |

试一试：

```
SELECT 1      -- 切换到1号数据库
SELECT 0      -- 切回0号数据库（默认）
DBSIZE        -- 查看当前数据库有多少个key
```

---

# 第三章：基本操作命令

## 3.1 增删改查四兄弟

Redis 里最基本的操作就是存数据和取数据。

### SET —— 存数据（增/改）

```
SET 键名 值
```

示例：
```
SET name "zhangsan"
SET age 25
SET city "beijing"
```

解释：
- `SET name "zhangsan"` —— 把"zhangsan"这个值存到名为"name"的抽屉里
- 如果"name"这个抽屉之前有东西，会被覆盖（所以SET既能增也能改）

> **生活比喻**：SET 就像你往一个贴了标签的抽屉里放东西。如果抽屉里已经有东西，就被新的替换掉。

### GET —— 取数据（查）

```
GET 键名
```

示例：
```
GET name
```

返回：`"zhangsan"`

如果取一个不存在的key，返回 `(nil)`，意思是"空的、不存在"。

> **生活比喻**：GET 就像你看抽屉标签，然后打开抽屉拿东西。如果这个标签的抽屉不存在，就什么也拿不到。

### DEL —— 删数据（删）

```
DEL 键名
```

示例：
```
DEL name
```

返回：`(integer) 1` —— 表示删掉了1个

> **生活比喻**：DEL 就像把整个抽屉连同里面的东西一起扔掉。

### 修改数据

Redis 里没有专门的"修改"命令，直接用 SET 覆盖就行：

```
SET name "zhangsan"    -- 先存
SET name "lisi"        -- 再覆盖，name就变成lisi了
```

## 3.2 其他常用命令

### KEYS —— 查看所有键

```
KEYS *           -- 查看所有键
KEYS na*         -- 查看以na开头的键
KEYS *e          -- 查看以e结尾的键
```

> **注意**：在生产环境（真正工作的服务器）上不要用 `KEYS *`，因为它会扫描所有数据，数据量大时会让Redis卡住。但在学习阶段可以随便用。

### EXISTS —— 检查键是否存在

```
EXISTS name
```

返回 `1` 表示存在，返回 `0` 表示不存在。

### EXPIRE —— 设置过期时间

```
EXPIRE name 10
```

意思是：name 这个键 10秒后自动删除。

> **生活比喻**：就像快递柜的存放时限，超过时间自动清空。
> 这在缓存场景特别有用：比如验证码5分钟后自动失效。

### TTL —— 查看还剩多久过期

```
TTL name
```

返回值含义：
- 正数 —— 还剩多少秒
- `-1` —— 没有设置过期时间（永久存在）
- `-2` —— 这个键已经不存在了

### TYPE —— 查看键的数据类型

```
TYPE name
```

返回这个键存的是什么类型的数据（string、hash、list、set、zset）。

### FLUSHDB —— 清空当前数据库

```
FLUSHDB
```

删掉当前数据库里的所有数据（慎用！）

### FLUSHALL —— 清空所有数据库

```
FLUSHALL
```

删掉所有16个数据库里的所有数据（更慎用！）

## 3.3 基本命令速查表

| 命令 | 作用 | 示例 | 记忆口诀 |
|------|------|------|---------|
| SET | 存数据 | `SET name "zhangsan"` | **S**et = **S**tore（存） |
| GET | 取数据 | `GET name` | **G**et = **G**rab（拿） |
| DEL | 删数据 | `DEL name` | **D**el = **D**elete（删） |
| KEYS | 查所有键 | `KEYS *` | Keys = 钥匙（看有哪些抽屉） |
| EXISTS | 是否存在 | `EXISTS name` | Exists = 存在 |
| EXPIRE | 设过期 | `EXPIRE name 10` | Expire = 过期 |
| TTL | 剩余时间 | `TTL name` | TTL = Time To Live（还能活多久） |
| TYPE | 看类型 | `TYPE name` | Type = 类型 |
| FLUSHDB | 清当前库 | `FLUSHDB` | Flush = 冲水（冲走） |
| FLUSHALL | 清所有库 | `FLUSHALL` | All = 全部 |

---

# 第四章：五种数据类型

Redis 有5种数据类型，每种类型有不同的用途和命令。

> **生活比喻**：
> 就像厨房里有5种不同的容器：
> 1. **String** —— 杯子（装一个东西）
> 2. **Hash** —— 多格饭盒（一格装一样菜）
> 3. **List** —— 管道（两头都能进出）
> 4. **Set** —— 篮子（装一堆不重复的东西）
> 5. **Sorted Set** —— 排名表（每样东西带个分数，自动排序）

## 4.1 String（字符串）

### 什么是 String？

最简单的类型，一个键对应一个值。值可以是文字、数字、甚至图片（二进制）。

> **生活比喻**：就像一个杯子，里面只能装一种饮料。

### 常用命令

#### SET / GET（存和取）

```
SET username "admin"
GET username
```

#### INCR / DECR（加1 / 减1）

```
SET counter 10
INCR counter      -- counter变成11
INCR counter      -- counter变成12
DECR counter      -- counter变成11
```

> **生活比喻**：就像计数器，按一下加1（INCR），再按一下减1（DECR）。
> 应用场景：点赞数、阅读量、库存数量。

#### INCRBY / DECRBY（加N / 减N）

```
SET counter 10
INCRBY counter 5    -- counter变成15
DECRBY counter 3    -- counter变成12
```

#### APPEND（追加文字）

```
SET msg "hello"
APPEND msg " world"   -- msg变成"hello world"
```

#### STRLEN（查看字符串长度）

```
SET msg "hello"
STRLEN msg            -- 返回5（5个字符）
```

#### MSET / MGET（批量存 / 批量取）

```
MSET name "zhangsan" age 25 city "beijing"
MGET name age city
```

> **为什么要批量？**
> 一次设置/获取多个值，减少网络往返次数，效率更高。
> 就像去超市一次买齐所有东西，比跑三趟分别买省时间。

### String 应用场景

| 场景 | 怎么用 |
|------|--------|
| 缓存 | `SET article:1 "文章内容"` |
| 计数器 | `INCR page:view:home` |
| 分布式锁 | `SET lock "uuid" NX EX 30` |
| 存Session | `SET session:abc123 "用户信息"` |

---

## 4.2 Hash（哈希）

### 什么是 Hash？

一个键下面可以存多个"字段-值"对。

> **生活比喻**：
> - String 是一个杯子，只能装一种饮料
> - Hash 是一个**多格饭盒**，每个格子装一样菜
>
> 举例：
> ```
> user:1 这个Hash里面：
>   name → "zhangsan"
>   age  → 25
>   city → "beijing"
> ```
> 就像一个人的档案袋里有姓名、年龄、城市等信息。

### 为什么不用 String 代替？

你当然可以用三个 String 来存：
```
SET user:1:name "zhangsan"
SET user:1:age 25
SET user:1:city "beijing"
```

但用 Hash 更好：
- 管理方便：一次删掉整个用户用 `DEL user:1`，不用删三个
- 节省内存：Hash 结构在数据少时非常省内存
- 语义清晰：`user:1` 就是一个完整对象的集合

### 常用命令

#### HSET（存一个字段）

```
HSET user:1 name "zhangsan"
HSET user:1 age 25
HSET user:1 city "beijing"
```

解释：在 `user:1` 这个Hash里，设置字段 `name` 的值为 `zhangsan`。

#### HGET（取一个字段）

```
HGET user:1 name
```

返回：`"zhangsan"`

#### HGETALL（取所有字段和值）

```
HGETALL user:1
```

返回：
```
name
zhangsan
age
25
city
beijing
```

#### HMSET（批量存多个字段）

```
HMSET user:2 name "lisi" age 30 city "shanghai"
```

#### HMGET（批量取多个字段）

```
HMGET user:1 name age
```

返回：
```
zhangsan
25
```

#### HDEL（删一个字段）

```
HDEL user:1 city
```

删掉 user:1 里的 city 字段（其他字段还在）。

#### HLEN（看有几个字段）

```
HLEN user:1
```

返回字段数量。

#### HEXISTS（检查字段是否存在）

```
HEXISTS user:1 name
```

返回 `1` 表示存在，`0` 表示不存在。

#### HKEYS / HVALS（只看字段名 / 只看值）

```
HKEYS user:1     -- 返回所有字段名：name, age, city
HVALS user:1     -- 返回所有值：zhangsan, 25, beijing
```

#### HINCRBY（给某个字段的数字值加N）

```
HINCRBY user:1 age 1    -- age加1，变成26
```

### Hash 应用场景

| 场景 | 怎么用 |
|------|--------|
| 存用户信息 | `HSET user:1 name "zhangsan" age 25` |
| 存商品信息 | `HSET product:100 name "手机" price 2999` |
| 购物车 | `HSET cart:user1 product:100 2`（买了2个） |

### Hash 命令速查表

| 命令 | 作用 | 示例 |
|------|------|------|
| HSET | 存一个字段 | `HSET user:1 name "zhangsan"` |
| HGET | 取一个字段 | `HGET user:1 name` |
| HGETALL | 取所有字段 | `HGETALL user:1` |
| HMSET | 批量存 | `HMSET user:1 name "zhangsan" age 25` |
| HMGET | 批量取 | `HMGET user:1 name age` |
| HDEL | 删字段 | `HDEL user:1 city` |
| HLEN | 字段数量 | `HLEN user:1` |
| HEXISTS | 字段是否存在 | `HEXISTS user:1 name` |
| HKEYS | 所有字段名 | `HKEYS user:1` |
| HVALS | 所有值 | `HVALS user:1` |
| HINCRBY | 数字加N | `HINCRBY user:1 age 1` |

> **记忆技巧**：Hash 的命令都以 **H** 开头（H = Hash），后面跟操作名：
> - H + SET = HSET
> - H + GET = HGET
> - H + GET + ALL = HGETALL
> - H + DEL = HDEL
> - H + LEN = HLEN

---

## 4.3 List（列表）

### 什么是 List？

一个键对应一个**有序的**字符串列表，可以从两头插入和删除。

> **生活比喻**：
> List 就像一根**管子**，两头都开口：
> - 从左边塞东西进去（LPUSH）
> - 从右边塞东西进去（RPUSH）
> - 从左边拿东西出来（LPOP）
> - 从右边拿东西出来（RPOP）
>
> 先放进去的在中间，后放进去的在两端。

### 常用命令

#### LPUSH / RPUSH（从左/右插入）

```
LPUSH mylist "a"     -- 列表：a
LPUSH mylist "b"     -- 列表：b a（b从左边塞进来，排第一）
RPUSH mylist "c"     -- 列表：b a c（c从右边塞进来，排最后）
```

> **L = Left（左边），R = Right（右边）**

#### LRANGE（查看范围）

```
LRANGE mylist 0 -1    -- 查看所有元素
LRANGE mylist 0 2     -- 查看第1到第3个元素
```

解释：
- `0` —— 从第一个开始（Redis的编号从0开始）
- `-1` —— 到最后一个（-1代表倒数第一个，-2代表倒数第二个）

#### LPOP / RPOP（从左/右弹出）

```
LPOP mylist    -- 弹出左边的第一个元素（b），列表变成 a c
RPOP mylist    -- 弹出右边的最后一个元素（c），列表变成 a
```

> "弹出" = 取出来 + 从列表中删除。就像抽签，抽走一根就少一根。

#### LLEN（查看长度）

```
LLEN mylist    -- 返回列表里有多少个元素
```

#### LINDEX（按位置取值，不删除）

```
LINDEX mylist 0    -- 取第1个元素（不删除）
LINDEX mylist 1    -- 取第2个元素（不删除）
```

### List 应用场景

| 场景 | 怎么用 |
|------|--------|
| 消息队列 | `LPUSH msgs "消息1"`，`RPOP msgs`（先进先出） |
| 最新消息列表 | `LPUSH timeline "新消息"`，`LRANGE timeline 0 9`（取最新10条） |
| 操作日志 | `RPUSH logs "操作1"` |

> **消息队列原理**：
> 生产者从左边塞消息：`LPUSH queue "任务"`
> 消费者从右边取消息：`RPOP queue`
> 这样先塞进去的先被取出来（先进先出，FIFO）
> 就像排队买饭，先来的先买到。

### List 命令速查表

| 命令 | 作用 | 示例 |
|------|------|------|
| LPUSH | 左边插入 | `LPUSH mylist "a"` |
| RPUSH | 右边插入 | `RPUSH mylist "a"` |
| LPOP | 左边弹出 | `LPOP mylist` |
| RPOP | 右边弹出 | `RPOP mylist` |
| LRANGE | 查看范围 | `LRANGE mylist 0 -1` |
| LLEN | 查看长度 | `LLEN mylist` |
| LINDEX | 按位置取 | `LINDEX mylist 0` |

> **记忆技巧**：List 的命令以 **L** 或 **R** 开头：
> - L = Left（左边）
> - R = Right（右边）
> - LPUSH = Left + Push（左边推进去）
> - RPOP = Right + Pop（右边弹出来）

---

## 4.4 Set（集合）

### 什么是 Set？

一个键对应一个**不重复的**字符串集合，没有顺序。

> **生活比喻**：
> Set 就像一个**篮子**，里面装一堆东西，每样东西只能装一个（不能重复）。
> 你不在乎顺序，只在乎"有没有"。

### 和 List 的区别

| 对比项 | List | Set |
|--------|------|-----|
| 有没有顺序 | 有顺序 | 没顺序 |
| 能不能重复 | 能重复 | 不能重复 |
| 生活比喻 | 管道（有头有尾） | 篮子（一扔进去就行） |

### 常用命令

#### SADD（添加元素）

```
SADD myset "apple"
SADD myset "banana"
SADD myset "apple"    -- 重复添加，不会成功（集合不允许重复）
```

#### SMEMBERS（查看所有元素）

```
SMEMBERS myset
```

返回：
```
apple
banana
```

#### SISMEMBER（判断元素是否存在）

```
SISMEMBER myset "apple"    -- 返回1，存在
SISMEMBER myset "grape"    -- 返回0，不存在
```

#### SREM（删除元素）

```
SREM myset "banana"    -- 删掉banana
```

#### SCARD（查看元素个数）

```
SCARD myset    -- 返回集合里有几个元素
```

#### 集合运算（Set 的杀手锏）

这是 Set 最强大的功能：

```
-- 两个集合
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"

-- 交集（两个集合都有的）
SINTER set1 set2       -- 返回 b c

-- 并集（两个集合合在一起，去掉重复的）
SUNION set1 set2       -- 返回 a b c d

-- 差集（set1有但set2没有的）
SDIFF set1 set2        -- 返回 a
```

> **生活比喻**：
> - 交集 = 两个人共同认识的朋友
> - 并集 = 两个人认识的所有朋友（去重）
> - 差集 = 我认识但你认识的朋友

### Set 应用场景

| 场景 | 怎么用 |
|------|--------|
| 标签 | `SADD article:1:tags "技术" "Redis" "运维"` |
| 共同关注 | `SINTER user:1:follows user:2:follows`（共同关注的人） |
| 抽奖 | `SPOP lottery 1`（随机弹出1个人） |
| 去重 | 把数据放Set里，自动去重 |

### Set 命令速查表

| 命令 | 作用 | 示例 |
|------|------|------|
| SADD | 添加元素 | `SADD myset "a"` |
| SMEMBERS | 查看所有 | `SMEMBERS myset` |
| SISMEMBER | 是否存在 | `SISMEMBER myset "a"` |
| SREM | 删除元素 | `SREM myset "a"` |
| SCARD | 元素个数 | `SCARD myset` |
| SINTER | 交集 | `SINTER set1 set2` |
| SUNION | 并集 | `SUNION set1 set2` |
| SDIFF | 差集 | `SDIFF set1 set2` |

> **记忆技巧**：Set 的命令以 **S** 开头（S = Set）：
> - S + ADD = SADD
> - S + MEMBERS = SMEMBERS
> - S + INTERSECTION = SINTER（交集）
> - S + UNION = SUNION（并集）
> - S + DIFFERENCE = SDIFF（差集）

---

## 4.5 Sorted Set（有序集合，简称 Zset）

### 什么是 Sorted Set？

和 Set 一样不允许重复，但**每个元素带一个分数（score）**，Redis 会根据分数自动排序。

> **生活比喻**：
> Sorted Set 就像**考试排名表**：
> - 每个学生名字不重复（和Set一样）
> - 每个学生有一个分数
> - 自动按分数从低到高（或从高到低）排列
>
> 这就是为什么它最适合做排行榜！

### 常用命令

#### ZADD（添加元素，带分数）

```
ZADD ranking 100 "zhangsan"     -- zhangsan考了100分
ZADD ranking 90 "lisi"          -- lisi考了90分
ZADD ranking 85 "wangwu"        -- wangwu考了85分
```

#### ZRANGE（从小到大查看）

```
ZRANGE ranking 0 -1    -- 从小到大显示所有
```

返回：
```
wangwu
lisi
zhangsan
```

#### ZRANGE 带分数显示

```
ZRANGE ranking 0 -1 WITHSCORES
```

返回：
```
wangwu
85
lisi
90
zhangsan
100
```

#### ZREVRANGE（从大到小查看）

```
ZREVRANGE ranking 0 -1    -- 从大到小显示所有
```

返回：
```
zhangsan
lisi
wangwu
```

> **排行榜就是用 ZREVRANGE**，从第一名开始往下排。

#### ZSCORE（查看某个元素的分数）

```
ZSCORE ranking "zhangsan"
```

返回：`"100"`

#### ZRANK（查看排名，从小到大）

```
ZRANK ranking "zhangsan"
```

返回：`2`（第3名，因为从0开始数：wangwu=0, lisi=1, zhangsan=2）

#### ZREVRANK（查看排名，从大到小）

```
ZREVRANK ranking "zhangsan"
```

返回：`0`（第1名！因为从高分到低分排）

> **ZRANK vs ZREVRANK 记忆**：
> - ZRANK = 从低到高排（分数最低的是第0名）
> - ZREVRANK = 从高到低排（分数最高的是第0名）
> - R = Reverse（反转）
> - 做排行榜用 ZREVRANK（高分在前）

#### ZINCRBY（给某个元素加分数）

```
ZINCRBY ranking 5 "lisi"    -- lisi的分数加5，从90变成95
```

#### ZREM（删除元素）

```
ZREM ranking "wangwu"    -- 删掉wangwu
```

#### ZCARD（查看元素个数）

```
ZCARD ranking    -- 返回有几个元素
```

### Sorted Set 应用场景

| 场景 | 怎么用 |
|------|--------|
| 游戏排行榜 | `ZADD rank 9999 "玩家1"`，`ZREVRANGE rank 0 9`（前10名） |
| 热搜榜 | `ZINCRBY hot 1 "关键词"`（每次搜索加1分） |
| 延迟队列 | 分数用时间戳，到时间了就取出来执行 |
| 班级成绩排名 | `ZADD class 95 "张三" 88 "李四" 76 "王五"` |

### Sorted Set 命令速查表

| 命令 | 作用 | 示例 |
|------|------|------|
| ZADD | 添加（带分数） | `ZADD rank 100 "zhangsan"` |
| ZRANGE | 从小到大 | `ZRANGE rank 0 -1` |
| ZREVRANGE | 从大到小 | `ZREVRANGE rank 0 -1` |
| ZSCORE | 查分数 | `ZSCORE rank "zhangsan"` |
| ZRANK | 排名（从小到大） | `ZRANK rank "zhangsan"` |
| ZREVRANK | 排名（从大到小） | `ZREVRANK rank "zhangsan"` |
| ZINCRBY | 加分 | `ZINCRBY rank 5 "zhangsan"` |
| ZREM | 删除 | `ZREM rank "zhangsan"` |
| ZCARD | 元素个数 | `ZCARD rank` |

> **记忆技巧**：Sorted Set 的命令以 **Z** 开头（Z = Zset）：
> - Z + ADD = ZADD
> - Z + RANGE = ZRANGE（从小到大）
> - Z + REV + RANGE = ZREVRANGE（从大到小，REV=反转）
> - Z + SCORE = ZSCORE
> - Z + INCR + BY = ZINCRBY

---

## 4.6 五种数据类型大对比

| 类型 | 名字 | 特点 | 生活比喻 | 典型场景 | 命令前缀 |
|------|------|------|---------|---------|---------|
| String | 字符串 | 一对一 | 杯子 | 缓存、计数 | 无前缀 |
| Hash | 哈希 | 一对多（字段-值） | 多格饭盒 | 存对象 | H |
| List | 列表 | 有序、可重复 | 管道 | 消息队列 | L/R |
| Set | 集合 | 无序、不重复 | 篮子 | 去重、交集 | S |
| Zset | 有序集合 | 不重复、带分数 | 排名表 | 排行榜 | Z |

> **终极记忆口诀**：
> - **String 没前缀**（最基础，直接用）
> - **Hash 是 H**（H = Hash 多格盒）
> - **List 是 L/R**（L = Left，R = Right 两头进出）
> - **Set 是 S**（S = Set 篮子装东西）
> - **Zset 是 Z**（Z = 排行榜，Z是最后一个字母，代表"终极排名"）

---

# 第五章：持久化（把数据存到硬盘）

## 5.1 为什么需要持久化？

Redis 的数据存在**内存**里，内存的特点是**一断电就清空**。

> **生活比喻**：
> 就像你在白板上写笔记，写得很快，但一擦就没了。
> 持久化就是"把白板上的内容拍照/抄到笔记本上"，这样即使白板被擦了，笔记本上还有。

Redis 有两种持久化方式：

| 方式 | 名字 | 原理 | 生活比喻 |
|------|------|------|---------|
| RDB | 快照 | 定时把所有数据拍一张"照片"存到硬盘 | 拍照存档 |
| AOF | 日志 | 把每一条写操作记到日志文件里 | 写日记 |

## 5.2 RDB（快照模式）

### 原理

RDB 就是把 Redis 当前内存里的所有数据，打包成一个文件（dump.rdb）存到硬盘上。

> **生活比喻**：
> 就像给你家拍一张全家福照片。拍照的那一瞬间，所有人都在里面。
> 如果家被烧了（断电），拿照片可以重建一个一样的家。

### 触发 RDB 的方式

#### 方式1：手动执行 SAVE（会阻塞）

```
SAVE
```

执行后 Redis 会**卡住**（不能处理其他命令），直到保存完成。

> **生活比喻**：就像你停下所有工作去整理文件，期间不接电话、不回消息。

#### 方式2：手动执行 BGSAVE（不会阻塞）

```
BGSAVE
```

BG = Background（后台）。Redis 会**fork出一个子进程**去保存数据，主进程继续工作。

> **生活比喻**：你雇了个助手去整理文件，你自己继续干活，互不影响。

#### 方式3：自动触发（配置文件里设置）

在 `/etc/redis.conf` 里有这样的配置：

```
save 900 1     -- 900秒（15分钟）内有1个key变化，就自动保存
save 300 10    -- 300秒（5分钟）内有10个key变化，就自动保存
save 60 10000  -- 60秒内有10000个key变化，就自动保存
```

满足任何一个条件就会触发自动BGSAVE。

### RDB 文件在哪？

默认保存在 `/var/lib/redis/dump.rdb`。

### RDB 的优缺点

| 优点 | 缺点 |
|------|------|
| 文件小，恢复速度快 | 可能丢失最近几分钟的数据 |
| 适合做备份 | SAVE会阻塞（一般用BGSAVE避免） |
| 对性能影响小 | 不能实时保存 |

## 5.3 AOF（日志模式）

### 原理

AOF 就是把 Redis 执行的**每一条写命令**（SET、DEL、INCR等）都追加记录到一个文件里。

> **生活比喻**：
> 就像写日记，你做了什么就记什么：
> ```
> 9:00 SET name "zhangsan"
> 9:05 SET age 25
> 9:10 DEL name
> ```
> 如果Redis重启了，把日记从头到尾读一遍重做，数据就恢复了。

### 开启 AOF

在 `/etc/redis.conf` 中找到：

```
appendonly yes
```

把 `no` 改成 `yes`，然后重启 Redis：

```bash
systemctl restart redis
```

### AOF 文件在哪？

默认保存在 `/var/lib/redis/appendonly.aof`。

### AOF 的三种刷盘策略

在配置文件中有这一行：

```
appendfsync everysec
```

有三种选择：

| 策略 | 含义 | 丢数据风险 | 性能 |
|------|------|-----------|------|
| `always` | 每条命令都立刻写硬盘 | 不丢 | 最慢 |
| `everysec` | 每秒写一次硬盘 | 最多丢1秒 | 折中（推荐） |
| `no` | 让操作系统决定什么时候写 | 可能丢几秒 | 最快 |

> **生活比喻**：
> - `always` = 每说一句话就写下来（最安全但最累）
> - `everysec` = 每分钟整理一次笔记（折中，推荐）
> - `no` = 随心情写（最快但可能漏）

### AOF 的优缺点

| 优点 | 缺点 |
|------|------|
| 数据安全性高，最多丢1秒 | 文件比RDB大 |
| 可读性好（就是命令文本） | 恢复速度比RDB慢 |
| 可以做灾难恢复 | 写入对性能有影响 |

## 5.4 RDB vs AOF 大对比

| 对比项 | RDB | AOF |
|--------|-----|-----|
| 原理 | 拍快照 | 写日志 |
| 文件 | dump.rdb | appendonly.aof |
| 文件大小 | 小 | 大 |
| 恢复速度 | 快 | 慢 |
| 数据安全性 | 可能丢几分钟 | 最多丢1秒 |
| 生活比喻 | 拍照 | 写日记 |
| 推荐场景 | 备份、灾难恢复 | 日常运行 |

> **实际工作中**：通常 RDB 和 AOF **同时开启**，互为补充。
> - AOF 保日常数据安全
> - RDB 做定期备份

---

# 第六章：主从复制

## 6.1 什么是主从复制？

一台 Redis 当**主库（master）**，其他 Redis 当**从库（slave）**。主库写的数据会自动同步到从库。

> **生活比喻**：
> 就像老板（主库）写文件，秘书（从库）自动复印一份。
> 老板写什么，秘书就有什么。
> 但秘书不能自己写文件，只能看（从库只能读不能写）。

## 6.2 为什么要主从复制？

| 好处 | 解释 |
|------|------|
| **数据备份** | 主库挂了，从库还有数据 |
| **读写分离** | 写数据走主库，读数据走从库，分担压力 |
| **高可用基础** | 配合哨兵，主库挂了从库可以顶上 |

## 6.3 架构图

```
┌──────────────┐
│  hadoop101   │
│  (主库master) │ ←── 写数据
│  192.168.121.101 │
└──────┬───────┘
       │ 自动同步数据
       ▼
┌──────────────┐
│  hadoop102   │
│  (从库slave)  │ ←── 只能读数据
│  192.168.121.102 │
└──────────────┘
```

## 6.4 配置步骤

### 第 1 步：主库（hadoop101）配置

编辑 `/etc/redis.conf`：

```
bind 0.0.0.0              -- 允许其他机器连接
protected-mode no          -- 关闭保护模式
```

> **为什么要改 bind？**
> 默认 Redis 只允许本机（127.0.0.1）连接。从库要来同步数据，必须让从库能连进来。
> `0.0.0.0` 意思是"所有机器都能连"。

> **为什么要改 protected-mode？**
> 保护模式开启时，外部连接会被拒绝。关掉它才能让从库连进来。

重启 Redis：

```bash
systemctl restart redis
```

### 第 2 步：从库（hadoop102）配置

首先在 102 上安装 Redis（参考第二章）。

编辑 `/etc/redis.conf`：

```
bind 0.0.0.0
protected-mode no
slaveof 192.168.121.101 6379
```

逐行解释：
- `bind 0.0.0.0` —— 让哨兵能连进来检查状态
- `protected-mode no` —— 关闭保护模式
- `slaveof 192.168.121.101 6379` —— 告诉Redis："你的主库是192.168.121.101的6379端口"

重启 Redis：

```bash
systemctl restart redis
```

### 第 3 步：验证主从复制

#### 在从库（102）上验证

```bash
redis-cli
INFO replication
```

正确结果应该看到：
```
role:slave                          -- 角色：从库
master_host:192.168.121.101         -- 主库IP
master_port:6379                    -- 主库端口
master_link_status:up               -- 连接状态：正常
```

#### 在主库（101）上验证

```bash
redis-cli
INFO replication
```

正确结果应该看到：
```
role:master                         -- 角色：主库
connected_slaves:1                  -- 连了1个从库
slave0:ip=192.168.121.102,state=online  -- 从库信息
```

#### 数据同步测试

在主库（101）写数据：

```
SET test "hello from master"
```

在从库（102）读数据：

```
GET test
```

如果返回 `"hello from master"`，说明同步成功！

### 第 4 步：注意从库只能读不能写

在从库（102）上尝试写数据：

```
SET newkey "test"
```

会报错：`(error) READONLY You can't write against a read only slave.`

> 这是正常的！从库天生只能读不能写，就像秘书只能看老板的文件，不能自己改。

## 6.5 常见问题

### 问题1：master_link_status:down

**原因**：主库的 Redis 没有配置 `bind 0.0.0.0` 或 `protected-mode no`，从库连不进去。

**解决**：在主库的 `/etc/redis.conf` 里改好这两个设置，然后重启 Redis。

### 问题2：REPLICAOF 命令报错

**原因**：Redis 版本较老（低于5.0），不支持 `REPLICAOF` 命令。

**解决**：用 `SLAVEOF` 代替，功能完全一样。

### 问题3：主从同步延迟

**原因**：如果主库写入量很大，从库同步可能有延迟。

**解决**：这属于正常现象，网络带宽和从库性能都会影响延迟。

## 6.6 主从复制原理（面试常问）

```
主库 (master)                          从库 (slave)
┌──────────┐                        ┌──────────┐
│  Redis   │                        │  Redis   │
│          │  ① IO线程连接主库       │          │
│          │ ◄──────────────────── │  IO线程   │
│          │                        │          │
│  binlog  │  ② 发送新数据          │ relaylog │
│  (写日志) │ ────────────────────► │ (中继日志)│
│          │                        │          │
│          │                        │  SQL线程  │
│          │                        │  ③ 执行   │
└──────────┘                        └──────────┘
```

步骤解释：
1. 从库的 **IO线程** 连接主库，说"我要同步数据"
2. 主库把新的写操作发送给从库，从库存在 **relaylog（中继日志）** 里
3. 从库的 **SQL线程** 读取中继日志，把操作执行一遍，数据就同步了

> **生活比喻**：
> - IO线程 = 秘书的耳朵，听老板说什么
> - relaylog = 秘书的笔记本，记下老板说的话
> - SQL线程 = 秘书的手，照着笔记本抄一份

---

# 第七章：哨兵 Sentinel 高可用

## 7.1 什么是哨兵？

哨兵（Sentinel）是一个**独立的监控进程**，专门盯着主库，主库挂了就自动把从库提升成新主库。

> **生活比喻**：
> 哨兵就像公司的**董事会**：
> - 平时盯着老板（主库）有没有正常工作
> - 老板突然倒了（主库宕机），董事会立刻提拔副手（从库）当新老板
> - 老老板病好回来（主库恢复），不会抢回位置，而是当副手

## 7.2 为什么要哨兵？

没有哨兵的情况：
```
主库挂了 → 从库还在，但还是从库（只能读）→ 没人能写数据 → 系统瘫痪
```

有了哨兵的情况：
```
主库挂了 → 哨兵发现 → 自动把从库提升为主库 → 系统继续工作
```

## 7.3 架构图

```
┌──────────────┐     ┌──────────────┐
│  hadoop101   │     │  hadoop102   │
│  Redis主库   │◄────│  Redis从库   │
│              │ 同步 │              │
└──────┬───────┘     └──────┬───────┘
       │                    │
       │ 监控                │ 监控
       ▼                    ▼
┌──────────────┐     ┌──────────────┐
│ Sentinel     │     │ Sentinel     │
│  (101上)     │     │  (102上)     │
│  端口26379   │     │  端口26379   │
└──────────────┘     └──────────────┘
```

## 7.4 配置步骤

### 第 1 步：修改哨兵配置文件

在 **101 和 102** 上都修改 `/etc/redis-sentinel.conf`：

```
sentinel monitor mymaster 192.168.121.101 6379 1
protected-mode no
```

逐行解释：
- `sentinel monitor mymaster 192.168.121.101 6379 1`
  - `sentinel monitor` —— 让哨兵监控一个主库
  - `mymaster` —— 给主库起个名字（随便取，但要一致）
  - `192.168.121.101 6379` —— 主库的IP和端口
  - `1` —— quorum值（多少个哨兵同意才能触发切换）
  - 我们只有2个哨兵，所以设为1（1个同意就行）

- `protected-mode no` —— 关闭保护模式，让哨兵之间能通信

> **什么是 quorum（法定数）？**
> 就像投票，需要多少人同意才能通过决议。
> 设为1就是"只要有1个哨兵说主库挂了，就立刻切换"。
> 如果有3个哨兵，通常设为2（多数同意更安全）。

### 第 2 步：启动哨兵服务

在 **101 和 102** 上都执行：

```bash
systemctl start redis-sentinel
systemctl enable redis-sentinel
```

### 第 3 步：验证哨兵

在 101 上：

```bash
redis-cli -p 26379
SENTINEL masters
```

> **注意**：哨兵运行在 **26379** 端口（不是6379），所以要用 `-p 26379` 连接。

正确结果应该看到：
```
"name" → "mymaster"
"ip" → "192.168.121.101"
"port" → "6379"
"flags" → "master"
```

## 7.5 故障切换测试

### 第 1 步：杀掉主库

在 hadoop101 上：

```bash
systemctl stop redis
```

### 第 2 步：等待 30 秒

> 哨兵配置了 `down-after-milliseconds 30000`（30秒）。
> 意思是：哨兵发现主库没响应后，等30秒再确认，避免因为网络抖动误判。
>
> **生活比喻**：老板没接电话，不会马上宣布离职，等30秒再打一次确认。

### 第 3 步：验证从库是否变成主库

在 hadoop102 上：

```bash
redis-cli
INFO replication
```

正确结果：
```
role:master          -- 从slave变成了master！
```

然后试试写数据：
```
SET test "hello new master"
```

返回 `OK` 说明2号机已经成功上位，可以读写了！

### 第 4 步：恢复原主库，验证自动变从库

在 hadoop101 上：

```bash
systemctl start redis
```

等一会儿，在 101 上检查：

```bash
redis-cli INFO replication
```

你会发现 101 变成了 `role:slave`，它的主库变成了 102！

> 这就是哨兵的"自动回归"功能：
> - 老老板（101）回来后不会抢位置
> - 而是自动给新老板（102）当副手
> - 一切都是哨兵自动安排的

## 7.6 常见问题

### 问题1：failover-abort-no-good-slave

**原因**：从库的 Redis 没有配置 `bind 0.0.0.0`，哨兵通过IP地址连不上从库，认为"没有好的从库可以提升"。

**解决**：在从库的 `/etc/redis.conf` 里设置 `bind 0.0.0.0` 和 `protected-mode no`，重启 Redis。

### 问题2：哨兵不触发切换

**原因**：
1. 哨兵服务没启动（检查 `systemctl status redis-sentinel`）
2. 等待时间不够（默认30秒）
3. quorum设置过高（只有2个哨兵但设了quorum=2）

**解决**：逐一排查以上原因。

### 问题3：哨兵之间无法通信

**原因**：`protected-mode` 没有关闭，或者防火墙阻止了26379端口。

**解决**：
1. 确认 `protected-mode no`
2. 开放防火墙：`firewall-cmd --add-port=26379/tcp --permanent && firewall-cmd --reload`

## 7.7 哨兵的工作原理（面试常问）

```
时间线：
─────────────────────────────────────────────────►
  ① 正常运行      ② 主库挂了    ③ 哨兵确认    ④ 选举    ⑤ 切换
  哨兵盯着主库    哨兵发现      等30秒确认    投票选    从库变
  一切正常        主库没响应    主库真挂了    领导者    主库
```

详细步骤：

1. **监控**：哨兵每隔1秒给主库发PING
2. **发现异常**：主库没回PING
3. **主观下线（s_down）**：单个哨兵认为主库挂了
4. **客观下线（o_down）**：足够多的哨兵（≥quorum）都认为主库挂了
5. **选举领导者**：哨兵们投票选一个来执行切换
6. **执行切换**：领导者把从库提升为主库
7. **通知**：通知其他哨兵和客户端新主库是谁

> **s_down vs o_down**：
> - s_down（主观下线）= 一个哨兵觉得主库挂了（可能只是它自己网络不好）
> - o_down（客观下线）= 多个哨兵都确认主库挂了（真挂了）
> - 就像一个老师觉得学生没来上课（s_down），问了一圈其他老师都说没看到他（o_down）

---

# 第八章：知识总结与面试速查

## 8.1 Redis 全部命令速查表

### 通用命令

| 命令 | 作用 |
|------|------|
| `SET key value` | 存数据 |
| `GET key` | 取数据 |
| `DEL key` | 删数据 |
| `KEYS *` | 查看所有键 |
| `EXISTS key` | 键是否存在 |
| `EXPIRE key 秒数` | 设置过期时间 |
| `TTL key` | 查看剩余过期时间 |
| `TYPE key` | 查看数据类型 |
| `FLUSHDB` | 清空当前数据库 |
| `FLUSHALL` | 清空所有数据库 |
| `SELECT 编号` | 切换数据库（0-15） |
| `DBSIZE` | 当前数据库key数量 |
| `INFO replication` | 查看主从复制状态 |
| `SAVE` | 手动RDB保存（阻塞） |
| `BGSAVE` | 后台RDB保存（不阻塞） |

### String 命令

| 命令 | 作用 |
|------|------|
| `SET key value` | 存 |
| `GET key` | 取 |
| `INCR key` | 加1 |
| `DECR key` | 减1 |
| `INCRBY key n` | 加n |
| `DECRBY key n` | 减n |
| `APPEND key value` | 追加 |
| `STRLEN key` | 长度 |
| `MSET k1 v1 k2 v2` | 批量存 |
| `MGET k1 k2` | 批量取 |

### Hash 命令

| 命令 | 作用 |
|------|------|
| `HSET key field value` | 存一个字段 |
| `HGET key field` | 取一个字段 |
| `HGETALL key` | 取所有字段 |
| `HMSET key f1 v1 f2 v2` | 批量存 |
| `HMGET key f1 f2` | 批量取 |
| `HDEL key field` | 删字段 |
| `HLEN key` | 字段数量 |
| `HEXISTS key field` | 字段是否存在 |
| `HKEYS key` | 所有字段名 |
| `HVALS key` | 所有值 |
| `HINCRBY key field n` | 字段值加n |

### List 命令

| 命令 | 作用 |
|------|------|
| `LPUSH key value` | 左边插入 |
| `RPUSH key value` | 右边插入 |
| `LPOP key` | 左边弹出 |
| `RPOP key` | 右边弹出 |
| `LRANGE key start stop` | 查看范围 |
| `LLEN key` | 长度 |
| `LINDEX key index` | 按位置取 |

### Set 命令

| 命令 | 作用 |
|------|------|
| `SADD key member` | 添加元素 |
| `SMEMBERS key` | 查看所有 |
| `SISMEMBER key member` | 是否存在 |
| `SREM key member` | 删除元素 |
| `SCARD key` | 元素个数 |
| `SINTER k1 k2` | 交集 |
| `SUNION k1 k2` | 并集 |
| `SDIFF k1 k2` | 差集 |

### Sorted Set 命令

| 命令 | 作用 |
|------|------|
| `ZADD key score member` | 添加（带分数） |
| `ZRANGE key 0 -1` | 从小到大 |
| `ZREVRANGE key 0 -1` | 从大到小 |
| `ZSCORE key member` | 查分数 |
| `ZRANK key member` | 排名（从小到大） |
| `ZREVRANK key member` | 排名（从大到小） |
| `ZINCRBY key n member` | 加分 |
| `ZREM key member` | 删除 |
| `ZCARD key` | 元素个数 |

### 主从复制命令

| 命令 | 作用 |
|------|------|
| `SLAVEOF host port` | 设置主库（老版本命令） |
| `REPLICAOF host port` | 设置主库（新版本命令，功能相同） |
| `INFO replication` | 查看主从状态 |

### 哨兵命令

| 命令 | 作用 |
|------|------|
| `SENTINEL masters` | 查看所有主库 |
| `SENTINEL master name` | 查看某个主库 |
| `SENTINEL sentinels name` | 查看其他哨兵 |
| `SENTINEL slaves name` | 查看从库 |
| `SENTINEL failover name` | 手动触发切换 |

## 8.2 五种数据类型对比表

| 类型 | 命令前缀 | 特点 | 典型场景 | 生活比喻 |
|------|---------|------|---------|---------|
| String | 无 | 最简单，一对一 | 缓存、计数器 | 杯子 |
| Hash | H | 字段-值对，一对多 | 存对象信息 | 多格饭盒 |
| List | L/R | 有序、可重复 | 消息队列 | 管道 |
| Set | S | 无序、不重复 | 去重、交集 | 篮子 |
| Zset | Z | 不重复、带分数排序 | 排行榜 | 排名表 |

## 8.3 面试高频考点

### Q1：Redis 为什么快？

1. 数据在内存里，读写不用操作硬盘
2. 单线程，没有多线程切换开销
3. 用了IO多路复用技术（一个线程处理多个连接）
4. 数据结构设计高效

### Q2：Redis 和 MySQL 的区别？

| 对比项 | Redis | MySQL |
|--------|-------|-------|
| 存储位置 | 内存 | 硬盘 |
| 速度 | 极快 | 较慢 |
| 数据类型 | 5种 | 表格 |
| 持久化 | RDB+AOF | 默认存硬盘 |
| 适用场景 | 缓存、排行榜 | 持久存储 |

### Q3：RDB 和 AOF 的区别？

| 对比项 | RDB | AOF |
|--------|-----|-----|
| 原理 | 快照 | 日志 |
| 文件大小 | 小 | 大 |
| 恢复速度 | 快 | 慢 |
| 数据安全性 | 可能丢几分钟 | 最多丢1秒 |
| 生活比喻 | 拍照 | 写日记 |

### Q4：什么是主从复制？

一台Redis当主库（master），其他当从库（slave）。主库写的数据自动同步到从库。从库只能读不能写。

好处：数据备份、读写分离、高可用基础。

### Q5：什么是哨兵？

哨兵是独立监控进程，监控主库状态。主库挂了，哨兵自动把从库提升为主库，实现高可用。

工作流程：监控 → 发现异常 → 确认下线 → 选举领导者 → 执行切换

### Q6：s_down 和 o_down 的区别？

- s_down（主观下线）：单个哨兵认为主库挂了
- o_down（客观下线）：足够多哨兵（≥quorum）都认为主库挂了

### Q7：Redis 的五种数据类型分别适合什么场景？

| 类型 | 场景 |
|------|------|
| String | 缓存、计数器、分布式锁 |
| Hash | 存对象信息（用户、商品） |
| List | 消息队列、最新消息列表 |
| Set | 去重、共同好友、抽奖 |
| Zset | 排行榜、热搜榜、延迟队列 |

## 8.4 Redis 学习路线回顾

```
第1步：安装Redis              ✓ 已完成
  └─ yum install epel-release + redis

第2步：基本操作                ✓ 已完成
  └─ SET / GET / DEL / KEYS / EXPIRE

第3步：五种数据类型            ✓ 已完成
  └─ String / Hash / List / Set / Zset

第4步：持久化                  ✓ 已完成
  └─ RDB（快照）+ AOF（日志）

第5步：主从复制                ✓ 已完成
  └─ 一主一从，自动同步数据

第6步：哨兵高可用              ✓ 已完成
  └─ 自动故障切换，主库挂了从库上位
```

---

> **文档总结**：
> Redis 核心就是6个模块：安装 → 基本操作 → 五种数据类型 → 持久化 → 主从复制 → 哨兵高可用。
> 记住：Redis 是"内存里的数据库"，快但会断电丢失，所以需要持久化。
> 一台 Redis 不够稳，所以需要主从复制做备份。
> 主库会挂，所以需要哨兵自动切换。
> 这就是 Redis 高可用的完整链路！
