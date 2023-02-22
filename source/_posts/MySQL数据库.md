---
title: MySQL数据库
date: 2021-07-08 20:40:23
tags: [MySQL,数据库]
categories: [Java,MySQL,数据库]
---

### Mysql架构

#### 数据存储目录

/var/lib/mysql目录下

- InnoDB: 	8.0会产生1个xxx.idb 	5.7会产生2个文件xxx.frm(表结构) xxx.idb	(独立表空间模式存储在ibdata1中)
- MyISAM:    3个文件 	1.5.7中:xxx.frm,8.0中xxx.sdi 	2.xxx.MYD(数据信息) 	4.xxx.MYI(索引信息)

<!--more-->

#### 逻辑架构

MySQL是典型的c/s架构，即 Client/Server架构，服务器端程序使用的mysqld。

![](https://s2.loli.net/2022/04/10/w3Rz4dstWVSqkvm.png)

- 连接层: 系统（客户端）访问MySQL服务器前，做的第一件事就是建立TCP连接。
  经过三次握手建立连接成功后，MySQL服务器对TCP传输过来的账号密码做身份认证、权限获取。
- 服务层: 第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。
- 引擎层

##### SQL执行流程

![](https://s2.loli.net/2022/04/10/Ep3GgOoaUed8zJX.png)

Oracle提出了共享池的概念,通过共享池来判断是软解析,还是硬解析

##### 数据库缓冲池

InnoDB存储引擎是以页为单位来管理存储空间的，我们进行的增删改查操作其实本质上都是在访问页面。而磁盘I/o需要消耗的时间很多，而在内存中进行操作，效率则会高很多，为了能让数据表或者索引中的数据随时被我们所用，DBMS 会申请*占用内存来作为数据缓冲池*，在真正访问页面之前，需要把在磁盘上的页缓存到内存中的 *Buffer Pool*之后才可以访问。

![](https://s2.loli.net/2022/04/11/umTojzQqXcJ1OWA.png)

#### 存储引擎

##### **InnoDB:具备外键支持功能的事务存储引擎**

- InnoDB是MysQL的默认事务型引擎，它被设计用来处理大量的短期(short-lived)事务。可以确保事务的完整提交(Commit)和回滚(Rollback)。
- 更新、删除操作，应优先选择lnnoDB存储引擎。
- 对比MylSAM的存储引擎，InnoDB写的处理效率差一些，并且会占用更多的磁盘空间以保存数据和索引。
- MyISAM只缓存索引，不缓存真实数据;InnoDB不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响。

##### **MyISAM:非事务处理存储引擎**

- MyISAM提供了大量的特性，包括全文索引、压缩、空间函数(GIS)等，但MylSAM不支持事务、行级锁、外键，有一个毫无疑问的缺陷就是崩溃后无法安全恢复。
- 优势是访问的速度快，对事务完整性没有要求或者以SELECT、INSERT为主的应用

##### Archive:用于数据存档

- archive是归档的意思，仅仅支持插入和查询两种功能（行被插入后不能再修改)。
- 拥有很好的压缩机制，使用zlib压缩库，在记录请求的时候实时的进行压缩，经常被用来作为仓库使用。
- 根据英文的测试结论来看，同样数据量下，`Archive表比MyISAM表要小大约75%`，比支持事务处理的InnoDB表小大约83%。

##### Blackhole:丢弃写操作,读操作会返回空内容

- Blackhole引擎没有实现任何存储机制，它会丢弃所有插入的数据，不做任何保存。
- 但服务器会记录Blackhole表的日志，所以可以用于复制数据到备库，或者简单地记录到日志。但这种应用方式会碰到很多问题，因此并不推荐。|

##### CVS:存储数据时,逗号分隔各个数据

- CSV引擎可以将普通的CSV文件作为My SQL的表来处理，但不支持索引
- CSV引擎可以作为一种数据交换的机制，非常有用。
- 对于数据的快速导入、导出是有明显优势的。

##### Memory:数据置于内存的表

- Memory采用的逻辑介质是内存，响应速度很快，但是当mysqld守护进程崩溃的时候数据会丢失。另外，要求存储的数据是数据长度不变的格式，比如，Blob和Text类型的数据不可用(长度不固定的)。

##### Federated:访问远程表

- Federated引擎是访问其他MysQL服务器的一个代理，尽管该引擎看起来提供了一种很好的跨服务器的灵活性，但也经常带来问题，因此默认是禁用的。

##### Merge:管理各个MyISAM表构成的表集合

##### NDB:MySQL集群专用存储引擎

- 也叫做NDB Cluster存储引擎，主要用于MysQL Cluster分布式集群环境，类似于Oracle的RAC集群。

### InnoDB数据存储结构

#### 页

1. lnnoDB将数据划分为若干个面, lnnoDB中页的大小默认为16KB。
2. 以页作为磁盘和内存之间交互的基本单位，也就是一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。也就是说，**在数据库中，不论读一行，还是读多行，都是将这些行所在的页进行加载。也就是说，数据库管理存储空间的基本单位是页(Page)**，数据库V/O操作的最小单位是页。一个页中可以存储多个记录

####  页的上层结构

在数据库中，还存在着区(Extent)、段(Segment)和表空间（Tablespace)的概念。行、页、区、段、表空间的关系如下图所示:

![](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204171515764.png)

- 碎片区: 碎片区中的页不属于同一段，碎片区直属于**表空间**
- 区(Extent)是比页大一级的存储结构，***存放连续的页，在InnoDB存储引擎中***，一个区会分配64个连续的页。因为InnoDB中的页大小默认是16KB，所以一个区的大小是64*16KB= 1MB。
- 段（Segment)由一个或多个区组成，***区分叶子节点和非叶子节点的区的集合***，区在文件系统是一个连续分配的空间(在InnoDB中是连续的64个页)，不过在段中不要求区与区之间是相邻的。段是数据库中的分配单位，不同类型的数据库对象以不同的段形式存在。当我们创建数据表、索引的时候，就会相应创建对应的段，比如创建一张表时会创建一个表段，创建一个索引时会创建一个索引段。
- 表空间（Tablespace)是一个逻辑容器([idb文件](####数据存储目录))，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间。数据库由一个或多个表空间组成，表空间从管理上可以划分为系统表空间、用户表空间、撤销表空间、临时表空间等。

- 系统表空间: 与独立表空间类似，MySQL进程只有一个系统表空间，会额外记录系统信息

**为某个段分配存储空间的策略是这样的:**

1. 在刚开始向表中插入数据的时候，段是从某个碎片区以单个页面为单位来分配存储空间的。
2. 当某个段已经占用了32个碎片区页面之后，就会申请以完整的区为单位来分配存储空间。


#### 页的内部结构

1. 页如果按类型划分的话，常见的有`数据页（保存B+树节点)`、系统页、Undo 页和事务数据页等。数据页是我们最常使用的页。

2. 数据页的16KB大小的存储空间被划分为七个部分，分别是文件头(File Header)、页头(Page Header)、最大最小记录(Infimum+supremum)、用户记录(User Records)、空闲空间(Free Space)、页目录(PageDirectory)和文件尾(File Tailer) 。

  ![](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204171544472.png)

![](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204171545764.png)

#### InnoDB行格式(记录格式)

#####  COMPACT行格式

在MySQL 5.1版本中，默认设置为Compact行格式。一条完整的记录其实可以被分为记录的额外信息和记录的真实数据两大部分。

![image-20220417204627307](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204172046388.png)

##### Dynamic和Compressed行格式

 行溢出

##### Redundant行格式

![image-20220420171406913](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204201714032.png)

### (重点)索引及调优

#### 索引的创建原则

**适合创建索引的情况:**

1. 频繁作为WHERE查询条件的字段
2. 经常GROUP BY 和 ORDER BY 的列
3. UPDATE , DELETE  的 WHERE 条件列
4. DISTINCT字段需要创建索引
5. 区分度高(散列性高)的列

**不适合创建索引的情况:**

1. WHERE使用不到的列
2. 有大量重复数据的列
3. 避免对经常更新的列
4. 不建议用无序的列
5. 不要定义冗余重复的索引
6. 建议单张表不要超过6个索引

#### 性能分析工具

##### **查看系统性能参数:**

| SHOW [GLOBAL\|SESSION] STATUS LIKE '参数'    | 作用                  |
| -------------------------------------------- | --------------------- |
| Connection                                   | 连接MySQL服务器的次数 |
| Uptime                                       | MySQL服务器上线时间   |
| Slow_queries                                 | 慢查询的次数          |
| Innodb_rows_read\|inserted\|updated\|deleted | 执行xx操作的行数      |
| Com_select\|insert\|update\|delete           | xx操作的次数          |
| Last_query_cost                              | SQL成本查询           |

#####  **Profile:**

| 命令                                 | 作用            |
| ------------------------------------ | --------------- |
| select @@profiling                   | 查看是否开启    |
| set profiling =1                     | 开启            |
| show profiles/profile [for query id] | 查看SQL执行时间 |

##### **EXPLAIN:**

`mysql> show variables like '%slow_query_log'`开启慢查询日志

**type: **system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range> index > ALL

SQL性能优化的目标: 至少要达到range级别，要求是ref级别，最好是consts级别。(阿里巴巴开发手册要求)

##### Trace分析优化器执行计划:

默认关闭	开启:`SET optimizer_trace= "enabled=on " , end_markers_in_json=on ; `  

设置最大能使用内存`set optimizer_trace_max_mem_size=100000;`

查看`select * from information_schema . optimizer_trace\G`

##### MySQL监控分析视图-sys schema 

#### 索引优化与查询优化

- 索引失效、没有充分利用到索引—―索引建立
- 关联查询太多JOIN(设计缺陷或不得已的需求)——sQL优化
- 服务器调优及各个参数设置（缓冲、线程数等）――调整my.cnf
- 数据过多——分库分表

SQL查询优化的技术有很多，但是大方向上完全可以分成**物理查询优化**和**逻辑查询优化**两大块。

- 物理查询优化是通过索引和才连接方式等技术来进行优化，这里重点需要掌握索引的使用。
- 逻辑查询优化就是通过SQL等价变换提升查询效率，直白一点就是说，换一种查询写法执行效率可能更高。

##### 关联查询优化

##### 子查询优化

##### 排序优化

1. SQL中，可以在WHERE子句和ORDER BY子句中使用索引，目的是在WHERE子句中避免全表扫描

2. 尽量使用Index完成ORDER BY排序。如果WHERE和ORDER BY后面是相同的列就使用单索引列;如果不同就使用联合索引。

3. 无法使用Index时，需要对FileSort方式进行调优。

   **FileSort排序:**双路排序和单路排序

   1. 提高sort_buffer_size
   2. 提高max_length_for_sort_data : 返回列的总长度大于此参数, 使用双路算法, 否则单路
   3. Order by 时 select *是大忌,最好只查询需要的字段


##### 覆盖索引

##### 索引条件下推ICP

默认开启

回表之前使用失效索引对数据进行过滤

##### 其他优化情况

1. EXISTS 和 IN:小表驱动大表

   ```sql
    SELECT * FROM A WHERE cc IN (SELECT cc FROM B) 	#A>B
   
   SELECT * FROM A WHERE cc EXISTS (SELECT cc FROM B WHERE B.cc=A.cc)	#B>A
   ```

   当A小于B时，用EXISTS。因为EXISTS的实现，相当于外表循环，实现的逻辑类似于:

   ```python
   for i in A
   	for j in B
       	if j.cc == i.cc then...
   ```

   当B小于A时用IN，因为实现的逻辑类似于:

   ```python
   for i in B
   	for j in A
       	if j.cc == i.cc then...
   ```

2. COUNT(*)与COUNT(具体字段)效率:

   InnoDB中尽量使用二级所用。因为主键采用的索引是聚簇索引，聚簇索引包含的信息多，二级索引只会加载部分字段。空间占用更小。对于COUNT(*)和COUNT(1)来说，它们不需要查找具体的行，只是统计行数，系统会自动采用占用空间更小的二级索引来进行统计。

3. LIMIT 1对优化影响:
   - 针对的是会扫描全表的SQL语句，加上LIMIT 1的时候，当找到一条结果的时候就不会继续扫描了，会加快查询速度。
   - 如果数据表已经对字段建立了唯一索引，那么可以通过索引进行查询，不会全表扫描，就不需要加上LIMIT1了。

4. 多使用COMMIT

   只要有可能，在程序中尽量多使用COMMIT，这样程序的性能得到提高，需求也会因为COMMIT所释放的资源而减少。

   COMMIT所释放的资源:

   - 回滚段上用于恢复数据的信息
   - 被程序语句获得的锁
   -  redo / undo log buffer中的空间
   - 管理上述3种资源中的内部花费

#### 数据库的设计规范 

##### 泛式

- 超键:能唯─标识元组的属性集叫做超键。
- 候选键:如果超键不包括多余的属性，那么这个超键就是候选键。·主键:用户可以从候选键中选择一个作为主键。
- 外键∶如果数据表R1中的某属性集不是R1的主键，而是另一个数据表R2的主键，那么这个属性集就是数据表R1的外键。
- 主属性:包含在任一候选键中的属性称为主属性。
- 非主属性:与主属性相对，指的是不包含在任何一个候选键中的属性。

###### 第一范式

第一范式主要是确保数据表中每个字段的值必须具有**原子性**，也就是说数据表中每个字段的值为不可再次拆分的最小数据单元。

###### 第二范式

第一范式的基础上，还要满足**数据表里的每一条数据记录，都是可唯一标识的。而且所有非主键字段，都必须完论依赖主键，不能只依赖主键的一部分。**如果知道主键的所有属性的值，就可以检索到任何元组(行)的任何属性的任何值。(要求中的主键，其实可以拓展替换为候选键)。

###### 第三范式

在第二范式的基础上，确保数据表中的每一个非主键字段都**和主键字段直接相关**，也就是说，要求数据表中的所有非主键字段不能依赖于其他非主键字段。通俗地讲，该规则的意思是所有非主键属性之间不能有依赖关系，必须相互独立。

##### 反范式化

增加冗余信息提高查询效率:

1. 这个冗余字段不需要经常进行修改
2. 这个冗余字段查询的时候不可或缺。(eg:历史快照)

### 事物

#### 基础知识

##### 事务的状态

- 活动的(active)
- 部分提交的(partially committed)
- 失败的(failed)
- 终止的(aborted)
- 提交的(committed)

##### 显式事务

| 命令                               | 作用         |
| ---------------------------------- | ------------ |
| START TRASNSACTION \| BEGIN        | 开启一个事务 |
| ROLLBACK\| ROLLBACK TO [SAVEPOINT] | 回滚事务     |
| COMMIT                             | 提交事务     |
| SAVEPOINT 保存点名称               | 创建保存点   |
| RELEASE SAVEPOINT xxx              | 删除保存点   |

##### 隐式事务

#### 隔离级别

> 对于同时运行的多个事务，当访问数据库中相同的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题
>
> - 脏读：T1修改了T2修改但未提交的数据，若 T2回滚，T1读取的内容是无效的
>
> - 脏读：T1读取了T2修改但未提交的数据，若T2回滚，T1读取的内容是无效的
> - 不可重复读：T1读取了之后T2更新了数据，T1再读取数据不同
> - 幻读：T1读取了之后T2插入了数据，T1再读取数据不同

##### MySQL的隔离级别

默认：REPEATABLE READ(可重复读)

![img](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204281752260.png)

![image-20220428175322713](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204281753769.png)

| 作用                 | 命令                                                         |
| -------------------- | ------------------------------------------------------------ |
| 查看当前隔离级别     | `select @@transaction_isolation`                             |
| 设置当前连接隔离级别 | `set transcation isolation level read committed`|`set transaction_isolation = '隔离级别'`[READ-UNCOMMITED \| READ-COMMITTED \| REPEATABLE-READ \| SERRIALIZEABLE] |
| 设置全局隔离级别     | `set global transcation isolation level read committed`      |

#### 事物日志

- 事务的隔离性由锁机制实现

- 而事务的原子性、一致性和持久性由事务的redo日志和undo日志来保证。
  - REDO LOG称为重做日志，提供再写入操作，恢复提交事务修改的页操作，用来保证事务的持久性
  - UNDO LOG称为回滚日志，回滚行记录到某个特定版本，用来保证事务的原子性、一致性。
- redo log:是存储引擎层(innodb)生成的日志，记录的是"**物理级别**"上的**页修改操作**，比如页号xxx、偏移量yyy写入了zzz数据。主要为了保证数据的可靠性;
- undo log:是存储引擎层(innodb)生成的日志，记录的是**逻辑操作**日志，比如对某一行数据进行了INSERT语句操作，那么undo log就记录一条与之相反的DELETE操作。主要用于事务的回滚(undo log记录的是每个修改操作的逆操作)和**一致性非锁定读**(undo log回滚行记录到某种特定的版本---MVCC，即多版本并发控制)。

##### redo日志

-  在执行事务的过程中，每执行一条语句，就可能产生若干条redo日志，这些日志是按照产生的**顺序写入磁盘的**，也就是使用顺序Io，效率比随机IO快。
- redo log跟bin log的区别，redo log是存储引擎层产生的，而bin log是数据库层产生的。假设一个事务，对表做10万行的记录插入，在这个过程中，一直不断的往redo log顺序记录，而bin log不会记录，直到这个事务提交，才会一次写入到bin log文件中。

###### redo的组成

- 重做日志的缓冲(redo log buffer), 保存在内存中, 是易丢失的

  在服务器启动时就向操作系统申请了一大片称之为redo log buffer的连续内存空间,这片内存空间被划分成若干个连续的redo log block 。一个redo log block占用512字节大小。

![image-20220429094110371](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204290941476.png)

- 重做日志文件( redo log file)，保存在硬盘中，是持久的。

###### redo的刷盘策略

![image-20220429094535061](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204290945110.png)

redo log buffer刷盘到redo log file的过程并不是真正的刷到磁盘中去，只是刷入到文件系统缓存

针对这种情况，InnoDB给出`innodb_flush_log_at_trx_commit`参数，该参数控制commit提交事务时，如何将redo log buffer中的日志刷新到redo log file中。它支持三种策略:

- 设置为0∶表示每次事务提交时不进行刷盘操作。(系统默认master thread每隔1s进行一次重做日志的同步)
- 设置为1∶表示每次事务提交时都将进行同步，刷盘操作（默认值)
- 设置为2∶表示每次事务提交时都只把redo log buffer内容写入page cache，不进行同步。由OS自己决定什么时候同步到磁盘文件。

![image-20220429100900436](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202204291009490.png)

##### undo日志

- 回滚数据
- MVCC

#### 锁

- MVCC: 读-写操作彼此不冲突, 性能更高
- 加锁: 读-写操作彼此需要排队, 影响性能

一般情况下采用MVCC来解决读-写操作并发执行的问题，但是业务在某些特殊情况下，要求必须采用加锁的方式执行。

##### 从对数据操作的类型（读\写）分:

- 读锁（S共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
- 写锁（X排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。

##### **从对数据操作的粒度分:**

- 表锁: 表级S-X锁, 意向锁, 自增锁, 元数据锁
- 行锁: 记录锁, 间隙锁, 临键锁, 插入意向锁
- 页锁: 

##### 从态度划分:

- 悲观锁:  
- 乐观锁: 不 采用锁机制, 采用程序来实现
  1. 版本号机制
  2. 时间戳机制

##### 从加锁方式划分:

- 隐式锁: INSERT操作下通过隐式锁保护这条记录在提交前不被别的事务访问, 冲突时转化为显式锁
- 显式锁:  `select * from performance_schema.data_lock_waits`能够查询到

#### 多版本并发控制(MVCC)

**<font color="red">MVCC的实现依赖于: 隐藏字段(事务id), Undo Log(多版本), Read View(并发控制)</font>**

MVCC主要是为了更好的方式去处理读-写冲突，有读写冲突时，不加锁，非阻塞并发读，这个读指的就是快照读,而非当前读。当前读实际上是一种加锁的操作，是悲观锁的实现。而MVCC本质是采用乐观锁思想的一种方式。

##### 快照读-当前读

- 快照读: 不加锁的简单的SELECT都属于快照读，即不加锁的非阻塞读;
- 当前读: 读取的是记录的最新版本（最新数据，而不是历史版本的数据），读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。加锁的SELECT，或者对数据进行增删改都会进行当前读。

##### ReadView

ReadView就是事务A在使用MVCC机制进行快照读操作时产生的读视图。当事务启动时，会生成数据库系统当前的一个快照，InnoDB为每个事务构造了一个数组,用来记录并维护系统当前活跃事务的ID(启动了但还没提交)。

##### RedView结构

1. `creator_trx_id` 创建这个Read View的事务ID。
2. `trx_ids` 表示在生成ReadView时当前系统中**活跃**的读写事务的事务id列表。
3. `up_limit_id` 活跃的事务中最小的事务ID。
4. `low_limit_id` 表示生成ReadView时系统中应该分配给下一个事务的id值(trx8+1)。

​	举例:

![image-20220506105946271](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202205061059382.png)

##### ReadView规则

- 如果被访问版本的trx_id属性值与ReadView中的`creator_trx_id`值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
- 如果被访问版本的trx_id属性值小于ReadView中的`up_limit_id`值，表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。
- 如果被访问版本的trx_id属性值大于或等于ReadView中的`low_limit_id`值，表明生成该版本的事务在当前事务生成Readview后才开启，所以该版本不可以被当前事务访问。
- 如果被访问版本的trx_id属性值在ReadView的`up_limit_id`和`low_limit_id`之间，那就需要判断一下trx_id属性值是不是在`trx_ids`列表中。
  - 如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问。
  - 如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。

##### MVCC操作流程

1. 首先获取事务自己的版本号，也就是事务ID;
2. 获取ReadView;
3. 查询得到的数据，然后与ReadView中的事务版本号进行比较;
4. 如果不符合ReadView规则，就需要从 Undo Log中获取历史快照;
4. 最后返回符合规则的数据。

### 日志

- 慢查询日志: 记录所有执行时间超过long_query_time的所有查询，方便我们对查询进行优化。
- 通用查询日志: 记录所有连接的起始时间和终止时间，以及连接发送给数据库服务器的所有指令，对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助。
- 错误日志: 记录MySQL服务的启动、运行或停止MySQL服务时出现的问题，方便我们了解服务器的状态，从而对服务器进行维护。
- 二进制日志: 记录所有更改数据的语句，可以用于主从服务器之间的数据同岁，以及服务器遇到故障时数据的无损失恢复。
- 中继日志: 用于主从服务器架构中，从服务器用来存放主服务器二进制日志内容的一个中间文件。从服务器通过读取中继日志的内容，来同步主服务器上的操作。
- 数据定义语句日志: 记录数据定义语句执行的元数据操作。

 除二进制日志外，其他日志都是文本文件。默认情况下，所有日志创建于MySQL数据目录中。

### 主从复制

1. 配置

```properties
[mysqld]
#[必须]主服务器唯一ID
server-id = 1
#[必须]启用二进制日志, 指定路径
log-bin = /log/mysqlbin
```

2. 建立账户并授权

```sql
#在主机中执行主从复制的命令 5.7
grant replication slave on *.* to 'slave1'@ '从机数据库IP' identified by 'abc123' 
```

```sql
create user 'slave1'@ '%' identified by 'abc123'
grant replication slave on *.* to 'slave1'@ '%'
alter user 'slave1'@ '%' identified with mysql_native_password by 'abc123' 
```

3. 从机配置 log_file=`show master status`

```sql
change master to master_host='主机的IP', master_user='主机用户名',master_password='用户密码',master_log_file='binlog.', master_log_pos='具体值'
```

4. `start slave`	`show status slave/g`

### 备份与恢复

- 物理备份︰备份数据文件，转储数据库物理文件到某一目录。物理备份恢复速度比较快，但占用空间比较大，MySQL中可以用xtrabackup工具来进行物理备份。

```mysql
#备份
flush tables with read lock;
cp -r #复制文件
unlock tables;
#恢复 给与mysql备份文件访问权限
chown -R mysql.mysql database1
```



- 逻辑备份:对数据库对象利用工具进行导出工作，汇总入备份文件内。逻辑备份恢复速度慢，但占用空间小，更灵活。MySQL中常用的逻辑备份工具为mysqldump。逻辑备份就是备份sql语句，在恢复的时候执行备份的saql语句实现数据库数据的重现。

```mysql
#备份
mysqldump -u 用户名称 -h 主机名称 备份数据库名[databse1]>databse1.sql
#恢复
mysql -u root -p datavase1<database1.sql
```
