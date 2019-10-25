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

## ## Analysis of interview questions
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

For the direct operation of uesrs, the general requirement of Internet enterprises is that each request must be completed within 200ms, and they are almost imperceptible to users.

If **uses MQ**, system a will send three messages to MQ queue continuously. If it takes 5ms, system A will take 3 + 5 = 8ms from receiving a request to returning a response to the user. For the user, it feels like clicking a button. After 8ms, it will return directly. Cool! The website is well done, so fast!

![mq-4](/images/mq-4.png)

#### Peak clipping
From 0:00 to 23:00 every day, system A is calm, and the number of concurrent requests per second is 50. Results from 12:00 to 13:00 every time, the number of concurrent requests per second suddenly increased to 5K+. But the system is directly based on MYSQL, a large number of requests flow into mysql, and about 5K SQL are executed for MYSQL every second.

Ordinary MySQL, carrying up to 2K requests per second is almose the same. If the request reaches 5K per second, MySQL may be killed directly, resulting in system crash, and users will no longer be able to use the system.

In general, it's almost as long as you carry 2K requests per second. If you request 5K requests per second, you may directly kill mysql, which causes the system to crash, and users will no longer be able to use the system. But after the peak period, in the afternoon, it will become a low peak period. It may be that there are only 1W users operating on the website at the same time. The number of requests per second may also be 50, which has little pressure on the while system.

![mq-5](/images/mq-5.png)

If MQ is used, 5K requests per second are writeen to MQ, and system a processes 2K requests per second at most, because MySQL processes 2K requests per second at most. System a slowly pulls requests from MQ, and pulls 2K requests per second. It is OK not to exceed the maximum number of requests it can handle per second. In this way, system a will never hang up even in peak hours. However, when MQ has 5K requests in every second and 2 K requests out, the result is that at the peak of noon(one hour), there may be hundreds of thousands or even millions of requests in MQ. 

![mq-6](/images/mq-6.png)

This short peak tim ebacklog is OK, because after the peak time, 50 requests enter MQ every second, but system a still processes at the rate of 2K requests per second. So as soon as the peak period is over, system a will quickly solve the backlog of messages.

### What are the advantages and disadvantages of message queuing
As metioned above, **has its corresponding advantages in special scenarios** namely **decouply**, **asynchronous**, **peak shaving**.

The disadvantages are as follows:

- Reduced system availability    
The more external dependencies the system introduces, the easier it is to hang up. You are te interface of system a calling three systems of BCD. The four systems of ABCD are good. There is no problem. You add an MQ in. In case MQ hangs up, how about MQ hagn up? When MQ hangs up, thw whole system crashes, you are not finished? To ensure the high availability of message queues, [click here](/docs/high-concurrency/how-to-ensure-high-availability-of-message-queues.md).

- Increase of system complexity    
How can you [ensure that the message is not consumed repeqtedly](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md) by adding MQ? How to deal with [message loss](/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)? How to deal with message loss and ensure the order of message delivery? Big head, big head, a lot of probles, pain.

- Consustency issues    
When system A finishes processing and returns to success directly, everyone thinks that your request is successful. But the problem is that if the three systems of BCD and the two systems of BD succeed in writing the database, the system C fils in writing the database. What's the whole problem? You don't have the same data.

So message queuing is actually a very complex architecture. You have many advantages when you introduce it, but you also have to amke various additional technical solutions and architectures to avoid the disadvantages. After doing this, you will dins that the system complexity has increased by an order of magnitude, maybe 10 times more complex. But at the critical moment, it still has to be used.

### What are the advantages and disadvantages of Kafka, ActiveMQ, RabbitMQ and RocketMQ?

| Features | ActiveMQ | RabbitMQ | RocketMQ | Kafka |
|---|---|---|---|---|
| 单机吞吐量 | 万级，比 RocketMQ、Kafka 低一个数量级 | 同 ActiveMQ | 10 万级，支撑高吞吐 | 10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic 数量对吞吐量的影响 | | | topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
| 时效性 | ms 级 | 微秒级，这是 RabbitMQ 的一大特点，延迟最低 | ms 级 | 延迟在 ms 级以内 |
| 可用性 | 高，基于主从架构实现高可用 | 同 ActiveMQ | 非常高，分布式架构 | 非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性 | 有较低的概率丢失数据 | 基本不丢 | 经过参数优化配置，可以做到 0 丢失 | 同 RocketMQ |
| 功能支持 | MQ 领域的功能极其完备 | 基于 erlang 开发，并发能力很强，性能极好，延时很低 | MQ 功能较为完善，还是分布式的，扩展性好 | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |


To sum up, after various comparisons, there are the following suggestions:

MQ should be introduced into general business systems. At first, we used ActiveMQ, but now we don't use it very much. Without the verification of large-scale throughput scenarios, the community is not veryactive, so let's forget it. I don't recommend it personally.

Later, we began to use RabbitMQ, but it is true that Erlang language has prevented a large number of Jave engineers from further studying and controlling it. For the company, it is almost uncontrollable, but it is true that people are open-source, relatively stable support, and highly active.

But now more and more conpanies will use RocketMQ. It's really good. After all, it's made by Alibaba, bu the community may have the risk of suddenly turning yellow (RocketMQ has bee donated to [Apache](https://github.com/apache/rocketmq), but the activity on GitHub is not high) and they have absolute confudence in their own company's technical strength. RocketMQ is recommended, otherwise they will go back to the old. It's a practical RbbitMQ. People have an active open source community. It's definitely not yellow.

So **small and medium-sized companies** have general technical strength and not particularly high technical challenges. RabbitMQ is a good choice; **large companies** have strong infrastructure R & D strength and RocketMQ is a good choice.

If it is the real-time computing, log collection and other scenarios in the **big data field**, Kafka is the industry standard, absolutely no problem, the community activity is very high, absolutely not yellow, and it is almost the factual specification in this fiel fall over the world.
