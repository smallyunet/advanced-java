# Internet Java Engineer Advanced Knowledge Completely Literacy<sup>[©](https://github.com/yanglbme)</sup>

Most of the content of this project comes from ZhongHuaShiShan, and the copyright belongs to the author. The content covers knowledge in the fields of high concurrency, distribution, high availability, micro-service, etc..

## High concurrency architecture
### [Message queue](/docs/high-concurrency/mq-interview.md)
- [Why use message queues? What are the advantages and disadvantages of message queues? What are the advantages and disadvantages of KafKa, ActiveMQ, RabbitMQ and RocketMQ?](/docs/high-concurrency/why-mq.md)
- [How to ensure high abailability of message queues?](/docs/high-concurrency/how-to-ensure-high-availability-of-message-queues.md)
- [How to ensure that messages are not consumed repeatedly?（how to guarantee the idempotence of message consumption）](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md)
- [Hwo to ensure the reliable transmission of messages? (How to deal with message loss?)](/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)
- [How to ensure the order of messages?](/docs/high-concurrency/how-to-ensure-the-order-of-messages.md)
- [How to solve the problem of message queue delay and expiration? What should I do when the message queue is full? There are millions of messages standing in the backlog for several hours. How to solve it?](/docs/high-concurrency/mq-time-delay-and-expired-failure.md)
- [If you are asked to write a message queue, how to design the architecture? Say what you think.](/docs/high-concurrency/mq-design.md)

### [Search Engines](/docs/high-concurrency/es-introduction.md)
- [Can you explain the principle of distributed architecture of ES?](/docs/high-concurrency/es-architecture.md)
- [How does ES write data? How does ES query data work? What about Lucene at the bottom? Do you understand the inverted index?](/docs/high-concurrency/es-write-query-search.md)
- [In the case of a large amount of data (billons of levels), how can a query efficiency improve?](/docs/high-concurrency/es-optimizing-query-performance.md)
- [es 生产集群的部署架构是什么？每个索引的数据量大概有多少？每个索引大概有多少个分片？](/docs/high-concurrency/es-production-cluster.md)

### Cache
- [How is caching used in projects? What are the consequences of improper use of cache?](/docs/high-concurrency/why-cache.md)
- [What is the difference between Redis and Memcached? What is the threading model of Redis? Why is single-threaded Redis much more efficient than multi-threaded Memcached?](/docs/high-concurrency/redis-single-thread-model.md)
- [What data types are there for Redis? In which scenarios are they suitable for use?](/docs/high-concurrency/redis-data-types.md)
- [What are Redis' expiration policies? Handwritten LRU code implementation?](/docs/high-concurrency/redis-expiration-policies-and-lru.md)
- [How to ensure that Redis is highly concurrent and highly available? Can Redis' master-slave replication principle be introduced? Can Redis's sentinel principle be introduced?](/docs/high-concurrency/how-to-ensure-high-concurrency-and-high-availability-of-redis.md)
- [How many ways does Redis persist? What are the advantages and disadvantages of different persistence mechanisms? How is the specific underlying layer of the persistence mechanism implemented?](/docs/high-concurrency/redis-persistence.md)
- [Redis 集群模式的工作原理能说一下么？在集群模式下，Redis 的 key 是如何寻址的？分布式寻址都有哪些算法？了解一致性 hash 算法吗？如何动态增加和删除一个节点？](/docs/high-concurrency/redis-cluster.md)
- [What is the avalancle, penetration and breakdown of redis? What happens when redis crashes? How should the system deal with this situation? How to deal with readis penetration?](/docs/high-concurrency/redis-caching-avalanche-and-caching-penetration.md)
- [如何保证缓存与数据库的双写一致性？](/docs/high-concurrency/redis-consistence.md)
- [What are the concurrent competition problems of redis? How to solve this problem? Do you know CAS scheme of redis transaction?](/docs/high-concurrency/redis-cas.md)
- [How is Redis deployed in a production envirnoment?](/docs/high-concurrency/redis-production-environment.md)

### Sub library table
- [为什么要分库分表（设计高并发系统的时候，数据库层面该如何设计）？用过哪些分库分表中间件？不同的分库分表中间件都有什么优点和缺点？你们具体是如何对数据库如何进行垂直拆分或水平拆分的？](/docs/high-concurrency/database-shard.md)
- [现在有一个未分库分表的系统，未来要分库分表，如何设计才可以让系统从未分库分表动态切换到分库分表上？](/docs/high-concurrency/database-shard-method.md)
- [如何设计可以动态扩容缩容的分库分表方案？](/docs/high-concurrency/database-shard-dynamic-expand.md)
- [分库分表之后，id 主键如何处理？](/docs/high-concurrency/database-shard-global-id-generate.md)

### Read/write separation
- [How to realize the read-write separation of MySQL? What is the principle of MySQL master0slave replication? How to solve the dalay problem of MySQL master-slave synchronization?](/docs/high-concurrency/mysql-read-write-separation.md)

### High concurrency system
- [How to design a high concurrency system?](/docs/high-concurrency/high-concurrency-design.md)

## Distributed system
### [Interview battery](/docs/distributed-system/distributed-system-interview.md)
### System split
- [Why do system splits occur? How to do a system split? Can I use Dubbo after splitting?](/docs/distributed-system/why-dubbo.md)

### Distributed service framework
- [说一下 Dubbo 的工作原理？注册中心挂了可以继续通信吗？](/docs/distributed-system/dubbo-operating-principle.md)
- [Dubbo 支持哪些序列化协议？说一下 Hessian 的数据结构？PB 知道吗？为什么 PB 的效率是最高的？](/docs/distributed-system/dubbo-serialization-protocol.md)
- [Dubbo 负载均衡策略和集群容错策略都有哪些？动态代理策略呢？](/docs/distributed-system/dubbo-load-balancing.md)
- [What is Dubbo's spi thought?](/docs/distributed-system/dubbo-spi.md)
- [如何基于 Dubbo 进行服务治理、服务降级、失败重试以及超时重试？](/docs/distributed-system/dubbo-service-management.md)
- [分布式服务接口的幂等性如何设计（比如不能重复扣款）？](/docs/distributed-system/distributed-system-idempotency.md)
- [分布式服务接口请求的顺序性如何保证？](/docs/distributed-system/distributed-system-request-sequence.md)
- [How to design a Dubbo like RPC framework?](/docs/distributed-system/dubbo-rpc-design.md)

### Distributed lock
- [Zookeeper 都有哪些应用场景？](/docs/distributed-system/zookeeper-application-scenarios.md)
- [使用 Redis 如何设计分布式锁？使用 Zookeeper 来设计分布式锁可以吗？以上两种分布式锁的实现方式哪种效率比较高？](/docs/distributed-system/distributed-lock-redis-vs-zookeeper.md)

### Distributed transaction
- [分布式事务了解吗？你们如何解决分布式事务问题的？TCC 如果出现网络连不通怎么办？XA 的一致性如何保证？](/docs/distributed-system/distributed-transaction.md)

### Distributed session
- [How to implement distributed sessions during cluster deployment?](/docs/distributed-system/distributed-session.md)

## High avalibrary architecture
- [Introduction to Hystrix](/docs/high-availability/hystrix-introduction.md)
- [电商网站详情页系统架构](/docs/high-availability/e-commerce-website-detail-page-architecture.md)
- [Hystrix 线程池技术实现资源隔离](/docs/high-availability/hystrix-thread-pool-isolation.md)
- [Hystrix 信号量机制实现资源隔离](/docs/high-availability/hystrix-semphore-isolation.md)
- [Hystrix 隔离策略细粒度控制](/docs/high-availability/hystrix-execution-isolation.md)
- [深入 Hystrix 执行时内部原理](/docs/high-availability/hystrix-process.md)
- [基于 request cache 请求缓存技术优化批量商品数据查询接口](/docs/high-availability/hystrix-request-cache.md)
- [基于本地缓存的 fallback 降级机制](/docs/high-availability/hystrix-fallback.md)
- [深入 Hystrix 断路器执行原理](/docs/high-availability/hystrix-circuit-breaker.md)
- [深入 Hystrix 线程池隔离与接口限流](/docs/high-availability/hystrix-thread-pool-current-limiting.md)
- [基于 timeout 机制为服务接口调用超时提供安全保护](/docs/high-availability/hystrix-timeout.md)

### High availability system
- How to design a highly available system?

### Current limiting
- How to limit current? How is it done at work? What's the implementation?

### Fuse
- How to fuse?
- What are the fuse frames? Do you know the specific implementation principle?
- [熔断框架如何做技术选型？选用 Sentinel 还是 Hystrix？](/docs/high-availability/sentinel-vs-hystrix.md)

### Downgrade
- How to downgrade?

## Microservice architecture
- [A description of microservice architecture](/docs/micro-services/microservices-introduction.md)
- [从单体式架构迁移到微服务架构](/docs/micro-services/migrating-from-a-monolithic-architecture-to-a-microservices-architecture.md)
- [微服务的事件驱动数据管理](/docs/micro-services/event-driven-data-management-for-microservices.md)

### Spring Cloud Microservice architecture
- What is a microservice? How do microservices communicate independently?
- What are the differences between A and B?
- Spring Boot and Spring Cloud，how do you understand them?
- What is service fusing? What is service degradation?
- What are the advantages and disadvantages of microservices? What are the pitfalls you have encountered in project development?
- What do you know about the microservice technology stack?
- Eureka and Zookeeper can both provide service registration and discovery. What's the difference between them?
- ......
