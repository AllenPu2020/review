# 数据库

###  常见数据库、协议默认端口号

Database | port
- | :-
MySQL | **3306**
Redis | **6379**
MongoDB | **27017**
http | **80**
https | **443**
SQLServer | **1433**
Oracle | **1521**

### 数据库的三范式

- **一、字段不可拆分**
    - 表的字段不可再拆分成更小的字段，原子性

- **二、完全依赖主键**
    - 在满足第一范式的基础上，非主键必须完全依赖主键，而不是仅仅依赖主键的一部分。

- **三、直接依赖主键**
    - 满足第二范式并且每个字段都不间接依赖于主键列。


### 事务的特性

- 1、**原子性(A)**：事务中的全部操作在数据库中是**不可分割**的, 要么全部完成，要么都不执行。
- 2、**一致性(C)**：几个*并行执行*的事务, 其执行结果必须与按*某一顺序串行执行*的结果**相一致**。
- 3、**隔离性(I)**：事务的执行**不受其他事务的干扰**，事务执行的中间结果对其他事务必须是透明的。
- 4、**持久性(D)**：对于任意已提交事务, 系统必须保证该事务对数据库的改变**不被丢失**，即使数据库出现故障。

### 事务隔离性的4种级别
> 数据库事务的隔离级别有4种，由低到高分别为Read uncommitted、Read committed、Repeatable read、Serializable<br>
大多数数据库默认的事务隔离级别是Read committed。如Sql Server, Oracle<br>
Mysql的默认隔离级别是Repeatable read

No.|class|描述|导致的问题
-|-|-|-
1 | Read uncommitted | 读未提交 | 脏读
2 | Read committed | 读提交 | 不可重复读
3 | Repeatable Read | 重复读 | 幻读
4 | Serializable | 序列化 | -

- Read uncommitted
    - 一个事务可以读取另一个未提交事务的数据

    > 事例：老板要给程序员发工资，程序员的工资是3.6万<br>但是发工资时老板不小心按错了数字，按成3.9万，该钱已经打到程序员账户，但是事务还没有提交。<br>就在这时，程序员去查看自己这个月的工资，发现比往常多了3千元，以为涨工资了非常高兴。<br>但是老板及时发现了不对，马上回滚，将数字改成3.6万再提交

    > 分析：实际程序员这个月的工资还是3.6万，但是他看到的是老板还没提交事务时的数据3.9万<br>
    这就是**脏读**

- Read committed
    - 一个事务要等另一个写操作事务提交后才能读取数据

    > 事例：程序员拿工资卡埋单时（事务开启），收费系统事先检测到他的卡里有3.6万<br>
    就在这个时候！！妻子要把钱全部转出充当家用，并提交<br>
    当收费系统准备扣款时，再检测卡里的金额，发现已经没钱了<br>
    第二次检测金额当然要等待妻子转出金额事务提交完<br>
    程序员就会很郁闷，明明卡里是有钱的…

    > 分析：这就是读提交，若有事务对数据进行更新（UPDATE）操作时，读操作事务要等待这个更新操作事务提交后才能读取数据，可以解决脏读问题。<br>
    但在这个事例中，出现了一个事务范围内两个相同的查询却返回了不同数据，这就是不可重复读

- Repeatable Read
    - 在开始读取数据（事务开启）时，不再允许修改操作

    > 事例：程序员拿工资卡埋单时（事务开启，不允许其他事务的UPDATE修改操作）<br>
    这个时候他的妻子不能转出金额了。接下来收费系统就可以扣款了

    > 分析：重复读可以解决不可重复读问题。
    不可重复读对应的是修改，即UPDATE操作，但是可能还会有幻读问题
    因为幻读问题对应的是插入INSERT操作，而不是UPDATE操作

    > 幻读出现的时机<br>
    程序员某一天去消费，花了2千元<br>
    然后妻子去查看他今天的消费记录（全表扫描FTS，妻子事务开启），看到确实是花了2千元<br>
    就在这个时候，程序员花了1万买了一部电脑，即新增(INSERT)了一条消费记录，并提交<br>
    当妻子打印程序员的消费记录清单时（妻子事务提交），发现花了1.2万元，似乎出现了幻觉，这就是幻读<br>

- Serializable
    - 最高的事务隔离级别
    - 事务串行化顺序执行
    - 可以避免脏读、不可重复读与幻读
    - 效率低下，比较耗数据库性能，一般不使用



### 视图的作用
- 提高了重用性，就像一个函数
- 对数据库重构，却不影响程序的运行
- 提高了安全性能，可以对不同的用户
- 让数据更加清晰

### 数据库索引

> 通俗的说，数据库索引好比是一本书前面的目录，能加快数据库的**查询速度**

- 数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询、更新数据库表中数据。
- 索引的实现通常使用 `B_TREE`。
    - 存储引擎不会再去扫描整张表得到需要的数据；
    - 从根节点开始，根节点保存了子节点的指针，存储引擎会根据指针快速寻找数据。
- 太多的索引会影响更新和插入的速度

### 数据库优化

- 优化索引、SQL 语句、分析慢查询

- 设计表的时候严格根据数据库的设计范式来设计数据库

- 使用缓存，把经常访问到的数据而且不需要经常变化的数据放在缓存中，能节约磁盘 IO

- 优化硬件；采用 SSD，使用磁盘队列技术(RAID0,RAID1,RDID5)等

- 采用 MySQL 内部自带的表分区技术，把数据分层不同的文件，能够提高磁盘的读取效率

- 垂直分表；把一些不经常读的数据放在一张表里，节约磁盘I/O

- 主从分离读写；采用主从复制把数据库的读操作和写入操作分离开来

- 分库分表分机器（数据量特别大），主要的的原理就是数据路由

- 选择合适的表引擎，参数上的优化

- 进行架构级别的缓存，静态化和分布式

- 不采用全文索引

- 采用更快的存储方式，例如 NoSQL存储经常访问的数

### 数据库查询优化
- **储存引擎选择**：
    - 如果数据表**需要事务**处理，应该考虑使用 **`InnoDB`**，因为它完全符合 ACID 特性。
    - 如果**不需要事务**处理，使用默认存储引擎 **`MyISAM`** 是比较明智的

- **分表分库**，**主从**

- **对查询进行优化**
    - 要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上**建立索引**
    - 应尽量**避免**在 `where` 子句中对字段**进行`null`值判断**，否则将导致引擎放弃使用索引而进行全表扫描
    - 应尽量**避免**在 `where` 子句中使用 `!=` 或 `<>` 操作符，否则将引擎放弃使用索引而进行全表扫描
    - 应尽量**避免**在 `where` 子句中使用 `or` 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描

- `Update` 语句，如果只更改 1、2 个字段，**不要 `Update` 全部字段**，否则频繁调用会引起明显的性能消耗，同时带来大量日志
- 对于多张大数据量（这里几百条就算大了）的表 JOIN，要**先分页再 `JOIN`**，否则逻辑读会很高，性能很差。


### 说一下MySQL数据库存储的原理

- **存储过程**是一个可编程的函数，它在数据库中创建并保存
- 它可以有SQL语句和一些特殊的控制结构组成
- 当希望在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的
- 数据库中的存储过程可以看做是对编程中面向对象方法的模拟。它允许控制数据的访问方式。

- 存储过程通常有以下优点：
    - 1) 能实现较快的执行速度。
    - 2) 允许标准组件是编程。
    - 3) 可以用流控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算。
    - 4) 可被作为一种安全机制来充分利用。
    - 5) 能减少网络流量。

### MySQL和Redis高可用性

- `MySQL Replication` 是 `MySQL` 官方提供的**主从同步**方案，用于将一个MySQL实例的数据，同步到另一个实例中。
    - `Replication` 为保证数据安全做了重要的保证，也是现在运用最广的 MySQL 容灾方案。Replication 用两个或以上的实例搭建了 MySQL 从复制集群，提供单点写入，多点读取的服务，实现了读的 scale out.

- `Sentinel` 是 `Redis` 官方为集群提供的高可用解决方案。
    - 在实际项目中可以使用 `sentinel` 去做 `redis` 自动故障转移，减少人工介入的工作量。另外 `sentinel`也给客户端提供了监控消息的通知，这样客户端就可根据消息类型去判断服务器的状态，去做对应的适配操作。

- 下面是是 Sentinel 主要功能列表 ：
    - `Monitoring`：Sentinel 持续检查集群中的 master、slave 状态，判断是否存活。
    - `Notification`：在发现某个 redis 实例死的情况下，Sentinel能通过    API 通知系统管理员或其他程序脚本。
    - `Automatic failover`：如果一个 `master` 挂掉后，`sentinel`立马启动故障转移，把某个 `slave` 提升为 `master`。其他的 `slave`重新配置指向新 `master`。`Configuration provider`：对于客户端来说 `sentinel` 通知是有效可信赖的。客户端会连接 `sentinel` 去请求当前 `master` 的地址，一旦发生故障 `sentinel` 会提供新地址给客户端。


### 怎样解决数据库`高并发`问题
- **分表分库**
- **数据库索引**
- **`redis` 缓存数据库**
- **读写分离**
- **负载均衡集群**：将大量的并发请求分担到多个处理节点。由于单个处理节点的故障不影响整个服务，负载均衡集群同时也实现了高可用性。

### 怎样解决`海量数据的存储和访问`造成系统设计瓶颈的问题

- **水平切分数据库**：可以降低单台机器的负载，同时最大限度的降低了宕机造成的损失
- **分库**: 降低了单点机器的负载
- **分表**: 提高了数据操作的效率
- **负载均衡**：可以降低单台机器的访问负载，降低宕机的可能性
- **集群**：解决了数据库宕机带来的单点数据库不能访问的问题
- **读写分离**：最大限度了提高了应用中读取数据的速度和并发量

### MySQL集群的优缺点
- 优点：
    - a) 99.999%的高可用性
    - b) 快速的自动失效切换
    - c) 灵活的分布式体系结构，没有单点故障
    - d) 高吞吐量和低延迟
    - e) 可扩展性强，支持在线扩容
- 缺点：
    - a) 存在很多限制，比如：不支持外键
    - b) 部署、管理、配置很复杂
    - c) 占用磁盘空间大，内存大
    - d) 备份和恢复不方便
    - e) 重启的时候，数据节点将数据load到内存需要很长时间

### MySQL集群搭建步骤

> 简介：MySQL 群集需要有一组计算机，每台计算机的角色可能是不一样的。MySQL群集中有三种节点：管理节点、数据节点和 SQL 节点。群集中的某计算机可能是某一种节点，也可能是两种或三种节点的集合。这三种节点只是在逻辑上的划分，所以它们不一定和物理计算机是一一对应的关系。管理节点（也可以称管理服务器）主要负责管理数据节点和SQL节点，还有群集配置文件和群集日志文件。它监控其他节点的工作状态，能够启动、关闭或重启某个节点。其他节点从管理节点检索配置数据，当数据节点有新事件时就把事件信息发送给管理节点并写入群集日志。数据节点用于存储数据。SQL 节点跟一般的 MySQL 服务器是一样的，我们可以通过它进行SQL 操作。用的 MysqlServer 已经不能满足群集的要求，配置群集需要使用MySQLCluster。

- MySQLCluster 的配置
    - 首先找三台电脑，或者是开三个虚拟机，管理节点部署在一台机子上，其他两台每台都部署一个数据节点和一个 SQL 节点。这里以两台机子举例，其中一台机器 A（IP 为 192.168.193.90）部署管理节点、数据节点和 SQL 节点，另一台机器 B（IP 为192.168.193.50）部署数据节点和 SQL 节点
    - 其实最好不要将管理节点跟数据节点部署到一台机子上，因为如果数据节点宕机会导致管理节点也不可用，整个 MySQL 群集就都不可用了。所以一个 MySQL 群集理想情况下至少有三台服务器，将管理节点单独放到一台服务器上。暂以两台举例，只是为了说明三种节点的配置启动方法
    - 将上面下载的安装包解压，并改文件夹名为 mysql，因为需要多次在命令行中操作，所以名字改短后更容易输入
    - 配置管理节点，配置数据节点，配置 SQL 节点
    - 启动管理节点，启动数据节点，启动 SQL 节点
    - 测试 MySQLCluster：我们需要测试三种情况：
        - 1. 在任一 SQL 节点对数据节点进行操作后，各数据节点是否能够实现数据同步。例如，我们在机器 A 上新创建一个数据库 myDB，然后再建一个表 student（新建表如下命令：createtable student (idint(2)) engine=ndbcluster），插入若干数据，接着我们到机器 B上查看是否能看到新的数据库myDB和新的表student以及插入数据
        - 2. 当关闭任一数据节点后，在所有 SQL 节点中进行操作是否不受其影响。例如，我们关闭机器 A 上的数据节点服务，在两台主机上应该能够继续对数据库进行各种操作。
        - 3. 关闭某数据节点进行了数据库操作，然后重新启动，所有 SQL节点的操作是否正常。
。

### 数据库负载均衡

> 负载均衡集群是由一组相互独立的计算机系统构成，通过常规网络或专用网络进行连接，由路由器衔接在一起，各节点相互协作、共同负载、均衡压力，对客户端来说，整个群集可以视为一台具有超高性能的独立服务器。

- 1) 实现原理
    - 实现数据库的负载均衡技术，首先要有一个可以控制连接数据库的控制端。在这里，它截断了数据库和程序的直接连接，由所有的程序来访问这个中间层，然后再由中间层来访问数据库。这样，我们就可以具体控制访问某个数据库了，然后还可以根据数据库的当前负载采取有效的均衡策略，来调整每次连接到哪个数据库。

- 2) 目的：实现多数据库数据同步

    > 对于负载均衡，最重要的就是所有服务器的数据都是**实时同步**的。

    > 这是一个集群所必需的，因为，如果数不据实时、不同步，那么用户从一台服务器读出的数据，就有别于从另一台服务器读出的数据，这是不能允许的。所以必须实现数据库的数据同步。这样，在查询的时候就可以有多个资源，实现均衡。

    > 比较常用的方法是 `Moebius for SQLServer`集群，`Moebius for SQL Server` 集群采用将核心程序驻留在每个机器的数据库中的办法，这个核心程序称为 Moebius for SQL Server 中间件，主要作用是监测数据库内数据的变化并将变化的数据同步到其他数据库中。

    > 数据同步完成后客户端才会得到响应，同步过程是并发完成的，所以同步到多个数据库和同步到一个数据库的时间基本相等；另外同步的过程是在事务的环境下完成的，保证了多份数据在任何时刻数据的一致性。正因为 `Moebius` 中间件宿主在数据库中的创新，让中间件不但能知道数据的变化，而且知道引起数据变化的 `SQL` 语句，根据 `SQL` 语句的类型智能的采取不同的数据同步的策略以保证数据同步成本的最小化。

    - 数据**条数很少**，数据**内容也不大**，则直接同步数据
    - 数据**条数很少**，但是里面包含**大数据类型**，（文本、二进制数据等），则先对数据进行**压缩**然后再同步，从而减少网络带宽的占用和传输所用的时间。
    - 数据**条数很多**
        - 中间件会拿到造成数据变化的SQL语句，对SQL语句进行解析，
        - 分析其执行计划和执行成本，并选择是同步数据还是同步SQL语句到其他的数据库中。
        - 此种情况应用在对表结构进行调整或者批量更改数据的时候非常有用。

- 3) 优缺点
    - 优点
        - **扩展性强**：当系统要更高数据库处理速度时，只要简单地增加数据库服务器就可以得到扩展
        - **可维护性**：当某节点发生故障时，系统会自动检测故障并转移故障节点的应用，保证数据库的持续工作。
        - **安全性**：因为数据会同步的多台服务器上，可以实现数据集的冗余，通过多份数据来保证安全性。另外它成功地将数据库放到了内网之中，更好地保护了数据库的安全性
        - **易用性**：对应用来说完全透明，集群暴露出来的就是一个IP
    - 缺点
        - 不能够按照 Web 服务器的处理能力分配负载
        - 负载均衡器(控制端)故障，会导致整个数据库系统瘫痪



### 你用的MySQL是哪个引擎，各引擎间有什么区别

- 一、`InnoDB` 支持事务，`MyISAM` 不支持，这一点是非常之重要
    - 事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了
- 二、MyISAM适合查询以及插入为主的应用，InnoDB适合频繁修改以及涉及到安全性较高的应用
- 三、InnoDB支持外键，MyISAM不支持
- 四、MyISAM是默认引擎，InnoDB需要指定
- 五、InnoDB不支持FULLTEXT类型的索引
- 六、InnoDB中不保存表的行数，如`selectcount(*)fromtable`时，`InnoDB` 需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好
的行数即可。
    - 注意的是，当`count(*)`语句包含`where`条件时`MyISAM`也需要扫描整个表；
- 七、对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在`MyISAM`表中可以和其他字段一起建立联合索引；
- 八、清空整个表时，InnoDB是一行一行的删除，效率非常慢。MyISAM则会重建表；
- 九、InnoDB 支持行锁（某些情况下还是锁整表，如 `update table set a=1 where user like '%lee%'`)

### `MySQLdb.connect`中 `cursorclass = MySQLdb.cursors.DictCursor` 参数用途

- `cursorclass` 是创建 `cursor` 对象的参数，执行查询后每条记录的结果以列表形式表示。

- `cursors` 参数为 `MySQLdb.cursors.DictCursor` 表示按字典形式返回查询记录。

### 用conn.execute()执行sql返回什么

- 返回执行 SQL 语句后受影响的行数

### cursor.execute查询结果后，有哪些方法可以获得结果

- `fetchone()`
- `fetchall()`
- `fetchmany()` 可指定返回个数 `fetchmany(3)`

### Redis基本类型、相关方法
> Redis 支持五种数据类型：string（字符串）、hash（哈希）、list（列表）、set（集合）及 zset(sorted set：有序集合)

- `String`
    - `String` 是 `Redis` 最为常用的一种数据类型，`String` 的数据结构为 `key/value` 类型，`String` 可以包含任何数据。
    - 常用命令: `set`, `get`, `decr`, `incr`, `mget`
- `Hash`
    - `Hash` 类型可以看成是一个 `key/value` 都是 `String` 的 `Map` 容器。
    - 常用命令：`hget`, `hset`, `hgetall`
- `List`
    - `List` 用于存储一个有序的字符串列表，常用的操作是向队列两端添加元素或者获得列表的某一片段。
    - 常用命令：`lpush`, `rpush`, `lpop`, `rpop`, `lrange`
- `Set`
    - `Set`可以理解为一组无序的字符集合，`Set`中相同的元素是不会重复出现的，相同的元素只保留一个。
    - 常用命令：`sadd`, `spop`, `smembers`, `sunion`
- `Sorted Set`（有序集合）
    - 有序集合是在集合的基础上为每一个元素关联一个分数，Redis通过分数为集合中的成员进行排序。
    - 常用命令：`zadd`, `zrange`, `zrem`, `zcard`

### Redis的应用场景
- 取**最新N个**数据的操作
- 排行榜应用,取 `TOP N` 操作
- 需要**精准设定过期时间**的应用
- 计数器应用
- `uniq` 操作,获取某段时间所有数据排重值
- Pub/Sub 构建实时消息系统
- 构建队列系统
- 缓存

### Redis事务
> Redis 事务允许一组命令在单一步骤中执行。

- 事务有两个属性
    - 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
    - Redis 事务是原子的。意味着要么所有的命令都执行，要么都不执行。

- 一个事务从开始到执行会经历以下三个阶段：
    - 开始事务
    - 命令入队
    - 执行事务

### Redis默认端口、有几个库、默认过期时间、Value最多可以容纳的数据长度
- 6379

- 16个（0-15）

- 永不过期

    > 一般情况下，当配置中开启了超出最大内存限制就写磁盘的话，那么没有设置过期时间的key可能会被写到磁盘上。假如没设置，那么 REDIS 将使用 `LRU` 机制，将内存中的老数据删除，并写入新数据。

- 512M



### Redis缓存命中率计算
- Redis提供了`INFO`这个命令，能够随时监控服务器的状态
- 只用`telnet`到对应服务器的端口，执行命令：`telnet localhost 6379 info`

> 在输出的信息里面有这几项和缓存的状态比较有关系：

item | value
- | -:
**keyspace_hits** | **14414110**
**keyspace_misses** | **3228654**
used_memory | 433264648
expired_keys | 1333536
evicted_keys | 1547380

- 通过计算`hits`和`miss`，我们可以得到缓存的命中率：14414110/(14414110+3228654)=81%
- 一个缓存失效机制，和过期时间设计良好的系统，命中率可以做到95%以上

### Redis由于存储的指纹过多怎么办

- 设置生存时间

- 定时清理

- 持久化

- 主从

### Redis预防攻击
- 主从、持久化存储、不以root账户启动、设置复杂密码、不允许 key方式登录


### Redis为什么需要做数据持久化处理

- Redis是内存型数据库，容量有限
- 内存在断电时会丢失所有数据，不安全
- 数据的使用一般不用Redis



### Redis和MySQL的区别

- `Redis` 是内存数据库，数据保存在内存中，速度快。
- `MySQL` 是关系型数据库，持久化存储，存放在磁盘里面，功能强大。检索的话，会涉及到一定的IO，数据访问也就慢。


### MongoDB

- `MongoDB` 特征
    - `MongoDB` 是一个**面向文档**的数据库系统。使用 **`C++`编写**，**不支持`SQL`** ，但有自己功能强大的查询语法。
    - `MongoDB` 使用`BSON`作为**数据存储**和**传输**的格式。
        - `BSON` 是一种类似 `JSON` 的二进制序列化文档，支持嵌套对象和数组。
    - `MongoDB` 对比 `MySQL`
        - `document` 对应 `MySQL` 的 `row`，`collection` 对应 `MySQL` 的 `table`

- 应用场景:
    - **网站数据**：`mongo` 非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
    - **缓存**：由于性能很高，`mongo`也适合作为信息基础设施的缓存层。在系统重启之后，由`mongo`搭建的持久化缓存可以避免下层的数据源过载。
    - **大尺寸、低价值的数据**：使用传统的关系数据库存储一些数据时可
能会比较贵，在此之前，很多程序员往往会选择传统的文件进行存储。
    - **高伸缩性的场景**：mongo 非常适合由数十或者数百台服务器组成的
数据库。
    - **对象及 `JSON` 数据的存储**：mongo 的 BSON 数据格式非常适合文
档格式化的存储及查询。
    - 重要数据：`MySQL`，一般数据：`MongoDB`，临时数据：`Redis`
    - **更快的视图**
        - 对于关系数据表而言，`mongodb` 是提供了一个更快速的视图 `view`；而对于 PHP 程序而言，mongodb 可以作为一个持久化的数组来使用，并且这个持久化的数组还可以支持排序、条件、限制等功能。
    - **代替 `mysql` 的部分功能**
        - 主要一个思考点就是： 把 `mongodb` 当作 `mysql` 的一个 view（视图），`view`是将表数据整合成业务数据的关键。
        - 比如说对原始数据进行报表，那么就要先把原始数据统计后生成view，在对 view 进行查询和报表。

- 不适合场景
    - **高度事务性的系统**
        - 例如银行或会计系统。传统的关系型数据库目前还是更适用于需要大量原子性复杂事务的应用程序。
    - **传统的商业智能应用**
        - 针对特定问题的 `BI`数据库会对产生*高度优化查询*方式。对于此类应用，*数据仓库*可能是更合适的选择。
    - **需要 `SQL` 的地方**
    - **重要数据，关系数据**

- 优点
    - **弱一致性**（最终一致），更能保证用户的访问速度
    - **文档结构的存储方式**，能够更便捷的获取数据
    - **内置 `GridFS`**，高效存储二进制大对象 (比如照片和视频)支持复制集、主备、互为主备、自动分片等特性
    - **动态查询**
    - **全索引支持**,扩展到内部对象和内嵌数组

- 缺点
    - **不支持事务**
    - **占用空间过大**
    - **维护工具不够成熟**

### Redis和MongoDB优缺点

- **各自特征**
    - `MongoDB` 和 `Redis`都是 `NoSQL`，采用结构型数据存储。
    - 二者在使用场景中，存在一定的区别，这也主要由于二者在**内存映射**的处理过程，**持久化**的处理方法不同。
    - MongoDB集群部署，更多的考虑到**集群**方案，Redis更偏重于**进程顺序写入**，虽然支持集群，也仅限于*主-从*模式.

- **Redis优点**
    - **读写性能优异**
    - **支持数据持久化**，支持 `AOF` 和 `RDB` 两种持久化方式
    - 支持**主从复制**，主机会自动将数据同步到从机，可以进行读写分离。
    - **数据结构丰富**：除了支持 `string` 类型的 `value` 外还支持`string`、`hash`、`set`、`sortedset`、`list` 等数据结构。

- **Redis缺点**
    - Redis**不具备自动容错和恢复功能**，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。

    - 主机宕机，宕机前有**部分数据未能及时同步**到从机，切换 IP后还会引入数据不一致的问题，降低了系统的可用性。

    - Redis的主从复制采用**全量复制**

        > 复制过程中主机会fork出一个子进程对内存做一份快照，并将子进程的内存快照保存为文件发送给从机，这一过程需要确保主机有足够多的空余内存。若快照文件较大，对集群的服务能力会产生较大的影响，而且复制过程是在从机新加入集群或者从机和主机网络断开重连时都会进行，也就是网络波动都会造成主机和从机间的一次全量的数据复制，这对实际的系统运营造成了不小的麻烦。

    - Redis**较难支持在线扩容**

        > 在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

- **MongoDB优点**
    - **弱一致性**（最终一致），更能保证用户的访问速度
    - **文档结构的存储方式**，能够更便捷的获取数据
    - **内置 `GridFS`**，高效存储二进制大对象 (比如照片和视频)支持复制集、主备、互为主备、自动分片等特性
    - **动态查询**
    - **全索引支持**,扩展到内部对象和内嵌数组

- **MongoDB缺点**
    - **不支持事务**
    - **占用空间过大**
    - **维护工具不够成熟**


### MySQL 命令行
> db_name: 数据库名<br>
tb_name: 数据表名<br>
c: 列名<br>
c_type: 数据类型<br>
c_cons: 约束constraint<br>
v: 值<br>
condition: 条件<br>
<br>

#### 终端命令
> `***` 表示密码 /可以不直接输入，回车后输入看不见/ <br>
`root` 表示用户名

- 连接
    - `mysql -uroot -p***`

- 备份
    - `mysqldump -uroot -p*** db_name > db_name_new.sql`

- 恢复
    - `mysql -uroot -p*** db_name_new < db_name.sql`    

#### database
- 查看所有
    - `show databases;`

- 查看当前
    - `select database();`

- 使用数据库
    - `use db_name;`

- 创建数据库
    - `create database db_name charset='utf8';`

- 删除数据库
    - `drop database db_name;`

#### table
- 查看所有
    - `show tables;`

- 查看表结构
    - `desc tb_name;`

- 查看表创建
    - `show crate table tb_name;`

- 创建表
    - `create table tb_name(c c_type c_cons, ...);`

- 修改表
    - 添加字段
        - `alter table tb_name add c c_type;`

    - 修改字段名
        - `alter talbe tb_name change c c_new c_type c_cons;`

    - 修改字段类型、约束
        - `alter table tb_name modify c c_type c_cons;`

    - 删除字段
        - `alter table tb_name drop c;`

- 删除表
    - `drop table tb_name;`

#### data
- 查看所有
    - `select * from tb_name;`

- 查看指定列
    - `select c1,c2 ... from tb_name;`

- 增加
    - 单行全列
        - `insert into tb_name values(v1,v2,...);`

    - 单行部分列
        - `insert into tb_name(c1,c2, ...) values(v1,v2, ...);`

    - 多行全列
        - `insert inot tb_name values(v1,v2, ...),(v1,v2, ...) ...;`

    - 多行部分列
        - `insert inot tb_name(c1,c2, ...) values(v1,v2, ...),(v1,v2, ...) ...;`

- 修改
    - `update tb_name set c1=v1, c2=v2, ... where condition;`

- 删除
    - 硬删除
        - `delete from tb_name where condition;`

    - 软删除(推荐)
        - `update tb_name set is_delete=1 where condition;`

#### data query
- 别名 `as`
    - 列取别名
        - `select c1 as 别名1, c2 as 别名2 ... from tb_name;`

    - 表取别名
        - `slect t.c1, t.c2 ... from tb_name as t;`

- 查询去重 `distinct`
    - `select distinct c1,c2 ... from tb_name;`

- 聚合函数 `count()` `max()` `min()` `sum()` `avg()`
    - `slect count(*) from tb_name;`
    - `slect sum(age)/count(*) from tb_name;`

- 条件

    > 比较/逻辑/模糊/范围/判空

    - 比较 `= > >= < <= !=`
        - `... where id > 3;`
        - `... where id = 3;`
        - `... where name != 'Alice';`

    - 逻辑 `and or not`
        - `... where id>3 and id_delete=0;`
        - `... where age>18 or gender=0;`

    - 模糊 `like`

        > `%` 任意数量<br> `_` 1个

        - `... where name like 'A%';`
        - `... where name like '王_';`
        - `... where name like 'A%' and name like '%n';`

    - 范围 `in` `between and`
        - `... where id in (1,2,3);` 非连续范围
        - `... where id between 3 and 8;` 连续范围

    - 判空 `is null` `is not null`
        - `... where age is null;`
        - `... where height is not null;`

    - 优先级
        - 括号 > not > 比较 > and > or

- 分组 `group by`
    - `select c1,c2 from db_name group by c1;`

    - 分组
        - `select gender from students group by gender;`

    - 分组+串连函数 `group_concat`
        - `select gender, group_concat(name) from students group by gender;`

    - 分组+聚合函数 `count/max/min/sum/avg`
        - `select gender, avg(age) from students group by gender;`
        - `select gender, count(*) from students group by gender;`

    - 分组+包含函数 `having`
        
        > `having` 是对分组以后的数据进行筛选<br>
          `where` 是对分组前的数据进行筛选

        - `select gender, count(*) from stucents group by gender having count(*)>2;`
    
    - 分组+统计 `with rollup`
        - `select gender, count(*) from students group by gender with rollup;`
        - `select gender, count(*) from students group by gender with rollup;`

- 排序 `order by` `asc` `desc`
    - `select * from tb_name where condition order by c1 asc|desc, c2 asc|desc ...;`

- 分页
    - `select * from tb_name limit start,count;`

    - 每页m条，求第n页的数据
        - `select * from db_name where is_delete=0 limit (n-1)*m, m;`

- 连接
    - `select * from db_name_a as a inner/left/right join db_name_b as b on a.c1=b.c2;`

- 自关联
    - 数据表和自身进行内连接查询

    - `select a.* from db_name as a inner join db_name ab b on a.c1=b.c2 where b.c3='xxx';`

- 子查询
    - 查询语句的嵌套
    - 标量子查询
        - `select * from students where age > (select avg(age) from students);`

    - 列级子查询
        - `select name from classes where id in (selcet cls_id from students);`

    - 行级子查询
        - `select * from students where (height,age) = (select max(height),max(age) from students);`

- 完整的查询语句
```SQL
select distinct *
from db_name
where ...
group by ... having ...
order by ...
limit start, count
```

- 执行顺序
    - from db_name
    - where
    - group by
    - select distinct *
    - having
    - order by
    - limit start, count

### Python-MySQL交互


### Redis 命令行

> k: key(键)<br>
v: value(值)<br>
f: field(属性)<br>
m: member(元素) 类似于value<br><br>
index: 列表元素的索引<br>
start: 列表元素的开始索引<br>
stop: 列表元素的结束索引<br>
count: 列表元素出现的次数<br>
score: 集合元素的权重<br>

#### key
- 查找
    - `keys pattern`

- 是否存在
    - `exists k`

- 查看类型
    - `type k`

- 删除键
    - `del k1 k2 ...`

- 过期时间
    - `expire k v`

- 查看过期时间
    - `ttl k`

#### string
- 设置键值
    - `set k v`

- 设置多个键值
    - `mset k1 v1 k2 v2 ...`

- 获取键值
    - `get k`

- 获取多个键值
    - `mget k1 k2 ...`

- 设置键值及过期时间（秒）
    - `setex k t v`

- 追加值
    - `append k v`

#### hash
- 设置属性值
    - `hset k f v`

- 设置多个属性值
    - `hmset k f1 v1 f2 v2 ...`

- 获取属性
    - `hget k`

- 获取属性值
    - `hget k f`

- 获取多个属性值
    - `hmget k f1 f2 ...`

- 获取所有属性值
    - `hvals k`

- 删除多个属性
    - `hdel k f1 f2 ...`

#### list
> 头部=左侧=前<br> 尾部=右侧=后<br>

> 索引/获取都是从左侧开始

- 插入
    - 从左往右插入（逆序插入）：`lpush k v1 v2 ...`
    - 从右往左插入（正序插入）：`rpush k v1 v2 ...`

- 给定元素前/后插入
    - `linsert k before/after v_now v_new`

- 修改
    - `lset k index v`

- 范围获取（按照从左到右获取）
    - `lrange k start stop`

        > **左右端点都包含，与python列表语法不同**

        > start=0, 表示从第一个元素（左侧）开始

        > start/stop 可以为负数（指的是索引值，获取方向依然是从左到右）

        > 获取所有元素 `lrange k 0 -1` 

- 删除指定元素`value`
    - `lrem k count v`

        > count > 0 从左往右删除<br>
          count = 0 全部删除<br>
          count < 0 从右往左删除<br>

#### set
- 增加元素
    - `sadd k m1 m2 ...`

- 获取所有元素
    - `smembers k`

- 删除多个元素
    - `srem m1 m2 ...`

#### zset
- 增加元素
    - `zadd k score1 m1 score1 m2`

- 范围获取（按照权重从小到大获取）
    - `zrange key start stop`

        > 语法同`list`

- 权重范围获取
    - `zrangebyscore k score_min score_max`

- 获取权重
    - `zscore k m`

- 删除指定元素
    - `zrem k m1 m2 ...`

- 删除指定权重元素
    - `zremrangebyscore k score_min score_max`


### Python-Redis交互
- 与单个Redis交互
> pip install redis

```python
from redis import StrictRedis

# 构建StrictRedis对象
sr = StrictRedis(host='localhost', port=6379, password='xxx', db=0)

# 增删改查与命令行一致
sr.set('name', 'xiaoming')
```

- 与Redis集群交互
> pip install redis-py-cluster

```python 
from rediscluster import StrictRedisCluster

# 构建所有的节点，Redis会使用CRC16算法，将键和值写到某个节点上
startup_nodes = [
{'host': '192.168.26.128', 'port': '7000'},
{'host': '192.168.26.130', 'port': '7003'},
{'host': '192.168.26.128', 'port': '7001'},
]

# 构建StrictRedisCluster对象
src=StrictRedisCluster(startup_nodes=startup_nodes, decode_responses=True)

# 增删改查与命令行一致
result=src.set('name','xiaoming') 
```

### Django-Redis交互
- 安装模块

```bash
pip install django-redis-sessions
```

- 配置

```python
# 修改settings文件，增加以下项
SESSION_ENGINE = 'redis_sessions.session'
SESSION_REDIS_HOST = 'localhost'
SESSION_REDIS_PORT = 6379
SESSION_REDIS_DB = 2
SESSION_REDIS_PASSWORD = ''
SESSION_REDIS_PREFIX = 'session'
```

- 增加和获取

```python
  def session_set(request):
      request.session['name']='itheima'
      return HttpResponse('ok')


  def session_get(request):
      name=request.session['name']
      return HttpResponse(name)
```

```python
url(r'^session_set/$',views.session_set),
url(r'^session_get/$', views.session_get),
```

### Flask-Redis交互
> 核心: 利用pickle将数据序列化，以文字流的方式缓存至redis，要用的时候再取出来进行反序列化。

```python
import redis
from datetime import datetime
from flask import session,request
from ..models import db

import pickle


class Redis:
    @staticmethod
    def connect():
        r = redis.StrictRedis(host='localhost', port=6379, db=0)
        return r

    #将内存数据二进制通过序列号转为文本流，再存入redis
    @staticmethod
    def set_data(r,key,data,ex=None):
        r.set(key,pickle.dumps(data),ex)

    # 将文本流从redis中读取并反序列化，返回返回
    @staticmethod
    def get_data(r,key):
        data = r.get(key)
        if data is None:
            return None

        return pickle.loads(data)

@home.route('/detail/<int:id>')
def detail(id):
    today = datetime.now().date()
    list_goods = None

    # 缓存标记，在页面显示出来
    cached_redis_remark = ""


    r = Redis.connect()

    #商品详情
    key = "data-cached:detail-id-%d" % (id)
    #从redis读取缓存（不存在的商品不会缓存，因为还是获取得到None)
    goods = Redis.get_data(r, key)
    if current_app.config['ENABLE_CACHED_PAGE_TO_REDIS'] and goods:
        # flash("读取缓存数据")
        cached_redis_remark += "goods|"
    else:
        # flash("查询数据库")
        goods=Goods.query.get(id)
        #将时间转为字符串才能序列化
        simple_goods = goods
        if goods:
            simple_goods.timestamp = str(goods.timestamp)
        #缓存入redis
        if current_app.config['ENABLE_CACHED_PAGE_TO_REDIS']:
            # 加入redis缓存,5分钟过期
            Redis.set_data(r, key, simple_goods, current_app.config['EXPIRE_CACHED_PAGE_TO_REDIS'])



    #推荐商品列表
    #推荐数量
    recommend_count = 4 if goods else 12

    key = "data-cached:detail-recommend-%d" % recommend_count
    list_recommend_goods = Redis.get_data(r, key)
    if current_app.config['ENABLE_CACHED_PAGE_TO_REDIS'] and list_recommend_goods:
        # flash("读取缓存数据")
        cached_redis_remark += "recommend|"
    else:
        list_recommend_goods = Goods.query.filter(Goods.coupon_expire >= today).filter_by(effective=True).limit(recommend_count).all()
        # 缓存入redis
        if current_app.config['ENABLE_CACHED_PAGE_TO_REDIS']:
            # 加入redis缓存,5分钟过期
            Redis.set_data(r, key, list_recommend_goods, current_app.config['EXPIRE_CACHED_PAGE_TO_REDIS'])


    return render_template("home/detail.html", goods=goods,list_goods=list_recommend_goods,cached_redis_remark=cached_redis_remark)
```

### Scrapy-Redis交互
- 安装模块

```bash
pip install scrapy-redis
```

- 配置

```python
# 修改settings.py文件，增加以下项

# 指定redis数据库的连接参数
REDIS_URL = 'redis://:password@host:port/db'

# 指定管道
ITEM_PIPELINES = {
    # 'JD.pipelines.ExamplePipeline': 300,
    'scrapy_redis.pipelines.RedisPipeline': 400,
}

# 指定重复过滤器模块
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"

# 启用调度器模块
SCHEDULER = "scrapy_redis.scheduler.Scheduler"

# 保持任务队列
SCHEDULER_PERSIST = True
```

- 修改项目.py文件
    - 导入类
    - 修改继承
    - 注销允许的域和起始的url
    - 设置redis_key
    - 设置动态允许的域(`__init__`)

