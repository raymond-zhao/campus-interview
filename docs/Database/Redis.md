> 推荐阅读：[妈妈再也不担心我面试被 Redis 问得脸都绿了](https://mp.weixin.qq.com/s/gmV0Z4nW9NYh4cVpDNiI9w)

## Redis 简介

**Redis** (**Re**mote **Di**ctionary **S**erver) 是一个使用 **C 语言** 编写的，开源的 *(BSD许可)* 高性能 **非关系型** *(NoSQL)* 的 **键值对数据库**。

- 支持丰富的数据结构；
- 支持事务 、持久化、Lua 脚本、LRU 驱动事件、多种集群方案。

## Redis 的优缺点

> 优点

- **读写性能优异**， Redis 读 QPS `110000` ，写 QPS `81000` 。
- **支持数据持久化**，支持 AOF 和 RDB 两种持久化方式。
- **支持事务**，Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作合并后的原子性执行。
- **数据结构丰富**，支持 string、hash、set、zset、list 等数据结构。
- **支持主从复制**，主机会自动将数据同步到从机，可以进行读写分离。

> 缺点

- 数据库 **容量受到物理内存的限制**，不能用作海量数据的高性能读写，因此 Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。
- Redis **不具备自动容错和恢复功能**，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了 **系统的可用性**。
- **Redis 较难支持在线扩容**，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

## Redis 为什么这么快？

1. **纯内存操作**：读取不需要进行磁盘 I/O，所以比传统数据库要快上不少；
2. **单线程，无锁竞争**：没有线程的上下文切换，不会因为多线程的一些操作而降低性能；
3. **多路 I/O 复用模型，非阻塞 I/O**：采用多路 I/O 复用技术可以让单个线程高效的处理多个网络连接请求（尽量减少网络 IO 的时间消耗）；
4. **高效的数据结构，加上底层做了大量优化**：Redis 对于底层的数据结构和内存占用做了大量的优化，例如不同长度的字符串使用不同的结构体表示，HyperLogLog 的密集型存储结构等等。

## Redis 数据结构

> 🙋🙋🙋：[具体点的介绍点这里-Redis 内部数据结构](https://raymond-zhao.top/2020/06/19/2020-06-19-Redis-DataStructure/)

![基本数据结构](https://gitee.com/raymond-zhao/oss/raw/master/uPic/image-20201011075848447.png)

| Redis数据结构                                                | 底层实现                      | 应用场景                                                     |
| ------------------------------------------------------------ | ----------------------------- | ------------------------------------------------------------ |
| `String`                                                     | 整数（只有 long 类型） 或 SDS | 二进制安全，缓存静态文件、用作计数器（incr），统计次数（如网站访问）。 |
| `Hash`                                                       | 字典 或 压缩列表              | 存储对象，用户信息、商品信息等。                             |
| `List`                                                       | 双端链表 或 压缩列表 或 快表  | 消息队列、粉丝列表、关注列表等。                             |
| `Set`                                                        | 字典 或 整数集合              | 用于去重相关操作                                             |
| `ZSet`                                                       | 字典 或 压缩列表              | 在`Set`功能的基础之外，构建优先队列。                        |
| [HyperLoglog](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484012&idx=1&sn=3624989d388d17331e1f7ad78fc7a257&chksm=f9d5a661cea22f7781ed997d05afee8f28da52a0013691961915a2831780e481ae94b6e9a1ae&scene=21#wechat_redirect) |                               | 基数统计                                                     |
| [BloomFilter](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484022&idx=1&sn=a98c479b4cac96c6af45f219a7c0bde4&chksm=f9d5a67bcea22f6d03b30ce8660f3f3ded294390fd394e138a40a439d0f96d56c8dda2082203&scene=21#wechat_redirect) |                               | 大数据判断是否存在、解决缓存穿透、爬虫/邮箱等系统的过滤。    |
| [GeoHash](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484027&idx=1&sn=6237212c01a009be9a5ca88e0e50a46d&chksm=f9d5a676cea22f603e06d1e61d6293985c8ddb063621b1cdf2696da2eaeb1681e307ef2a3f20&scene=21#wechat_redirect) |                               | 地里定位，附近的人、附近的 xxx                               |
| bitMap                                                       |                               |                                                              |
| [Pub/Sub](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484039&idx=1&sn=99866e4cc6c842dc6d50d68a362d53a8&chksm=f9d5a68acea22f9cf6d972c23809520137957bd26ce7e6c6cb4f4ed11ddcbbc9938c1a56eb46&scene=21#wechat_redirect) |                               |                                                              |
| [Stream](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484039&idx=1&sn=99866e4cc6c842dc6d50d68a362d53a8&chksm=f9d5a68acea22f9cf6d972c23809520137957bd26ce7e6c6cb4f4ed11ddcbbc9938c1a56eb46&scene=21#wechat_redirect) |                               |                                                              |

## 字典的实现

> [具体点的介绍点这里-Redis 内部数据结构](https://raymond-zhao.top/2020/06/19/2020-06-19-Redis-DataStructure/)

## List底层实现-快表

> [Redis 列表 List 底层原理](https://zhuanlan.zhihu.com/p/102422311)
>
> Redis 3.2 版本起对 `list` 数据结构进行改造，使用 `quicklist`代替了 `ziplist` 和 `linkedlist`。

### 压缩列表

> 在 Redis 3.2 之后， list 中底层实现中的 ziplist 被 quicklist 所取代，但仍然是 zset 的底层实现之一，zset 的另一种实现是 跳跃表。

- ziplist 是特殊的双向链表，特殊之处在于：
- 每个 entry 的长度不一定相同；
- 没有维护双向指针 prev、next，而是存储上一个 entry 的长度与当前 entry 的长度，通过长度计算下一个元素的位置；
- 牺牲读取的性能，获得高效的存储空间，因为存储指针比存储 entry 长度更费内存。这是一种以时间换空间的思想；
- 压缩列表转化为双向链表的条件：
  - 试图往列表新添加一个字符串，且这个字符串的长度超过 `server.list_max_ziplist_value=64(可修改)`。
  - ziplist 包含的节点超过 `server.list_max_ziplist_entries=512(可修改)`

### 快表 - quicklist

> A generic doubly linked quicklist implementation.
> A doubly linked list of ziplists.（基于压缩链表的双向链表）
>
> 
>
> Quicklist 结合 Linkedlist 与 Ziplist 二者而实现。

**二者的优缺点**

- **双向链表 linkedlist：** 便于在表的两端进行 push 和 pop 操作，但是内存开销大，除了要保存数据外，还要保存两个指针；双向链表的各个节点是单独的内存块，地址不连续，容易产生内存碎片。
- **压缩列表 ziplist：**存储内存连续，所以存储效率较高。但是不利于修改操作，插入和删除操作需要频繁地申请和释放内存。当 ziplist 长度很大时，一次 `realloc` 可能会导致大量的数据复制。

**快表的特点：**

- Quicklist 是由 Ziplist 组成的双向链表，每个节点（Quicklist Node）使用 Ziplist 来保存数据。
- 具有双向链表的基本特征，包括 head、tail 结点，prev、next 指针。

## 跳跃表

> 参考文章：[跳跃表](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484000&idx=1&sn=a7e02adebea31535c3870cc514719493&chksm=f9d5a66dcea22f7b7cdf210993cbe6c057c0456967ac106d76c89fdd76ed5cb271c879a83f3e&scene=21#wechat_redirect)

### 为什么使用跳跃表？

- 因为 zset 要支持随机的插入和删除，所以不宜使用数组实现。
- **性能考虑：**在高并发的情况下，树形结构可能频繁需要执行 rebalance 这种设计整棵树的操作，而跳表的变化只涉及局部；
- **实现考虑：**在复杂度与红黑树相同的情况下，跳跃表实现起来更加简单，看起来也更加直观。

### 跳表的实现

- 跳表为了避免插入删除数据时的数据调整问题，它不要求上下相邻两层链表之间的节点个数有严格的对应关系，而是**为每个节点随机出一个层数（level）**。
  - 如果新插入的结点的高度为 **level**，则将该节点插入 **1~level** 层中的每一层中；
- 在插入过程中，新插入的结点并不会影响到其他结点的层数。因此插入操作只需要修改节点前后的指针，而不需要对多个节点都进行调整。

### 跳表的随机层数是如何产生的？

```c
// Redis 跳跃表默认允许最大的层数是 ZSKIPLIST_MAXLEVEL = 32
// 当 Level[0] 有 2^{64} 个元素时，才能达到 32 层。
int zslRandomLevel(void) {
    int level = 1;
    while ((random() & 0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level < ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}
```

- level1：期望目标 50% 概率
- level2：期望目标 25% 概率
- level3：期望目标 12.5% 概率
- ...

### 插入结点

1. 声明需要存储的变量
2. 搜索当前结点插入位置
3. 生成插入结点
4. 重排前向指针
5. 重排后向指针并返回

### 删除结点

- 搜索要删除的结点所处的位置
- 删除结点
- 修改前向指针与后向指针
- 注意更新最高层数 maxLevel

### 更新节点

- 当调用 `ZADD` 方法时，如果对应的 value 不存在，那就是插入过程；否则，调整 score 的值，走更新流程。
- 假设这个新的 score 值并不会带来排序上的变化，那就不需要调整位置，直接修改元素的 score 值，但是如果排序位置改变了，那就需要调整位置。
- **Redis 的调整策略：**把当前元素先删除，然后再插入。

### 元素排名如何实现？

- 跳表本身是有序的，Redis 在 skiplist 的 forward 指针上进行了优化，给每一个 forward 指针增加了 span 属性，用来表示 **从前一个结点沿着当前层的 forward 指针跳到这个结点中间会经过的结点，即 跨度。**
- Redis 在插入、删除操作时都会更新 `span` 值。
- 所以，沿着**搜索路径**，把所有经过结点的跨度`span`进行累加，就可以计算出当前元素的 `rank` 值。

## 布隆过滤器

> [亿级数据过滤和布隆过滤器](https://mp.weixin.qq.com/s?__biz=MzUyMTg0NDA2Ng==&mid=2247484022&idx=1&sn=a98c479b4cac96c6af45f219a7c0bde4&chksm=f9d5a67bcea22f6d03b30ce8660f3f3ded294390fd394e138a40a439d0f96d56c8dda2082203&scene=21#wechat_redirect)

## 使用缓存时可能出现的问题

一般来说有如下几个问题，回答思路遵照 **是什么** → **为什么** → **怎么解决**：

- 缓存雪崩问题；
- 缓存穿透问题；
- 缓存击穿问题；
- [缓存和数据库双写一致性问题](https://mp.weixin.qq.com/s/3Fmv7h5p2QDtLxc9n1dp5A)。

### 缓存雪崩问题

![缓存雪崩](https://tva1.sinaimg.cn/large/007S8ZIlly1ghsnwdnqolj30tv0ft0zg.jpg)

另外对于 **"Redis 挂掉了，请求全部走数据库"** 这样的情况，有如下的思路：

- **事发前**：实现 Redis 的高可用(主从架构 + Sentinel 或者 Redis Cluster)，尽量避免 Redis 挂掉这种情况发生。
- **事发中**：万一 Redis 真的挂了，我们可以设置本地缓存(ehcache) + 限流(hystrix)，尽量避免我们的数据库被干掉(起码能保证我们的服务还是能正常工作的)
- **事发后**：Redis 持久化，重启后自动从磁盘上加载数据，快速恢复缓存数据。

### 缓存穿透问题

![缓存穿透](https://tva1.sinaimg.cn/large/007S8ZIlly1ghsnxrax6cj30tw0ft7aq.jpg)

### 缓存和数据库双写一致性问题

![缓存和数据库双写一致性问题](https://tva1.sinaimg.cn/large/007S8ZIlly1ghsnznkhp6j30tw0fqgt6.jpg)

> 本系统的一致性解决方案：
>
> - 为所有缓存数据设置过期时间，数据过期下一次查询触发主动更新。
> - 读写数据的时候，加上分布式的读写锁。(读多写少时几乎无影响)

无论是双写模式还是失效模式，都会导致缓存的不一致问题，即多个实例同时更新会出事，怎么办?

- 如果是**用户维度**数据(订单数据、用户数据)，这种并发几率非常小，不用考虑这个问题，缓存数据加 上过期时间，每隔一段时间触发读的主动更新即可。
- 如果是菜单，商品介绍等**基础数据**，也可以去使用 `canal` 订阅 `binlog` 的方式。
- **缓存数据 + 过期时间** 足够解决大部分业务对于缓存的要求。
- 通过**加锁**保证并发**读写、写写**的时候按顺序排好队，读读无所谓，所以适合使用读写锁。(业务不关心脏数据，允许临时脏数据可忽略。)

**总结:**

- 能放入缓存的数据本就不应该是实时性、一致性要求超高的。所以缓存数据的时候加上过期时间，保证每天拿到当前最新数据即可。

- 不应该过度设计，增加系统的复杂性。

- 遇到实时性、一致性要求高的数据，就应该实时直接查数据库，即使这样速度相对于缓存会比较慢。

> 推荐阅读：[面试前必须要知道的 Redis 面试题 - 缓存与数据库双写一致性问题](https://mp.weixin.qq.com/s/3Fmv7h5p2QDtLxc9n1dp5A)

### 缓存击穿

- 是什么：在高并发的系统中，大量的请求同时查询一个 `key`(热点 `key`) 时，此时这个 `key` 正好失效了，就会导致大量的请求都打到数据库上面去。
- 为什么：`key` 设置了过期时间，`key` 又为热点`key`
- 怎么解决：在第一个查询数据的请求上使用一个 **互斥锁(mutex)** 来锁住它。其他的线程走到这一步拿不到锁就等着，等第一个线程查询到了数据，然后做缓存。后面的线程进来发现已经有缓存了，就直接走缓存。

```java
public String get(key) {
    String value = redis.get(key);
    if (value == null) { // 代表缓存值过期
        // 设置 3min 的超时，防止 del 操作失败的时候，下次缓存过期一直不能load db
        if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
            value = db.get(key);
            redis.set(key, value, expire_secs);
            redis.del(key_mutex);
        } else {  
            // 这个时候代表同时候的其他线程已经load db并回设到缓存了
            // 这时候重试获取缓存值即可
            sleep(50);
            get(key);  //重试
        }
    } else return value;
 }
```

> 这种 **互斥锁(mutex) ** 在单机情况下是有效的，但是在分布式情况下就要使用**分布式锁**。
>
> 本地锁，只能锁住当前线程，所以需要分布式锁。
>
> Spring Boot 所有的组件在容器中默认都是单例的，使用 `synchronized (this)` 可以实现加锁。
>
> 在每一个微服务中的`synchronized(this)`加锁的对象只是当前实例，但是并未对其他微服务的实例产生影响，即使每个微服务加锁后只允许一个请求，假如有 8 个微服务，仍然会有 8 个线程存在。

## 如何保证在查询缓存时只查一次DB

```java
/**
* SpringBoot 所有的组件在容器中默认都是单例的，使用 synchronized (this) 可以实现加锁。
*
* 得到锁之后 应该再去缓存中确定一次，如果没有的话才需要继续查询。
*
* 假如有100W个并发请求，首先得到锁的请求开始查询，此时其他的请求将会排队等待锁
* 等到获得锁的时候再去执行查询，但是此时有可能前一个加锁的请求已经查询成功并且将结果添加到了缓存中
*/
```

将核心操作封装为原子操作，保证锁的时序性，具体地讲，就是将

1. 确认缓存是否存在
2. 查询数据库
3. 将结果放入缓存

这三个操作封装为原子操作，必须当做一个事务来执行，放在同一把锁里面完成。

## Redis 持久化

> 将内存中的数据库状态保存到磁盘中。

### 持久化步骤

1. 客户端向数据库 **发送写命令**（数据在客户端内存中）
2. 数据库接收到客户端的 **写请求**（数据在服务器内存中）
3. 数据库 **调用系统 API** 将数据写入磁盘（数据在内核缓冲区中）
4. 操作系统将 **写缓冲区** 传输到 **磁盘控制器**（数据在磁盘缓存）
5. 操作系统的磁盘控制器将数据 **写入实际的物理媒介** （数据在磁盘中）

### RDB 持久化

- 优点：
  - 只有一个文件 `dump.rdb`，**方便持久化**。
  - **容灾性好**，一个文件可以保存到安全的磁盘。
  - **性能最大化**，`fork` 子进程来完成写操作，让主进程继续处理命令，所以使 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，保证了 Redis 的高性能。
  - 相对于数据集大时，比 AOF 的 **启动效率** 更高。
- 缺点：
  - **数据安全性低**。RDB 是间隔一段时间进行持久化，如果持久化之间 Redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候；

### AOF 持久化

- 优点：
  - **数据安全**，AOF 持久化可以配置 `appendfsync` 属性，有 `always`，每次更新命令都会记录到 AOF 文件。
  - 通过 Append 模式写文件，即使中途服务器宕机，可以通过 `redis-check-aof` 工具解决数据一致性问题。
  - AOF 的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）。
- 缺点：
  - AOF 文件比 RDB **文件大**，且 **恢复速度慢**。
  - **数据集大** 的时候，比 RDB **启动效率低**。

## Redis 数据恢复

**Redis** 数据恢复的优先级：

1. 如果只配置 AOF ，重启时加载 AOF 文件恢复数据；
2. 如果同时配置了 RDB 和 AOF ，启动只加载 AOF 文件恢复数据；
3. 如果只配置 RDB，启动将加载 dump 文件恢复数据。

拷贝 **AOF** 文件到 Redis 的数据目录，启动 redis-server AOF 的数据恢复过程：

- Redis 虚拟一个客户端，读取 AOF 文件恢复 Redis 命令和参数；
- 然后执行命令，进而恢复数据，这些过程主要在 `loadAppendOnlyFile()` 中实现。

拷贝 **RDB** 文件到 Redis 的数据目录，启动 redis-server 即可，因为 RDB 文件和重启前保存的是真实数据而不是命令状态和参数。

## 主从复制概念

- **主从复制**，是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。前者称为 **主节点(master)**，后者称为 **从节点(slave)**。

- 且数据的复制是 **单向** 的，只能由主节点到从节点。

- Redis 主从复制支持 **主从同步** 和 **从从同步** 两种，后者是 Redis 后续版本新增的功能，以减轻主节点的同步负担。

## 主从复制的作用

- **数据冗余：** 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复：** 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复 *(实际上是一种服务的冗余)*。
- **负载均衡：** 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 *（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点）*，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
- **高可用基石：** 除了上述作用以外，主从复制还是哨兵和集群能够实施的 **基础**，因此说主从复制是 Redis 高可用的基础。

## 主从复制原理

## 在主从复制时是如何保证数据一致的？

## Redis 内存回收策略

- `volatile-lru`：从已设置过期时间的数据集（`server.db[i].expires`）中挑选最久未使用的数据淘汰；
- `volatile-ttl`：从已设置过期时间的数据集（`server.db[i].expires`）中挑选将要过期的数据淘汰；
- `volatile-random`：从已设置过期时间的数据集（`server.db[i].expires`）中随机选取数据淘汰；
- `allkeys-lru`：当内存不足以容纳新写入数据时，在键空间中，移除最久未使用的 `key`；
- `allkeys-random`：从数据集（`server.db[i].dict`）中随机选取数据淘汰；
- `no-eviction`：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。

`volatile-xxx` 这三个淘汰策略使用的不是全量数据，有可能无法淘汰出足够的内存空间。在没有过期键或者没有设置超时属性的键的情况下，这三种策略和 `noeviction` 差不多。

4.0 版本后增加以下两种：

- `volatile-lfu`：从已设置过期时间的数据集(`server.db[i].expires`)中挑选最不经常使用的数据淘汰。
- `allkeys-lfu`：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 `key`。

一般的经验规则：

- 使用 `allkeys-lru` 策略：当预期请求符合一个幂次分布(二八法则等)，比如一部分的子集元素比其它元素被访问的更多时，可以选择这个策略。
- 使用 `allkeys-random` 策略：循环连续的访问所有的键时，或者预期请求分布平均（所有元素被访问的概率都差不多）。
- 使用 `volatile-ttl`：要采取这个策略，缓存对象的 `TTL` 值最好有差异。

`volatile-lru` 和 `volatile-random` 策略，当想要使用**单一**的 `Redis` 实例来同时实现**缓存淘汰和持久化**一些经常使用的键集合时很有用。

- 对未设置过期时间的键进行持久化保存，对设置了过期时间的键参与缓存淘汰。
- 不过一般运行两个实例是解决这个问题的更好方法。

为键设置过期时间也是需要消耗内存的，所以使用 `allkeys-lru` 这种策略更加节省空间，因为这种策略下可以不为键设置过期时间。

## Redis LRU

> [ Using Redis as an LRU cache](https://redis.io/topics/lru-cache)

**过期键的删除策略**有两种：

- 惰性删除：每次从键空间获取键时，都检查键是否过期，如果过期的话就删除该键，否则返回该键。
- 定期删除：每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。

**Redis LRU 配置参数：**

- `maxmemory`：配置 `Redis` 存储数据时指定限制的内存大小，比如 `100m`。当缓存消耗的内存超过这个数值时, 将触发数据淘汰。该数据配置为 $0$ 时，表示缓存的数据量没有限制, 即 $LRU$ 功能不生效。$64$ 位的系统默认值为 $0$，$32$ 位的系统默认内存限制为 $3GB$。
- `maxmemory_policy`：触发数据淘汰后的淘汰策略。
- `maxmemory_samples`：随机采样的精度，也就是随即取出 $key$ 的数目。该数值配置越大, 越接近于真实的 $LRU$ 算法，但是数值越大，相应消耗也变高，对性能有一定影响，样本值默认为 $5$ 。

## 近似 LRU 的具体实现

- [Redis作为LRU Cache的实现](https://developer.aliyun.com/article/63034)
- [Redis 内存回收策略和key失效机制](https://zhuanlan.zhihu.com/p/149528273)

全局 LRU 时钟

```c
#define REDIS_LRU_BITS 24
unsigned lruclock:REDIS_LRU_BITS; /* Clock for LRU eviction */
```

$Redis$ 并不会选择最久未被访问的键进行回收，而是尝试运行一个近似 $LRU$ 算法，**通过对少量键进行取样，然后回收其中的最久未被访问的键**。通过调整每次回收时的采样数量 `maxmemory-samples`（默认是 5 个），可以调整算法的精度。

每个 `Redis Object` 可以挤出 $24 bits$ 的空间，但 $24 bits$ 不够存储两个指针，但是够存储一个低位时间戳，`Redis Object` 以秒为单位存储了对象新建或者更新时的 `Unix Time`，也就是 `LRU Clock`，$24 bits$ 数据要溢出的话需要 $194$ 天，而缓存的数据更新非常频繁，已经足够。

> 可以通过设置值 `maxmemory-samples` 参数修改采样数量。

**Redis 3.0 优化**

- 新算法会维护一个候选池（大小为 16），池中的数据根据访问时间进行排序；
- 第一次随机选取的 `key` 都会放入池中；
- 随后每次随机选取的 key 只有在访问时间小于池中最小的时间才会放入池中，直到候选池被放满；
- 当放满后，如果有新的 key 需要放入，则将池中最后访问时间最大（最近被访问）的移除。

当需要淘汰时，需要从池中捞出最久没被访问的key淘汰掉就行了。

## Redis 分布式锁 - RedLock 

> [Distributed locks with Redis](https://redis.io/topics/distlock)

假设有一个 Redis Cluster，其中有 N 个 Redis  Master 实例。为了取到锁，客户端应该执行以下操作:

1. 获取当前 Unix 时间，以毫秒为单位。
2. 依次尝试从 N 个实例，使用**相同的 key** 和 **随机的 value** 获取锁。当向 Redis 设置锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。这样可以避免服务器端 Redis 已经挂掉的情况下，客户端还在持续等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个 Redis 实例。
3. 客户端使用当前时间减去开始获取锁时间，得到获取锁使用的时间。当且仅当从大多数的 Redis 节点都获取到锁，并且**使用时间**小于**锁失效时间**时，锁才算获取成功。
4. 如果取到了锁，key 的真正有效时间等于有效时间减去获取锁所使用的时间。
5. 如果因为某些原因，获取锁失败（没有在至少 $\frac{N}{2}+1$ 个 Redis 实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的 Redis 实例上进行**解锁**（即使某些 Redis 实例根本就没有加锁成功）。

## 参考

- [博客园-再见紫罗兰](https://www.cnblogs.com/linxiyue/p/10945216.html)

- [简书-后厂村老司机-Redis的缓存淘汰策略LRU与LFU](https://www.jianshu.com/p/c8aeb3eee6bc)
- [掘金-Takagi_san-Redis内存淘汰策略源码分析以及LFU/LRU实现](https://juejin.im/post/5e902c92e51d4546b847732b)