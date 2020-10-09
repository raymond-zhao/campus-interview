## CAP 定理

在理论计算机科学中，CAP 定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点：

- 一致性(Consistency): 等同于所有节点访问同一份最新的数据副本。
- 可用性(Availability): 每次请求都能获取到非错的响应，但是不保证获取的数据为最新数据。
- 分区容错性(Partition tolerance): 以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在 C 和 A 之间做出选择。

理解 CAP 理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了 C 性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了 A 性质。如果要同时保证 C 和 A，需要两个节点可以互相通信，但是这又会导致丧失 P 性质。

| 组件名    | 语言 | CAP   | 服务健康检查 | 对外暴露接口 | SpringCloud集成 |
| --------- | ---- | ----- | ------------ | ------------ | --------------- |
| Eureka    | Java | AP    | 可配支持     | HTTP         | 已集成          |
| Consul    | Go   | CP    | 支持         | HTTP/DNS     | 已集成          |
| Zookeeper | Java | CP    | 支持         | 客户端       | 已集成          |
| Nacos     | Java | CP+AP | 支持         | HTTP/DNS     | 已集成          |

> 问题：各个组件具体是怎么保证 CAP 的？
>

## Raft 一致性算法

> - [动画演示](http://thesecretlivesofdata.com/raft/)
>
> - [Raft GitHub](https://raft.github.io/)
>
> - [Raft 论文](https://raft.github.io/raft.pdf)

- 一个结点可以有三种状态：Follower(随从)、Candidate(候选者)、Leader(领导)
- 开始时所有的结点都处于 Follower 状态
- 如果 Followers 没有收到 Leader 的信息，他们就可以成为 Candidate
- 接下来 Candidate 就会请求其他结点进行投票
- 其他的 Followers(结点) 就会用投票来回复
- 如果 Candidate 获取了大多数结点的支持，它就会成为 Leader 结点

> 上面的过程称为 Leader Election(领导选举)，接下来对于这个系统的所有操作都将会经过这个 Leader 。

- 每一次改变都会成为结点的 log 中的一条记录(Entry)
- 如果 log entry 没有被提交的话，自然也就不会更新 结点的值
- 如果要提交 log entry 的话，首先需要复制刚才的操作到 Follower 结点，然后 Leader 会等待大多数结点写入这条 entry，然后 Leader 会通知 Followers 说 entry 修改已被提交。
- 这个时候集群(Cluster) 就会就系统的状态达成共识，进入一致性状态。

> 上面的过程叫做 Log Replication。

### Leader Election

- 在 Raft 中控制选举过程的有两种计时设置
- 一个叫做 election timeout，这个计时是指一个 Follower 等待成为 Candidate 所需要的时间。这个时间是 150ms~300ms 之间的随机值。
- 首先完成选举倒计时的结点(Follower)将会成为 Candidate，然后开始进行下一轮的选举。
- 成为 Candidate 的结点首先给自己投票，然后发送请求投票 (Request Vote) 信息给其他结点。
- 收到信息的结点在本轮投票中如果还未投票的话，就会投票给 Candidate，然后重新开始倒计时(election timeout)。
- 一旦 Candidate 结点拥有大部分结点的投票支持后它就会成为 Leader
- Leader 开始发送 Append Entries(添加条目) **消息**给它的 Followers
- 这些消息以 **heartbeat timeout**(心跳时间) 指定的时间间隔发送
- Followers 然后会给每一条 Append Entries 的消息予以回应
- 这个过程将会持续到 Follower 停止收到 heartbeat 并且成为 Candidate 为止

> 需要多数结点的投票保证了在每轮选举中只会产生一个 Leader，如果两个结点都同时成为 Candidate，将会产生分裂投票(脑裂?)

举个例子，如果在 4 个结点中（A、B、C、D），两个结点（A、B）同时成为了 Candidate，并且同时发送了请求投票的信息，并且同时又各自得到了剩余的两个结点（C、D）中其中之一的支持。这样的话，在本轮投票中 A、B 加上自身的投票与其支持者的投票都各自有了两票，并且不能够再获得其他的投票。

出现这种情况的话将会开始**新一轮的投票**，在本轮选举中可能就会产生**多数投票**的结果，选举出新的 Leader。

### Log Replication

- 一旦选举出 Leader 的话，我们就需要复制**所有的修改(all changes)** 给系统中的所有结点。
- 这是通过与 Heartbeats(心跳检测) 中所用到的 Append Entries 实现的(它们都是用的 Append Entries)。
- 首先，一个客户端(Client)发送修改(change)给 Leader，这条修改被写进 Leader's Log，然后这些改变将会在下一个 Heartbeat 时发送给它的 Followers。
- 一旦多数 Followers 确认了修改之后 entry 将会被提交，响应(Response) 也会被发送给 客户端(Client)。

> Raft 甚至在 network partitions(不同网络中，可能出现了网络故障)时也能够保持一致性。
>
> 因为分区，现在有不同选举阶段的 Leader，当在网络故障阶段即使不同分区中的 Leader 和 Followers 都接收到了修改请求也能够保持一致性。

比如集群中有 5 个结点，因为网络分成了2、3两个区，它们的多数值是 3，具有 2 个结点的集群中即使收到了修改也不会提交 entry，因为得不到 3 个支持，会保持 uncommitted。

这是因为，在网络恢复之后，将会校验不同分区中 Leader 的选举次数(election term)，具有较低选举次数的 Leader(并且没有) 将会甘拜下风，让 Term 值更大的 Leader 成为新的 Leader，但是之前与 Term 较小值同区的结点的 **未提交的条目(uncommitted entries)** 将会回滚(rollback)，然后去同步新 Leader 的 log。然后系统会再次进入一致性状态。

