## ThreadLocal

[ThreadLocal](https://mp.weixin.qq.com/s/WKaUzChzj2PIcqiw05jcIA)

## 线程池

[进程、线程与线程池](https://raymond-zhao.top/2020/07/19/2020-07-19-JUC-ProcessAndThread/)

## volatile

[volatile 底层实现原理](https://raymond-zhao.top/2020/08/20/2020-08-19-volatile/)

## AQS 源码笔记

[AQS 源码笔记，主要看获取锁与释放锁的流程](https://raymond-zhao.top/2020/07/10/2020-07-10-Java-AQS/)

## 阻塞队列 BlockingQueue

- 主要用于 **生产者-消费者** 模式
  - 在多线程场景时**生产者**线程在**队列尾部添加元素**，而**消费者**则在**队列头部消费元素**；
  - 通过这种方式能够达到将任务的生产和消费进行隔离的目的。

- 提供可阻塞的入队和出队操作
  - 获取操作：在队列为空时，获取元素的线程会等待，直到队列变为非空；
  - 插入操作：当队列已满时，生产元素的线程会等待，直到队列变为非满。

在阻塞队列不可用时，上面的操作提供了 4 种处理方式：

| 操作 | 抛出异常    | 返回特殊值 | 一直阻塞 | 超时退出               |
| ---- | ----------- | ---------- | -------- | ---------------------- |
| 插入 | `add(e)`    | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| 删除 | `remove()`  | `poll()`   | `take()` | `poll(time, unit)`     |
| 查看 | `element()` | `peek()`   | 不可用   | 不可用                 |

> 阻塞队列原理：如果队列是空的，则消费者等待，如果队列是满的，则生产者等待。生产者、消费者得知队列的状态是 JDK 通过**通知模式**实现的。就是当生产者往满的队列添加元素时会被阻塞，当消费者消费了队列元素后，会通知生产者队列现在可用。
>
> 常见的阻塞队列：

- `ArrayBlockingQueue` ： 一个由数组结构组成的有界阻塞队列。
  - 底层基于 `final Object[] items` 实现；
  - 可传入 `fair` 构造为**公平/非公平** 阻塞队列；
  - 使用 ReentrantLock 来保证所有元素的访问安全、两个 Condition 熟悉用于 `take()` 与 `put` 操作；
- `LinkedBlockingQueue` ： 一个由链表结构组成的有界阻塞队列，默认值为 `Integer.MAX_VALUE`;
- `PriorityBlockingQueue` ：一个支持优先级排序的无界阻塞队列；
  - 默认自然序，也可自定义排序。
- `DelayQueue` ：一个使用优先队列实现的、支持延迟获取元素的无界阻塞队列；
  - 队列中的元素必须实现 Delayed 接口；
  - 用于设计缓存系统：可以用 DelayQueue 保存缓存元素的有效期，使用一个线程循环查询 DelayQueue，一旦能从 DelayQueue 中获取元素时，表示缓存有效期已到；
  - 用于处理超时任务：比如下订单后长期未支付，自动关闭订单。
- `SynchronousQueue` ：一个不存储元素的阻塞队列，每个 `put(..)` 操作必须等待一个 `take()` 操作，否则不能继续添加元素。
- `LinkedTransferQueue` ： 一个由链表结构组成的无界阻塞队列。
  - 多了 `transfer()` 和 `tryTransfer()` 方法；
  - `transfer()`：
    - 如果当前有消费者正在等待接收元素，直接把生产者生产的元素 transfer 给消费者；
    - 如果没有消费者在等待，将生产者生产的元素存放在队列尾部，知道该元素被消费者消费了才返回。
  - `tryTransfer()`：试探生产者掺入的元素能否直接传给消费者。
    - 如果没有消费者等待消费元素，则返回 false；
    - 与 `transfer()` 的区别是，本方法无论消费者是否接收，都立即返回，而 `transfer()` 方法是必须等到消费者消费了才返回。
- `LinkedBlockingDeque` : 一个由链表结构组成的双向阻塞队列。

