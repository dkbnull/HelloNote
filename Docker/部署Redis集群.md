## 部署Redis集群
~~~shell
# 创建网卡
docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建六个redis配置
for port in $(seq 1 6); \
do
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

# 启动6个redis容器
for port in $(seq 1 6); \
do
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
done

# *可选 单个启动
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 进入redis
docker exec -it redis-1 sh

# 创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1

# 连接redis
redis-cli -c

# 集群信息
cluster info

# 当前连接节点所属集群的配置信息
cluster nodes

# 查看集群信息
redis-cli --cluster check ip:port
~~~

## 哈希取余分区

2亿条记录就是2亿个k.y，假设有3台机器构成一个集群，用户每次读写操作都是根据公式：hash(key) % N个机器台数，计算出哈希值，用来决定数据映射到哪一个节点上。

**优点**

简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。

使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡+分而治之的作用。

**缺点**

原来规划好的节点，进行扩容或者缩容时比较麻烦，每次数据变动导致节点有变动，映射关系需要重新进行计算，在服务器个数固定不变时没有问题，如果需要弹性扩容或故障停机，原来的取模公式就会发生变化。

此时地址经过取余运算的结果将发生很大变化，根据公式获取的服务器也会变得不可控。

某个redis机器宕机了，由于台数数量变化，会导致hash取余全部数据重新洗牌。

## 一致性哈希算法分区

为了在节点数目发生改变时尽可能少的迁移数据

将所有的存储节点排列在收尾相接的Hash环上，每个key在计算Hash后会顺时针找到临近的存储节点存放。

而当有节点加入或退出时仅影响该节点在Hash环上顺时针相邻的后续节点。

**步骤**

* 算法构建一致性哈希环
* 服务器ip节点映射
* key落到服务器的落键规则

**优点**

* 容错性
* 扩展性

加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。

**缺点**

* 数据倾斜

数据的分布和节点的位置有关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果。

## 哈希槽分区

为什么会出现

一致性哈希算法的数据倾斜问题

哈希槽实质就是一个数组，数组[0,2^14-1]形成hash slot空间

能干什么

解决均匀分配的问题，在数据和节点之间又加入了一层，把这层称为哈希槽(slot)，用于管理数据和节点之问的关系，现在就相当于节点上放的是槽，槽里放的是数据

槽解决的是粒度问题，相当于把粒度变大了，这样便于数据移动。

哈希解决的是映射问题，使用key的哈希值来计算所在的槽，便于数据分配。

名少个hash槽

一个集群只能有16384个植，编号0-16383 (0-2414-1)。

这些槽会分配给集群中的所有主节点，分配策略没有要求。可以指定哪些编号的槽分配给哪个主节点。集群会记录节点和槽的对应关系。解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取余，余数是几key就落入对应的槽里。slot= CRC16(key) % 16384，以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了



Redis 集群中内置了 16384 个哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在 Redis 集群中放置一个 key-value时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，也就是映射到某个节点上。

## 主从扩容

* 新建6377、6378两个节点+新建后启动+查看是否8节点

~~~shell
docker run -p 6377:6379 -p 16377:16379 --name redis-7 \
-v /mydata/redis/node-7/data:/data \
-v /mydata/redis/node-7/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.17 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6378:6379 -p 16378:16379 --name redis-8 \
-v /mydata/redis/node-8/data:/data \
-v /mydata/redis/node-8/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.18 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
~~~

* 进入6377容器实例内部

~~~shell 
docker exec -it redis-7 sh
~~~

* 将新增的6377节点(空槽号)作为master节点加入原集群

~~~shell
redis-cli --cluster add-node 172.38.0.17:6379 172.38.0.11:6379
~~~

* 检查集群情况第1次

~~~shell
redis-cli --cluster check ip:port
~~~

* 重新分派槽号

~~~shell
redis-cli --cluster reshard ip:port
~~~

* 检查集群情况第2次

为什么6377是3个新的区间，以前的还是连续?

重新分配成本太高，所以前3家各自匀出来一部分，从6371/6372/6373三个旧节点分别匀出1364个坑位给新节点6377

* 为主节点6377分配从节点6378

~~~shell
redis-cli --cluster add-node 172.38.0.18:6379 172.38.0.17:6379 --cluster-slave --cluster-master-id 新主机节点id
~~~

* 检查集群情况第3次

## 主从缩容

* 检查集群情况1，获得6378的节点ID
* 将6378删除，从集群中将4号从节点6378删除

~~~shell
redis-cli --cluster del-node 172.38.0.18:6379 从机6378节点di
~~~

* 将6377的槽号清空，重新分配，本例将清出来的槽号都给6371

~~~shell
redis-cli --cluster reshard 172.38.0.11:6379
~~~

* 检查集群情况第二次

* 将6377删除

~~~shell
redis-cli --cluster del-node 172.38.0.17:6379 主机6377节点di
~~~

* 检查集群情况第三次



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/130022026](https://blog.csdn.net/dkbnull/article/details/130022026)

微信：[https://mp.weixin.qq.com/s/YoiWd3tygrYZ_dLxaqpOlQ](https://mp.weixin.qq.com/s/YoiWd3tygrYZ_dLxaqpOlQ)

---