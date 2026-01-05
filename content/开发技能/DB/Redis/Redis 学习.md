## 1. Redis 概念基础
### 1.1 redis 是什么
Redis是一款内存高速缓存数据库。Redis全称为：**Remote Dictionary Server**（远程数据服务），使用C语言编写，Redis是一个key-value存储系统（键值存储系统），支持丰富的数据类型，如：String、list、set、zset、hash。

Redis是一种支持key-value等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。

### 1.2 redis 的特点
- **读写性能优异**
	- Redis 能读的速度是110000次/s,写的速度是81000次/s
- **数据类型丰富**
	- Redis 支持 String, List, Set，Zset，Hash等数据类型操作
- **原子性**
	- Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作合并后的原子性执行
- **丰富的特性**
	- Redis支持 publish/subscribe, 通知, key 过期等特性
- **持久化**
	- Redis支持 RDB, AOF 等持久化方式
- **发布订阅**
	- Redis支持发布/订阅模式
- **分布式**
	- Redis Cluster

### 1.3 常见应用场景
Redis 最为常见的一个应用场景就是用作**缓存**，缓存可以**显著提升访问速度，降低数据库压力**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250628175942788.png)

更详细的场景介绍见[微信公众平台](https://mp.weixin.qq.com/s/srkd73bS2n3mjIADLVg72A)

## 2. Redis 客户端
### 2.1 命令行客户端
Redis 提供了一个命令行客户端 `redis-cli`，我们可以使用他连接到 Redis 服务器，然后通过命令与服务器进行各种交互，例如数据的增删查改。下面介绍命令的基本用法：

**启动客户端**
```bash
redis-cli -h 127.0.0.1 -p 6379
```
说明：
- `-h <hostname>`选项用于声明 Redis 服务器的主机名或 IP 地址，默认值为`127.0.0.1`
- `-p <port>`选项用于声明 Redis 服务器监听的端口号，默认值为`6379`
**测试连接状态**
`ping` 命令可用于测试连接状态，语法如下：
```bash
ping
```
说明：若连接正常，则会返回 pong
**退出客户端**
`quit`命令可用于断开客户端与服务端的连接，并退出客户端，语法如下：
```bash
quit
```
### 2.2 图形化客户端
Redis 官方推荐的图形化客户端为**RedisInsight**，该客户端开源免费，且功能强大。
## 3. Redis 常用数据类型及命令
### 3.1 通用命令
- **查看所有键**
`keys`命令可用于查看所有键，语法如下：
```bash
keys pattern
```
**说明**：pattern 用于匹配 key，其中 `*` 表示任意个任意字符，`?`表示一个任意字符。
**示例**：
```bash
127.0.0.1:6379>KEYS *
1)"k3"
2)"k2"
3)"k1"
```
**注意**：该命令会遍历 Redis 服务器中保存的所有键，因此当键很多时会影响整个 Redis 服务的性能，线上环境需要谨慎使用。

- **键总数**
`dbsize`可用于查看键的总数，语法如下：
```bash
dbsize
```

- **判断键是否存在**
`exists`命令可用于判断一个键是否存在，语法如下：
```bash
exists key
```
**说明**：若键存在则返回1，不存在则返回0.

- **删除键**
`del`可用于删除指定键，语法如下：
```bash
del key [key ...]
```
**说明**：返回值为删除键的个数，若删除一个不存在的键，则返回0

- **设置键的过期时间**
```bash
expire key seconds
```
**说明**：key 在 seconds 秒后过期

- **查询键的剩余过期时间**
```bash
ttl key
```
**说明**：`ttl`的含义为 time to live，用于查询一个定时键的剩余存活时间，返回值以秒为单位。若查询的键的未设置过期时间，则返回`-1`，若查询的键不存在，则返回`-2`.

- **去掉 key 的过期时间**
```bash
persist key
```

- **查询 key 的类型**
```bash
type key
```
**说明**：返回 key 的类型

### 3.2 **数据库管理命令**
Redis 默认有编号为0~15的16个逻辑数据库，每个数据库之间的数据是相互独立的，所有连接默认使用的都是0号数据库。

- **切换数据库**
`select` 命令可用于切换数据库，语法如下
```bash
select index
```
**说明**：若 index 超出范围，会报错

- **清空数据库**
`flushdb`命令会清空当前所选用的数据库，`flushall`命令会清空0~15号所有的数据库。
**注意**：生产环境慎用

### 3.3 数据类型简介
首先对 redis 来说，所有的 key（键）都是字符串。我们在谈基础数据结构时，讨论的是存储值的数据类型，主要包括常见的5种数据类型，分别是：String、List、Set、Zset、Hash
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251044052.png)

| 结构类型       | 结构存储的值                | 结构的读写能力                                                              |
| ---------- | --------------------- | -------------------------------------------------------------------- |
| String 字符串 | 可以是字符串、整数或浮点数         | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作；                                 |
| List 列表    | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行 push 和 pop 操作，读取单个或多个元素；根据值查找或删除元素；                         |
| Set 集合     | 包含字符串的无序集合            | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等                          |
| Hash 散列    | 包含键值对的无序散列表           | 包含方法有添加、获取、删除单个元素                                                    |
| Zset 有序集合  | 和散列一样，用户存储键值对         | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 |

### 3.4 5种基础类型
#### String 字符串
> String 是 redis 中最基本的数据类型，一个key对应一个value。

String类型是二进制安全的，意思是 redis 的 string 可以包含任何数据。如数字，字符串，jpg图片或者序列化的对象。
Redis 中的 string 类型保存的是字节序列（Sequence of bytes），因此任意类型的数据，只要经过序列化之后都可以保存到 Redis 的 string 类型中，包括文本、数字甚至是一个对象。
##### 1.图例
下图是一个String类型的实例，其中键为hello，值为world

<img src="https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251140786.png" width="300" height="200">

##### 2.常用命令
**set**
`set`命令用于添加 String 类型的键值对，具体语法如下
```bash
SET key value [NX|XX] [EX seconds|PX milliseconds]
```
各项含义如下：
- NX：仅在 key 不存在时 set
- XX：仅在 key 存在时 set
- EX seconds：设置过期时间，单位为秒
- PX milliseconds：设置过期时间，单位为毫秒

**get**
`get`命令用于获取某个 string 类型的键对应的值，具体语法如下
```bash
GET key
```

**incr**
`incr`命令用于对数值做自增操作，具体语法如下
```bash
INCR key 
```
若 key 对应的 value 是整数，则返回自增后的结果，若不是整数则报错，若 key 不存在则创建并返回1。

**decr**
`decr`命令用于对数值做自减操作，具体语法如下
```bash
DECR key
```
若 key 对应的 value 是整数，则返回自减后的结果，若不是整数则报错，若 key 不存在则创建并返回-1.

| 命令     | 简述          | 使用                |
| ------ | ----------- | ----------------- |
| GET    | 获取存储在给定键中的值 | GET name          |
| SET    | 设置存储在给定键中的值 | SET name value    |
| DEL    | 删除存储在给定键中的值 | DEL name          |
| INCR   | 将键存储的值加1    | INCR key          |
| DECR   | 将键存储的值减1    | DECR key          |
| INCRBY | 将键存储的值加上整数  | INCRBY key amount |
| DECRBY | 将键存储的值减去整数  | DECRBY key amount |

##### 3.应用场景
- **缓存**： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis 作为缓存层，mysql 做持久化层，降低 mysql 的读写压力。
- **计数器**：redis 是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
- **session**：常见方案 spring session + redis 实现 session 共享，

#### List 列表
> Redis 中的 List 其实就是链表（Redis 用双端链表实现 List）。

使用 List 结构，我们可以轻松地实现最新消息排队功能（比如新浪微博的 TimeLine）。List 的另一个应用就是消息队列，可以利用 List 的 PUSH 操作，将任务存放在 List 中，然后工作线程再用 POP 操作将任务取出进行执行。
##### 1.图例
<img src="https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251151264.png" width="300" height="200">

##### 2.常用命令
**① 添加元素**
向集合中添加元素的命令有 `lpush`、`rpush`、`linsert`，各命令的功能与用法如下
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250628222507887.png)
`lpush`
该命令用于向 list 左侧添加元素，语法如下
```bash
lpush key element [element ...]
```
示例
```bash
lpush l1 a b c
```

`rpush`
该命令用于向 list 右侧添加元素，语法如下
```bash
rpush key element [element ...]
```

`linsert`
该命令用于向 list 指定位置添加元素，语法如下
```bash
linsert key before|after pivot element
```
参数说明：
`key`：列表的 key 名
`before|after`：插入的位置：在`pivot`之前或之后
`pivot`：要查找的基准值
`element`：要插入的新元素
示例
```bash
linsert l1 after b new
```

**② 查询元素**
查询 list 元素的命令有 `lindex` 和`lrange`，各命令的功能和用法如下
`lindex`
该命令用于获取指定索引位置的元素，语法如下
```bash
lindex key index
```
说明：index 从左到右依次是0，1，2...，从右到左依次是-1，-2，-3...
`lrange`
该命令用于获取指定范围内的元素列表，**包含 start 和 stop 位置的元素**，语法如下
```bash
lrange key start stop
```
示例
获取 list 全部元素，命令如下
```bash
lrange l1 0 -1
```

**③ 删除元素**
删除 list 元素的命令有`lpop`、`rpop`、`lrem`，各命令的功能与用法如下
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250628223721149.png)
`lpop`
该命令用于移除并返回 list 左侧元素，语法如下
```bash
lpop key [count]
```
说明：count 参数表示移除元素的个数
`rpop`
该命令用于移除并返回 list 右侧的元素，语法如下
```bash
rpop key [count]
```
`lrem`
该命令用于移除 list 中的指定元素，语法如下
```bash
lrem key count element
```
说明：count 参数表示要移除的 element 元素的个数（list 中可以存在多个相同的元素），count 的用法如下：
- 若 count > 0，则从左到右删除最多 count 个 element 元素
- 若 count < 0，则从右到左删除最多 count（的绝对值）个 element 元素
- 若 count = 0，则删除所有的 element 元素

**④ 修改元素**
`lset`命令可用于修改指定索引位置的元素，语法如下
```bash
lset key index element 
```

**⑤ 其他**
`llen`命令可用于查看 list 长度，语法如下
```bash
llen key
```

`ltrim` 命令用于按照索引范围修剪列表
```bash
ltrim key start end
```
例子：保留索引下标从1到4的列表
```bash
ltrim listkey 1 4
```


| 命令     | 简述                                                             | 使用               |
| ------ | -------------------------------------------------------------- | ---------------- |
| RPUSH  | 将给定值推入到列表右端                                                    | RPUSH key value  |
| LPUSH  | 将给定值推入到列表左端                                                    | LPUSH key value  |
| RPOP   | 从列表的右端弹出一个值，并返回被弹出的值                                           | RPOP key         |
| LPOP   | 从列表的左端弹出一个值，并返回被弹出的值                                           | LPOP key         |
| LRANGE | 获取列表在给定范围上的所有值                                                 | LRANGE key 0 -1  |
| LINDEX | 通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推 | LINDEX key index |
##### 3.列表使用技巧
- lpush+lpop=Stack(栈)
- lpush+rpop=Queue（队列）
- lpush+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）
##### 4.应用场景
- **微博TimeLine**: 有人发布微博，用 lpush 加入时间轴，展示新的列表信息
- **社交应用**中，可使用 list 缓存每个用户发布的最新的 N 条记录
- **异步消息队列**

#### Set 集合
> Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)
##### 1.图例
<img src="https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251330118.png" width="300" height="200">


##### 2.常用命令
**① 集合内**
`sadd`
该命令用于向 set 中添加元素，语法如下
```bash
sadd key member [member ...]
```

`smembers`
该命令用于查询 set 中的全部元素，语法如下
```bash
smembers key
```

`srem`
该命令用于移除 set 中的指定元素，语法如下
```bash
srem key member [member ...]
```

`spop`
该命令随机移除并返回 set 中的 n 个元素，语法如下
```bash
spop key [count]
```

`srandmember`
该命令随机返回 set 中的 n 个元素（不删除），语法如下
```bash
srandmember key [count]
```

`scard`（Cardinality，基数）
该命令用于查询 set 中的元素个数，语法如下
```bash
scard key
```

`sismember`
该命令用于元素是否在 set 中， 语法如下
```bash
sismember key element
```


**② 集合间**
`sinter`
该命令用于计算多个集合的交集，语法如下
```bash
sinter key [key ...]
```

`sunion`
该命令用于计算多个集合的并集，语法如下
```bash
sunion key [key ...]
```

`sdiff`
该命令用于计算多个集合的差集，语法如下
```bash
sdiff key [key ...]
```

| 命令        | 简述                        | 使用                   |
| --------- | ------------------------- | -------------------- |
| SADD      | 向集合添加一个或多个成员              | SADD key value       |
| SCARD     | 获取集合的成员数                  | SCARD key            |
| SMEMBERS  | 返回集合中的所有成员                | SMEMBERS key member  |
| SISMEMBER | 判断 member 元素是否是集合 key 的成员 | SISMEMBER key member |

##### 3.应用场景
- **标签**（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。
- **点赞，或点踩，收藏等**，可以放到set中实现
- set 可用于计算共同关注好友，随机抽奖系统

#### Hash 散列
> Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。
##### 1.图例
<img src="https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251339897.png" width="300" height="200">

##### 2.常用命令
`hset`
该命令用于向 hash 中增加键值对，语法如下
```bash
hset key field value [field value ...]
```

`hget`
该命令用于获取 hash 中某个键对应的值，语法如下
```bash
hget key field
```

`hdel`
该命令用于删除 hash 中的指定的键值对，语法如下
```bash
hdel key field [field ...]
```

`hlen`
该命令用于查询 hash 中的键值对个数，语法如下
```bash
hlen key
```

`hexists`
该命令用于判断 hash 中的某个键是否存在，语法如下
```bash
hexists key field
```

`hkeys`
该命令用于返回 hash 中所有的键，语法如下
```bash
hkeys key
```

`hvals`
该命令用于返回 hash 中所有的值， 语法如下
```bash
hvals key
```

`hgetall`
该命令用于返回 hash 中所有的键与值，语法如下
```bash
hgetall key
```

| 命令      | 简述                   | 使用                            |
| ------- | -------------------- | ----------------------------- |
| HSET    | 添加键值对                | HSET hash-key sub-key1 value1 |
| HGET    | 获取指定散列键的值            | HGET hash-key key1            |
| HGETALL | 获取散列中包含的所有键值对        | HGETALL hash-key              |
| HDEL    | 如果给定键存在于散列中，那么就移除这个键 | HDEL hash-key sub-key1        |

##### 3.应用场景
**缓存**： 相比 string 更节省空间，能直观的维护缓存信息，如用户信息，视频信息等

#### Zset 有序集合
Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序
##### 1.图例
<img src="https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/202506251349262.png" width="300" height="200">

##### 2. 底层实现机制
ZSet 底层由 **两个数据结构** 组成，分别用于不同的访问需求：
1. **哈希表（dict）**
结构：`member -> score`
- 用于快速通过 `member` 查询 score：**O(1)**
- 例子：`user1 -> 100`

2. **跳表（skiplist）**
结构：按 score 升序排序的链表结构（支持层级跳跃）
- 用于：
    - `ZRANGE`、`ZREVRANGE` 排序访问
    - `ZRANGEBYSCORE` 区间访问
    - `ZRANK` 获取 member 的排名
- 插入/删除/范围查找性能约为：**O(log N)**
**插入时，两个结构同步插入：**
- 将 member -> score 写入 hash 表；
- 将 member + score 插入跳表。
**这样可以兼顾**：
- 精确查找效率（hash dict）
- 排序/排名/范围查询效率（skiplist）

##### 3.常用命令
`zadd`
该命令用于向 zset 中添加元素，语法如下
```bash
ZADD key [NX|XX] score member
```
说明：
- NX：仅当 member 不存在时才 add
- XX：仅当 member 存在时才 add

`zcard`
该命令用于计算 zset 中的元素个数，语法如下
```bash
zcard key
```

`zscore`
该命令用于查看某个元素的分数，语法如下
```bash
zscore key member
```

`zrank/zrevrank`
这组命令用于计算元素的排名，其中 zrank 按照 score 的升序排序，zrevrank 则按照降序排序，语法如下
```bash
zrank/zrevrank key member
```
**说明： 名次从0开始**

`zrem`
该命令用于删除元素，语法如下
```bash
zrem key member [member ...]
```

`zincrby`
该命令用于增加元素的分数，语法如下
```bash
zincrby key increment member
```

`zrange`
该命令用于查询指定区间范围的元素，语法如下
```bash
zrange key start stop [byscore] [rev] [limit offset count] [withscores]
```
**说明：**
- **start/stop**：用于指定查询区间，但是在不同模式下，其代表的含义也不同
	- 默认模式下，`start~stop`表示的是名次区间，且该区间为闭区间。名次从0开始，且可为负数，-1表示倒数第一，-2表示倒数第二，以此类推。
	- byscore 模式下（声明了 byscore 参数），则`start~stop`表示的就是分数区间，该区间默认仍为闭区间。在该模式下，可以在`start`或`stop`前增加`(`来表示开区间，例如`(1(5`，表示的就是`(1,5)`这个开区间。除此之外，还可以使用`-inf`和`+inf`表示负无穷和正无穷。
- **byscore**：用于切换到分数模式。
- **rev**：表示降序排序。
- **limit**：该选项只用于 byscore 模式下使用 rev 参数需要注意查询范围， start 应大于 stop
- **withscores**：用于打印分数

| 命令     | 简述                           | 使用                             |
| ------ | ---------------------------- | ------------------------------ |
| ZADD   | 将一个带有给定分值的成员添加到有序集合里面        | ZADD zset-key 178 member1      |
| ZRANGE | 根据元素在有序集合中所处的位置，从有序集合中获取多个元素 | ZRANGE zset-key 0-1 withccores |
| ZREM   | 如果给定元素成员存在于有序集合中，那么就移除这个元素   | ZREM zset-key member1          |

##### 4.应用场景
**排行榜**：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行

### 3.5 3种特殊类型
#### HyperLogLogs（基数统计）
> Redis 2.8.9 版本更新了 Hyperloglog 数据结构

- **什么是基数？**

举个例子，A = {1, 2, 3, 4, 5}， B = {3, 5, 6, 7, 9}；那么基数（不重复的元素）= 1, 2, 4, 6, 7, 9； （允许容错，即可以接受一定误差）

- **HyperLogLogs 基数统计用来解决什么问题**？

这个结构可以非常省内存的去统计各种计数，比如注册 IP 数、每日访问 IP 数、页面实时UV、在线用户数，共同好友数等

- **它的优势体现在哪**？

一个大型的网站，每天 IP 比如有 100 万，粗算一个 IP 消耗 15 字节，那么 100 万个 IP 就是 15M。而 HyperLogLog 在 Redis 中每个键占用的内容都是 12K，理论存储近似接近 2^64 个值，不管存储的内容是什么，它一个基于基数估算的算法，只能比较准确的估算出基数，可以使用少量固定的内存去存储并识别集合中的唯一元素。而且这个估算的基数并不一定准确，是一个带有 0.81% 标准错误的近似值（对于可以接受一定容错的业务场景，比如IP数统计，UV等，是可以忽略不计的）

- **命令使用**

| 命令                                        | 简述                                  | 使用                                          |
| ----------------------------------------- | ----------------------------------- | ------------------------------------------- |
| PFADD key element [element ...]           | 向`key`对应的HyperLogLog 中添加元素          | PFADD myUniqueSet element1 element2         |
| PFCOUNT key                               | 获取`key`对应的HyperLogLog 中的基数，即唯一元素的数量 | PFCOUNT myUniqueSet                         |
| PFMERGE destkey sourcekey [sourcekey ...] | 将多个HyperLogLog集合合并到一个`destkey`中     | PFMERGE mergedSet myUniqueSet1 myUniqueSet2 |
- 应用场景

**唯一用户访问统计**：HyperLogLog 类型非常适合用来统计一段时间内访问网站或应用的唯一用户数量
**事件独立性分析**：HyperLogLog 可以用于分析不同事件的独立性，例如，不同来源的点击事件是否来自相同的用户群体

#### Bitmap（位存储）
> Bitmap 即位图数据结构，都是操作二进制位来进行记录，只有0 和 1 两个状态。

##### 1.基本命令

| 命令                                    | 简述                                                  | 使用                               |
| ------------------------------------- | --------------------------------------------------- | -------------------------------- |
| SETBIT key offset value               | 对`key`指定的`offset`位置设置位值。`value`可以是0或1               | SETBIT myBitmap 100 1            |
| GETBIT key offset                     | 获取`key`在指定`offset`位置的位值                             | GETBIT myBitmap 100              |
| BITCOUNT key [start end]              | 计算`key`中位值为1的数量。可选地，可以指定一个范围`[start end]`来计算该范围内的位值 | BITCOUNT myBitmap                |
| BITOP operation destkey key [key ...] | 对一个或多个键进行位操作（AND, OR, XOR, NOT）并将结果存储在`destkey`中    | BITOP AND resultBitmap key1 key2 |

##### 2.应用场景
**状态监控**：Bitmap 类型可以用于监控大量状态，例如用户在线状态、设备状态等
**功能开关**：Bitmap 类型可以用于控制功能开关，例如 A/B 测试、特性发布等

#### Geospatioal（地理位置）
> Redis 的 GEO 数据结构用于存储地理位置信息，它允许用户进行各种基于地理位置的操作，如查询附近的位置、计算两个地点之间的距离等。

##### 1.基本命令

| 命令                                                                             | 简述                                | 使用                                                                         |
| ------------------------------------------------------------------------------ | --------------------------------- | -------------------------------------------------------------------------- |
| GEOADD key longitude latitude member                                           | 向`key`对应的GEO集合中添加带有经纬度的成员`member` | GEOADD myGeoSet 116.407526 39.904030 "Beijing"                             |
| GEOPOS key member [member ...]                                                 | 返回一个或多个成员的地理坐标                    | GEOPOS myGeoSet "Beijing"                                                  |
| GEODIST key member1 member2 [unit]                                             | 计算两个成员之间的距离                       | GEODIST myGeoSet "Beijing" "Shanghai"                                      |
| GEOHASH key member [member ...]                                                | 返回一个或多个成员的Geohash表示               | GEOHASH myGeoSet "Beijing"                                                 |
| GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [WITHHASH] | 查询给定位置周围指定半径内的所有成员                | GEORADIUS myGeoSet 116.407526 39.904030 500 km WITHCOORD WITHDIST WITHHASH |

##### 2.应用场景
**附近地点搜索**：GEO类型可以用于实现基于地理位置的搜索功能，如查找附近的餐馆、影院等。
**用户定位与导航**：GEO类型可以用于记录用户的地理位置，并提供导航服务。

### 3.6 Stream 类型
#### 3.6.1 Stream 的结构
每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250820145051792.png)
解释：
- `Consumer Group` ：消费组，使用 `XGROUP CREATE` 命令创建，一个消费组有多个消费者(Consumer), 这些消费者之间是竞争关系。
- `last_delivered_id` ：游标，每个消费组会有个游标 `last_delivered_id`，任意一个消费者读取了消息都会使游标 `last_delivered_id` 往前移动。
- `pending_ids` ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 `pending_ids` 记录了当前已经被客户端读取的消息，但是还没有 `ack` (Acknowledge character：确认字符）。如果客户端没有ack，这个变量里面的消息ID会越来越多，一旦某个消息被ack，它就开始减少。这个`pending_ids`变量在Redis官方被称之为PEL，也就是Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。
补充：
- `消息ID`：消息ID的形式是 timestampInMillis-sequence，例如1527846880572-5，它表示当前的消息在毫米时间戳 1527846880572时产生，并且是该毫秒内产生的第5条消息。消息ID可以由服务器自动生成，也可以由客户端自己指定，但是形式必须是整数-整数，而且必须是后面加入的消息的ID要大于前面的消息ID。
- `消息内容`: 消息内容就是键值对，形如 hash 结构的键值对，这没什么特别之处。



#### 3.6.2 生产和消费
##### ① 基本的增删查改
`XADD` - 添加消息到末尾
```bash
# * 号表示服务器自动生成 ID，后面顺序跟着一堆 key/value
xadd codehole * name laoqian age 30
```

`XTRIM` - 对流进行修剪，限制长度


`XDEL` - 删除消息
```bash
xdel codehole 1527849609889-0
```

`XLEN` - 获取流包含的元素数量，即消息长度
```bash
xlen codehole
```

`XRANGE` - 获取消息列表，会自动过滤已经删除的消息
```bash
xrange codehole - + # -表示最小值，+表示最大值

xrange codehole 1527849629172-0 + # 指定最小消息ID的列表

xrange codehole - 1527849629172-0  # 指定最大消息ID的列表
```

`XREVRANGE` - 反向获取消息列表，ID 从大到小

`XREAD` - 以阻塞或非阻塞方式获取消息列表

##### ② 单一消费者的消费
Redis设计了一个单独的消费指令xread，可以将Stream当成普通的消息队列(list)来使用。使用xread时，我们可以完全忽略消费组(Consumer Group)的存在，就好比Stream就是一个普通的列表(list)。
```bash
# 从 Stream 头部读取两条消息
127.0.0.1:6379> xread count 2 streams codehole 0-0
1) 1) "codehole"
   2) 1) 1) 1527851486781-0
         2) 1) "name"
            2) "laoqian"
            3) "age"
            4) "30"
      2) 1) 1527851493405-0
         2) 1) "name"
            2) "yurui"
            3) "age"
            4) "29"
# 从 Stream 尾部读取一条消息，这里不会返回任何消息
127.0.0.1:6379> xread count 1 streams codehole $
(nil)
# 从尾部阻塞等待新消息到来，下面的指令会堵住，直到新消息到来
127.0.0.1:6379> xread block 0 count 1 streams codehole $
# 重新打开一个窗口，在这个窗口往 stream 里发消息
127.0.0.1:6379> xadd codehole * name youming age 60
1527852774092-0
# 在切换到前面的窗口，可以看到阻塞解除了，返回了新的消息内容
# 并且还显示了一个等待时间
127.0.0.1:6379> xread block 0 count 1 streams codehole $
1) 1) "codehole"
   2) 1) 1) 1527852774092-0
         2) 1) "name"
            2) "youming"
            3) "age"
            4) "60"
(93.11s)
```
客户端如果想要使用xread进行顺序消费，一定要记住当前消费到哪里了，也就是返回的消息ID。下次继续调用xread时，将上次返回的最后一个消息ID作为参数传递进去，就可以继续消费后续的消息。

block 0表示永远阻塞，直到消息到来，block 1000表示阻塞1s，如果1s内没有任何消息到来，就返回nil
```bash
127.0.0.1:6379> xread block 1000 count 1 streams codehole $
(nil)
(1.07s)
```
##### ③ 消费组的消费
- **消费组消费图**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250820160745130.png)


- **相关命令**
	- XGROUP CREATE - 创建消费者组
	- XREADGROUP GROUP - 读取消费者组中的消息
	- XACK - 将消息标记为"已处理"
	- XGROUP SETID - 为消费者组设置新的最后递送消息ID
	- XGROUP DELCONSUMER - 删除消费者
	- XGROUP DESTROY - 删除消费者组
	- XPENDING - 显示待处理消息的相关信息
	- XCLAIM - 转移消息的归属权
	- XINFO - 查看流和消费者组的相关信息；
	- XINFO GROUPS - 打印消费者组的信息；
	- XINFO STREAM - 打印流信息

- **创建消费组**
	Stream 通过`xgroup create`指令创建消费组(Consumer Group)，需要传递起始消息ID参数用来初始化`last_delivered_id`变量。
	```bash
	xgroup create codehole cg1 0-0 # 表示从头开始消费
	ok
	# $ 表示从尾部开始消费，只接收新消息，当前stream消息会全部忽略
	xgroup create codehole cg2 $
	ok
	```
- **消费消费组**
	Stream 提供了 `xreadgroup` 指令可以进行消费组的组内消费，需要提供消费组名称、消费者名称和起始消息ID。它同 xread 一样，也可以阻塞等待新消息。读到新消息后，对应的消息ID就会进入消费者的PEL(正在处理的消息)结构里，客户端处理完毕后使用 xack 指令通知服务器，本条消息已经处理完毕，该消息ID就会从PEL中移除。
```bash
# > 号表示从当前消费组的 last_delivered_id 后面开始读
# 每当消费者读取一条消息，last_delivered_id 变量就会前进
xreadgroup GROUP cg1 c1 count 1 streams codehole >
```
#### 3.6.3 监控状态
Stream提供了XINFO来实现对服务器信息的监控，可以查询：
- 查看队列信息
```bash
Xinfo stream mq
```
- 消费组信息
```bash
Xinfo groups mq
```
- 消费者组成员信息
```bash
Xinfo CONSUMERS mq mqGroup
```

### 3.7 对象机制
#### 3.7.1 为什么 Redis 会设计 redisObject 对象？
一些命令, 比如 `DEL`、 `TTL` 和 `TYPE`, 可以用于任何类型的键；要正确实现这些命令, 必须为不同类型的键设置不同的处理方式: 比如说, 删除一个列表键和删除一个字符串键的操作过程就不太一样。
以上的描述说明, **Redis 必须让每个键都带有类型信息, 使得程序可以检查键的类型, 并为它选择合适的处理方式**.
比如说， 集合类型就可以由字典和整数集合两种不同的数据结构实现， 但是， 当用户执行 ZADD 命令时， 他/她应该不必关心集合使用的是什么编码， 只要 Redis 能按照 ZADD 命令的指示， 将新元素添加到集合就可以了。
这说明, **操作数据类型的命令除了要对键的类型进行检查之外, 还需要根据数据类型的不同编码进行多态处理**.

为了解决以上问题, **Redis 构建了自己的类型系统**, 这个系统的主要功能包括:
- redisObject 对象.
- 基于 redisObject 对象的类型检查.
- 基于 redisObject 对象的显式多态函数.
- 对 redisObject 进行分配、共享和销毁的机制
#### 3.7.2 redisObject 数据结构
redisObject 是 Redis 类型系统的核心, 数据库中的每个键、值, 以及 Redis 本身处理的参数, 都表示为这种数据类型.
```c
/*
 * Redis 对象
 */
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 编码方式
    unsigned encoding:4;
    // LRU - 24位, 记录最末一次访问时间（相对于lru_clock）; 或者 LFU（最少使用的数据：8位频率，16位访问时间）
    unsigned lru:LRU_BITS; // LRU_BITS: 24
    // 引用计数
    int refcount;
    // 指向底层数据结构实例
    void *ptr;
} robj;
```
下图对应上面的结构
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250823155253805.png)
**其中 type、encoding 和 ptr 是最重要的三个属性**。
- **type 记录了对象所保存的值得类型**，它的值可能是以下常量中的一个：
```c
/*
* 对象类型
*/
#define OBJ_STRING 0 // 字符串
#define OBJ_LIST 1 // 列表
#define OBJ_SET 2 // 集合
#define OBJ_ZSET 3 // 有序集
#define OBJ_HASH 4 // 哈希表
```
- **encoding记录了对象所保存的值的编码**，它的值可能是以下常量中的一个：
```c
/*
* 对象编码
*/
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* 注意：版本2.6后不再使用. */
#define OBJ_ENCODING_LINKEDLIST 4 /* 注意：不再使用了，旧版本2.x中String的底层之一. */
#define OBJ_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6  /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8  /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9 /* Encoded as linked list of ziplists */
#define OBJ_ENCODING_STREAM 10 /* Encoded as a radix tree of listpacks */
```
- **ptr是一个指针，指向实际保存值的数据结构**，这个数据结构由type和encoding属性决定。举个例子， 如果一个redisObject 的type 属性为`OBJ_LIST` ， encoding 属性为`OBJ_ENCODING_QUICKLIST` ，那么这个对象就是一个Redis 列表（List)，它的值保存在一个QuickList的数据结构内，而ptr 指针就指向quicklist的对象；

下图展示了redisObject 、Redis 所有数据类型、Redis 所有编码方式以及底层数据结构之间的关系
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250823155517512.png)

- **lru属性: 记录了对象最后一次被命令程序访问的时间**
**空转时长**：当前时间减去键的值对象的lru时间，就是该键的空转时长。Object idletime命令可以打印出给定键的空转时长
如果服务器打开了maxmemory选项，并且服务器用于回收内存的算法为volatile-lru或者allkeys-lru，那么当服务器占用的内存数超过了maxmemory选项所设置的上限值时，空转时长较高的那部分键会优先被服务器释放，从而回收内存。

### 3.8 底层数据结构
#### 3.8.1 简单动态字符串 - sds
##### SDS 定义
> 这是一种用于存储二进制数据的一种结构, 具有动态扩容的特点

- **SDS 的总体概览**如下图：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250823160623579.png)
其中`sdshdr`是头部, `buf`是真实存储用户数据的地方. 另外注意, 从命名上能看出来, 这个数据结构除了能存储二进制数据, 显然是用于设计作为字符串使用的, 所以在buf中, 用户数据后总跟着一个\0. 即图中 `"数据" + "\0"` 是为所谓的buf。

- **SDS 相关的结构：**
SDS有五种不同的头部. 其中sdshdr5实际并未使用到. 所以实际上有四种不同的头部, 分别如下:
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250823160829564.png)
其中：
- `len`保存了SDS保存字符串的长度
- `buf[]`数组用来保存字符串的每个元素
- `alloc` 分别以 uint8,uint16,uint32,uint64 表示整个 SDS，除过头部与末尾的\0，剩余的字节数
- `flags` 始终为一字节，以低三位标示着头部的类型，高5位未使用

##### 为什么使用SDS
- 常数复杂度获取字符串长度
- 杜绝缓冲区溢出
- 减少修改字符串的内存重新分配次数
- 二进制安全
- 兼容部分 C 字符串函数
#### 3.8.2 压缩列表 - ZipList


#### 3.8.3 快表 - QuickList


#### 3.8.4 字典/哈希表 - Dict


#### 3.8.5 整数集 - IntSet



#### 3.8.6 跳表 - ZSkipList
##### 介绍
跳表是一种多层的**有序链表结构**，每一层是下层的“加速索引”。  
每个节点除了指向下一个节点外，可能还会**跨层链接**，形成如下结构：
```bash
Level 3:  A ------------------------> Z
Level 2:  A --------> G ---------> Z
Level 1:  A --> C --> G --> M --> Z
```
从上到下查找就像：
1. 先在高层快速跳转缩小范围；
2. 再回到底层精确查找。
##### 跳表结构
跳表节点结构 `zskiplistNode`：
```c
typedef struct zskiplistNode {
    sds ele;                  // 元素（member）
    double score;             // 分数（用于排序）
    struct zskiplistNode *backward;  // 后退指针
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned int span;    // 距离（用于排名）
    } level[];
} zskiplistNode;
```
##### 时间复杂度

| 操作    | 时间复杂度                |
| ----- | -------------------- |
| 查找    | O(log N)             |
| 插入/删除 | O(log N)             |
| 范围查询  | O(log N + k)（k 是结果数） |
和红黑树相当，但跳表实现简单、插入更快、更适合**并发和高性能场景**。
##### 高度和概率
Redis 的跳表是 **随机化的**，每个节点插入时会“掷硬币”决定它的层数。
- 默认最大高度：32 层
- 每一层的概率（p）：0.25
换句话说：
- 每个节点有 1/4 的概率有第 2 层指针；
- 有 1/16 的概率有第 3 层指针；
这种随机分布使得跳表性能接近平衡树。
##### 对比红黑树

| 特性    | 跳表（Skip List） | 红黑树（Red-Black Tree） |
| ----- | ------------- | ------------------- |
| 实现复杂度 | 简单            | 复杂（旋转、染色）           |
| 插入性能  | 更快            | 较慢（需调整平衡）           |
| 查询性能  | 相当            | 相当                  |
| 内存占用  | 略高            | 略低                  |
| 并发友好性 | 好（局部操作）       | 差（全树变动）             |
> 红黑树虽然常用，但跳表实现更简单、更稳定，适合 Redis 的高性能场景。

##### 总结

| 项目    | 说明             |
| ----- | -------------- |
| 应用场景  | ZSet、排行榜、范围查询  |
| 优势    | 简单、性能稳定、支持范围查找 |
| 数据结构  | 多层链表 + 随机高度    |
| 查询复杂度 | O(log N)       |


## 4. Redis 持久化
### 4.1 RDB 持久化
#### 4.1.2 RDB 介绍
RDB 全称 Redis Database Backup file（Redis 数据备份文件），也被叫做 Redis 数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当 Redis 实例故障重启后，从磁盘读取快照文件，恢复数据。
快照文件称为 RDB 文件，默认是保存在当前运行目录。
```bash
save # 由 Redis 主进程来执行RDB，会阻塞所有命令

bgsave # 开启子进程执行 RDB，避免主进程受到影响
```
Redis 停机时会执行一次 RDB

Redis 内部有触发 RDB 的机制，可以在 redis.conf 文件中找到，格式如下：
```bash
# 900秒内，如果至少有1个key被修改，则执行bgsave，如果是save "" 则表示禁用RDB
save 900 1
save 300 10
save 60 10000
```
RDB 的其他配置也可以在 redis.conf 文件中设置：
```bash
# 是否压缩，建议不开启，压缩也会消耗 cpu，磁盘的话不值钱
rdbcompression yes

# RDB 文件名称
dbfilename dump.rdb

# 文件保存的路径目录
dir ./
```

#### 4.1.2 RDB 的 fork 原理
bgsave 开始时会 fork 主进程得到子进程，子进程共享主进程的内存数据。完成 fork 后读取内存数据并写入 RDB 文件。
fork 采用的是 copy-on-write 技术：
- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作。
图示：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817141823363.png)

#### 4.1.3 RDB 总结
**RDB 方式 bgsave 的基本流程？**
- fork 主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的 RDB 文件
- 用新 RDB 文件替换旧的 RDB 文件
**RDB 会在什么时候执行？save 60 1000 代表什么含义？**
- 默认是服务停止时
- 代表 60 秒内至少执行 1000 次修改则触发 RDB
**RDB 的缺点？**
- RDB 执行间隔时间长，两次 RDB 之间写入数据有丢失的风险
- fork 子进程、压缩、写出 RDB 文件都比较耗时
### 4.2 AOF 持久化
#### 4.2.1 AOF 介绍
AOF 全称为 Append Only File（追加文件）。Redis 处理的每一个写命令都会记录在 AOF 文件，可以看做是命令日志文件。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817142806055.png)
AOF 默认是关闭的，需要修改 redis.conf 配置文件来开启 AOF：
```bash
# 是否开启AOF功能，默认是no
appendonly yes
# AOF 文件的名称
appendfilename "appendonly.aof"
```
AOF 的命令记录的频率也可以通过 redis.conf 文件来配置：
```bash
# 表示每执行一次写命令，立即记录到 AOF 文件
appendfsync always
# 写命令执行完先放入 AOF 缓冲区，然后表示每隔1秒将缓冲区数据写到 AOF 文件，是默认方案
appendfsync everysec
# 写命令执行完先放入 AOF 缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

| 配置项      | 刷盘时机   | 优点          | 缺点             |
| -------- | ------ | ----------- | -------------- |
| Always   | 同步刷盘   | 可靠性高，几乎不丢数据 | 性能影响大          |
| everysec | 每秒刷盘   | 性能适中        | 最多丢失1秒数据       |
| no       | 操作系统控制 | 性能最好        | 可靠性较差，可能丢失大量数据 |
#### 4.2.2 `bgrewriteaof` 命令介绍
因为是记录命令，AOF 文件会比 RDB 文件大的多。而且 AOF 会记录对同一个 key 的多次写操作，但只有最后一次写操作才有意义。通过执行 `bgrewriteaof`命令，可以让 AOF 文件执行重写功能，用最少的命令达到相同效果。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817144123503.png)
Redis 也会在触发阈值时自动去重写 AOF 文件。阈值也可以在 redis.conf 中配置：
```bash
# AOF 文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF 文件体积最小多大以上才触发重写
auto-aof-rewrite-min-size 64mb
```


### 4.3 对比
RDB 和 AOF 各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会结合两者来使用。

|             | RDB                    | AOF                              |
| ----------- | ---------------------- | -------------------------------- |
| **持久化方式**   | 定时对整个内存做快照             | 记录每一次执行的命令                       |
| **数据完整性**   | 不完整，两次备份之间会丢失          | 相对完整，取决于刷盘策略                     |
| **文件大小**    | 会有压缩，文件体积小             | 记录命令，文件体积很大                      |
| **宕机恢复速度**  | 很快                     | 慢                                |
| **数据恢复优先级** | 低，因为数据完整性不如 AOF        | 高，因为数据完整性更高                      |
| **系统资源占用**  | 高，大量 CPU 和内存消耗         | 低，主要是磁盘IO资源，但AOF重写时会占用大量CPU和内存资源 |
| **使用场景**    | 可以容忍数分钟的数据丢失，追求更快的启动速度 | 对数据安全性要求较高常见                     |
### 4.4 RDB+AOF 混合持久化
#### 4.4.1 RDB、AOF 共存
**RDB、AOF 两者可否共存？共存听谁的？**
```bash
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
```
从 redis.conf 配置文件中可以查看到，RDB和AOF两者共存时，主要听 AOF 的，因为 AOF 有更好的耐久性保证。

#### 4.4.2 数据恢复顺序和加载流程
> 在同时开启 rdb 和 aof 持久化时，重启时只会加载 aof 文件，不会加载 rdb 文件

![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250825094754060.png)
#### 4.4.3 如何选择
RDB 持久化方式能够在**指定的时间间隔**能对数据进行**快照存储**。
AOF 持久化方式**记录每次对服务器写的操作**，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF 命令以 redis 协议追加保存每次写的操作到文件末尾。
#### 4.4.4 同时开启两种持久化方式
在这种情况下，当 redis 重启的时候会**优先载入 AOF 文件**来恢复原始的数据，因为在通常情况下 **AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整**。
RDB 的数据不实时，同时使用两者时服务器重启也只会找 AOF 文件。那要不要只使用 AOF 方式？作者建议不要，因为 **RDB 更适合用于备份数据库**（AOF在不断变化不好备份），留着 rdb 作为一个万一的手段。
#### 4.4.5 RDB + AOF 混合方式
结合了 RDB 和 AOF 的优点，既能快速加载又能避免丢失更多的数据。
1. 开启混合方式设置
设置 `aof-use-rdb-preamble` 的值为 yes，yes 表示开启，设置为 no 表示禁用
2. RDB + AOF 的混合方式 ----> 结论：**RDB 镜像做全景持久化，AOF 做增量持久化**
先使用 RDB 进行快照存储，然后使用 AOF 持久化记录所有的写操作，当重写策略满足或手动触发重写的时候，将最新的数据存储为新的 RDB 记录。这样的话，重启服务的时候会从 RDB 和 AOF 两部分恢复数据，既保证了**数据完整性**，又**提高了恢复数据的性能**。简单来说：混合持久化方式产生的文件一部分是 RDB 文件，一部分是 AOF 文件。----> **AOF 包括了 RDB 头部 + AOF 混写**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250825101043824.png)

## 5. Redis 事务
### 5.1 介绍
Redis 事务：可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，**按顺序地串行化执行而不会被其它命令插入，不许加塞**
### 5.2 应用
一个队列中，一次性、顺序性、排他性的执行一系列命令
### 5.3 Redis 事务 VS 数据库事务
**单独的隔离操作**：Redis 的事务仅仅是保证事务里的操作会被连续独占的执行，redis 命令执行是单线程架构，在执行完事务内所有指令前是不可能再去同时执行其他客户端的请求的。
**没有隔离级别的概念**：因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这种问题
**不保证原子性**：Redis 的事务不保证原子性，也就是不保证所有指令同时成功或同时失败，只有决定是否开始执行全部指令的能力，没有执行到一半进行回滚的能力
**排它性**：Redis 会保证一个事务内的命令依次执行，而不会被其他命令插入
### 5.4 使用
#### 5.4.1 常用命令

| 命令                  | 描述                                                 |
| ------------------- | -------------------------------------------------- |
| DISCARD             | 取消事务，放弃执行事务块内的所有命令                                 |
| EXEC                | 执行所有事务块内的命令                                        |
| MULTI               | 标记一个事务块的开始                                         |
| UNWATCH             | 取消 WATCH 命令对所有 key 的监视                             |
| WACHT key [key ...] | 监视一个（或多个）key，如果在事务执行之前这个（或这些）key 被其他命令所改动，那么事务将被打断 |

#### 5.4.2 常见情况
##### 1. 正常执行
```bash
MULTI
...
EXEC
```
##### 2. 放弃事务
```bash
MULTI
...
DISCARD
```

##### 3. 全体连坐
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250903152219402.png)

##### 4. 冤头债主
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250903152651920.png)

与传统数据库事务有区别：不一定要么一起成功要么一起失败（即不具有原子性）
##### 5. watch 监控
**介绍**
Redis 使用 watch 来提供乐观锁定，类似于 CAS（Check-and-Set）
> 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会 block 直到它拿到锁。
> 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据
> **乐观锁策略：提交版本必须 大于 记录当前版本才能执行更新**

**watch 用法**
redis 的乐观锁是通过`watch`实现的
- `watch`会在执行事务前监视一个或多个 key
- 如果事务执行前这些 key 被其他客户端修改了，事务会自动失败
- 程序需要重新读取数据、重新执行事务
类似于 Mysql 的乐观锁版本号机制。

```bash
WATCH key1 [key2 ...]
```
- 监视一个或多个 key
- 监视会一直有效，直到：
	- 执行`EXEC`
	- 执行`DISCARD`
	- 执行`UNWATCH`
	- 连接断开

**unwatch 用法**
```bash
UNWATCH
```
- 取消对所有 key 的监视
- 如果你决定放弃乐观锁，必须在事务前调用
- 一旦执行`EXEC`或`DISCARD`，`UNWATCH`会被自动触发
**小结**
一旦执行了`EXEC`，之前加的监控锁都会被取消掉了，当客户端连接丢失的时候（比如退出链接），所有东西都会被取消监视 

### 5.5 总结
**开启**：以`MULTI`开始一个事务
**入队**：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
**执行**：由`EXEC`命令触发事务


## 6. Redis 管道
### 6.1 介绍
管道（pipeline）可以一次性发送多条命令给服务端，服务端依次处理完毕后，通过一条响应一次性将结果返回，通过减少客户端与 redis 的通信次数来实现降低往返延时时间。pipeline 实现的原理是队列，先进先出特性就保证数据的顺序性。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250907174912947.png)
> 批处理命令变种优化措施，类似 Redis 的原生批命令（mget 和 mset）

### 6.2 案例演示
### 6.3 总结
#### 6.3.1 Pipeline 与原生批量命令(mset等 )对比
- 原生批量命令是原子性（例如：mset，mget），**pipeline 是非原子性**
- 原生批量命令一次只能执行一种命令，pipeline 支持批量执行不同命令
- 原生批命令是服务端实现，而 pipeline 需要服务端与客户端共同完成

#### 6.3.1 Pipeline 与事务对比
- 事务具有原子性，管道不具有原子性
- 管道一次性将多条命令发送到服务器，事务是一条一条的发，事务只有在接收到`exec`命令后才会执行，管道不会
- 执行事务时会阻塞其他命令的执行，而执行管道中的命令时不会
#### 6.3.1 使用 Pipeline 注意事项
- pipeline 缓冲的指令只是会依次执行，不保证原子性，如果执行中指令发生异常，将会继续执行后续的指令
- 使用 pipeline 组装的命令个数不能太多，不然数据量过大客户端阻塞的时间可能过久，同时服务端此时也被迫回复一个队列答复，占用很多内存

## 7. Redis 发布订阅
### 7.1 介绍
定义：是一种消息通信模式：发送者（PUBLISH）发送消息，订阅者（SUBSCRIBE）接收消息，可以实现进程间的消息传递

### 7.2 应用场景
**Redis 客户端可以订阅任意数量的频道，类似微信关注多个公众号**
当有新消息通过 PUBLISH 命令发送给频道 channel1 时：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250915112844360.png)
所有订阅了频道 channel1 的客户端均会接收到新消息

**发布/订阅其实是一个轻量的队列，只不过数据不会被持久化，一般用来处理实时性较高的异步消息**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250915113038899.png)


### 7.3 常用命令
| 序号  | 命令及描述                                              | 描述               | 备注                               | 响应                                          |
| --- | -------------------------------------------------- | ---------------- | -------------------------------- | ------------------------------------------- |
| 1   | `PSUBSCRIBE pattern [pattern ...]`<br>             | 订阅一个或多个符合给定模式的频道 |                                  |                                             |
| 2   | `PUBSUB subcommand [argument [argument ...]] `<br> | 查看订阅与发布系统状态      |                                  |                                             |
| 3   | `PUBLISH channel message`<br>                      | 将信息发送到指定的频道      |                                  |                                             |
| 4   | `PUNSUBSCRIBE [pattern [pattern ...]]`<br>         | 退订所有给定模式的频道      |                                  |                                             |
| 5   | `SUBSCRIBE channel [channel ...]`<br>              | 订阅给定的一个或多个频道的信息  | **推荐先执行订阅后再发布，订阅成功之前发布的消息是收不到的** | 订阅的客户端每次可以收到一个3个参数的消息：消息的种类、始发频道的名称、实际的消息内容 |
| 6   | `UNSUBSCRIBE [channel [channel ...]]`<br>          | 指退订给定的频道         |                                  |                                             |
### 7.4 总结
 Redis 可以实现消息中间件 MQ 的功能，通过发布订阅实现消息的引导和分流。但不推荐使用该功能，应交给专业的 MQ 中间件处理。
 **Pub/Sub 缺点**：
 1. 发布的消息在 Redis 系统中不能持久化，因此，必须先执行订阅，再等待消息发布。如果先发布了消息，那么该消息由于没有
 2. 消息只管发送，对于发布者而言消息是即发即失的，不管接收，也没有 ACK 机制，无法保证消息的消费成功。
 以上的缺点导致 redis 的 pub/sub 模式就像个小玩具，在生产环境中几乎无用武之地，为此 Redis5.0 版本新增了 Stream 数据结构，不但支持多播，还支持数据持久化，相比 Pub/Sub 更加的强大。

## 8. Redis 主从（replica）
### 8.1 搭建主从架构
单节点 Redis 的并发能力是有上限的，要进一步提高 Redis 的并发能力，就需要搭建主从集群，实现读写分离。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817150443146.png)
**总结**：
假设有A、B两个 Redis 实例，如何让 B 作为 A 的 slave 节点？
- 在 B 节点执行命令： `slaveof A的IP A的port`

### 8.2 主从数据同步原理
#### 8.2.1 全量同步
主从第一次同步是**全量同步**：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817160920399.png)

**master 如何判断 slave 是不是第一次来同步数据？** 这里会用到两个很重要的概念：
- `Replication id`：简称 replid，是数据集的标记，id 一致则说明是同一数据集。每一个 master 都有唯一的 replid，slave 则会继承 master 节点的 replid。
- `offset`：偏移量，随着记录在 repl_baklog 中 的数据增多而逐渐增大。slave 完成同步时也会记录当前同步的 offset。如果 slave 的offset 小于 master 的 offset，说明 slave 数据落后于 master，需要更新。
因此 slave 做数据同步，必须向 master 声明自己的 replication id 和 offset，master 才可以判断到底需要同步哪些数据

> master 如何判断 slave 节点是不是第一次来做数据同步？

![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817161822999.png)

**总结：**
简述全量同步的流程？
- slave 节点请求增量同步
- master 节点判断 replid，发现不一致，拒绝增量同步
- master 将完整内存数据生成 RDB，发送 RDB 到 slave
- slave 清空本地数据，加载 master 的 RDB
- master 将 RDB 期间的命令记录在 repl_backlog，并持续将 log 中的命令发送给 slave
- slave 执行接收到的命令，保持与 master 之间的同步
#### 8.2.2 增量同步
主从第一次同步是全量同步，但如果 slave 重启后同步，则执行增量同步
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817202519527.png)

> 注意：`repl_backlog` 大小有上限，写满后覆盖最早的数据。如果 slave 断开时间过久，导致数据被覆盖，则无法实现增量同步，只能再次全量同步。

可以从以下几个方面来优化 Redis 主从集群：
- 在 master 中配置 `repl-diskless-sync yes` 启用**无磁盘复制**，避免全量同步时的磁盘IO
> 无磁盘复制：master fork 出一个子进程直接在内存里生成 RDB 数据流；不落到磁盘，而是直接通过网络 socket 推送给从节点；从节点收到数据后加载到内存，完成同步
- Redis 单节点上的内存占用不要太大，减少 RDB 导致的过多磁盘IO
- 适当提高 repl_backlog 的大小，发现 slave 宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个 master 上的 slave 节点数量，如果实在是太多 slave，则可以采用主-从-从链式结构，减少 master 压力
主-从-从链式结构：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817203610818.png)

**总结**
简述全量同步和增量同步区别？
- 全量同步：master 将完整内存数据生成 RDB，发送 RDB 到 slave。后续命令则记录在 repl_backlog，逐个发送给 slave
- 增量同步：slave 提交自己的 offset 到 master，master 获取 repl_backlog 中从 offset 之后的命令给 slave
什么时候执行全量同步？
- slave 节点第一次连接 master 节点时
- slave 节点断开时间太久，repl_backlog 中的 offset 已经被覆盖时
什么时候执行增量同步？
- slave 节点断开又恢复，并且在 repl_backlog 中能找到 offset 时

## 9. Redis 哨兵（sentinel）
### 9.1 哨兵的作用和原理
#### 9.1.1 哨兵的作用
Redis 提供了哨兵（Sentinel）机制来实现主从集群的自动故障恢复。哨兵的结构和作用如下：
- **监控**：Sentinel 会不断检查你的 master 和 slave 是否按预期工作
- **自动故障恢复**：如果 master 故障，Sentinel 会将一个 slave 提升为 master。当故障实例恢复后也以新的 master 为主
- **通知**：Sentinel 充当 Redis 客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给 Redis 的客户端
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817205500599.png)
- **配置提供**：客户端可以连接 Sentinel 来获取当前 Master 的地址。
#### 9.1.2 服务状态监控
Sentinel 基于心跳机制检测服务状态，每隔1秒内向集群的每个实例发送 ping 命令：
- 主观下线：如果某 sentinel 节点发现某实例未在规定时间响应，则认为该实例**主观下线**
- 客观下线：若超过指定数量（quorum）的 sentinel 都认为该实例主观下线，则该实例**客观下线**。quorum 值最好超过 Sentinel 实例数量的一半。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817205913485.png)

#### 9.1.3 选举新的 master
一旦发现master 故障，sentinel 需要在 slave 中选择一个作为新的 master，选择依据是这样的：
- 首先会判断 slave 节点与 master 节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该 slave 节点
- 然后判断 slave 节点的 slave-priority 值，越小优先级越高，如果是0则永不参与选举
- 如果 slave-priority 一样，则判断 slave 节点的 offset 值，越大说明数据约新，优先级越高
- 最后是判断 slave 节点的运行 id 大小，越小优先级越高。

#### 9.1.4 实现故障转移
当选中了其中一个 slave 为新的 master 后（例如 slave），故障的转移的步骤如下：
- sentinel 给备选的 slave 节点发送 `slaveof no one` 的命令，让该节点成为 master
- sentinel 给所有其它 slave 发送 `slaveof 192.168.150.101 7002` 命令，让这些 slave 成为新 master 的从节点，开始从新的 master 上同步数据
- 最后，sentinel 将故障节点标记为 slave，当故障节点恢复后会自动成为新的 master 的 slave 节点。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250817214026062.png)

#### 9.1.5 总结
Sentinel 的三个作用是什么？
- 监控
- 故障转移
- 通知
Sentinel 如何判断一个 redis 实例是否健康？
- 每隔1秒发送一次 ping 命令，如果超过一定时间没有响应则认为是主观下线
- 如果大多数 sentinel 都认为实例主观下线，则判定服务下线
故障转移步骤有哪些？
- 首先选定一个 slave 作为新的 master，执行 `slaveof no one`
- 然后让所有节点都执行 `slaveof 新 master`
- 修改故障节点配置，添加 `slaveof 新 master`

## 10. Redis 集群（cluster）
### 10.1 介绍
**定义**：
**由于数据量过大，单个 Master 复制集**难以承担，因此需要对多个复制集进行集群，形成水平扩展，每个复制集只负责存储整个数据集的一部分，这就是 Redis 的集群，其作用是提供在多个 Redis 节点间共享数据的程序集。
**总结**：
1. Redis 集群是一个提供在多个 Redis 节点间共享数据的程序集
2. Redis 集群可以支持多个 Master

**哨兵+主从模式：**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250915134325541.png)

**集群模式：**
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250915134358186.png)
### 10.2 集群作用
- Redis 集群支持多个 Master，每个 Master 又可以挂载多个 Slave
- 由于 Cluster 自带 Sentinel 的故障转移机制，内置了高可用的支持，**无需再去使用哨兵功能**
- 客户端与 Redis 的节点连接，不再需要连接集群中所有的节点，只需要任意连接集群中的一个可用节点即可
- **槽位 slot** 负责分配到各个物理服务节点，由对应的集群来负责维护节点、插槽和数据之间的关系

### 10.3 集群算法-分片-槽位slot
#### 10.3.1 redis 集群的槽位 slot
Redis 集群的数据分片
Redis 集群没有使用一致性 hash，而是引入了哈希槽的概念。
Redis 集群有16384个哈希槽，每个key通过crc16校验后对16384取模来决定放置哪个槽。集群的每个节点负责一部分hash槽
举个例子，比如当前集群有3个节点，那么：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250917103954545.png)

#### 10.3.2 redis 集群的分片
**分片是什么？**
使用Redis集群时我们会将存储的数据分散到多台redis机器上，这称为分片；简言之，集群中的每个Redis实例都被认为是整个数据的一个分片。
**如何找到给定 key 的分片？**
为了找到给定key的分片，我们对key进行CRC16（key）算法处理并通过对总分片数量取模。然后，使用**确定性哈希函数**，这意味着给定的key**将多次始终映射到同一个分片**，我们可以推断将来读取特点key的位置。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250917103954545.png)

#### 10.3.3 槽位slot和分片的优势
**最大优势：方便扩缩容和数据分派查找**
这种结构很容易添加或者删除节点，比如要添加个节点D，需要从节点A，B，C中的部分槽到D上。如果想移除节点A，需要将A中的槽移到B和C节点上，然后将没有任何槽的A节点从集群中移除即可。由于从一个节点将哈希槽移动到另一个节点并不会停止服务，所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250917103954545.png)

#### 10.3.4 slot 槽位映射
##### 哈希取余分区
2亿条记录就是2亿个k，v，单机不行必须要分布式多机。假设有3台机器构成一个集群，用户每次读写操作都是根据公司：`hash(key) % N` 个机器台数，计算出哈希值，用来决定数据映射到哪一个节点上。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250917110806284.png)

**优点：**
简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡+分而治之的作用。
**缺点：**
原来规划好的节点，进行扩容或者缩容就比较麻烦，不管扩缩，每次数据变动导致节点有变动，映射关系需要重新进行计算，在服务器个数固定不变时没有问题。如果需要弹性扩容或故障停机的情况下，原来的取模公式就会发生变化：Hash(key)/3会变成Hash(key)/?。此时地址经过取余运算的结果将发生很大变化，根据公式获取的服务器也会变得不可控。
某个redis机器宕机了，由于台数数量变化，会导致hash取余全部数据重新洗牌。

##### 一致性哈希算法分区
**是什么**
一致性哈希算法在1997年由麻省理工学院中提出的，设计目标是为了解决分布式缓存数据变动和映射问题，某个机器宕机了，分母数量改变了，自然取余数就行
**应用**
提出一致性Hash解决方案。目的是当服务器个数发生变动时，尽量减少影响客户端到服务器的映射关系
**步骤**
① 算法构建一致性哈希环
一致性哈希环
> 一致性哈希算法必然有个hash函数并按照算法产生hash值，整个算法的所有可能哈希值会构成一个全量集，这个集合可以成为一个hash空间[0,2^32-1]，这个是一个线性空间，但是在算法中，我们通过适当的逻辑控制将它首尾相连（0=2^32），这样让它逻辑上形成了一个环形空间。

它也是按照使用取模的方法，前面笔记介绍的节点取模是对节点（服务器）的数量进行取模。而一致性 Hash 算法是对 2^32 取模，简单来说，一致性 Hash 算法将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数 H 的值空间为 0-2^32-1（即哈希值是一个 32 位无符号整型），整个哈希环如下图：整个空间按顺时针方向组织，圆环的正上方的点代表0，0点右侧的第一个点代表1，以此类推，2、3、4……直到2^32-1，也就是说0点左侧的第一个点代表2^32-1，0和2^32-1在零点中方向重合，我们把这个由 2^32 个点组成的圆环称为 Hash 环。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923160918113.png)

② redis 服务器IP节点映射
将集群中各个 IP 节点映射到环上的某一个位置。
将各个服务器使用 Hash 进行一个哈希，具体可以选择服务器的 IP 或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。假如4个节点 NodeA、B、C、D，经过 IP 地址的哈希函数计算（hash(ip)），使用 IP 地址哈希后在环空间的位置如下：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923161313397.png)

③ key 落到服务器的落键规则
当我们需要存储一个 kv 键值对时，首先计算 key 的 hash 值，hash(key)，将这个 key 使用相同的函数 Hash 计算出哈希值并确定此数据在环上的位置，**从此位置沿环顺时针“行走”**，第一台遇到的服务器就是其应该定位到的服务器，并将该键值对存储在该节点上。
如有 Object A、Object B、Object C、Object D 四个数据对象，经过哈希计算后，在环空间上的位置如下：根据一致性 Hash 算法，数据 A 会定位到 Node A 上，B 被定位到 Node B 上，C 被定位到 Node C 上，D 被定位到 Node D 上。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923161537022.png)

**优点**
① 一致性哈希算法的**容错性**
假设 Node C 宕机，可以看到此时对象 A、B、D不会受到影响。一般的，在一致性 Hash 算法中，如果一台服务器不可用，则**受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据**，其他不会受到影响。简单说，就是 C 挂了，受到影响的只有 B、C 之间的数据**且这些数据会转移到 D 进行存储**。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923162805048.png)

② 一致性哈希算法的**扩展性**
数据量增加了，需要增加一台节点 NodeX，X的位置在A和B之间，那受到影响的也就是A到X之间的数据，重新把A到X的数据录入到X上即可，不会导致hash取余全部数据重新洗牌。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923163403196.png)

**缺点**
① 一致性哈希算法的**数据倾斜**问题
一致性Hash算法在服务节点太少时，容易因为节点分布不均匀而造成数据倾斜（被缓存的对象大部分集中缓存在某一台服务器上）问题，例如系统中只有两台服务器：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923163615907.png)

**总结**
为了在节点数目发生改变时尽可能少的迁移数据
将所有的存储节点排列在首尾相接的 Hash 环上，每个 key 在计算 Hash 后会**顺时针**找到临近的存储节点存放。而当有节点加入或退出时仅影响该节点在Hash环上**顺时针相邻的后续节点**。

优点：
加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。
缺点：
数据的分布和节点的位置有关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果。

##### 哈希槽分区
**介绍**
为什么出现
为了解决一致性哈希算法的数据倾斜问题
哈希槽实质就是一个数组，数组[0,2^14-1]形成 hash slot 空间。

能干什么
解决均匀分配的问题，在数据和节点之间又加入了一层，把这层称为哈希槽（slot），用于管理数据和节点之间的关系，现在就相当于节点上放的是槽，槽里放的是数据。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923165154288.png)
槽解决的是粒度问题，相当于把粒度变大了，这样便于数据移动。哈希解决的是映射问题，使用key的哈希值来计算所在的槽，便于数据分配

多少个 hash 槽
一个集群只能有16384个槽，编号0-16383（0-2^14-1）。这些槽会分配给集群中的所有主节点，分配策略没有要求。

集群会记录节点和槽的对应关系，解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取模，余数是几key就落入对应的槽里。`HASH_SLOT = CRC16(key) mod 16384`。以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了。

**哈希槽计算**
Redis 集群中内置了 16384 个哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在 Redis 集群中放置一个 key-value时，redis先对key使用crc16算法算出一个结果然后用结果对16384求余数[ CRC16(key) % 16384]，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，也就是映射到某个节点上。如下代码，key之A 、B在Node2， key之C落在Node3上
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923165538026.png)
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250923165544291.png)

### 10.4 集群环境案例

### 10.5 集群常用操作命令和 CRC16 算法分析

- **作用**：实现**数据分片 (Sharding)**，将数据分布到多个节点，解决单机 Redis 容量和性能瓶颈，实现水平扩展。
- **核心原理**：
	- **哈希槽 (Hash Slot)**：Redis Cluster 预设了 16384 个哈希槽。
	- **数据分片**：对每个 key 计算 CRC16 校验码，然后对 16384 取模，决定该 key 存储在哪个槽。每个 Redis 节点负责一部分哈希槽。
	- **去中心化**：集群中的每个节点都知道所有其他节点负责的槽位，客户端可以连接任意节点，如果 key 不在当前节点，会被重定向到正确的节点。
## 11. 缓存穿透、击穿、雪崩
- **缓存穿透 (Cache Penetration)**
    - **现象**：查询一个**不存在**的数据。由于缓存中没有，请求会一直穿透到数据库，导致数据库压力增大。
    - **解决方案**：
        1. **缓存空对象**：如果数据库查询结果为空，也在缓存中设置一个空值（但设置较短的过期时间）。
        2. **布隆过滤器 (Bloom Filter)**：在访问缓存前，通过布隆过滤器判断数据是否存在，不存在则直接返回。
- **缓存击穿 (Cache Breakdown)**
    - **现象**：一个**热点 Key** 在失效的瞬间，大量并发请求同时涌入，直接打到数据库上。
    - **解决方案**：
        1. **加锁**：使用分布式锁，只允许一个线程去查询数据库并重建缓存，其他线程等待。
        2. **热点数据永不过期**：对热点数据不设置过期时间，或通过后台任务异步更新。
- **缓存雪崩 (Cache Avalanche)**
    - **现象**：**大量 Key** 在同一时间集中失效，导致大量请求瞬间全部打到数据库上。
    - **解决方案**：
        1. **随机化过期时间**：在基础过期时间上增加一个随机值，避免集中失效。
        2. **构建高可用缓存**：使用 Redis Cluster 或 Sentinel 模式，避免 Redis 整体宕机。
        3. **服务降级/限流**：当缓存失效时，通过限流组件（如 Hystrix、Sentinel）暂时阻止部分请求访问数据库。
## 12. 缓存一致性
- **先更新数据库，再删除缓存 (Cache-Aside Pattern)**：
    - **流程**：读请求先查缓存，没有则查数据库并写入缓存；写请求先更新数据库，然后删除缓存。
    - **优点**：相对简单，能保证最终一致性。
    - **问题**：可能存在短暂不一致（更新数据库后，删除缓存前，有读请求读到旧数据）。
    - **为什么是删除缓存而不是更新缓存？** 因为更新缓存的开销更大，且如果一个值依赖多个数据源，更新逻辑会很复杂。
- **延时双删**：
    - **流程**：先删除缓存 -> 再更新数据库 -> 延迟一段时间（如 500ms）后再次删除缓存。
    - **目的**：防止在主从数据库延时的情况下，旧数据被再次写入缓存。
- **订阅 Binlog，异步更新/删除缓存**：
    - **流程**：通过 Canal 等工具订阅 MySQL 的 binlog，当数据库发生变更时，异步通知服务去更新或删除缓存。
    - **优点**：实现了解耦，可靠性高。
    - **缺点**：架构复杂，引入了新的中间件。
## 13. 过期删除策略和内存淘汰策略
### 13.1 设置 Redis 键过期事件
### 13.2 Redis 过期时间的判定
### 13.3 过期删除策略
#### 13.3.1 定时删除
#### 13.3.2 惰性删除
#### 13.3.3 定期删除
### 13.4 Redis 过期删除策略
### 13.5 内存淘汰策略
#### 13.5.1 设置 Redis 最大内存
#### 13.5.2 设置内存淘汰方式
### 13.6 总结

当 `maxmemory` 限制被达到时，Redis 会根据配置的淘汰策略来删除 Key。
- **noeviction**：默认策略，不删除任何数据，对写操作返回错误。
- **allkeys-lru**：从所有 Key 中，移除最近最少使用的 (LRU)。
- **volatile-lru**：从设置了过期时间的 Key 中，移除最近最少使用的。
- **allkeys-random**：从所有 Key 中，随机移除。
- **volatile-random**：从设置了过期时间的 Key 中，随机移除。
- **volatile-ttl**：从设置了过期时间的 Key 中，移除剩余生存时间最短的。
- **allkeys-lfu** (Redis 4.0+): 从所有 Key 中，移除最不经常使用的 (LFU)。
- **volatile-lfu** (Redis 4.0+): 从设置了过期时间的 Key 中，移除最不经常使用的。
## 14. Redis 实现分布式锁
- **核心命令**：`SET key value NX PX milliseconds`
    - `NX`: 只在 key 不存在时才设置成功（保证原子性）。
    - `PX`: 设置过期时间（防止死锁）。
- **实现**：
    1. **加锁**：`SET lock_key random_value NX PX 30000`。如果返回 OK，则获取锁成功。`random_value` 是为了保证只有加锁的客户端才能解锁。
    2. **解锁**：使用 Lua 脚本保证原子性。先 `GET` key 的值，判断是否与自己加锁时设置的 `random_value` 相等，如果相等，则执行 `DEL`。
```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```
- **可重入锁**：可以使用 Hash 结构，`key` 是锁名，`field` 是线程 ID，`value` 是重入次数。













