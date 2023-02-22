---
title: Redis相关使用
date: 2020-11-13 21:47:16
tags: [Java,tools]
categories: Linux
---

### Redis简介

Redis是现在最受欢迎的NoSQL数据库之一，Redis是一个使用ANSI C编写的开源、包含多种数据结构、支持网络、基于内存、可选持久性的键值对存储数据库，其具备如下特性：

<!--more-->

- 基于内存运行，性能高效
- 支持分布式，理论上可以无限扩展
- key-value存储系统
- 开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API

### Nosql

#### 概述

- NoSQL(NoSQL = Not Only SQL )，意即"不仅仅是SQL"，泛指非关系型的数据库。
- NoSQL不依赖业务逻辑方式存储,而以简单的key-value模式存储,因此大大的增加了数据库的扩展能力
- 不遵循SQL标准
- 不支持ACID(原子性,一致性,独立性,持久性)
- 远超SQL的性能

#### NoSQL适用场景

- 对数据的高并发读写
- 海量数据的读写
- 对数据高可扩展性的

#### 不适用场景

- 需要事务支持
- 基于sql的结构化查询存储,处理复杂的关系,需要及席查询(条件查询)

### 使用

#### 启动命令

|             命令              |       作用        |
| :---------------------------: | :---------------: |
|     systemctl start redis     |     启动redis     |
|     systemctl stop redis      |     停止redis     |
|    systemctl status redis     | 查看redis运行状态 |
|             ps ef             |    grep redis     |
|           redis-cli           | 开启(本机)客户端  |
| redis-cli -h 127.0.0.1 -p6379 |    开启客户端     |

#### 使用命令

|          命令          |                      作用                      |
| :--------------------: | :--------------------------------------------: |
|         keys *         |               查询当前库的所有键               |
|      exists <key>      |               判断某个键是否存在               |
|       type <key>       |                  查看键的类型                  |
|       del  <key>       |                   删除某个键                   |
| expire <key> <seconds> |          为键值设置过期的时间,单位秒           |
|       ttl <key>        | 查看还有多少秒过期,-1表示永不过期,-2表示已过期 |
|         dbsize         |           查看当前数据库的key的数量            |
|        Flushdb         |                   清空当前库                   |
|        Flushall        |                   清空所有库                   |

#### 数据操作

|                  命令                   |                             作用                             |
| :-------------------------------------: | :----------------------------------------------------------: |
|                get <key>                |                         查询对应键值                         |
|            set <key><value>             |                          添加键值对                          |
|           append <key><value>           |                   将value最佳到指定键值末                    |
|              strlen <key>               |                         获得值的长度                         |
|           setnx <key><value>            |                 只有在key不存在时设置key的值                 |
|            incr / decr <key>            | 将key中存储的数字值增1/减1(只能对数字值操作,如果为空,新增值为1/-1) |
|      incrby / decrby <key> <步长>       |               将key中存储的数字增减,自定义步长               |
| mset<key1><value1> / mget <key1> <key2> |                         多个添加获取                         |
|       getrange <key><start><end>        |              获取值的范围,类似java中的substring              |
|      setrange <key><start><value>       |       用<value>覆写<key>所存储的字符串值,从<start>开始       |

 默认16个数据库,类似数组下标从0开始,初始默认使用0号库

使用 select <id> 来切换数据库

### 五种数据结构

![](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201204163643.png)

#### 1.String 字符串类型

是redis中最基本的数据类型，一个key对应一个value。

String类型是二进制安全的，意思是 redis 的 string 可以包含任何数据。如数字，字符串，jpg图片或者序列化的对象。

使用：get 、 set 、 del 、 incr、 decr 等

实战场景：

1. 缓存： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。

2. 计数器：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。

3. session：常见方案spring session + redis实现session共享

#### 2.Hash

是一个Mapmap，指值本身又是一种键值对结构，如 value={ {field1,value1},......fieldN,valueN} }

使用：所有hash的命令都是 h  开头的   hget 、hset 、 hdel 等

```shell
127.0.0.1:6379> hset user name1 hao
(integer) 1
127.0.0.1:6379> hset user email1 hao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
127.0.0.1:6379> hget user user
(nil)
127.0.0.1:6379> hget user name1
"hao"
127.0.0.1:6379> hset user name2 xiaohao
(integer) 1
127.0.0.1:6379> hset user email2 xiaohao@163.com
(integer) 1
127.0.0.1:6379> hgetall user
1) "name1"
2) "hao"
3) "email1"
4) "hao@163.com"
5) "name2"
6) "xiaohao"
7) "email2"
8) "xiaohao@163.com"
```

实战场景：

1. 缓存： 能直观，相比string更节省空间，的维护缓存信息，如用户信息，视频信息等。

#### 3.链表 

List 说白了就是链表（redis 使用双端链表实现的 List），是有序的，value可以重复，可以通过下标取出对应的value值，左右两边都能进行插入和删除数据。

```shell
127.0.0.1:6379> lpush mylist 1 2 ll ls mem
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "mem"
2) "ls"
3) "ll"
4) "2"
5) "1"
127.0.0.1:6379>
```

实战场景：

1.timeline：例如微博的时间轴，有人发布微博，用lpush加入时间轴，展示新的列表信息。

#### 4.Set  集合

集合类型也是用来保存多个字符串的元素，但和列表不同的是集合中 

1. 不允许有重复的元素
2. 集合中的元素是无序的，不能通过索引下标获取元素
3. 支持集合间的操作，可以取多个集合取交集、并集、差集

使用：命令都是以s开头的 sset 、srem、scard、smembers、sismember

```shell
127.0.0.1:6379> sadd myset hao hao1 xiaohao hao
(integer) 3
127.0.0.1:6379> SMEMBERS myset
1) "xiaohao"
2) "hao1"
3) "hao"
127.0.0.1:6379> SISMEMBER myset hao
(integer) 1
```

实战场景;

1. 标签（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。

2. 点赞，或点踩，收藏等，可以放到set中实现

#### 5.zset 有序集合

有序集合和集合有着必然的联系，保留了集合不能有重复成员的特性，区别是，有序集合中的元素是可以排序的，它给每个元素设置一个分数，作为排序的依据。

使用： 有序集合的命令都是 以 z 开头  zadd 、 zrange、 zscore

```shell
127.0.0.1:6379> zadd myscoreset 100 hao 90 xiaohao
(integer) 2
127.0.0.1:6379> ZRANGE myscoreset 0 -1
1) "xiaohao"
2) "hao"
127.0.0.1:6379> ZSCORE myscoreset hao
"100"
```

实战场景：

1. 排行榜：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行。

### 相关配置

#### ip地址的绑定(bind)

- 默认情况下bind = 127.0.0.1只能接受本机访问,不写无限制接收任何ip访问(生产环境要写应用服务器的地址)
- 如果开启了protexted-mode,在没有设定bind ip 且没有设密码的情况下,只允许接收本机访问

#### tcp-backlog

- 请求到达后至少接受进程处理前的队列
- backlog队列总和 = 未完成三次握手队列 + 已经完成三次握手的队列
- 高并发环境tcp-backlog设置值跟超时时限内的Redis吞吐量决定

#### timeout

- 一个空闲的客户端维持多少秒会关闭,0为永不关闭

#### TCP keepalive

- 对访问客户端的一种心跳检测,每n秒检测一次,官方推荐为60秒

#### requirepass

- 设置永久密码

### Jedis

#### 导入依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.11</version>
</dependency>
```

#### 连接Redis

 ```java
Jedis jedis = new Jedis("xxx.xxx.xx.xx",6379);
jedis.ping();
jedus.close();
 ```

### Redis事务

#### 定义

- Redis事务是一个单独的隔离操作: 事务中的所有命令都会序列化, 按顺序的执行,事务在执行的过程中, 不iu被其他客户端发送来的命令请求所打断

- Redis事务的主要作用就是串联多个命令防止别人的命令插队

####  Multi, Exec, discard

- 从输入Multi命令开始, 输入的命令都会依次进入命令队列中, 但不会执行, 直到输入Exec后, Redis会将之前的命令队列中的命令依次执行.
- 组队的过程中可以通过discard来放弃组队(回滚).

![](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201115155543.png)

![](https://s2.loli.net/2022/04/07/SjUIoKfFGyzsi1L.png)



#### 事务的错误处理

- 组队中某个命令如果出现了报告错误, 执行时整个的队友队列都会被取消

![事务的错误处理](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201115155931.png)

- 如果执行阶段某个命令报出了错误, 则只有报错的命令不会被执行, 而其他的命令都会执行,不会回滚

![事务的错误处理](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201115160127.png)

#### WATCH (监视)

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个)key ，如果在事务执行之前这个(或这些) key被其他命令所改动，那么事务将被打断，unwatch取消监视

### Redis持久化

Redis提供了2个不同形式的持久化方式

#### RDB(Redis DataBase)

- 在指定的时间间隔内将内存中的数据集快照(Snapshot)写入磁盘，它恢复时是将快照文件直接读到内存里。
- Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能,如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

- RDB保存的文件在redis.conf中配置文件名称,默认为dump.rdb,保存路径也可修改

- RDB的保存策略 

  save	900	1(900秒内保存1次)

#### AOF(Append Of File)

- 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
- AOF默认不开启, 需要在配置文件中配置  apeendonly 为 yes
- AOF与RDB同时开启, 以AOF为准

### 集群

#### 主从复制

- 作用: 	1.读写分离(主机写从机读)	 2.容灾的快速恢复	

- 复制原理: Slave启动成功连接到master后会发送一个sync命令,master接到后启动后台的存盘进程,将所有数据传送到slave

![](https://s2.loli.net/2022/04/04/s3u9aZlbWU6xIp2.png)cc

| 命令                   | 作用             |
| ---------------------- | ---------------- |
| info replication       | 查看主机运行状态 |
| slaveof 127.0.0.1 6379 | 设置从机         |
| slaveof no one         | 讲从机变为主机   |

注意:本地测试时配置redis.windows-service.conf(redis.conf) 文件 bind IP地址（不要用127.0.0.1）

redis-cli -h IP地址 -p 端口号 -a 密码  

#### 哨兵模式(分片集群)

1. redis安装目录下创建`sentinel.conf`文件

2. 写入`sentinel monitor mymaster 127.0.0.1 1`	"1"表示至少多少个哨兵同意迁移的数量

3. 启动哨兵 `redis-sentinel sentinel.cof`

4. 主机选择 优先级在redis.conf 中默认: `slave-prioriy `( replica-prioriy 新 ) 100，值越小优先级越高

5. Java中使用

   ![](https://s2.loli.net/2022/04/04/TyqZxJYLtFd51lU.png)

#### 使用

Redis集群实现了对Redis 的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis集群通过分区( partition )来提供一定程度的可用性( av ailability ): 即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求。·

##### 配置(redis.conf)

| 配置项                              | 作用                                  |
| ----------------------------------- | ------------------------------------- |
| cluster-enabled yes                 | 打开集群模式                          |
| cluster-config-file nodes-6379.conf | 设定节点配置文件名                    |
| cluster-node-timeout 15000          | 设定节点失联时间,超过时间自动主从切换 |

创建集群: redis-cli -cluster create ---replicas 1 ip1:port ip2:port ip3:port ip4:port ip5:port ip6:portc	(1,2,3主3,4,5从),

一个集群至少要有三个主节点	"1"表示希望为集群中的每个主节点创建—个从节点。分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个P地址上.

##### 命令

| 命令                 | 作用               |
| -------------------- | ------------------ |
| redis-cli -c -p 6379 | 集群方式连接       |
| cluster nodes        | 查看集群中节点信息 |

##### Jedis使用

即使连接的不是主机，集群会自动切换主机 存储。主机写，从机读。无中心化主从集群。无论从哪台主机写的数据，其他主机上都能读到数据。

![](https://s2.loli.net/2022/04/05/7N46pY3kZhJBRzC.png)

### 过期键删除策略

Redis keys过期有两种方式: 被动和主动方式

- 主动: 当客户端访问它时, key会被发现并主动过期
- 被动: Redis每秒做10次随机测试20个Keys进行过期检测, 删除过期Keys, 如多余25%过期, 再次抽20个

### 缓存回收

Redis中提供了一种内存淘汰策略，当内存不足时，Redis会根据相应的淘汰规则对key数据进行淘汰。Redis一共提供了8种淘汰策略

- **volatile-lru**，针对设置了过期时间的key，使用lru算法进行淘汰。
- **allkeys-lru**，针对所有key使用lru算法进行淘汰。
- **volatile-lfu**，针对设置了过期时间的key，使用lfu算法进行淘汰。
- **allkeys-lfu**，针对所有key使用lfu算法进行淘汰。
- **volatile-random**，从所有设置了过期时间的key中使用随机淘汰的方式进行淘汰。
- **allkeys-random**，针对所有的key使用随机淘汰机制进行淘汰。
- **volatile-ttl**，删除生存时间最近的一个键。
- **noeviction**，不删除键，值返回错误。(默认)

注: LRU---Least Recently Used 近期最少使用算法(最后一次被使用到发生调度的时间长短)			LFU----Least Frequently Used算法,即最少访问算法(一定时间段内页面被使用的频率)

### 缓存穿透,击穿,雪崩

![图片](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/061e2c04e0ebca3425dd75dd035b6b7b.png)
