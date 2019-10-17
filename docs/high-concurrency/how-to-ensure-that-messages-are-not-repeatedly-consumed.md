## Interview questions
How to ensure that messages are noe consumed repeatedly? In other words, how to ensure the idempotence of message consumption?

## Psychological analysis of interviewers
In fact, this is s avery common question. These two questions can be asked together. Since it's consumption message, we must consider whether it will be repeated consumption. Can we avoid repeated consumption? Or is it OK to repeat consumption without causing system exceptions? This is the basic problem in MQ field. In fact, it is still a question to ask you **how to ensure idempotence** by using message queuing. This is a question to be considered in your architecture.

## Analysis of interview questions
To answer this question, first of all, you don't need to hear about repeated information, so you have no idea. First of all, you can say about the possible repeated consumption problems.

First of all, for example, RabbitMQ, RocketMQ and Kafka may have the problem of repeated message consumption, which is normal. Because this problem is usually not guaranted by MQ itself, but by our development. Take Kafka, for example, and talk about how to re consume.

Kafka actually has an offset consept, that is, every message written in has an offset, which represents the sequence number of the message. Then after the consumer consumes the data, **every other period of time** (regular time), he will submit the offset of the message he has consumed, which means "I have consumed it. if I restart something next time, you will let me continue to cancel it from the last time. Pay the offset to continue consumption."

But there are always accidents. For example, what we often encounter in previous production is that you sometimes restart the system to see how you restart it. If you are in a bit of hurry, just kill the process and restart it. This will cause some messages of consumer to be processed, but the offset is not submitted in time, which is embarrassing. After the restart, a few messages will be consumed again.

Here's a chestnut.

There's a scene like this. Data 1/2/3 enters Kafka in turn, Kafka will assign an offset to each of the three data, representing the serial number of the data. We assume that the allocated offset is 152/153/154 in turn. When consumers consume from Kafka, they also consume in this order. If the consumer consumes the data with `offset = 153`, and is just about to submit the offset to zookeeper, then the consumer process is restarted. At this time, 1/2 of the offset of the consumed data is not submitted, and Kafka does not know that you have consumed the data `offset = 153`. Then after the restart, the consumer will look for Kafka and say, "Hey, man, you can give me the data of the last place I spent and continue to pass it to me." Since the previous offset was not submitted successfully, 1/2 of the data will be transmitted again. If the consumer does not de duplicate at this time, it will lead to repeated consumption.

![mq-10](/images/mq-10.png)

如果消费者干的事儿是拿一条数据就往数据库里写一条，会导致说，你可能就把数据 1/2 在数据库里插入了 2 次，那么数据就错啦。

其实重复消费不可怕，可怕的是你没考虑到重复消费之后，**怎么保证幂等性**。

举个例子吧。假设你有个系统，消费一条消息就往数据库里插入一条数据，要是你一个消息重复两次，你不就插入了两条，这数据不就错了？但是你要是消费到第二次的时候，自己判断一下是否已经消费过了，若是就直接扔了，这样不就保留了一条数据，从而保证了数据的正确性。

一条数据重复出现两次，数据库里就只有一条数据，这就保证了系统的幂等性。

幂等性，通俗点说，就一个数据，或者一个请求，给你重复来多次，你得确保对应的数据是不会改变的，**不能出错**。

所以第二个问题来了，怎么保证消息队列消费的幂等性？

其实还是得结合业务来思考，我这里给几个思路：

- 比如你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下好吧。
- 比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性。
- 比如你不是上面两个场景，那做的稍微复杂一点，你需要让生产者发送每条数据的时候，里面加一个全局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 Redis。如果消费过了，那你就别处理了，保证别重复处理相同的消息即可。
- 比如基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据插入只会报错，不会导致数据库中出现脏数据。

![mq-11](/images/mq-11.png)

当然，如何保证 MQ 的消费是幂等性的，需要结合具体的业务来看。