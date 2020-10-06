## 数据库范式

1. 第一范式：保证每一列都是原子的，不可再分的。
2. 第二范式：在第一范式的基础上，消除了非主属性对码的部分依赖。
3. 第三范式：在第二范式的基础上，消除了非主属性对码的传递依赖。
4. BC范式：在第三范式的基础上，消除对主码子集的依赖。

## MySQL 与 Redis 的对比

| Feature      | MySQL                                                    | Redis                        |
| ------------ | -------------------------------------------------------- | ---------------------------- |
| **类型上**   | 关系型数据库                                             | 非关系型数据库               |
| **作用上**   | 数据存储于磁盘，读写较慢。                               | 数据存储于内存，读写较快。   |
| **缓存**     | 有查询缓存，速度较慢。                                   | 内存缓存，速度较快。         |
| **适用场景** | 长期存储数据，具备关系型数据库的特性，如支持关联查询等。 | 存放热点数据，快速读取数据。 |
| **线程模型** | 每个线程一个连接                                         | 单线程、IO 多路复用。        |

## 数据库引擎的对比

下表有删改，详细参考官网。

[MySQL 8.0 - Chapter 16 Alternative Storage Engines](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)

| Feature                                          | MyISAM | Memory  | InnoDB | Archive | NDB   |
| ------------------------------------------------ | ------ | ------- | ------ | ------- | ----- |
| **B-tree 索引**                                  | Yes    | Yes     | Yes    | No      | No    |
| **聚簇索引**                                     | No     | No      | Yes    | No      | No    |
| **哈希索引**                                     | No     | Yes     | No     | No      | Yes   |
| **全文检索索引**                                 | Yes    | No      | Yes    | No      | No    |
| **事务**                                         | No     | No      | Yes    | No      | Yes   |
| **MVCC**                                         | No     | No      | Yes    | No      | No    |
| **锁的粒度**                                     | Table  | Table   | Row    | Row     | Row   |
| **支持外键**                                     | No     | No      | Yes    | No      | Yes   |
| **数据缓存**                                     | No     | N/A     | Yes    | No      | Yes   |
| **存储限制**                                     | 256TB  | RAM     | 64TB   | None    | 384EB |
| **分布式集群**                                   | No     | No      | No     | No      | Yes   |
| **数据压缩**                                     | Yes    | No      | Yes    | Yes     | No    |
| **分片复制支持** (Server 实现，而非存储引擎实现) | Yes    | Limited | Yes    | Yes     | Yes   |
| **T-tree 索引**                                  | No     | No      | No     | No      | Yes   |
| **索引缓存**                                     | Yes    | N/A     | Yes    | No      | Yes   |

## 索引分类

- 按底层数据结构：Hash Indexes、B-Tree Indexes。
- 按数据组织结构：聚簇（clustered）索引、非聚簇索引。
- 按索引包含的列：单列索引、联合索引。
- 按是否是主键分：主键索引、辅助索引。
- 按是否唯一分：唯一索引、非唯一索引。
- 其他：全文（Fulltext）索引。

## 主键与唯一索引的区别

## 索引的优缺点

> Note：B+ 树是文件系统数据结构，而像红黑树、AVL 树等都是内存数据结构。

**优点：**

- 大大减小了服务器需要扫描的数据量
- 帮助服务器避免排序和临时表
- 将随机 I/O 变成顺序 I/O

**缺点：**

- 虽然提高了查询速度，但是会降低更新表的速度。因为更新表时不仅要保存数据，还要保存索引文件。
- 索引文件会占用磁盘空间，但是只在索引文件非常巨大时影响才比较严重。
- 如果列的**区分度**不高，建立索引意义不大。
- 对于数据量比较小的表，大部分情况下全表扫描更高效。

## B+ 树与其他数据结构的对比

我们常用的数据结构有数组、链表、散列表、树等
1. 数组支持快速查找，但是需要占用大量的连续存储空间、为了保证顺序存储，增删元素可能需要移动大量数据；
2. 链表不用占用连续存储空间，但不支持快速查找，在非首尾结点增删元素也比较麻烦；
3. 散列表可以进行快速查找，但是只支持等值匹配，不支持区间查找和顺序查找；
4. 在双向链表之上加多层索引可以改造出跳表，跳表也可以实现快速查找，区间查询，顺序查询。如果忽略看过的 B+ 树与跳表的示意图，二者已十分接近。
5. 关于 B 树 与 B+ 树
     1. B 树遍历元素时效率低下，B+ 树只需要去遍历叶子节点就可以实现整棵树的遍历，而在 B树 中遍历数据则需要进行中序遍历。数据库中基于范围的查询是十分频繁，B 树不支持这样的操作或者说效率太低。
     2. B+ 树中间节点只存索引，不存数据，所以体积更小，在磁盘大小一定（如4KB）时，相比 B 树，磁盘可以存储更多 B+ 树节点，一次读入内存的数据项也更多，减少了 IO 次数。
     3. B+ 树的查找更稳定，数据都存储在叶子节点上，查找数据总是要走到叶子节点，而 B 树查找到数据后就返回。

## 为什么选择用 B+ 树

常用的 SQL 主要包括三类：
1. 根据某个值进行精确查找（**等值查询**）
2. 按照**区间查询**
3. 进行**顺序查找或逆序查找**

结合以上三点考虑，

- 哈希虽然能够提供 O(1) 的查询性能，但是只支持**等值查询**，不适用**范围查询**和**排序查询**，最终导致全表扫描；
- B+ 树中间节点只存索引，不存数据，所以体积更小，在磁盘页大小一定（如4KB）时，相比 B 树，单页磁盘可以存储更多 B+ 树节点，一次读入内存的数据项也更多，减少了 IO 次数。
- B+ 树的叶节点通过指针串联成双向链表，尽管等值匹配时效率不如**哈希**，但是 B+ 树是多路平衡查找树，树的高度可以很低，最坏查找效率也为**对数级别**，最主要的是 B+ 树非常适合**区间查询**与**顺序查询**。
- B 树的非叶子节点中存储数据，这导致在**查询连续数据时**可能会带来更多的随机 I/O，而 B+ 树的因为叶节点串联成双向链表，适合顺序 I/O。

## 主键自增与使用 UUID

> 当达到页面最大装填因子时，新数据将会插入到新的页面中。InnoDB 默认的最大装填因子为 15/16，其余 1/16 用于修改操作。

**采用主键自增时：**因为值是顺序的，所以新插入记录总是会在已有记录后面，所以：

- 因为主键自增有序，数据将会以顺序方式追加数据页，提高了页面的填充率，避免页面浪费。
- 因为是顺序追加，所以在新插入数据的地址计算方便，不需要因为寻址消耗太多的性能。
- 减少了页分裂与页面碎片的产生。

**采用 UUID 作为主键时：**新行主键不一定比已有主键大，所以需要为新行寻找存储地址，数据的无顺序性将会导致数据分布散乱。

- 待写入的目标也可能已经刷新到磁盘并且已从缓存中移除，或者还没有被加载到内存中，存储引擎需要在插入前先**找到磁盘**，然后将数据也**读取到内存**。在这个过程中将会导致大量的随机 IO。
- 因为数据写入乱序，将会导致频繁的**页面分裂**，进而导致大量的数据移动，以便为新行分配空间。
- 由于**无顺序性**以及频繁的**页面分裂**，将会使数据页变得**稀疏**，产生**页面碎片**。在有时使用 **OPTIMIZE TABLE** 来重建表并优化**页面填充**时也会浪费时间。

## 索引失效场景

1. 索引列参与了计算

2. 索引列使用了 MySQL 中的函数
3. 使用了前置模糊查询，like '%hello' 不走索引，like hello%'走索引
4. 索引列使用了正则表达式
5. 索引列进行了非等值查询，也就是范围查询(不满足最左匹配)。
6. 字符串与数字进行了比较
7. OR 条件连接时某些列没有加索引
8. MySQL 内部优化器如果估计全表扫描比索引更快，将放弃索引。

## 最左前缀匹配

常见问题：假如有联合索引 (a, b, c)，那么 (b, c)、(a, c)、(c)... 走索引吗？（在此假设用在 where 子句中，有 (a, b, c, d) 四列）。

- 一言以蔽之，只要查询包含了 `a` 就走索引，即使 `a` 与 `b、c` 逆序也没关系，如(c, a)，(c, b, a) 也走索引，否则不走。
- =、in 可以乱序。
- 测试 SQL 见参考资料部分。

## 事务特性

- 原子性（atomicity): 一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一部分操作，这就是事务的原子性。

- 一致性（consistency): 数据库总是从一个一致性的状态转换到另外一个一致性的状态。
- 隔离性（isolation): 一个事务所做的修改在最终提交以前，对其他事务是不可见的。
- 持久性（durability): 一旦事务提交，则其所做的修改就会永久保存到数据库中。此时即使系统崩溃，修改的数据也不会丢失。

## 事务隔离级别及其问题

[事务隔离级别及其问题](https://raymond-zhao.top/2020/08/01/2020-08-01-MySQL-TransactionAndLock/)

- `READ UNCOMMITTED`（未提交读）：事务中的修改，即使没有提交，对其他事务也都是可见的。事务可以读取未提交的数据，这也被称为脏读（`Dirty Read`）。
- `READ COMMITTED`（已提交读）：一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。这个级别有时候也叫做**不可重复读**。
- `REPEATABLE READ`（可重复读）：在**同一个事务中多次读取同样记录**的结果是一致的。
- `SERIALIZABLE`（可串行化）：会在读取的每一行数据上都加锁，所以可能导致大量的**超时和锁争用**的问题。

| 隔离级别           | 脏读 | 不可重复读 | 幻读 | 加锁读 |
| :----------------- | :--- | :--------- | :--- | :----- |
| `READ UNCOMMITTED` | YES  | YES        | YES  | NO     |
| `READ COMMITTED`   | NO   | YES        | YES  | NO     |
| `REPEATABLE READ`  | NO   | NO         | YES  | NO     |
| `SERIALIZABLE`     | NO   | NO         | NO   | YES    |

- 脏读：A 事务读取到了 B 事务未提交前的旧值，然后 B 又回滚了。
- 不可重复读：同一个事务两次读取所得到的的**记录内容**不一致，通常是由于另一个事务进行了 update 操作。
- 幻读：同一个事务两次读取所得到的的**记录条数**不一致，通常是由于另一个事务进行了 insert 操作。

## MVCC

[MVCC](https://raymond-zhao.top/2020/08/01/2020-08-01-MySQL-TransactionAndLock/)

- 通过保存数据在某个时间点的**快照**来实现的 (**Snapshot 快照图**) 来实现。
- 不同存储引擎的 MVCC 实现是不同的，典型的有**乐观并发控制**与**悲观并发控制**两种。
- InnoDB 的 MVCC，通过在每行记录后面保存两个隐藏的列来实现。一个保存了行的**创建时间**，一个保存行的**过期时间**。
- **INSERT：**为新插人的每一行保存**当前系统版本号**作为行版本。
- **DELETE：**为删除的每一行保存**当前系统版本号**作为行删除标识。
- **SELECT：**只査找**版本早于当前事务版本的数据行**，可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
- **UPDATE：**插入一行新记录，保存**当前系统版本号**作为行版本号，同时保存**当前系统版本号**到原来的行作为行**删除标识**。
- 只在 RR、RC 下工作。因为 RU 总是读取最新的数据行，而不是符合当前事务版本的数据行。而 SERIALIZABLE 则会对所有读取的行都加锁。

## 如何实现非阻塞读

- [MVCC](##MVCC)

## 快照读与当前读

- 执行 **SELECT** 操作时 InnoDB 默认会执行**快照读**，记录下此次 SELECT 的结果，之后再 SELECT 的则返回此次快照结果，即使其他事务提交了不会影响当前 SELECT 的数据，也就是**可重复读**。
- 对于**写操作** (update、insert、delete) 均采用**当前读**。执行这三个操作时会读取最新的记录，即使是别的事务新提交的数据也可以查询到。

## InnoDB 下 RR 如何防止幻读？

- MVCC + Next-Key Locks: `next-key locks` 由 `record locks`(索引加锁) 和 `gap locks`(间隙锁，每次锁住的不光是需要使用的数据，还会锁住这些数据附近的数据)。
- [MySQL 官方 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

## MySQL 锁的分类

- 按粒度划分：表级锁，行级锁，页级锁。
- 按读写划分：共享锁，排它锁。
- 按加锁方式划分：自动锁，显式锁。
- 按操作划分：DML 锁，DDL 锁。
- 按使用方式划分：乐观锁 (通过版本号)，悲观锁。

## 行级锁与间隙锁

- [MySQL 官方 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
- 用于锁住 `Index Record` 之间的间隙。

- 如果是`通过唯一索引来搜索一行记录`的时候，不需要使用 `Gap Locks`，此时 `Gap Locks` 降级为 `Record Locks`。
- `Gap S-Lock` 与 `Gap X-Lock` 是兼容的。
- `Gap Locks` 只能_**`阻止其他事务在该Gap中插入记录`**_，但**`无法阻止`**其他事务获取`同一个Gap` 上的 `Gap Lock`。
- 可以通过将事务隔离级别设置为 READ COMMITTED 禁用 `Gap Locks`。

## SQL 的执行过程

[SQL 的执行过程](https://www.cnblogs.com/lfri/p/12598339.html)

![img](https://img2020.cnblogs.com/blog/1365470/202003/1365470-20200330140529330-934488719.png)

1. 客户端通过**连接器**建立与 Server 的连接；
2. **查询缓存**，如果命中缓存则直接返回；
3. 经过**分析器**进行词法分析，语法分析，之后仍然会去再查一次**查询缓存**；
4. 如果查询缓存仍然不存在所需数据的话，通过**优化器**进行各种优化，包括重写査询、决定表的读取次序，选择合适索引等，而后生成执行计划；
5. 通过**执行器**执行执行计划，操作存储引擎数据访问接口，返回结果。

## 慢查询优化

[美团技术团队 - 慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)

**建索引的几大原则：**

- 最左前缀匹配原则
- = 和 in 可以乱序
- 尽量选择**区分度高**的列作为索引，区分度的公式是 count(distinct col)/count(*)，比例越大扫描的记录数越少，唯一索引的区分度是为 1。
- 索引列不要参与计算
- 尽量的扩展索引，不要新建索引。

**慢查询优化的基本思路：**

1. 通过设置 SQL_NO_CACHE，先看看查询是否真的很慢；
2. 使用 WHERE 条件进行单表查询，锁定**最小返回记录**表，找到**区分度**最高的字段；
3. 使用 EXPLAIN 查看执行计划，是否与 2 预期结果一直；
4. ORDER BY、LIMIT 形式的 SQL 语句让排序的表优先查；
5. 分析业务实际使用场景
6. 参照建索引时的几大原则添加索引
7. 观察优化结果，不满足要求则从头开始，否则结束。

## MySQL 三大日志

[MySQL 三大日志](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247495901&idx=2&sn=1e02ebaebcc3e6a420c267beb260e97b&chksm=ebd5cff1dca246e7cb17386d00f79ba92d5a7d68cd329ad5e9a9b710c7e614b0b21e16f8f5fc&mpshare=1&scene=24&srcid=0820BOU74VHoJGesRWPD6ykC&sharer_sharetime=1597917853827&sharer_shareid=459d60cb1cece625a6a0b23694c741b3#rd)

**BIN LOG：**用于记录数据库执行的**写入性**操作信息，以二进制的形式保存在磁盘中。通过追加的方式进行写入。使用场景主要有两个：

- 使用场景：
  - 主从复制：在 master 端开启 binlog，然后将 binlog 发送到各个 slave 端，slave 端执行 binlog 内容，从而达到主从数据一致。
  - 数据恢复：通过使用 mysqlbinlog 工具来恢复数据。

- 日志格式：
  - **STATMENT：**每一条会修改数据的 SQL 语句会记录到 binlog 中。
  - **ROW：**不记录每条 SQL 语句的上下文信息，需记录哪条数据被修改了。
  - **MIXED：**一般的复制使用 STATEMENT 模式保存 binlog，对于 STATEMENT 模式无法复制的操作使用 ROW 模式保存 binlog。

**REDO LOG：**只记录事务对数据页做了哪些修改，保证事务四大特性之一的**一致性**。

- 主要组成部分：MySQL 每执行一条 DML 语句，先将记录写入**内存**中的**日志缓冲**（redo log buffer），后续某个时间点再一次性将多个操作记录写到**磁盘**上的**日志文件**（redo log file）。
- 记录形式：大小固定，循环写入，当写到结尾时，会回到开头循环写日志。

**UNDO LOG：**记录数据的逻辑变化，保证事务四大特性之一的**原子性**。一条 INSERT 语句，对应一条 DELETE 的 undo log，对于每个 UPDATE 语句，对应一条相反的 UPDATE 的 undo log。也是 MVCC 实现的关键。

## MySQL 为什么选用 RR 做默认隔离级别？

[MySQL 为什么选用 RR 做默认隔离级别？](https://dominicpoi.com/2019/06/16/MySQL-1/)

> 这与主从复制中的 binlog 有关。

![image](https://static001.geekbang.org/resource/image/a6/a3/a66c154c1bc51e071dd2cc8c1d6ca6a3.png)

主库和从库之间有一个长连接，主库 A 内部有一个线程专门服务从库 B 的长连接，一个事务日志同步的完整过程是这样的：

1. 从库 B 上通过 change master 命令设置主库 A 的 IP、端口、用户名、密码，以及开始请求 binlog 的位置（包括文件名和日志偏移量）。
2. 从库 B 上执行 start slave 命令，启动两个线程（`io_thread` 和 `sql_thread`)，其中 `io_thread` 负责与主库建立连接。
3. 主库 A 校验完用户名密码后，按照从库 B 传过来的位置从本地读取 binlog 发给 B。
4. 从库 B 拿到 binlog 之后写到本地文件，称为中转日志（relay log）。
5. `sql_thread` 读取中转日志，解析出日志里的命令并执行。

> 而在这整个过程中，binlog 中的 STATEMENT 日志格式会导致主从不一致。
>
> 因为 STATEMENT 只记录了 SQL，重新恢复时，可能由于事务**提交顺序**问题，以及**索引选择**不同问题，在一些语句执行上会出现歧义。
>
> 而 ROW 格式则不会出现主从不一致问题，但是它在 MySQL 5.1 才引入。所以为了保证主从数据一致性，才设置为 RR 为默认级别。

## MySQL 主从复制

[MySQL 主从复制](##MySQL 为什么选用 RR 做默认隔离级别？)

> 就是上个问题里的图和过程。

## 排查死锁

[排查死锁](https://mp.weixin.qq.com/s/DChYk7WhbgMG4tRKf85G2g)

## 参考资料

- [面试官，你要是敢在问我B+树，别怪我不客气！！](https://mp.weixin.qq.com/s/DtbWSGnjt001JMR78tjt5A)
- [为什么 MySQL 使用 B+ 树](https://draveness.me/whys-the-design-mysql-b-plus-tree/)

```mysql
create table idx_test
(
    id int not null primary key,
    a  int not null,
    b  int not null,
    c  int not null,
    d  int null
);

create index idx_abc
    on idx_test (a, b, c);

explain select * from idx_test where a = 2; # Y, Extra: null
explain select * from idx_test where b = 2; # N, Extra: Using where
explain select * from idx_test where a = 2 and b = 2; # Y, Extra: null
explain select * from idx_test where a = 2 and b = 2 and c = 2; # Y, Extra: null
explain select * from idx_test where b = 2 and c = 2; # N, Extra: Using where
explain select * from idx_test where a = 2 and c = 2; # Y, Extra: Using index condition
explain select d from idx_test where a = 2 and c = 2; # Y, Extra: Using index condition
explain select a, b, c from idx_test where a = 2 and b = 2 and c = 2; # Y, Extra: Using index
explain select a, b, c from idx_test where c = 2 and b = 2 and a = 2; # Y, Extra: Using index
explain select a from idx_test where a = 2 and b = 2 and c = 2; # Y, Extra: Using index
explain select d from idx_test where a = 2 and b = 2 and c = 2; # Y, Extra: null
explain select d from idx_test where c = 2; # Y, Extra: Using where
explain select d from idx_test where c = 2 and a = 2; # Y, Extra: Using where
```

