# 常见问题
> https://blog.csdn.net/Butterfly_resting/article/details/89668661
> https://www.jianshu.com/p/65765dd10671
> https://ifeve.com/redis-lock/

## 1.为什么要用 redis /为什么要用缓存
```
高性能
操作内存，无io
高并发
直接操作缓存能够承受的请求是远远大于直接访问数据库的,所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。

```
## 2.redis 和 memcached 的区别
```
1.redis支持更丰富的数据类型（支持更复杂的应用场景)
2.Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而Memecache把数据全部存在内存之中
3.memcached没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 redis 目前是原生支持 cluster 模式的
4.Redis和Memcache在写入性能上面差别不大，读取性能上面尤其是批量读取性能上面Memcache更强
```
## 3.redis的底层数据结构
```
  Redis的字典底层使用哈希表实现，每个字典通常有两个哈希表，一个平时使用，另一个用于rehash时使用，使用链地址法解决哈希冲突。
  跳跃表通常是有序集合的底层实现之一，表中的节点按照分值大小进行排序。

```
## 4.相比于keys命令，scan命令两个的优缺点
```
https://www.jianshu.com/p/be15dc89a3e8
```
## 5.使用过Redis做异步队列么，你是怎么用的？
```
一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。
如果对方追问可不可以不用sleep呢？list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来。
如果对方追问能不能生产一次消费多次呢？使用pub/sub主题订阅者模式，可以实现1:N的消息队列。
如果对方追问pub/sub有什么缺点？在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。
如果对方追问redis如何实现延时队列？我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的这么详细。但是你很克制，然后神态自若的回答道：使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。

```
## 6. Redis如何做持久化的？
> RDB做镜像全量持久化，AOF做增量持久化。因为RDB会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要AOF来配合使用。在redis实例重启时，会使用RDB持久化文件重新构建内存，再使用AOF重放近期的操作指令来实现完整恢复重启之前的状态。


## 7.Redis的同步机制了解么？

## 8.是否使用过Redis集群，集群的原理是什么？

## 9. Pipeline有什么好处，为什么要用pipeline？
> 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
> pipeline可以将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性。

## 10.对方追问RDB的原理是什么？
> 你给出两个词汇就可以了，fork和cow。fork是指redis通过创建子进程来进行RDB操作，cow指的是copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

## 11.讲解下Redis线程模型
> I/O 多路复用程序负责监听多个套接字， 并向文件事件分派器传送那些产生了事件的套接字。
尽管多个文件事件可能会并发地出现， 但 I/O 多路复用程序总是会将所有产生事件的套接字都入队到一个队列里面， 然后通过这个队列， 以有序（sequentially）、同步（synchronously）、每次一个套接字的方式向文件事件分派器传送套接字： 当上一个套接字产生的事件被处理完毕之后（该套接字为事件所关联的事件处理器执行完毕）， I/O 多路复用程序才会继续向文件事件分派器传送下一个套接字。如果一个套接字又可读又可写的话， 那么服务器将先读套接字， 后写套接字.
[![img](https://img-blog.csdnimg.cn/20190429094050254.png)]

## 12.讲解下Redis线程模型
> Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的
Redis会将一个事务中的所有命令序列化，然后按顺序执行。
1.redis 不支持回滚“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。
2.如果在一个事务中的命令出现错误，那么所有的命令都不会执行；
3.如果在一个事务中出现运行错误，那么正确的命令会被执行。

1）MULTI命令用于开启一个事务，它总是返回OK。 MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。
2）EXEC：执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 nil 。
3）通过调用DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。
4）WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。
————————————————
版权声明：本文为CSDN博主「睶先森」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/Butterfly_resting/article/details/89668661


## 布隆过滤器
个名叫 Bloom 的人提出了一种来检索元素是否在给定大集合中的数据结构，这种数据结构是高效且性能很好的，但缺点是具有一定的错误识别率和删除难度。并且，理论情况下，添加到集合中的元素越多，误报的可能性就越大

- 结构
它看作由二进制向量（或者说位数组）和一系列哈希函数两部分组成的数据结构

- 原理
使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。
根据得到的哈希值，在位数组中把对应下标的值置为 1。

## 上述Redis分布式锁的缺点
在redis master实例宕机的时候，可能导致多个客户端同时完成加锁。

## 分区不同实现方式
客户端分区：客户端直接选择正确节点读写指定键。很多Redis客户实现了这种分区方式。
代理辅助分区：是指我们的客户端通过Redis协议把请求发送给代理，而不是直接发送给真正的Redis实例服务器。这个代理会确保我们的请求根据配置分区策略发送到正确的Redis实例上，并返回给客户端。Redis和Memcached的代理都是用Twemproxy （译者注：这是twitter开源的一个代理框架）来实现代理服务分区的。
查询路由：是指你可以把一个请求发送给一个随机的实例，这时实例会把该查询转发给正确的节点。通过客户端重定向(客户端的请求不用直接从一个实例转发到另一个实例，而是被重定向到正确的节点)，Redis集群实现了一种混合查询路由。
