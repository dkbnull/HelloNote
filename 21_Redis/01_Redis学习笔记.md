# NoSQL

## 什么是NoSQL

NoSQL = not only sql

泛指非关系型数据库

关系型数据库：表格，行、列记录

## 为什么用NoSQL

用户的个人信息、地理位置、用户数据等爆发式增长，这些数据类型的存储不需要一个固定的格式，不需要多余的操作就可以横向扩展，就需要用NoSQL数据库。

## 特点

1、方便扩展，数据之间无关系

2、大数据量高性能

3、数据类型多样型，不需事先设计数据库

4、没有固定的查询语言

5、一致性

## 四大分类

### KV键值对

**Redis**

### 文档型数据库

**BSON格式，和JSON一样**

**MongoDB**，一个基于分布式文件存储的数据库，主要用于处理大量的文档，是一个介于关系型数据库和非关系型数据库中间的产品，是非关系型数据库中功能最丰富、最像关系型数据库的产品

### 列存储

**HBase**

分布式文件系统

### 图形关系数据库

不是存图形，放的是关系，比如社交网络、广告推荐

**Neo4j**，InfoGrid

# Redis

官网：[https://redis.io/](https://redis.io/)

文档：[https://redis.io/docs/about/](https://redis.io/docs/about/)

# 安装

安装教程：[https://www.runoob.com/redis/redis-install.html](https://www.runoob.com/redis/redis-install.html)

## Windows

下载地址：[https://github.com/tporadowski/redis/releases](https://github.com/tporadowski/redis/releases)

## Linux

~~~sh
wget http://download.redis.io/releases/redis-7.2.4.tar.gz
或
#下载文件重命名为redis-7.2.4.tar.gz
wget https://github.com/redis/redis/archive/7.2.4.tar.gz

tar -zxvf redis-7.2.4.tar.gz
cd redis-7.2.4

#安装依赖环境
# yum install gcc-c++

#安装服务
#redis默认安装目录/usr/local/bin
make
# make install

#redis默认不是后台启动，需修改配置
vi redis.conf
	daemonize yes
~~~

# 用处

1、内存存储、持久化。内存中是断电即失的，所以持久化很必要（机制：rdb、aof）

2、效率高，可以用于高速缓存

3、发布订阅系统

4、地图信息分析

5、计时器、计数器（浏览量）

# 特性

1、多样的数据类型

2、持久化

3、集群

4、事务

# 命令/Redis Key

[https://redis.io/commands/](https://redis.io/commands/)

~~~sh
#测试连接
ping

#存
set key value

#取
get key

#查看数据库所有key
keys *

#清空当前数据库
flushdb

#清空全部数据库
flushall

#判断key是否存在，返回1存在0不存在
exists key

#删除key
del key

#移动key到指定数据库，1表示目标数据库，返回1表示移动成功
move key 1

#设置过期时间，10s后过期
expire key 10

#展示key剩余过期时间，返回-2表示已过期
ttl key

#查看key是什么类型
type key
~~~

# 性能测试

**redis-benchmark**

~~~sh
redis-benchmark [option] [option value]
~~~

| 选项                | 描述                                       | 默认值    |
| :------------------ | :----------------------------------------- | :-------- |
| **-h**              | 指定服务器主机名                           | 127.0.0.1 |
| **-p**              | 指定服务器端口                             | 6379      |
| **-s**              | 指定服务器 socket                          |           |
| **-c**              | 指定并发连接数                             | 50        |
| **-n**              | 指定请求数                                 | 10000     |
| **-d**              | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| **-k**              | 1=keep alive 0=reconnect                   | 1         |
| **-r**              | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| **-P**              | 通过管道传输 <numreq> 请求                 | 1         |
| **-q**              | 强制退出 redis。仅显示 query/sec 值        |           |
| **--csv**           | 以 CSV 格式输出                            |           |
| **-l（小写字母L）** | 生成循环，永久执行测试                     |           |
| **-t**              | 仅运行以逗号分隔的测试命令列表。           |           |
| **-I（大写字母i）** | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

~~~sh
redis-benchmark -h localhost -p 6379 -c 100 -n 100000

====== SET ======
  100000 requests completed in 1.46 seconds			#对10w个写入请求进行测试
  100 parallel clients								#100个并发客户端
  3 bytes payload									#每次写入3个字节
  keep alive: 1										#只有1台服务器处理请求，单机性能

89.61% <= 1 milliseconds
99.58% <= 2 milliseconds
99.70% <= 8 milliseconds
99.73% <= 9 milliseconds
99.77% <= 10 milliseconds
99.81% <= 11 milliseconds
99.86% <= 12 milliseconds
99.90% <= 13 milliseconds
99.90% <= 18 milliseconds
99.92% <= 19 milliseconds
99.95% <= 20 milliseconds
100.00% <= 21 milliseconds
100.00% <= 21 milliseconds							#所有请求在21ms处理完成
68540.09 requests per second						#每秒处理68540.09个请求
~~~

# 基础知识

## Redis默认有16个数据库

**redis.conf**中

~~~conf
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
~~~

默认使用**第0个数据库**，可以使用**select**进行切换

~~~sh
127.0.0.1:6379> select 3		#切换数据库
OK
127.0.0.1:6379[3]> dbsize		#查看数据库大小
(integer) 0
~~~

## Redis是单线程的

Redis是基于内存操作，CPU不是Redis的性能瓶颈。Redis的瓶颈是根据机器的内存和网络带宽

Redis 不同版本之间采用的线程模型是不一样的，Redis4.0 版本之前使用的是单线程模型，4.0 版本之后增加了多线程的支持。

在4.0之前虽然Redis是单线程，也只是说它的网络 I/O 线程以及 Set 和 Get 操作是由一个线程完成的。持久化、集群同步还是使用其他线程来完成。

4.0 之后添加了多线程的支持，主要是体现在大数据的异步删除功能上，例如 unlink key、flushdb async、flushall async 等



核心：Redis将所有的数据全部放在内存中，所以说单线程去操作效率是最高的。多线程会产生CPU上下文切换，耗时。对于内存系统来说，没有上下文切换的效率是最高的。多次读写都是在一个CPU上

# 五大数据类型

## String/字符串

使用场景：value可以是字符串、数字

* 计数器
* 统计多单位的数量
* 粉丝数
* 对象缓存存储

### 命令

[https://redis.io/commands/?group=string](https://redis.io/commands/?group=string)

~~~sh
#追加字符串，返回字符串长度，key不存在则set
append key value

#获取字符串长度
strlen key

#截取字符串，[start,end]
getrange key start end

#替换指定位置开始的字符串
setrange key offset value

#########################

#i++
incr key

#i--
decr key

#i+10
incrby view 10

#i-5
decrby view 5

#########################
#设置过期时间，set with expire
setex key seconds value

#不存在再设置，set if not exist，常用在分布式锁中
setnx key value

#########################

#批量设置
mset key value

#批量获取
mget key

#批量，原子性操作，同时成功或失败
msetnx key value

#########################

#先get再set
getset key value
~~~

## List/列表

基本的数据类型：列表

可以用作栈、队列、阻塞队列

实际上是一个链表 before node after，left、right都可以插入值

* key不存在，则新建链表
* key存在，则新增
* 移除所有值，链表为空，则不存在
* 在两边插入或修改，效率最高；中间元素，效率相对低



消息队列：lpush，rpop

栈：lpush，lpop

### 命令

[https://redis.io/commands/?group=list](https://redis.io/commands/?group=list)

~~~sh
#存，插入到列表头部
lpush key value [value ...]
#存，插入到列表尾部
rpush key value [value ...]

#取
lrange key start stop

#移除列表第一个元素
lpop key
#移除列表最后一个元素
rpop key

#########################

#通过下标获取list中某一个值
lindex key index

#获取列表长度
llen key

#########################

#移除指定值，指定个数的value
lrem key count value

#########################

#通过下标截取指定长度，[start,stop]
ltrim key start stop

#########################

#移除列表的最后一个元素，再移动到新的列表中
rpoplpush source destination

#########################

#是否存在
exists key

#更新值，list不存在时更新会报错
lset key index value

#将value插入到list中某个元素前面或后面
linsert key BEFORE|AFTER pivot value
~~~

## Set/集合

set中的值不能重复

### 命令

[https://redis.io/commands/?group=set](https://redis.io/commands/?group=set)

~~~sh
#存
sadd key member [member ...]

#取
smembers key

#值是否在set中
sismember key member

#获取元素个数
scard key

#移除集合中指定元素
srem key member [member ...]

#随机抽选指定个数的元素
srandmember key [count]

#随机移除指定个数的元素
spop key [count]

#########################

#移动指定元素
smove source destination member

#########################

#数学集合类
#差集
sdiff key [key ...]
#交集
sinter key [key ...]
#并集
sunion key [key ...]
~~~

## Hash/哈希

Map集合，本质与String类型没有太大区别，还是一个简单的key-value

存放变更数据，尤其是类似用户信息，经常变动的信息

hash更适合对象的存储；string更适合字符串存储

### 命令

[https://redis.io/commands/?group=hash](https://redis.io/commands/?group=hash)

~~~sh
#存
hset key field value
#取
hget key field

#批量存
hmset key field value [field value ...]
#批量取
hmget key field [field ...]
#获取全部数据
hgetall key

#删除指定的key，对应value会同步删除
hdel key field [field ...]

#获取hash字段数量
hlen key

#字段是否存在
hexists key field

#########################

#获取所有field
hkeys key

#获取所有value
hvals key

#########################

#增加
hincrby key field increment

#########################

#不存在再设置，set if not exist，常用在分布式锁中
hsetnx key field value
~~~

## Zset/有序集合

在set基础上，增加了一个值set k1 v1 ，Zset k1 score1 v1

存储班级成绩表、工资表

带权重进行判断，普通消息1，重要消息2

排行榜，取Top N

### 命令

[https://redis.io/commands/?group=sorted-set](https://redis.io/commands/?group=sorted-set)

~~~sh
#存，支持多个值
zadd key [NX|XX] [CH] [INCR] score member [score member ...]
#取
zrange key start stop [WITHSCORES]
#倒序取
zrevrange key start stop [WITHSCORES]
 
#获取指定区间内的成员，从小到大，-inf -∞，+inf +∞
zrangebyscore key min max [WITHSCORES] [LIMIT offset count]

#移除指定元素
zrem key member [member ...]

#获取指定集合的个数
zcard key

#获取指定区间的成员数量
zcount key min max
~~~

# 三种特殊数据类型

## Geospatial indices/地理位置

朋友圈、附近的人、距离计算

Redis的Geo v3.2后推出的功能

可以推算地理位置的信息，两地之间的距离，附近的人

### 命令

~~~sh
#GEOADD
#添加地理位置，经度、纬度、名称
#有效的经度范围：from -180 to 180
#有效的纬度范围：from -85.05112878 to 85.05112878
geoadd key longitude latitude member [longitude latitude member ...]

#GEODIST
#两个地址之间的直线距离
#m 米
#km 千米
#mi 英里
#ft 英尺
geodist key member1 member2 [unit]

#GEOHASH
#获取指定城市的经纬度字符串，二维转一维，如果两个字符串越接近，则距离越近
geohash key member [member ...]

#GEOPOS
#获取指定城市的经纬度
geopos key member [member ...]

#GEORADIUS
#以给定的经纬度为中心，找出某一半径内的元素
#WITHDIST 显示直线距离 WITHCOORD 显示他人经纬度 COUNT 筛选指定数量的结果
georadius key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC | DESC] [STORE key | STOREDIST key]

GEORADIUS_RO

#GEORADIUSBYMEMBER
#找出位于指定元素周围的其他元素
georadiusbymember key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC | DESC] [STORE key | STOREDIST key]

GEORADIUSBYMEMBER_RO

GEOSEARCH

GEOSEARCHSTORE
~~~

### 示例

~~~sh
127.0.0.1:6379> geoadd city 116.41 39.91 beijing
(integer) 1
127.0.0.1:6379> geoadd city 121.48 31.24 shanghai
(integer) 1
127.0.0.1:6379> geoadd city 120.39 36.07 qingdao
(integer) 1
127.0.0.1:6379> geoadd city 117.01 36.67 jinan
(integer) 1

127.0.0.1:6379> geopos city qingdao
1) 1) "120.38999944925308228"
   2) "36.07000093381353878"

127.0.0.1:6379> geodist city qingdao jinan km
"309.9630"

127.0.0.1:6379> georadius city 110 30 1000 km
1) "jinan"
127.0.0.1:6379> georadius city 110 30 2000 km
1) "shanghai"
2) "jinan"
3) "qingdao"
4) "beijing"
127.0.0.1:6379> georadius city 110 30 1000 km withdist
1) 1) "jinan"
   2) "986.6424"
   
127.0.0.1:6379> georadiusbymember city beijing 1000 km
1) "jinan"
2) "qingdao"
3) "beijing"
~~~

### 扩展

GEO底层实现原理为Zset，可以使用Zset命令来操作GEO

~~~sh
127.0.0.1:6379> zrange city 0 -1
1) "shanghai"
2) "jinan"
3) "qingdao"
4) "beijing"
127.0.0.1:6379> zrem city shanghai
(integer) 1
127.0.0.1:6379> zrange city 0 -1
1) "jinan"
2) "qingdao"
3) "beijing"
~~~

## HyperLogLog/基数统计算法

基数：不重复的元素，可以接受误差，0.81%错误率

Redis v2.8.9后支持

HyperLogLog，基数统计的算法，数据结构

优点：占用内存是固定的，2^64 不同的元素的基数，只需12KB内存

网页的UV，一个人访问一个网站多次，还是算作一个人

传统方式，set保存用户id，然后统计set中的元素作为标准。但保存大量的用户id，会比较麻烦

### 命令

~~~sh
#PFADD
#创建一组元素
pfadd key element [element ...]

#PFCOUNT
#统计元素的基数数量
pfcount key [key ...]

PFDEBUG

#PFMERGE
#合并两组元素，并集
pfmerge destkey sourcekey [sourcekey ...]

PFSELFTEST
~~~

### 示例

~~~sh
127.0.0.1:6379> pfadd pfset a b c d e f
(integer) 1
127.0.0.1:6379> pfcount pfset
(integer) 6
127.0.0.1:6379> pfadd pfset2 e f g h i j k l
(integer) 1
127.0.0.1:6379> pfcount pfset2
(integer) 8
127.0.0.1:6379> pfmerge pfset3 pfset pfset2
OK
127.0.0.1:6379> pfcount pfset3
(integer) 12
~~~

## Bitmap/位存储

位存储

两个状态的，都可以使用Bitmap

Bitmap，位图，数据结构，操作二进制位来进行记录，只有0 1 两个状态

### 命令

~~~sh
#SETBIT
#存
set key value [expiration EX seconds|PX milliseconds] [NX|XX]

#GETBIT
#取
getbit key offset

#BITCOUNT
#统计
bitcount key [start end]

BITFIELD
BITFIELD_RO
BITOP
BITPOS
~~~

### 示例

记录一周打卡情况

~~~sh
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 1
(integer) 0
127.0.0.1:6379> setbit sign 3 0
(integer) 0
127.0.0.1:6379> setbit sign 4 0
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0
127.0.0.1:6379> getbit sign 3
(integer) 0
127.0.0.1:6379> getbit sign 6
(integer) 1
127.0.0.1:6379> bitcount sign
(integer) 4
~~~

# 事务

Redis事务本质：一组命令的集合

一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行

一次性、顺序性、排他性 执行一系列命令

所有命令在事务中，并没有立即被执行，只有发起执行命令的时候才会执行

**Redis单条命令是保证原子性的，但是事务不保证原子性**

**Redis事务没有隔离级别的概念**

Redis事务：

* 开启事务    **multi**
* 命令入队
* 执行事务    **exec**

~~~sh
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 ve
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v2"
4) OK
~~~

取消事务：**discard**

事务中的命令都不被执行

~~~sh
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> discard
OK
127.0.0.1:6379> get k4
(nil)
~~~

编译时异常，命令错误，事务中所有命令都不被执行

~~~sh
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> getset k2
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
~~~

运行时异常，语法错误，执行命令时，其他命令可以正常执行，错误命令抛出异常

~~~sh
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> get k3
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range
2) OK
3) OK
4) "v3"
~~~

# 悲观锁

认为什么时候都会出问题，无论做什么都加锁

# 乐观锁

认为什么时候都不会出现问题，所以不会加锁

更新数据时判断一下，在此期间是否有人修改过这个数据

watch 监视 unwatch 放弃监视

* 获取version

* 更新时比较version

# Jedis

Redis官方推荐的Java连接开发工具，使用Java操作Redis中间件

## 导入依赖

~~~xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>5.1.2</version>
</dependency>
~~~

## 编码测试

~~~java
Jedis jedis = new Jedis("127.0.0.1", 6379);

jedis.watch("key");

try{
    Transaction multi = jedis.multi();
    multi.set("key", "value");
    multi.exec();
}catch (Exception e){
    multi.discard();
}finally{
    jedis.close();
}
~~~

## 常用API



# Spring Boot 整合Redis

[https://blog.csdn.net/dkbnull/article/details/137062282](https://blog.csdn.net/dkbnull/article/details/137062282)

# Redis.conf

启动时通过配置文件来启动

~~~conf
# unit单位，大小写不敏感

# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

################################## INCLUDES ###################################

# 可以引入其他文件
# include /path/to/local.conf
# include /path/to/other.conf
# include /path/to/fragments/*.conf

################################## NETWORK #####################################

#
# Examples:
#
# bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses
# bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6
# bind * -::*                     # like the default, all available interfaces

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# 绑定的ip
bind 127.0.0.1 -::1

# 保护模式，保证安全性
protected-mode yes

# 端口
port 6379

################################# GENERAL #####################################

# 是否后台运行，以守护进程方式运行，默认为no
daemonize yes

# 如果以后台方式运行，需指定pid文件
pidfile /var/run/redis_6379.pid

# 日志级别，notice生产环境适用
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
# nothing (nothing is logged)
loglevel notice

# 日志文件名
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# 数据库数量，默认16
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

# 是否显示logo
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
always-show-logo no

################################ SNAPSHOTTING  ################################

# 持久化，在规定的时间内，执行了多少次操作，则会持久化到文件.rdb 
# redis是内存数据库，如果没有持久化，
# save <seconds> <changes> [<seconds> <changes> ...]
#
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 change was performed
#   * After 300 seconds (5 minutes) if at least 100 changes were performed
#   * After 60 seconds if at least 10000 changes were performed
#
# save 3600 1 300 100 60 10000

# 如果持久化出错，是否继续工作
stop-writes-on-bgsave-error yes

# 是否压缩rdb文件，需要消耗一些cpu资源
rdbcompression yes

# 保存rdb文件时，进行错误的检查校验
rdbchecksum yes

# rdb保存的文件名
dbfilename dump.rdb

# rdb文件保存的目录
dir ./

################################## SECURITY ###################################

# 设置登录密码，默认无密码。
# auth 密码登录
# requirepass foobared

################################### CLIENTS ####################################

# 能连接到redis的最大客户端数
# maxclients 10000

############################## MEMORY MANAGEMENT ################################

# 最大内存容量
# maxmemory <bytes>

# 内存达到上限后处理策略
# volatile-lru -> 只对设置了过期时间的key进行LRU（默认值）
# allkeys-lru -> 剔除LRU算法的key
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> 随机删除即将过期的key
# allkeys-random -> 随机删除
# volatile-ttl -> 删除即将过期的
# noeviction -> 永不删除，返回错误
# maxmemory-policy noeviction

############################## APPEND ONLY MODE ###############################

# 默认不开启AOF模式，默认使用rdb方式持久化，在大部分情况下，rdb够用
appendonly no

# 持久化文件的名字
appendfilename "appendonly.aof"

# appendfsync always	# 每次修改都会sync，消耗性能
appendfsync everysec	# 每秒执行1次sync，可能会丢失这1s的数据
# appendfsync no		# 不同步sync，操作系统自己同步数据，速度最快

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
~~~

# 持久化

Redis是内存数据库，如果不将内存中的数据保存到磁盘中，那么一旦服务器进程退出，服务器中的数据库状态就会消失。

## RDB(Redis Database)

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能会丢失。

默认使用RDB，一般情况不需要修改配置

保存的文件，**dump.rdb**

生产环境可对其进行备份

在主从复制中，从机上备用

**触发机制：**

* save规则满足情况下，会自动触发rdb
* 执行flushall命令
* 退出redis

**恢复机制：**

只需将dump.rdb文件放到Redis启动目录就可以，Redis启动时会自动检查dump.rdb恢复其中的数据

查看需要存放的位置

~~~sh
127.0.0.1:6379> config get dir
1) "dir"
2) "/user/local/bin"
~~~

~~~conf
# 持久化，在规定的时间内，执行了多少次操作，则会持久化到文件.rdb 
# redis是内存数据库，如果没有持久化，
# save 3600 1 300 100 60 10000

# 如果持久化出错，是否继续工作
stop-writes-on-bgsave-error yes

# 是否压缩rdb文件，需要消耗一些cpu资源
rdbcompression yes

# 保存rdb文件时，进行错误的检查校验
rdbchecksum yes

# rdb保存的文件名
dbfilename dump.rdb

# rdb文件保存的目录
dir ./
~~~

**优点：**

* 适合大规模的数据恢复
* 对数据的完整性要求不高

**缺点：**

* 需要一定的时间间隔进行操作，如果Redis意外宕机，那最后一次修改的数据就丢失了
* fork进程的时候，会占用一定的内存空间

## AOF(Append Only File)

将我们所有的命令都记录下来，恢复的时候就把这个文件全部再执行一次

以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

保存的文件，**appendonly.aof**

默认不开启，需手动配置，`appendonly yes`

如果aof文件有错误，Redis无法启动，可使用**redis-check-aof  --fix**修复文件

~~~conf
# 默认不开启AOF模式，默认使用rdb方式持久化，在大部分情况下，rdb够用
appendonly no

# 持久化文件的名字
appendfilename "appendonly.aof"

# appendfsync always	# 每次修改都会sync，消耗性能
appendfsync everysec	# 每秒执行1次sync，可能会丢失这1s的数据
# appendfsync no		# 不同步sync，操作系统自己同步数据，速度最快

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
~~~

aof默认文件无限追加，文件会越来越大。如果aof文件大于64m，fork一个新的进程来将文件进行重写

**优点：**

* 每一次修改都同步，文件的完整性更好
* 每秒同步一次，可能丢失一秒数据
* 从不同步，效率最高

**缺点：**

* 相对于数据文件来说，aof远远大于rdb，修复速度更慢
*  aof运行效率比rdb慢，所以Redis默认配置的是rdb

## 小结

1、RDB持久化方式能够在指定的时间间隔内对数据进行快照存储

2、AOF持久化方式记录每次对服务器写的操作，当服务器重启时会重新执行这些命令来恢复原始的数据。AOF会以Redis协议追加保存每次写的操作到文件末尾，Reds还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。

3，**只做缓存，如果只希望数据在服务器运行的时候存在，可以不使用任何持久化**

4、同时开启两种持久化方式

* 在这种情况下，当Redis重启时会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集会比RDB文体保存的数据集更完整。
* RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件

5、性能建议

* 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
* 如果Enable AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单，只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的，只要硬盘允许，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设置到5G以上，默认超过原大小100%大小重写可以改到适当的数值
* 如果不Enable AOF，仅置 Master-Slave Replication 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master Slave 同时挂掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个

# 发布订阅

Redis发布订阅（pub/sub）是一种**消息通信模式**，发送者pub发送消息、订阅者sub接收消息

Redis客户端可以订阅任意数量的频道

* 消息发送者

* 频道

* 消息订阅者

## 命令

用于构建即时通讯应用，比如网络聊天室、实时广播、实时提醒等

~~~sh
#订阅一个或多个符合给定模式的频道。
PSUBSCRIBE pattern [pattern ...]

#查看订阅与发布系统状态。
PUBSUB subcommand [argument [argument ...]]

#将信息发送到指定的频道。
PUBLISH channel message

#退订所有给定模式的频道。
PUNSUBSCRIBE [pattern [pattern ...]]

#订阅给定的一个或多个频道的信息。
SUBSCRIBE channel [channel ...]

#退订给定的频道。
UNSUBSCRIBE [channel [channel ...]]
~~~

## 测试

**订阅者**

~~~sh
127.0.0.1:6379> SUBSCRIBE test
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "test"
3) (integer) 1
		#等待读取推送的信息
1) "message"	#消息
2) "test"		#哪个频道的消息
3) "hello"		#消息具体内容
~~~

**发送者**

~~~sh
127.0.0.1:6379> PUBLISH test hello
(integer) 1
~~~

## 使用场景

* 实时消息系统
* 实时聊天，频道当做聊天室，将信息回显给所有人即可
* 订阅、关注系统

再复杂的场景可以使用消息中间件MQ

# 主从复制

## 概念

**数据的复制是单向的，只能由主节点到从节点。**Master以写为主，Slave以读为主。

**==默认情况下，每台Redis服务器都是主节点==**，且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

**主从复制的作用主要包括：**

* 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
* 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复，实际上是一种服务的究余
* 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点据供写服务，由从节点提供读服务，分担服务器负裁；尤其是在写少读多的场最下，通过多个从节点分担读负裁，可以大大提高Redis服务器的并发量。
* 高可用（集群）基石：主从复制是哨兵和集群能够实施的基础，因此说主从复制是redis高可用的甚础



**一般来说，单台Redis最大使用内存不应该超过20G**

## 环境配置

只配置从库，不用配置主库

~~~sh
127.0.0.1:6379> info replication		#查看当前库信息
# Replication
role:master
connected_slaves:0
master_replid:30b557e69a8d72fd1a28baed4d7b72b37ee981c1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
~~~

**修改配置文件**

~~~config
port 6379

pidfile /var/run/redis_6379.pid

logfile "6379.log"

dbfilename dump6379.rdb
~~~

## 配置从机

**默认情况下，每台Redis服务器都是主节点**，一般情况只配置从机就可以

~~~sh
#配置主机ip，端口号
SLAVEOF host port
~~~

真实的主从配置应该在配置文件中配置，这样的话是永久的。使用命令，是暂时的

~~~conf
replicaof <masterip> <masterport>

masterauth <master-password>
~~~



主机可以写，从机不能写只能读。

主机断开连接，从机依旧连接到主机，但是没有写操作。主机如果回来了，从机依旧可以直接获取到主机写的信息

如果是使用命令行配置的从机，从机重启了，就会变成主机。只要再变回从机，立马就会从主机中获取值

## 复制原理

Slave 启动成功连接到 master 后会发送一个sync同步命令

Master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，**master将传送整个数据文件到slave。并完成一次完全同步**。

**全量复制**：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中

**增量复制**：master继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行

## 树形结构

上一个M链接下一个S

主 <--- 从/主 <--- 从

从/主 部分依旧是从节点



主机断开连接，可以使用`SLAVEOF no one`让自己变成主机，其他节点可手动连接到当前最新的主节点。这时如果主机恢复，需要重新连接

~~~sh
#自己成为主节点
SLAVEOF no one
~~~

# 哨兵模式

主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，需要人工干预，费时费力，还会造成一段时间内服务不可用。不推荐，优先考虑哨兵模式。

v2.8开始提供了sentinel哨兵架构来解决这个问题

能够后台监控主机是否故障，如果故障了，根据投票数**自动将从库转为主库**

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，他会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**

哨兵的作用

* 通过发送命令，让Redis服务器返回监控器运行状态，包括主服务器和从服务器
* 当哨兵检测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让他们切换主机

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，可以使用多个哨兵进行监控，各个哨兵之间还会进行监控，形成多哨兵模式

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover处理，仅仅是哨兵1主观的认为主服务器不可用，这个现象是主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover（故障转移）操作。切换成功后，就会通过发布订阅模式，让各个哨兵吧自己监控的从服务器实现切换主机，这个过程成为客观下线。

## sentinel.conf

哨兵配置文件

~~~conf
#sentinel monitor 被监控的名称 ip port 1
sentinel monitor masterredis 127.0.0.1 6379 1
~~~

后面的数字1，代表主机挂了，从机投票看让谁替换为主机，票最多的，成为主机

~~~sh
#启动
redis-sentinel sentinel.conf
~~~

如果master主机宕机，这时就会从从机中随机选择一个服务器

如果主机回归，只能归并到新的主机下，当做从机，这就是哨兵模式的规则

## 优点

* 哨兵集群，基于主从复制模式，所有的主从配置优点，他都有
* 主从可以切换，故障可以转移，系统的可用性就会更好
* 哨兵模式就是主从模式的升级，手动到自动

## 缺点

* Redis不好在线扩容，集群容量一旦到达上限，在线扩容就十分麻烦
* 实现哨兵模式的配置其实是很麻烦

## 配置

~~~conf
# Example sentinel.conf
# 哨兵sentinel实例运行的端口，默认26379 如果有哨兵集群，需配置每个哨兵的端口
port 26379
# 哨兵sentinel的工作目录
dir /tmp

# 哨兵sentinel监控的redis主节点的 ip port
# master-name 可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000

# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1

# 故障转移的超时时间 failover-timeout 可以用在以下这些方面：
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# shell编程
# sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh

# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh # 一般都是由运维来配
置！
~~~

# Redis 集群

[https://blog.csdn.net/dkbnull/article/details/130022026](https://blog.csdn.net/dkbnull/article/details/130022026)

# 缓存穿透

查不到

用户想要查询一个数据，发现Redis内存数据库中没有，也就是缓存没有命中，于是向持久层数据库查询，发现也没有，于是本次查询失败。

当用户很多的时候，缓存都没有命中，于是都去请求持久层数据库，这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

### 解决方案

**缓存空对象**

当存储层不命中后，即使返回空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源

存在问题

* 如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键
* 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响

**布隆过滤器**

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力

# 缓存击穿

量太大，缓存过期

与缓存穿透的区别，缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就击破缓存，直接请求数据库，数据库瞬间压力过大，就像在一个屏障上凿开了一个洞

### 解决方案

**设置热点数据永不过期**

从缓存层面来看，没有设置过期时间，所以不会出现热点key过期后产生的问题

**加互斥锁**

使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此指需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大

# 缓存雪崩

是指在某一时间段，缓存集中过期失效，或者Redis宕机

比如双十一零点，商品时间比较集中的放入了缓存，加入缓存一个小时。那么到了凌晨一点的时候，这批商品的缓存就都过期了。而对于这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会到达存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况

其实集中过期，倒不是非常致命，比较致命的缓存雪崩是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

一般可以停掉一些服务，保证主要的服务可用

### 解决方案

**Redis高可用**

搭建集群（异地多活）

**限流降级**

在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

**数据预热**

在正式部署之前，先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/139610450](https://blog.csdn.net/dkbnull/article/details/139610450)

知乎：[https://zhuanlan.zhihu.com/p/702829923](https://zhuanlan.zhihu.com/p/702829923)

微信：[https://mp.weixin.qq.com/s/iASczvllGo7J__n5srYlnA](https://mp.weixin.qq.com/s/iASczvllGo7J__n5srYlnA)

---

