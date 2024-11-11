---
title: Redis知识点总结
date: 2021-08-21
tags: 
  - notes
  - redis
categories: technology
keywords: 'basic redis'
---

# 一、安装、配置及Jedis客户端

1. 安装：[docker](https://hub.docker.com/_/redis)
2. 配置：暂无
3. Jedis客户端：
   
    ```jsx
    <dependency>
      <groupId>redis.clients</groupId> 
      <artifactId>jedis</artifactId>
      <version>5.0.0</version>
    </dependency>
    ```
    

# 二、基本功能

## 2.1 字符串(string)

字符串类型的值可以为字符串、数字或二进制。Redis提供命令支持常规的字符串操作，如`append`，`range`等。当实际类型为数字时，还可对它进行数学运算，如自增自减的原子操作。

```bash
# 自增1/自减1
set incr_t1 1
incr incr_t1
decr decr_t1

# 增加指定值/减少指定值
set incrby_t1 1
incrby incrby_t1 5
decrby decrby_t1 2
```

## 2.2 列表(list)

每个节点都包含一个字符串，可以从链表的两端推入或者弹出元素等。

- `lpush`，`rpush`：将元素推进左侧或者右侧。
- `lpop`，`rpop`：将元素从左端或者右端弹出。
- `lindex`：获取指定位置的元素。
- `lrange`：获取给定范围上的元素。

```bash
# ----普通列表操作----
# rpush key value [value...]: 将一个或多个值推入列表的右端
# 推入一个值
rpush rpushtest rpushtestvalue1
# 推入多个值
rpush rpushtest rpushtestvalue2 rpushtestvalue3
# rpushtestvalue1 rpushtestvalue2 rpushtestvalue3
lrange rpushtest 0 100

# lpush key value [value...]: 将一个或多个值推入列表的左端
lpush lpushtest lpushtestvalue1
lpush lpushtest lpushtestvalue2 lpushtestvalue3
# lpushtestvalue3 lpushtestvalue2 lpushtestvalue1
lrange lpushtest 0 100

# rpop key: 移除并返回最右端的元素
rpush rpoptest rpoptestvalue1 rpoptestvalue2 rpoptestvalue3
rpop rpoptest
# rpoptestvalue1 rpoptestvalue2
lrange rpoptest 0 100

# lpop key: 移除并返回最左端的元素
rpush lpoptest lpoptestvalue1 lpoptestvalue2 lpoptestvalue3
lpop lpoptest
# lpoptestvalue2 lpoptestvalue3
lrange lpoptest 0 100

# lindex key offset: 返回下标为offset的元素
lpush lindextest lindextestvalue1 lindextestvalue2 lindextestvalue3
# 注意这里造数据用的是lpush
# lindextestvalue3
lindex lindextest 0

# lrange key start end: 返回从start到end且包括start和end的元素
lpush lrangetest lrangetestvalue1 lrangetestvalue2 lrangetestvalue3
# 注意这里造数据用的是lpush
# lrangetestvalue3 lrangetestvalue2 lrangetestvalue1
lrange lrangetest 0 100

# ltrim key start end: 修剪列表, 只保留从start到end且包括start和end的元素
lpush ltrimtest ltrimtestvalue1 ltrimtestvalue2 ltrimtestvalue3
ltrim ltrimtest 1 1
# ltrimtestvalue2
lrange ltrimtest 0 100

# ----阻塞式列表操作----
# 阻塞式列表操作会阻塞执行命令的客户端, 直到有其他客户端给列表添加元素为止. 对于阻塞弹出命令和弹出并推入命令, 最常见的用例就是消息传递(messaging)和任务队列(task queue).
# blpop key [key ...] timeout: 从第一个非空列表中弹出位于最左端的元素, 或者在timeout秒之内阻塞并等待可弹出的元素出现
# 在一个redis-cli下执行下方命令. 命令执行后这个redis-cli开始阻塞
blpop blpoptest 100000
# 打开另一个redis-cli, 执行下方命令. 命令执行后立即看到前面一个redis-cli结束阻塞, 并输出结果
lpush blpoptest blpoptestvalue

# brpop key [key ...] timeout: 和blpop类似, 但这个是从右边弹出
brpop brpoptest 100000
rpush brpoptest rpoptestvalue

# ----列表间操作----
# rpoplpush source-key dest-key: 从source-key列表中弹出位于最右端的元素, 然后将这个元素推入dest-key列表的最左端
lpush rpoplpush-src rpoplpush-srcValue
rpoplpush rpoplpush-src rpoplpushtest-des
# rpoplpush-srcValue
lindex rpoplpushtest-des 0

# brpoplpush source-key dest-key timeout: 从source-key列表中弹出位于最右端的元素, 然后将这个元素推入dest-key列表的最左端. 如果source-key为空, 那么redis客户端将会阻塞等待timeout秒
# redis-cli 1
brpoplpush brpoplpushtest-src brpoplpushtest-des 100000
# 打开另一个redis-cli 2
lpush brpoplpushtest-src brpoplpushtest-srcValue
# brpoplpushtest-srcValue
lindex brpoplpushtest-des 0
```

## 2.3 集合(set)

包含独一无二的字符串元素的数据结构，可以添加，获取或者移除单个元素，检查元素是否存在于集合中，**计算交集，并集，差集，**从集合里面随机获取元素等。

- `sadd`：将元素添加到集合。
- `srem`：从集合里面移除元素。
- `sismember`：快速检查一个元素是否已经在集合中。
- `smembers`：获取集合包含的所有元素（如果集合包含的元素非常多，这个命令的执行速度可能会很慢，所以应该谨慎地使用这个命令）。

```bash
# ----基本集合操作----
# sadd key item [item ...]: 将一个或多个元素添加到集合里面
sadd saddtest saddtestmember1 saddtestmember2
# saddtestmember1 saddtestmember2
smembers saddtest

# srem key item [item ...]: 从集合里面移除一个或多个元素
sadd sremtest sremtestvalue1 sremtestvalue2
# sremtestvalue1 sremtestvalue2
smembers sremtest
srem sremtest sremtestvalue1
# sremtestvalue2
smembers sremtest

# sismember key item: 检查元素item是否在于集合key里面
sadd sismembertest sismembertestvalue1 sismembertestvalue2
# 0 表示不在集合中
sismember sismembertest aaa
# 1 表示在集合中
sismember sismembertest sismembertestvalue1

# scard key: 返回集合包含的元素数量
sadd scardtest scardtestvalue1 scardtestvalue2
# 2 两个元素
scard scardtest

# smembers key: 返回集合包含的所有元素
sadd smembertest smembertestvalue1 smembertestvalue2
# smembertestvalue1 smembertestvalue2
smembers smembertest

# spop key: 随机地移除集合中的一个元素
# 先造多点测试数据
sadd spoptest spoptestvalue1 spoptestvalue2
sadd spoptest spoptestvalue3 spoptestvalue4
sadd spoptest spoptestvalue5 spoptestvalue6
sadd spoptest spoptestvalue7 spoptestvalue8
# 会随机移除一个元素, 并返回这个移除的元素
spop spoptest

# smove source-key dest-key item: 如果source-key包含item, 那么从source-key中移除item, 并将item添加到集合dest-key中
sadd smovetest-src smovetestValue1 smovetestValue2
smove smovetest-src smovetest-dest smovetestValue2
# smovetestValue1
smembers smovetest-src
# smovetestValue2
smembers smovetest-dest

# ----处理多个集合操作----
# sdiff key [key ...]: 返回存在于第一个集合, 但不存在于其他集合中的元素(差集运算)
sadd sdifftest-0 sdifftestValue1 sdifftestValue2
sadd sdifftest-1 sdifftestValue1
# sdifftestValue2
sdiff sdifftest-0 sdifftest-1

# sdiffstore dest-key key [key ...]: 将存在于第一个集合, 但不存在于其他集合中的元素存储到dest-key键里面
sadd sdiffstoretest-0 sdiffstoretestValue1 sdiffstoretestValue2
sadd sdiffstoretest-1 sdiffstoretestValue1
sdiffstore sdiffstoretest-dest sdiffstoretest-0 sdiffstoretest-1
# sdiffstoretestValue2
smembers sdiffstoretest-dest

# sinter key [key ...]: 返回同时存在于所有集合中的元素(交集运算)
sadd sintertest-0 sintertestValue1 sintertestValue2
sadd sintertest-1 sintertestValue2 sintertestValue3
# sintertestValue2
sinter sintertest-0 sintertest-1

# sinterstore dest-key key [key ...]: 将同时存在于所有集合中的元素存储到dest-key键里面
sadd sinterstoretest-0 sinterstoretestValue1 sinterstoretestValue2
sadd sinterstoretest-1 sinterstoretestValue2 sinterstoretestValue3
sinterstore sinterstoretest-dest sinterstoretest-0 sinterstoretest-1
# sinterstoretestValue2
smembers sinterstoretest-dest

# sunion key [key ...]: 返回至少存在于一个集合中的元素(并集操作)
sadd suniontest-0 suniontestValue1 suniontestValue2
sadd suniontest-1 suniontestValue2 suniontestValue3
# suniontestValue1 suniontestValue2 suniontestValue3
sunion suniontest-0 suniontest-1

# sunionstore dest-key key [key ...]: 将至少存在于一个集合中的元素存储到dest-key键里面
sadd sunionstoretest-0 sunionstoretestValue1 sunionstoretestValue2
sadd sunionstoretest-1 sunionstoretestValue2 sunionstoretestValue3
sunionstore sunionstoretest-dest sunionstoretest-0 sunionstoretest-1
# sunionstoretestValue1 sunionstoretestValue2 sunionstoretestValue3
smembers sunionstoretest-dest
```

## 2.4 哈希(hash)

- `hset`：将一个键值对添加到一个hashmap中。
- `hget`：获取指定hashmap中指定键的键值对的值。
- `hgetall`：获取指定hashmap中所有键值对。
- `hdel`：删除指定键值对中指定的键的键值对。

注意：如果想要获取所有键值对，`hgetall`和`hkeys`与`hvals`都能实现相同的功能，但是如果散列包含的值非常大，那么可以先使用`hkeys`取出包含的所有键，然后再使用`hget`一个接一个地去除键的值，从而避免一次获取多个大体积的值导致服务器阻塞。

```bash
# hmget key field1 [field2 ...]: 从散列key里面获取一个field或者多个field的值
hmset hmgettest hmgettestField1 hmgettestValue1 hmgettestField2 hmgettestValue2
# hmgettestValue1 hmgettestValue2
hmget hmgettest hmgettestField1 hmgettestField2

# hmset key field1 fieldValue1 [field2 fieldValue2 ...]: 为散列里面的一个或多个键设置值
hmset hmsettest hmsettestField1 hmsettestValue1 hmsettestField2 hmsettestValue2
# hmsettestValue1 hmsettestValue2
hmget hmsettest hmsettestField1 hmsettestField2

# hdel key field1 [field2 ...] 删除散列key里面的一个或多个field
hmset hdeltest hdeltestField1 hdeltestValue1 hdeltestField2 hdeltestValue2
hdel hdeltest hdeltestField1
# nil 因为该field已经被删除了
hmget hdeltest hdeltestField1

# hlen key: 返回散列key包含的值对数量
# 2
hmset hlentest hlentestField1 hlentestValue1 hlentestField2 hlentestValue2
hlen hlentest

# 创建测试数据供下面几个命令使用
hmset hashtest hexistsTestField existsTestFieldValue hincrbyTestField 1 hincrbyfloatTestField 2
# hexists key field: 检查给定键是否存在于散列中
# 1 表示存在
hexists hashtest hexistsTestField
# 0 表示不存在
hexists hashtest hexistsTestField-noexists

# hkeys key: 获取散列包含的所有键
# hexistsTestField hincrbyTestField hincrbyfloatTestField
hkeys hashtest

# hvals key: 获取散列包含的所有值
# existsTestFieldValue hincrbyTestValue hincrbyfloatTestValue
hvals hashtest

# hgetall key: 获取散列包含的所有键值对
# hexistsTestField existsTestFieldValue ... 键值对的方式先后出现
hgetall hashtest

# hincrby key field increment: 将键key的散列的field的值加上整数increment
# 4 表示现在的值为4
hincrby hashtest hincrbyTestField 3

# hincrbyfloat key field increment: 将键key的散列的field的值加上浮点数increment.
# 6.2 表示现在的值为6.2
hincrbyfloat hashtest hincrbyfloatTestField 2.2
```

## 2.5 有序集合(zset)

有序集合和散列一样，都用于存储键值对：有序集合的键被称为成员(member)，**每个成员都是各不相同的；**而有序集合的值则称为分值(score)，**分值必须为浮点数。**有序集合是Redis里面唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排列顺序来访问元素的结构。

- `zadd`：添加一个元素，注意区分`member`和`score`。
- `zrange`：获取元素集合，可以根据分值进行排序：`zrange zset-key 0 -1 withscores`。
- `zrangebyscore`：根据分值来获取有序集合中的一部分元素。`zrangebyscore zset-key 0 800 withscores`：获取分值在0 到 800 的元素。
- `zrem`：如果给定成员存在有序集合，那么移除这个成员。

散列存储的是键和值之间的映射，而有序集合存储的是成员与分值之间的映射，并提供了分值的处理命令。以及根据分值大小有序地获取或扫描成员和分值的命令。

```bash
# 创造测试数据
zadd zsettest 1.9 member1 2.8 member2

# zadd key score member [score member ...]: 将带有给定分值的成员添加到有序集合里面
zadd zsettest 1.2 member3 2.9 member4

# zrem key member [member ...]: 从有序集合里面移除给定的成员, 并返回被移除成员的数量
zrem zsettest member1

# zcard key: 返回有序集合包含的成员数量
zcard zsettest

# zincrby key increment member
# zcount key min max
# zrank key member
# zscore key member
# zrange key start stop [withscores]: zrange key 0 -1 表示返回所有成员，不计算分值区间
# zrevrank key member
# zrevrange key start stop [withscores]
# srangebyscore key min max [withscores] [limit offset count]
# zrevrangebyscore key max min [withscores] [limit offset count]
# zremrangebyrank key start stop
# zremrangebyscore key min max
```

## 2.6 bitmaps

redis提供bitmaps来实现对位的操作，它不是一种数据结构，实际上它就是字符串。bitmaps单独提供了一套命令，所以在redis中使用bitmaps和使用字符串的方法不太相同。

```bash
# 使用方式
setbit key offset value

# 如下两条命令生成这样的bitmap：01001
setbit bitmapstest 1 1
setbit bitmapstest 4 1

# 获取值
getbit bitmapstest 1

# 获取指定范围值为1的个数
bitcount bitmapstest 5 10

# 其他操作：两个key交集并集等，第一个值为target的偏移量等操作，具体查文档
```

## 2.7 hyperloglog

用在可容忍一定精准度下的统计操作。消耗的内存极小，大约为12kb。redis中它不是一种数据结构，实际上它就是字符串。

```bash
# 使用方式
# 往hyperloglog中添加元素，成功返回1
pfadd key element [elements ...]

# 计算hyperloglog独立总数
pfcount key [key ...]

# 求出多个hyperloglog的并集并赋值给destkey
pfmerge destkey sourcekey [sourcekey ...]
```

## 2.8 geo

支持存储地理位置信息的功能，存储的信息会包括地理位置的经度、纬度、成员。redis中它不是一种新的数据结构，实际上是zset。

```bash
# 增加地理位置
geoadd cities:locations 116.28 39.55 beijing
geoadd cities:locations 102.11 35.12 shanghai

# 获取地理位置信息
geopos cities:locations beijing

# 获取两个地理位置间距离
geodist cities:locations beijing shanghai
```

## 2.9 发布订阅

订阅者订阅频道，发送者负责向频道发送消息，每当有消息被发送至给定频道时，频道的所有订阅者都会收到消息。Redis提供如下五个发布与订阅命令：

| 命令 | 用例与描述 |
| --- | --- |
| subscribe | subscribe channel [channel …]: 订阅给定的一个或多个频道 |
| unsubscribe | unsubscribe [channel [channel …]]: 退订给定的一个或多个频道, 如果执行时没有给定任何频道, 那么退订所有频道 |
| publish | publish channel message: 向给定频道发送消息 |
| psubscribe | psubscribe pattern [pattern …]: 订阅与给定模式相匹配的所有频道 |
| punsubscribe | punsubscribe [pattern [pattern …]]: 退订给定的模式, 如果执行时没有给定任何模式, 那么退订所有模式 |

```bash
# 在一个redis-cli中订阅两个频道:
subscribe testchannel1 testchannel2

# 在另一个redis-cli中执行发布消息:
publish testchannel1 'hello from publisher'
publish testchannel2 'hello from the same publisher'

# 在第一个redis-cli可以收到第二个redis-cli中发布的消息
```

和专业消息队列系统比，无法实现消息堆积和回溯，负载均衡也是个问题。如果业务场景可以容忍这些问题，那么可以使用。

## 2.10 事务

redis中有5个事务相关的命令，分别是`watch`，`unwatch`，`multi`，`exec`和`discard`。事务可以让一个客户端在不被其他客户端打断的情况下执行多个命令。**被`multi`命令和`exec`命令包围的所有命令会一个接一个地执行，直到所有命令都执行完毕为止，当一个事务执行完毕之后，Redis才会处理其他客户端的命令。**

```bash
multi
# QUEUED 说明命令被入队了
set multitest 100
# QUEUED
get multitest
# 这时候才真正执行
exec

# 在exec之前, 如果使用另一个redis-cli来get或者set multitest的话, 是可以正常作用到这个key的
```

`watch`：如果在事务开始之前`watch`了某个key，那么在事务执行期间，该key的值发生了改变，`exec`时不会执行事务。

redis事务不保证原子性：

- queue命令时出现错误（如语法错误），整个事务无法执行
  
    ```bash
    multi
    set key1 value
    # "QUEUED"
    
    put key2 value
    # 报错，事务不能执行
    ```
    
- queue命令时没有出现错误，但是运行时出现错误，错误的命令不执行，其他命令正常执行
  
    ```bash
    multi
    set key1 value
    # "QUEUED"
    
    incr key1 
    # "QUEUED"
    
    set key2 value
    # "QUEUED"
    
    exec
    
    # 执行到incr key1时会报错，但是仍然会继续执行set key2 value
    ```
    

## 2.11 lua脚本

lua脚本在redis中是原子执行的，执行过程中不会插入其他命令。lua脚本可以帮助用户创造出自己定制的命令，并可以将这些命令常驻在redis内存中，实现复用效果。lua脚本可将多条命令一次性打包，有效地减少网络开销。

使用redis脚本的方式：`eval`和`evalsha`，两者的区别是，后者需要先使用`load`命令将lua脚本加载到redis中，完成后会得到一个sha1，后续执行`evalsha sha1 ...`即可执行脚本。

```bash
eval 脚本内容 key个数 key列表 参数列表
```

lua脚本里面可以使用`redis.call(...)`来执行redis命令，如`redis.call("set", "hello", "world")`。也可使用`redis.pcall(...)`，两者区别在于：执行失败时，前者脚本执行结束并返回错误，后者忽略错误继续执行脚本。

## 2.12 pipeline

命令执行需要经过：发送命令 -> 命令排队 -> 命令执行 -> 返回结果，n个命令需要经过n次该流程。使用pipeline，客户端可以在发送命令时一次性发送所有命令，服务端执行完所有命令后，再一次性将结果返回。由于服务端会缓存命令结果到所有命令都执行完毕，因此命令数量过多时会很耗费内存。

## 2.13 其他命令

- `sort`：对列表和集合进行排序，返回排序后的数据（不改变原有数据）：
  
    ```bash
    # 排序列表
    rpush sortforlisttest 10 2 3 9 1
    # 1 2 3 9 10
    sort sortforlisttest
    
    # 排序集合
    sadd sortforsettest 2 3 1 9 4 10
    # 1 2 3 4 9 10
    sort sortforsettest
    ```
    
- 键的过期时间：可以通过`del key`命令显示删除无用的键，也可以为键设置过期时间，让这个键在指定时间后被自动删除
  
    ```bash
    # persist key: 移除键的过期时间
    # ttl key: 查看给定键距离过期还有多少秒
    # expire key seconds: 让给定键在指定的秒数之后过期
    # expire key timestamp: 让给定的键在timestamp时间戳的时候过期
    # pttl key: 距离过期还有多少毫秒
    # pexpire key millseconds: 让给定键在指定的毫秒数之后过期
    # pexpire key timestamp: 在毫秒级的timestamp时间戳的时候过期
    ```
    

# 三、持久化

Redis持久化是将Redis内存数据库中的数据保存到磁盘以便在重启Redis时进行数据恢复的过程。因为Redis是内存数据库，数据通常存储在RAM中，如果没有持久化的过程，服务器重启后数据就全丢了。

## 3.1 RDB

RDB文件，一个紧凑压缩的二进制文件，代表Redis在某个时间点上的整个数据集（数据快照）。可手动触发生成或自动触发生成。

- 手动触发：
    - `save`命令：阻塞当前Redis服务器，直到RDB过程完成为止，对内存比较大的实例会造成长时间阻塞（因此线上不建议使用）。
    - `bgsave`命令：Redis进程执行fork操作创建子进程，通过cow来处理持久化逻辑，阻塞只发生在fork阶段，时间很短。
- 自动触发：
    - 设置`save`配置选项: 如`save 60 10000`会从Redis最近一次创建快照之后开始算起, 当“60秒之内有10000次写入(距离上次成功生成快照已经过了60秒, 且在此期间有超过10000次写入)”这个条件满足的时候, 自动触发`BGSAVE`命令.
    - 从节点执行全量复制操作，主节点自动执行`bgsave`生成RDB文件并发送给从节点。
    - 执行`shutdown`命令时，如果没有开启AOF持久化功能自动执行`bgsave`

优缺点：

- 优点：性能高；冷启动快速，因为不需要逐个重放命令。
- 缺点：数据丢失，如果服务器在两次RDB快照之间崩溃，那么会丢失最后一次快照后的所有数据。

## 3.2 AOF

以追加日志的方式，将每个写操作追加到日志文件中，记录数据变更的命令。重启时逐个执行AOF文件中的命令即可恢复数据状态。

优缺点：

- 优点：数据完整性高
- 缺点：性能慢，包括每个命令都要写以及崩溃恢复的逐个命令重放

# 四、主从复制、哨兵、集群

## 4.1 主从复制

分布式系统中为了解决单点问题，通常会把数据复制到多个副本部署到其他机器，满足故障恢复和负载均衡等需求。主从复制下从节点的加入主要解决了两个问题：一是作为从节点的备份，转移主节点的故障。二是读写分离分担主节点的压力。

### 4.1.1 配置

参与复制的Redis实例被划分为主节点（master）和从节点（slave）。默认情况下每个节点都是主节点。每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点。

建立复制：

1. 配置文件加入`slaveof {masterHost} {masterPort}`随Redis启动生效
2. 在redis-server启动命令后加入`-slaveof {masterHost} {masterPort}`生效
3. 直接使用命令： `slaveof {masterHost} {masterPort}`

断开复制：

1. 在从节点上执行`slaveof no one`
2. 断开复制之后数据不会被清除，但是如果从新连接主节点，则当前节点数据会被清除

### 4.1.2 拓扑

1. 一主一从：主节点出现宕机时，从节点提供故障转移
2. 一主多从：读写分离，读占比比较多场景都可用，耗时长的读操作可在从服务器做，如`keys`命令
3. 树状主从：避免多从节点同步对主节点带来性能干扰

### 4.1.3 原理

从节点使用`slaveof`命令和主节点建立复制关系之后，复制过程开始运作，整个过程为：

1. 保存主节点信息：ip，port，状态等
2. 和主节点建立网络连接
3. 发送ping命令检查网络连接是否可用
4. 主节点将数据全部发送给从节点
5. 从节点获取到初始化数据后，后续主节点会持续将写命令发送给从节点，保证主从数据一致性

全量复制：用于初次复制场景，将主节点数据一次性发送给从节点

部分复制：用于主从复制中因网络闪断等原因造成的数据丢失场景，从节点再次连上主节点之后，如果条件允许，主节点会补发数据给从节点。因补发的数据远远小于全量数据，可以有效避免全量复制的过高开销。

### 4.1.4 故障转移

1. 客户端与主节点的连接失败，从节点与主节点的连接失败，复制中断
2. 挑选一个从节点，执行`slaveof no one`使其成为新的主节点
3. 更新应用方的主节点信息为新挑选的主节点
4. 使用`slaveof`命令，让其他从节点去复制新的主节点
5. 原来的主节点恢复后，让他复制新的主节点

## 4.2 哨兵

主从复制模式下，一旦主节点由于故障不能提供服务，需要人工将从节点晋升为主节点，同时还要通知应用方更新主节点地址，生产上这种处理方式可用性很差，Redis2.8以后提供的Redis Sentinel架构就是Redis的高可用实现方案，用于处理这种问题。就是说Sentinel的存在就是为了自动处理故障发现和故障转移的。

Sentinel架构：

1. 包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其余的Sentinel节点进行监控
2. 当发现节点不可达时，会对节点做下线标识，如果被标识的节点是主节点，则会和其他Sentinel节点进行协商
3. 当大多数Sentinel节点都认为主节点不可达时，它们会选举出一个Sentinel节点来完成自动故障转移的工作，并将这个变化通知Redis应用方

### 4.2.1 Leader节点选举

对主节点做出客观下线之后，需要一个Sentinel节点来完成故障转移，这个节点由选举得出，选举算法为Raft。

## 4.3 集群

Redis的分布式解决方案，有效解决单机内存，并发，流量等瓶颈。思想为：定义范围哈希槽（16384个），集群内每个节点对应一部分的槽，所有的键根据哈希函数映到槽内。

### 4.3.1 节点通信

分布式存储中需要提供维护节点元数据信息的机制，所谓元数据是指：节点负责哪些数据，是否出现故障等状态信息。常见的元数据维护方式分为：集中式和P2P方式。Redis集群采用P2P的Gossip协议，原理是节点彼此不断通信交换信息，一段时间后所有的节点都会知道集群完整信息。

Gossip协议：职责为信息交换，信息交换的载体为节点彼此发送的Gossip消息，类型有ping消息、pong消息、meet消息、fail消息等。

- meet消息：用于通知新节点加入，消息发送者通知接收者加入到当前集群，meet消息通信正常完成后，接收节点会加入到集群中并进行周期性的ping、pong消息交换。
- ping消息：集群内交换最频繁的消息，集群内每个节点每秒向多个其他节点发送ping消息，用于检测节点是否在线和交换彼此状态信息。ping消息发送封装了自身节点和部分其他节点的状态数据。
- pong消息：当接收到ping、meet消息时，作为响应消息回复给发送方确认消息正常通信。pong消息内部封装了自身状态数据。节点也可以向集群内广播自身的pong消息来通知整个集群对自身状态进行更新。
- fail消息：当节点判定集群内另一个节点下线时，会向集群内广播一个fail消息，其他节点接收到fail消息后会把对应节点更新为下线状态。

### 4.3.2 伸缩原理

Redis集群可以实现对节点的灵活上下线控制，原理可抽象为槽和对应数据在不同节点之间灵活移动。扩容从宏观角度看就是将一些槽分配给新的节点并将属于这些槽的数据移动到新的节点，缩容反之。扩容的步骤为：

1. 启动新的节点
2. 使用`cluster meet`命令将新节点加入集群，此时新加入的节点都是都是主节点，但由于没有负责的槽，不能接受任何读写操作。可以为它迁移槽和数据实现扩容，也可以让它作为其他主节点的从节点负责故障转移
3. 迁移槽和数据（迁移过程集群可以正常提供读写服务）：
    - 对目标节点发送`cluster setslot {slot} importing {sourceNodeId}`命令，让目标节点准备导入槽的数据
    - 对源节点发送`cluster setslot {slot} migrating {targetNodeId}`命令，让源节点准备导出槽的数据
    - 源节点循环执行`cluster getkeysinslot {slot} {count}`命令，获取count个属于槽`{slot}`的键
    - 在源节点上执行`migrate {targetIp} {targetPort}...`命令把获取的键通过流水线机制批量迁移到目标节点上
    - 重复执行步骤三和四直到槽下所有的键值数据迁移到目标节点
    - 向集群内所有主节点发送`cluster setslot {slot} node {targetNodeId}`命令，通知槽分配给目标节点。为保证槽节点映射变更及时传播，需要遍历发送给所有主节点更新被迁移的槽指向新节点
4. 为扩容的主节点设置从节点

上述步骤都要手动执行，可以使用redis-trib工具自动执行

缩容步骤略。

### 4.3.3 请求路由

1. 集群模式下，Redis接收任何键相关命令时首先计算键对应的槽，再根据槽找出对应的节点，如果节点是自身，则处理键命令，否则回复MOVED重定向错误，通知客户端请求正确的节点。使用redis-cli时加入`c`参数支持自动重定向。
2. smart客户端：本地维护slot到节点的映射，每次访问key时，计算key对应的slot就可以拿到节点直接访问，当收到MOVED响应时会更新本地映射。
3. JedisCluster：Java下的smart客户端

### 4.3.4 故障转移

1. 故障发现：
    - 集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong消息作为响应。如果在cluster-node-timeout时间内通信一直失败，则发送节点会认为接收节点存在故障，把接收节点标记为主观下线状态
    - 某个节点判断另一个节点主观下线后，相应节点状态会随消息在集群内传播，当半数以上持有槽的主节点都标记某个节点是客观下线时，触发客观下线流程
    - 客观下线：向集群广播一条fail消息，通知所有的节点将故障节点标记为客观下线（谁发的消息？）
2. 故障恢复：故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它的从节点中选出一个替换它，从而保证集群高可用。

# 五、应用场景

基本上有几种应用场景：

1. 利用Redis数据存储于内存而内存高速读写的特性
    - 缓存
    - 分布式锁
2. 利用Redis提供的多种数据结构，方便地对数据进行各种计算操作
    - 排行榜
    - 集合计算
    - 计数器（如库存扣减）
    - 消息队列

## 5.1 缓存

### 5.1.1 缓存穿透

缓存穿透指的是在使用缓存机制的应用中，当某个请求需要获取的数据在缓存中不存在，而且请求是恶意或者频繁的，导致每次请求都要访问后端数据库。

解决方案：

- 第一层使用布隆过滤器：提前哈希数据保存到布隆过滤器中，不存在布隆过滤器中的数据一定不在后端库中，可以直接返回。在布隆过滤器中的键可能也不在库中，但是还会经过第二层过滤
- 第二层缓存空对象，并设置过期时间，避免使用同一个不存在的key做攻击

### 5.1.2 缓存雪崩

缓存雪崩是指在缓存中大量的缓存键在同一时间失效（或同时被删除），导致大量的请求直接访问后端存储（如数据库），从而对后端存储造成了巨大的负载压力。

解决方案：

- 设置随机缓存过期时间，避免热点缓存同时失效

### 5.1.3 缓存击穿

缓存击穿是指在使用缓存的应用程序中，**某个**缓存键的失效或过期导致了大量的请求同时访问后端存储（如数据库），从而对后端存储系统造成了巨大的负载压力。缓存击穿通常是由于热点数据的高并发访问和缓存中的数据失效引起的。

解决方案：

- 热点键设置永不过期

## 5.2 分布式锁

- 使用`setnx`命令加锁：
    - 为防止死锁，需要设置过期时间
    - 设置过期时间和加锁不是一个原子操作, 期间宕机会导致死锁, 使用lua脚本
    - 新版本可使用set命令加锁, 加上px和nx参数, 可避免使用lua脚本
- 加锁的key为锁的key，value为标识当前线程的uuid，作用：解锁时根据该值判断是否当前线程加的锁，从而防止锁过期后其他线程拿到了锁，然后当前线程又误释放了别人的锁
- 解锁：判断值是否当前线程才解锁，因为可能执行时间超过超时时间，锁已经释放了，另外判断锁是否当前线程持有再解锁也不是原子操作，需要用lua脚本
- 续期线程：续期锁过期时间，但也有可能下一个续期时间还没到，锁已经过期了，续期不了，所以加锁时设置value为线程的uuid还是必要的
- 加锁失败：如果当前获取锁的线程是当前进程，则线程移到等待队列，挂起；如果获取锁的线程不是当前进程，则启动重试线程，隔一段时间去看一下锁对应的键在不在，不在说明被释放了，可唤醒当前进程等待队列的头部线程，去竞争锁。

## 5.3 计数器

可以对 String 进行自增自减运算，从而实现计数器功能。

# 六、其他

## 6.1 运维

### 6.1.1 概念

- Redis工作线程模型为单线程
- Redis单机每秒支持10万QPS
- 慢查询：Redis中慢查询统计只统计命令执行的耗时，命令传输及命令排队耗时不会统计。`slowlog-log-slower-than`表示执行时间长于这个值就记录相应的命令，单位为微秒。命令存放点为一个列表，`slowlog-max-len`这个命令表示那个列表的长度为多少，超过该长度则移除表头元素。操作方式：
  
    ```bash
    # 配置
    # 方式一：命令
    config set slowlog-log-slower-than 20000
    config set slowlog-max-len 1000
    # 将配置持久化到本地文件
    config rewrite
    # 方式二：配置文件
    
    # 查询
    # 慢查询列表
    slowlog get
    # 慢查询列表长度
    slowlog len
    # 慢查询日志重置（即清空慢日志列表）
    slowlog reset
    ```
    

### 6.1.2 命令

```bash
# 查看所有键
keys *
# 键总数
dbsize
# 检查键是否存在
exists key
# 删除键
del key
# 键过期
expire key seconds
# 键类型
type key
# 内部编码类型
object encoding key
# 键重命名
rename key newkey
# 随机返回一个键
randomkey
# 键过期
expire key seconds
expireat key timestamp
# 查询剩余过期时间
ttl
pttl
# 切换数据库
select dbIndex
# 清除数据库
flushdb # 清除当前数据库
flushall # 清除所有数据库
# 查看内存消耗情况
info memory
```

### 6.1.3 内存回收策略

1. 删除过期键对象：
    - 惰性删除：对键进行操作时，若发现键已经过期则删除键并取消执行
    - 定时删除：定期到过期字典拿key，判断是否过期，若是则删除它
2. 内存溢出控制策略：当Redis所用内存达到maxmemory上限时，触发相应的溢出控制策略：
    - noeviction：默认策略，不会删除任何数据，拒绝写入并返回客户端错误信息
    - volatile-lru：根据lru算法删除设置了超时属性的键，直到腾出足够空间，如果没有可删除的键，回退到noeviction策略
    - allkeys-lru：根据lru算法删除键，不管数据是否设置过期时间，直到腾出足够空间为止
    - volatile-random：随机删除过期键，直到腾出空间为止
    - allkeys-random：随机删除所有键，直到腾出足够空间为止
    - volatile-ttl：根据减值对象的ttl属性，删除最近要过期的数据，如果没有，回退到noeviction策略

内存溢出控制策略设置方式为：`config set maxmemory-policy {policy}`，如：`config set maxmemory-policy volatile-ttl`

## 6.2 参考资料

1. [redis官方文档](https://redis.io/documentation)
2. [你了解 Redis 的三种集群模式吗？](https://segmentfault.com/a/1190000022808576)
3. [深入剖析Redis系列](https://juejin.cn/post/6844903661223542792)
4. [Redis集群的原理和搭建](https://www.jianshu.com/p/c869feb5581d)
5. [50道Redis面试题史上最全，以后面试再也不怕问Redis了](https://juejin.cn/post/6844903817796911112)
6. 书籍：Redis开发与运维，[Redis设计与实现](https://www.processon.com/mindmap/61f7afc60e3e7407d4beab40)，[掘金小册”Redis深度历险”](https://www.processon.com/mindmap/621b54f11e08533fc3b0e5f0)，[极客时间“Redis核心技术与实战”](https://www.processon.com/mindmap/621b5438e401fd520c17fa1f)