## 什么是幂等性？

接口幂等性就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生副作用。

## 哪些情况需要防止？

- 用户多次点击按钮
- 用户页面回退再次提交
- 用户刷新页面(POST操作时)
- 微服务互相调用，由于网络问题导致请求失败，Feign 触发重试机制。

## 什么情况需要幂等？

### 天然幂等

在 SQL 中，有些操作是天然幂等的。

- select * from table where id = ?; 即读操作天然幂等。
- update tab1 set col=1 where col2=2;也幂等。
- delete from user where user_id = 1; 也幂等。
- insert into user (user_id, name) values (1, 'a')，如果 user_id 为唯一主键，即重复执行上面的业务，只会插入一条数据，具备幂等；如果 user_id 不是唯一主键，则不幂等。

不幂等

- update tab1 set col1 = col1+1 where col2 = 2; 每次执行结果都会变化，不幂等。

> 订单号需要设置唯一索引保证幂等。

## 幂等解决方案

### Token机制

- 服务端需要提供发送 token 的接口。在分析业务时，可能会存在幂等问题的业务，就必须**在业务执行前，先去获取 token，服务器会把 token 保存到 redis 中。**
- 然后**请求业务接口时，把 token 携带过去**，一般放在请求头部中。
- 服务器**判断 token 是否在 redis 中**
  - 存在则表示第一次请求，然后删除 token，继续执行业务。
  - 不存在则表示是重复操作，直接返回重复标记给 Client，这样就保证了业务代码不被重复执行。

> 令牌存在两个地方，服务器一个，页面一个。

![下订单流程图](https://tva1.sinaimg.cn/large/007S8ZIlly1gihu3va0luj30mq0kota9.jpg)

**危险性**

- 先删除 token 还是后删除 token？
  - 先删除可能导致，业务确实没有执行，重试时带上之前的 token，由于 token 在第一次请求时已经删除，导致后来的请求不能成功。
  - 后删除可能导致，业务处理成功，但是服务闪断，出现超时，没有删除掉 token，别人继续重试，导致业务执行两遍。
  - **选择**：先删除 token，如果业务调用失败，就重新获取 token 再次请求。
- token 获取、比较、删除必须保证原子性
  - redis.get(token)、token.equals、redis.del(token) 可能导致在高并发下，都 get 到同样的数据，而且判断也成功，继续业务并发执行。
  - 可以在 redis 使用 lua 脚本完成这个操作。

```lua
if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end;
```

> 那么，为什么 lua 脚本能够保证原子性？
>
> Redis uses the same Lua interpreter to run all the commands. Also Redis guarantees that a script is executed in an atomic way: no other script or Redis command will be executed while a script is being executed. This semantic is similar to the one of [MULTI](https://redis.io/commands/multi) / [EXEC](https://redis.io/commands/exec). From the point of view of all the other clients the effects of a script are either still not visible or already completed.
>
> 
>
> 上面的话就是原因，当 Redis 可以保证在 Lua 脚本执行时，其他脚本或者 Redis 命令都不会被执行。这有点像 Redis 事务中的 MULTI/EXEC。对于其他的客户端来说，Lua 脚本对于他们来说要么是不可见的，要么是已经完成的。

### 各种锁机制

**1、数据库悲观锁**

```mysql
select * from xxx where id = xx for update;
```

一般伴随事务一起使用，数据锁定时间可能会很长，需要根据实际情况选用。另外要注意的是，ID 字段一定是主键或者被加了唯一索引，不然可能造成锁表，处理起来很麻烦。

**2、数据库乐观锁**

适合用在更新的场景中，版本号机制。主要用于读多写少的场景。

```mysql
update xx set x = x + 1, version = version + 1 where id = 2 and version = 1;
```

在操作库存前先获取当前商品的 version 版本号，然后操作的时候带上 version 版本号。

在第一次操作库存时，得到 version 为 1，调用库存服务 version 变成 2；但返回给订单服务出现了问题，订单服务有一次发起调用库存服务。当订单服务传入的 version 还是 1，再执行上面的 SQL 时将不会执行成功，因为 version 已经改变为 2。

**3、业务层分布式锁**

如果多台机器可能在同一时间内同时处理相同的数据，比如多台机器定时任务都拿到了相同数据处理，这个时候就可以使用分布式锁，锁定此数据，处理完成后释放锁，获取到锁的必须先判断这个数据是否被处理过。

### 各种唯一性约束

**1、数据库唯一性约束**

插入数据，应该按照唯一索引进行插入。比如订单号，相同的订单就不可能有两条记录插入。在数据库层面防止重复。

这个机制利用了数据库的主键唯一约束的特性，解决了在 insert 场景下的幂等问题。**但主键的要求不是自增的数据，这样就需要业务生成全局唯一的主键。**

如果是分库分表场景下，路由规则要保证相同请求下，落地在同一个数据库和同一张表中，要不然数据库主键约束就不气效果了，因为不同的数据库和表主键不相关。

**2、redis set 防重**

很多数据需要处理，只能被处理一次。比如可以计算数据的 MD5 将其放入 redis 的 set，每次处理数据，先看这个 MD5 是否已经存在，存在就不处理。

### 防重表

使用订单号 orderNo 作为去重表的唯一索引，把唯一索引插入去重表，再进行业务操作，且它们在同一个事务中。

这样保证了重复请求时，因为去重表有唯一约束，导致请求失败，避免了幂等问题。这里要注意的是，去重表和业务表应该在**同一个库**中，这样就保证了在同一个事务，即使业务操作失败了，也会把去重表的数据回滚，保证了**数据一致性**。

### 全局请求唯一 ID

调用接口时，生成一个唯一 id，redis 将数据保存到集合中(去重)，存在即被处理过。可以使用 Nginx 设置每一个请求的唯一 ID(没办法做防重处理)。

```nginx
proxy_set_header X-Requested-Id $request_id;
```

