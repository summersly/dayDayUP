# Redis

remote dictionary server：C语言编写的，开源的，高性能非关系型NoSQL键值对数据库。

**作为数据库、缓存、消息中间件**

**优点：**

+ 读写性能优异；
+ 支持数据持久化：AOF和RDB两种方式；
+ 支持事务；
+ 数据结构丰富：5种类型；
+ 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

**缺点：**

+ 数据库容量收到物理内存的限制；
+ 不具备自动容错和恢复功能，可能会导致主从数据不一致的情况；
+ 较难支持在线扩容；

**为什么要用缓存？**

+ 高性能，操作缓存比硬盘读写要快很多；
+ 高并发，直接操作缓存能够承受的请求是远远大于直接访问数据库的。
+ 用redis缓存而不用map或者guava的原因：使用自带的map或者guava实现的缓存属于本地缓存，生命周期随着JVM的销毁而结束，在多实例的情况下，缓存不能保持一致性。而redis实现的是分布式缓存。

**Redis为什么这么快？**

+ 内存操作；
+ 数据结构简单，操作也简单；
+ 采用单线程；
+ 使用多路复用的IO模型，非阻塞；

**应用场景总结**

+ 计数器：string进行自增自减运算，从而实现计数器的功能。redis适合频繁读写的计数量。
+ 缓存：将热点数据存放到内存中，设置最大使用量和淘汰策略来保证缓存的命中率。
+ 会话缓存session/token：统一存储多台应用服务器的会话信息，当应用服务器不再存储用户的会话信息，也就不再具有状态。通过redis，用户可以请求任意的应用服务器，从而实现高可用。
+ 全页缓存FPC：比如说magento提供了一个插件，使用redis作为全页缓存的后端。
+ 查找表：DNS记录使用redis进行存储，但是查找表的内容不能失效。
+ 消息队列：list双端队列来实现，但是最好还是使用kafka，rabbitMQ等消息中间件。
+ 分布式锁实现：分布式场景下，可以使用redis自带的setnx命令来实现分布式锁，或者使用官方提供的redlock分布式锁实现。



## 理论知识

### Nosql

not only sql

四种类型：

+ 键值对类型；
+ 文档型；bson格式，mongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档。是一个介于关系型和非关系型中间的产品，功能最丰富，最像关系型数据库。
+ 列存储数据库；HBase，分布式文件系统；
+ 图形关系数据库；存的不是图形，是关系，比如朋友圈关系；Neo4j



redis-benchmark自带的压力测试工具



### 基本操作

redis默认有16个数据库，默认使用第0个；

6379是默认端口。

```shell
## 切换数据库
select 3
## 查看DB大小
DBSIZE
## 添加
set name lkdjfldkj
## 获取
get name
## 查看所有key
keys *
## 清空数据库
flushdb
flushall
## 检查key是否存在
EXISTS name
## 设置过期时间，单位s
EXPIRE name 10
## 查看还剩多少时间 
ttl name
## 查看类型
type name

```



### 数据类型

1. 字符串：实际上可以是简单字符串、json、xml、数字、二进制，最大不能超过512M。可用于缓存、计数器（访问播放量）、共享session、限速（短信接口的频率）。
2. 哈希：一种键值对机构，相对于字符串序列化缓存信息更加直观，并且在操作上更便捷，常用于用户信息等管理。
3. 列表：多个有序的元素，redis的列表是双端队列，元素可以重复。可以用于消息队列、文章列表。
4. 集合：set中不允许有重复的元素，并且是无序的，不能通过索引下标获取元素，用于标签（tag，用户的兴趣）、生成随机数抽奖、社交网络图（sadd+sinter）
5. 有序集合：zset，元素不能重复，但是可以排序，它给每个元素设置一个分数（double类型，分数可以重复），作为排序的依据。多用于排行榜。**集合和有序集合是通过哈希表来实现的，所有增删查都是O1**
6. 

#### String

Redis并没有使用C的字符串表示（C是字符串是以`\0`空字符结尾的字符数组），而是自己构建了一种**简单动态字符串**（simple dynamic string，SDS）的抽象类型，并作为Redis的默认字符串表示。SDS的定义在Redis 3.2版本之后有一些改变，由一种数据结构变成了5种数据结构，**会根据SDS存储的内容长度来选择不同的结构**，以达到节省内存的效果。

+ 常数复杂度来获取字符串的长度，本身就记录了len；
+ 杜绝缓冲区溢出，减少修改字符串时带来的内存重分配次数；空间预分配和惰性空间释放两种优化策略，解决了字符串凭借和截取的空间问题；
+ 以处理二进制的方式来处理存放在`buf`数组里的数据，不会对里面的数据做任何的限制。但是C字符串中间是不能包含空字符的。
+ 

```shell
## 字符串类型的使用
set key1 v1
get key1 ## "v1"
APPEND key1 "hello" ##(Integer) 7
get key1 ## "v1hello"
STRLEN key1 ## (Integer) 7 返回长度
GETRANGE key1 0 3 ## 获取【0，3】的字符
GETRANGE key1 0 -1 ## 获取所有字符
SETRANGE key1 1 xxx ## 替换指定位置开始的字符 ，1，2，3位置变成xxx

## setex (set with expire) 设置过期时间
setex key3 30 "hello"
## setnx (set if not exist) 如果不存在，在分布式锁中常使用
setnx mykey "redis" ## 如果不存在，创建成功
setnx mykey "xcsdfaf" ## 已经存在，创建失败

set view 0
incr view
get view ## "1"
decr view
get view ## "0"
INCRBY view 10 ## 加10
DECRBY view 10 

## 批量生产
mset k1 v1 k2 v2 k3 v3
mget k1 k2 k3
msetnx k1 v1 k4 v4 ## 原子性操作，会一起失败，k4不会创建

## 对象
set user:1 {name:zhangsan,age:3} ## 设置一个user:1对象，值为json字符来保存一个对象
mset user:1:name zhangsan user:1:age 2
mget user:1:name user:1:age
## user:{id}:{field} xxx

## getset
getset db redis ## 先get然后set，由于一开始db不存在，返回nil
get db ## "redis"
getset db mangodb ## 值存在，先返回原值，然后修改值 "redis"
get db ## "mangodb"
```



#### List

redis的链表是自己定义的。

双向链表，pre，next指向前后节点，表头指针head，表尾指针tail，以及链表长度计数器len，还有复制节点、释放节点、对比节点的函数。

```shell
LPUSH list one ## 从左插入
LPUSH list two
LPUSH list three
LRANGE list 0 -1
## 返回 “three“ ”two” “one”
RPUSH list right ## 从右插入
LRANGE list 0 -1
## 返回 “three” "two" "one" "right"
LPOP list ## 移除左边第一个
RPOP list ## 移除右边第一个
## lindex
lindex list 1 ## 通过下标来查找
lindex list 0
## Llen
Llen list #返回列表的长度
## Lrem 移除
lrem list 1 one #移除list集合中指定个数的value
lrem list 2 three 
## trim 截取指定下标的长度
ltrim mylist 0 3 ## 截取【0，3】
##rpoplpush 移除list最后一个元素，移动到目标列表中
rpoplpush mylist targetlist
## lset list 0 item #如果0不存在，或者列表不存在就会报错
## linsert 
```



#### Set

整数集合intset是redis用于保存整数值的集合抽象数据结构，可以保存

```shell
## 添加
sadd myset "hello" [member2]
## 查看所有值
SMEMBERS myset
## contains判断
SISMEMBER myset hello 
## 随机抽出一个元素
SRANDMEMBER myset
## 随机抽出两个元素
SRANDMEMBER myset 2
## 随机删除元素
spop myset
## 删除集合中一个或者多个元素
SREM myset member1 [member2]
## 将一个指定的值，移动到另一个set集合
SMOVE sourceSet targetSet member
## 获取集合的成员数
SCARD myset

## 交集 并集 差集
SDIFF key1 [key2]
SDIFFSTORE destination key1 [key2]
SINTER key1 [key2]
SINTERSTROE destination key1 [key2]
SUNION key1 [key2]
SUNION destination key1 [key2]
```



#### Zset/Sorted Set

一个普通的单链表查询一个元素的时间复杂度为O(N)，即便该单链表是有序的。使用**跳跃表（SkipList）**是来解决查找问题的，它是一种有序的数据结构，不属于平衡树结构，也不属于Hash结构，它通过在每个节点维持多个指向其他节点的指针，而达到快速访问节点的目的。

跳跃表是有序集合的底层实现之一，如果有序集合包含的元素较多，或者元素的成员是比较长的字符串的时候，redis会使用跳跃表来做有序集合的底层实现。



```shell
## 增 删 
ZADD key score1 member1 [score2 member2]
ZADD myzset 1 hhh 2 wawawa 3 jijiji
## 向有序集合添加一个或多个成员，或者更新已存在成员的分数
ZREM key member [member ...]
## 移除有序集合中的一个或多个成员
## 移除某个区间内的元素：可以是按照字典序，下标，分数
ZREMRANGEBYLEX key min max
ZREMRANGEBYRANK key start stop
ZREMRANGEBYSCORE key min max

## 获取所有的元素
ZCARD key
## 排序 按照socre 排序 withscore就是返回时带上score值
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]
ZRANGEBYSCORE salary -inf +inf
ZRANGEBYSCORE salary (1 (5 #(1,5) 不加（就表示闭区间
## 通过字典区间返回有序集合的成员 # min可以用 - 缺省， [c (c 一个是闭区间一个是开区间
ZRANGEBYLEX key min max [LIMIT offset count]
## zrange返回有序集合中指定区间内的元素 start stop都是下标值
ZRANGE key start stop [WITHSCORES]
## 
ZRANK key member

## 从高到低排序
# 分数区间内的元素，分数从高到低排序
ZREVRANGEBYSCORE key max min [WITHSCORES]
# 通过排序后的下标来获取元素，
ZREVRANGE key start stop [WITHSCORES]
# 返回指定元素的rank下标，(分数从大到小)
ZREVRANK key member

## 修改分数 对指定成员的分数加上增量 increment
ZINCRBY key increment member
## 返回某个元素的分数值
ZSCORE key member
## 返回分数区间的元素数量
ZCOUNT key min max
## 返回字典区间内的元素数量
ZLEXCOUNT key min max

## 交集 并集
ZINTERSTORE destination numkeys key [key ...]
ZUNIONSTORE destination numkeys key [key ...]

ZSCAN key cursor [MATCH pattern] [COUNT count]
# 迭代有序集合中的元素（包括元素成员和元素分值）
```



#### Map

字典，用于保存键值对的抽象数据结构。字典中的每一个key都是唯一的。redis的键值对存储和Hash底层实现就是字典。

具体实现就是哈希表，通过一个dictEntry的数组。dictEntry内部有指向下一个哈希表节点的指针，形成链表，解决hash冲突问题（链地址法）。

dict中有两个hash表，一般使用一个表，另一个表是为了rehash的时候使用的（数量超过负载）。

```shell
## 写入，读取，单个或多个，所有
hset myhash field1 hello
hget myhash field1 
hmset myhash field1 value1 field2 value2
hmget myhash field1 field2
hgetall myhash
## 删除
HDEL key field1 [field2]
## 查看哈希表 key 中，指定的字段是否存在。
HEXISTS key field
## 查看所有的key
HKEYS myhash
## 查看所有的values
HVALS myhash
## 如果不存在not exist
HSETNX myhash field value

HINCRBY key field increment
## 为哈希表 key 中的指定字段的整数值加上增量 increment 。
HINCRBYFLOAT key field increment
##为哈希表 key 中的指定字段的浮点数值加上增量 increment 。

HLEN key
## 获取哈希表中字段的数量
```

#### geospatial地理位置

朋友的定位，周围的人。

```shell
## 添加
geoadd china:city 116.40 39.90 bj
## 获取指定的地点经纬度
geoget china:city bj [sh] [hz]
## 返回两个给定位置之间的直线 距离, 默认单位是m
geodist china:city beijing shanghai [km]
## 以给定的经度纬度为中心，指定半径内的元素
georadius china:city 110 30 500 km [withdist] [withcoord] [count 10]
## 以给定的元素为中心，查找
georadiusbymember china:city beijing 100km
## 返回11个字符的Geohash字符串
geohash china:city beijing [qita]
```

**Geospatial的底层是Zset，所有可以用Zset的方法来操作！**

   

#### HyperLogLog基数统计

**基数？** 不重复的元素，可以接收误差。

优点：占用的内存空间很小，官方文档说一般不超过12kb；

缺点：有误差，1%左右

```shell
## 添加
pfadd mykey a b c d e f
## 计数
pfcount mykey
## 合并 merge
pfmerge mykey3 mykey1 mykey2 [更多sourcekey]
##
```

#### Bitmaps位图

使用位存储：统计用户信息，活跃/不活跃，登录/未登录，打卡情况365天的情况。两个状态的都可以用bitmap

节省内存！

```shell
## 用bitmap来记录一周的打卡，0-6天
setbit sign 0 1
setbit sign 1 1
setbit sign 4 1

## 查看某一天是否打卡
getbit sign 4

## 统计打卡的天数
bitcount sign
```



### 事务

redis通过MULTI，EXEC，WATCH等命令来实现事务，

Redis事务本质：一组命令的集合

一个事物中所有命令都会被序列化，在事务执行过程中，会按照顺序执行。一次性、顺序性、排他性，执行一系列的命令。

**Redis的事物总是具有ACID中的一致性和隔离性，其他特性是不支持的。Redis的单条命令是原子性执行的，但事务不保证原子性，并且没有回滚。事务中任意命令执行失败，其余命令仍然执行。**

+ 事务中有命令格式报错error，事务中所有命令都不执行；

+ 事务中有命令执行失败error，事务中其他命令依旧执行。

Redis的事务没有隔离级别的概念。

所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会执行。Exec

**Redis事务的三个阶段**

1. 事务开始：MULTI
2. 命令入队
3. 事务执行：EXEC

取消事务 ：DISCARD 

WATCH 命令是一个乐观锁，可以为 Redis 事务提供 CAS行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。

```bash
watch money
multi
DECRBY money 10
INCRBY out 10
exec
##如果在事务提交之前，watch的money被改动了，事务不会被执行
```



### Jedis

Jedis是Redis官方推荐的java连接开发工具，使用Java操作Redis的中间件。maven导入jedis包

``` java
Transaction multi = jedis.multi();
try{
  multi.set("user","sffaef f");
  multi.exec();
} catch (Exception e){
  multi.discard();
} finally {
  jedis.close();
}
```



### SpringBoot整合Redis

Spring-boot-starter-data-redis底层使用的就是spring-data-redis和**lettuce（原来的jedis被替换为lettuce）**。

+ jedis采用直连，多线程操作不安全，需要jedis pool连接池
+ lettuce底层采用netty，实例StatefulRedisConnection可以在多个线程中并发访问，线程安全

SpringBoot所有的配置，都有一个自动配置类，在spring-boot-autoconfigure包中有一个叫spring.factories的，可以找到RedisAutoConfiguration.java，这个类中绑定了redisProperties.class 配置文件

在RedisAutoConfiguration.java中注入了RedisTemplate类，这个redisTmplate没有过多的设置 

配置连接池的时候要用lettuce，jedis已经不用了

```java
@Autowired
private RedisTemplate redisTemplate;

redisTemplate.opsForValue().set("myKey", "sdkfjsldkfj");

```



### 配置文件

```bash
## redis.conf 文件忽略大小写
## 可以添加其他目录下的配置文件
include /path/to/local.conf
## bind 网络 如果没有bind配置，redis会监听所有可用的网络连接
bind 127.0.0.1::1
## 是否受保护的模式 默认 是
protected-mode yes
## 默认端口 6379
port 6379

## GENERAL 通用
## 是否以守护进程的方式运行？
daemonize yes
## 如果以后台守护进程的方式运行，需要一个pid文件
pidfile /var/run/redis_6379.pid
## 日志 级别：debug verbose notice warning
loglevel notice
logfile "" #输出的日志文件名
## 数据库的数量
database 16
## 

## SNAPSHOTTING 快照
## 与持久化有关
save 900 1 ## 900秒内至少1个key进行修改就持久化
save 300 10  ## 300秒内10个
save 60 10000 ## 60秒内10000个就进行持久化
## 持久化出错了还是否要继续工作? yes:停止接收更新操作，否则没人注意到这里有错
stop-writes-on-bgsave-error yes
## 是否压缩rdb文件
rdbcompression yes
## rdb文件保存的时候，进行错误的检查校验
rdbchecksum
## rdb文件保存的地址
dir ./

## 主从复制相关的配置



## SECURITY 安全
## 密码设置 1. 配置文件设置
requiredpass 123456
## 2. 命令设置 有密码状态使用前需要auth
config set requirepass "123456"
auth 123456

## CLIENT 客户端
maxclients 10000
## 限制 配置最大的内存容量
maxmemory <bytes>
## 内存达到上限之后的处理策略:
## volatile-lru: 只对设置了过期时间的key进行LRU
## allkeys-lru: 所有key进行lru算法
## volatile-random: 随机删除即将过期的key
## allkeys-random: 随机删除
## volatile-ttl: 删除即将过期的key
## noeviction: 永不删除，返回错误
maxmemory-policy noevication

##append only 模式 AOF的配置
appendonly no # 默认是不开启aof模式的，默认是使用rdb方式持久化
appendfilename “appendonly.aof”
## appendonly的频率 
# always 每次修改都会同步，消耗性能
# everysec 每秒执行一次，可能会丢失这1s的数据
# no 不执行，操作系统自己同步数据，速度最佳
appendfsync everysec
```



### 持久化

持久化就是把内存的数据写到磁盘中去，为了重启或者故障之后进行数据恢复。Redis 提供两种持久化机制 快照RDB（默认） 和只追加文件 AOF 机制。快照就是获取存储在内存的数据在某个时间点上的副本，而AOF的实时性更好，只可能丢失一秒的数据。4.0之后redis支持两种方式混合的持久化，AOF重写的时候会直接吧RDB的内容写到AOF的开头，这样既能避免丢失过多的数据，也可以快速加载。

AOF重写实际上是通过读取数据库中的键值对来完成的，并不需要对现有的AOF文件进行任何读入、分析或者写入操作。

#### RDB

RDB：是Redis DataBase缩写快照

dump.rdb

RDB是Redis默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为dump.rdb。通过配置文件中的save参数来定义快照的周期。恢复的时候是将快照文件直接读取到内存中。

Redis会单独创建（fork）一个子进程来进行持久化，会先讲数据写入一个临时rdb文件中，待持久化过程结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，保证了极高的性能。如果需要进行大规模数据的恢复，而且对数据恢复的完整性不是非常敏感，RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化的数据可能会丢失。

触发rdb保存的机制：

+ save的规则被满足的情况下，会触发rdb规则；
+ 执行flushall命令；
+ 退出redis，也会自动触发；

如何从rbd文件恢复：

+ 将rdb文件保存在redis指定的目录

优点：

+ 适合大规模的数据恢复；备份dump.rdb文件就可以；
+ 比AOF更快；

缺点：

+ 需要时间间隔才能保存；最后一次的修改数据容易丢失；
+ fork进程的时候会占用内存空间；



#### AOF

append only file：以日志的形式把所有的命令记录下来（读操作不记录），恢复的时候把命令再执行一遍。

appendonly.aof文件

**如果AOF文件被破坏，有问题，redis会启动失败。使用redis-check-aof进行aof文件修复。**

优点：

+ 每次修改都同步，文件更完整；
+ 不需要同步，效率高

缺点：

+ 每秒同步一次，可能会丢失一秒的数据；
+ 相对于数据文件来说，AOF远大于RDB；

AOF文件重写：

+ auto-aof-rewrite-min-size：64MB，超过这个限制的时候会进行重写；
+ 



### 淘汰机制

#### 设置过期时间

+ 定期删除：每隔一段时间随机删除过期的key
+ 惰性删除：过期了但是没有被删除的key，在被查询的时候被删除

#### 内存淘汰机制-8个

+ volatile-lru
+ volatile-random
+ volatile-lfu
+ volatile-ttl
+ allkeys-lru
+ allkeys-random
+ allkeys-lfu
+ no-eviction

总结来说，基本是volatile（设置了过期时间），allkeys（所有key）和4个机制的组合：lru（最近最少使用），lfu（使用次数最少），random（随机，任意），ttl（将要过期的）



### 高并发问题

#### 缓存雪崩

缓存同一时间大面积过期失效，导致后面的请求全都落在数据库上，造成数据库短时间内承受大量请求而崩掉。

+ 选择合适的内存淘汰机制
+ 加锁：当一个线程需要去访问这个缓存的时候，如果发现缓存为空，则需要先去竞争一个锁，如果成功则进行正常的数据库读取和写入缓存这一操作，然后再释放锁，否则就等待一段时间之后，重新尝试读取缓存，如果还没有数据就继续去竞争锁。
+ 事后恢复缓存

#### 缓存穿透

大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有 经过缓存这一层。

+ 最基本的就是首先做好参数校验，一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的 数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等。
+ 布隆过滤器：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过 来，我会先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信 息给客户端，存在的话才会走下面的流程。**但缺点是具有一定的错误识别率和删除难度**



### 数据一致性

一般情况下我们都是这样使用缓存的:先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。这种方式很明显会存在缓存和数据库的数据不一致的情况。

**如何保证缓存和数据库的一致性**

+ 缓存一定要设置过期时间

+ 保证数据库和缓存的最终一致性就好，不必强求一致性（分布式读写锁）

+ 经典方案：Cache Aside Pattern：先更新数据库，再删除缓存

+ 首先，为什么要删除而不是更新缓存，这个在前面有分析，这里不再赘述。那为什么我们应当先更新数据库呢？因为缓存在数据持久化这方面往往没有数据库做得好，而且数据库中的数据是不存在过期这个概念的，我们应当以数据库中的数据为主，缓存因为有着过期时间这一概念，最终一定会跟数据库保持一致。

  

  

### key过大









