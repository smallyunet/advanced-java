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

If the consumer's job is to take a piece of data and write one in the database, it will lead to saying that you may have inserted 1/2 of the data into the database twice, then the data is wrong.

In fact, repeated consumption is not terrible. What's terrible is that you don't consider how to **guarantee idempotence after repeated consumption**.

Let's give you an example. Suppose you have a system that inserts a piece of data into the database when consuming a message. If you repeat a message twice, you will not insert two pieces of data. Is the data correct? But if you spend the second time, you can judge whether you have already consumed it. If you just throw it away, you will not keep a piece of data, so as to ensure the correctness of the data.

When a piece of data appears twice, there is only one piece of data in the database, which ensures the idempotence of the system.

Idempotence, to put it more simply, you need to make sure that the corresponding data will **not change** if you repeat a data or a request multiple times.

So the second question is, how to guarantee the idempotence of message queue consumption?

In fact, we have to think about it in combination with business. Here are some ideas:

- For example, if you want to write a data to the database, you need to check it according to the primary key first. If you have all the data, don't insert it. Update it.
- For example, if you write redis, it's OK. Anyway, every time it's set, natural idempotence.
- For example, you are not in the above two scenarios, which is a little more complicated. When you need to ask the producer to send each data, you need to add a globally unique ID, such as the order ID. After you consume it here, you need to check it in redis according to this ID. Have you consumed it before? If you haven't consumed it, you will process it. Then the ID will write redis. If you has been consumed, then you should not deal with it. Make sure not to deal with the same message repeatedly.
- For example, the unique key based on the database ensures that duplicate data will not be repeatedly inserted. Because there is only one key constraint, repeated data insertion will only report errors ans will not cause dirty data in the database.

![mq-11](/images/mq-11.png)

Of course, how to ensure that MQ consumption is idempotent needs to be combined with specific business.
