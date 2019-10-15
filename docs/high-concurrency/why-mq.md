## Interview questions
- Why use message queues?
- What are the advantages and disadvantages of message queues?
- What are the differences between Kafka, ActiveMQ, RabbitMQ and RocketMQ, and what scenarios are suitable for them?

## Interviewer Psycholological Analysis
In face, the interviewer maninly wants to see:

- **First**, do you know why message queuing is used in your system?    
Many candidates say that they uesd Redis and MQ in their projects, but in fact they don't know why they used it. In fact, to put ot plainly, it is for use, or the structure designed by others, he never thought about it from beginning to end.    
Those who haven't asked why they are structured must be people who don't usually think about it. Interviewers usually have a bad impression of such candidates. Because the interviewer is afraid that when you join the team, you will only do dull work and will not think for yourself.

- **Second**, since you use the message queue, do you know what's the advantage and disadvantage of using it?    
If you haven't thought about it, then you blindly get an MQ into the system, and there's a problem behind it. Did you slip away and leave a hole for the company? If you don't consider the drawbacks and risk of introducing a technology, the interviewer will recruit such candidates, which is basically a digger. I'm afraid you'll dig a lot of pits for a year and hop your own job, leaving endless.

- **Third**, since you used MQ, maybe some kind of MQ, did you do any research at that time?    
Don't foolishly pat your head to see your personal preferences and use an MQ, such as Kafka, without even investigating what kinds of MQ are popular in the industry. What are the advantages and disadvantages of each MQ? Each MQ **has no absolute good or bad**, but it depends on which scenaric can be used to **enhance strengths and avoid wearknesses, make use of its strengths and avoid its weaknesses**.    
If a candidate who does not consider technology selection is recruited into the team, leader gives him a task to design a system. He uses some technology in it, and may not consider the selection. The last technology may not be appropriate, but it is pitkeeping.

## Analysis of Interview Questions
### Why use message queues
In fact, it si to ask you what use scenarics are there in the message queue, and then what specific scenarios are in you project, and what are using the message queue in this scenario?

The interviewer asks you this questiong, **An expected answer** is to say what **business scenario** your company has, what technical challenges this business scenario has, if you don't use MQ, it may be troublesome, but now you use MQ after bringing you a lot of benefits.

Let's start with the common usage scenarios of message queues. In fact, there are many scenarios, but there are three core scenarios: **decoupling**, **asynchronization** and **peak shaving**.

#### Decoupling
Look at this scene. System A sends data to three systems of BCD and sends it through interface calls. What if the E system aslo wants this data? What if C system is not needed now? System A leader almost collapsed...

![mq-1](/images/mq-1.png)

In this scenario, system A is heavily coupled with other messy systems. System A generates a critical piece of data, and many systems need system A to send this data. System A shoud always consider the four BCDE systems. What if they hang up? Do you want to resend or save the message? Your hair is white!

If MQ is used, system A generates a piece of data and sends it to MQ, which system needs data to consume in MQ itself. If the new system needs data, it can be consumed directly from MQ; if a system does not need this data, it can cancel the consumption of MQ messages. In this way, system A does not need to consider who to send data to, maintain the code, or whether the call succeeds or fails over time.

![mq-2](/images/mq-2.png)

**Summary**: System A is completely decoupled from other system through a model of MQ, Pub/Sub publishing subscription messages.

**Interview Skills**: You need to consider whether there is a similar scenario in the system you are responsible for, that is, a system or a module that calls multiple systems or modules, and the calls between them are complex and difficult to maintain. But in fact, this call does not need to directly call the interface synchronously. If it is possible to use MQ to decouple it asynchronously, you need to consider whether you can use this MQ to decouple the system in your project. This is reflected in the resume, using MQ as decoupling.

#### Asynchronous
In another scenario, system A receives a request and needs to write libraries locally. It also needs to write libraries locally in three systems of BCD, three systems of BCD need to write libraries 3ms and three systems of BCD need to write libraries 300ms, 450ms and 200ms respectively. The total latency of the final request is 3 + 300 + 450 + 200 = 953ms, which is close to 1s. The user feels like he's doing something and he's dead and show. It is almost unacceptable for a user to initiate a request through a browser and wait for one second.

![mq-3](/images/mq-3.png)

一般互联网类的企业，对于用户直接的操作，一般要求是每个请求都必须在 200 ms 以内完成，对用户几乎是无感知的。

如果**使用 MQ**，那么 A 系统连续发送 3 条消息到 MQ 队列中，假如耗时 5ms，A 系统从接受一个请求到返回响应给用户，总时长是 3 + 5 = 8ms，对于用户而言，其实感觉上就是点个按钮，8ms 以后就直接返回了，爽！网站做得真好，真快！

![mq-4](/images/mq-4.png)

#### 削峰
每天 0:00 到 12:00，A 系统风平浪静，每秒并发请求数量就 50 个。结果每次一到 12:00 ~ 13:00 ，每秒并发请求数量突然会暴增到 5k+ 条。但是系统是直接基于 MySQL 的，大量的请求涌入 MySQL，每秒钟对 MySQL 执行约 5k 条 SQL。

一般的 MySQL，扛到每秒 2k 个请求就差不多了，如果每秒请求到 5k 的话，可能就直接把 MySQL 给打死了，导致系统崩溃，用户也就没法再使用系统了。

但是高峰期一过，到了下午的时候，就成了低峰期，可能也就 1w 的用户同时在网站上操作，每秒中的请求数量可能也就 50 个请求，对整个系统几乎没有任何的压力。

![mq-5](/images/mq-5.png)

如果使用 MQ，每秒 5k 个请求写入 MQ，A 系统每秒钟最多处理 2k 个请求，因为 MySQL 每秒钟最多处理 2k 个。A 系统从 MQ 中慢慢拉取请求，每秒钟就拉取 2k 个请求，不要超过自己每秒能处理的最大请求数量就 ok，这样下来，哪怕是高峰期的时候，A 系统也绝对不会挂掉。而 MQ 每秒钟 5k 个请求进来，就 2k 个请求出去，结果就导致在中午高峰期（1 个小时），可能有几十万甚至几百万的请求积压在 MQ 中。

![mq-6](/images/mq-6.png)

这个短暂的高峰期积压是 ok 的，因为高峰期过了之后，每秒钟就 50 个请求进 MQ，但是 A 系统依然会按照每秒 2k 个请求的速度在处理。所以说，只要高峰期一过，A 系统就会快速将积压的消息给解决掉。

### 消息队列有什么优缺点
优点上面已经说了，就是**在特殊场景下有其对应的好处**，**解耦**、**异步**、**削峰**。

缺点有以下几个：

- 系统可用性降低<br>
系统引入的外部依赖越多，越容易挂掉。本来你就是 A 系统调用 BCD 三个系统的接口就好了，人 ABCD 四个系统好好的，没啥问题，你偏加个 MQ 进来，万一 MQ 挂了咋整，MQ 一挂，整套系统崩溃的，你不就完了？如何保证消息队列的高可用，可以[点击这里查看](/docs/high-concurrency/how-to-ensure-high-availability-of-message-queues.md)。

- 系统复杂度提高<br>
硬生生加个 MQ 进来，你怎么[保证消息没有重复消费](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md)？怎么[处理消息丢失的情况](/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)？怎么保证消息传递的顺序性？头大头大，问题一大堆，痛苦不已。

- 一致性问题<br>
A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。

所以消息队列实际是一种非常复杂的架构，你引入它有很多好处，但是也得针对它带来的坏处做各种额外的技术方案和架构来规避掉，做好之后，你会发现，妈呀，系统复杂度提升了一个数量级，也许是复杂了 10 倍。但是关键时刻，用，还是得用的。

### Kafka、ActiveMQ、RabbitMQ、RocketMQ 有什么优缺点？

| 特性 | ActiveMQ | RabbitMQ | RocketMQ | Kafka |
|---|---|---|---|---|
| 单机吞吐量 | 万级，比 RocketMQ、Kafka 低一个数量级 | 同 ActiveMQ | 10 万级，支撑高吞吐 | 10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic 数量对吞吐量的影响 | | | topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
| 时效性 | ms 级 | 微秒级，这是 RabbitMQ 的一大特点，延迟最低 | ms 级 | 延迟在 ms 级以内 |
| 可用性 | 高，基于主从架构实现高可用 | 同 ActiveMQ | 非常高，分布式架构 | 非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性 | 有较低的概率丢失数据 | 基本不丢 | 经过参数优化配置，可以做到 0 丢失 | 同 RocketMQ |
| 功能支持 | MQ 领域的功能极其完备 | 基于 erlang 开发，并发能力很强，性能极好，延时很低 | MQ 功能较为完善，还是分布式的，扩展性好 | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |


综上，各种对比之后，有如下建议：

一般的业务系统要引入 MQ，最早大家都用 ActiveMQ，但是现在确实大家用的不多了，没经过大规模吞吐量场景的验证，社区也不是很活跃，所以大家还是算了吧，我个人不推荐用这个了；

后来大家开始用 RabbitMQ，但是确实 erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，活跃度也高；

不过现在确实越来越多的公司会去用 RocketMQ，确实很不错，毕竟是阿里出品，但社区可能有突然黄掉的风险（目前 RocketMQ 已捐给 [Apache](https://github.com/apache/rocketmq)，但 GitHub 上的活跃度其实不算高）对自己公司技术实力有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，人家有活跃的开源社区，绝对不会黄。

所以**中小型公司**，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择；**大型公司**，基础架构研发实力较强，用 RocketMQ 是很好的选择。

如果是**大数据领域**的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。
