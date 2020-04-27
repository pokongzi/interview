# 常见问题

参考
> https://www.jianshu.com/p/65765dd10671

##如何保证从消息队列里拿到的数据按顺序执行

通过算法，将需要保持先后顺序的消息放到同一个消息队列中，然后只用一个消费者去消费该队列。
rabbitMQ：拆分多个queue，每个queue一个consumer，就是多一些queue而已。或者就是一个queue对应一个consumer，然后这个consumer内部用内存队列做排队，然后分发给底层不同的worker来处理
kafka：一个topic，一个partition，一个consumer，内部单线程消费，写n个内存queue，然后N个线程分别消费一个内存queue即可