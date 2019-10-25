## Interview questions
How to ensure high availability of message queues?

## Psychological analysis of interviewers

If someone asks about your MQ knowledge, **high avaliability is a must**. [As mentioned above](/docs/high-concurrency/why-mq.md), MQ will lead to **reduced system abailiability**. So as long as you use MQ, some of the key points to be asked next must be how to solve the shortcomings of MQ.

If you are stupid and use an MQ, you have never considered all kinds of problems. The interviewer's felling for you is that you can only some techniques simply, without any thinking, and your impression is not very good at once. If such a student is recruited to be an ordinary younger brother with a salary of less then 20K and a senior engineer with a salary of 20K+, it will be a disaster. Let's design a system. There must be a lot of holes in it. The company will suffer losses in the accident, and the team will back up together.

## ## Analysis of interview questions
It's a good question to ask, because I can't ask you know to guarantee the high availability of Kafka. How to ensure the high availabililty of ActiveMQ? If an interviewer asks like this, it seems that he is not good at it. He may use RabbitMQ instead of Kafka. Why do you come up and ask him Kafka? Isn't that a clear and diffcult one?

So a good interviewer asks how to guarantee the high availability of MQ. This is which MQ you have used, and you will talk about your understanding of the high availability of that MQ.

### High availiability of RabbitMQ
RabbitMQ is quite representative, because **does high availability based on master-slave** (non ditributed). Let's take RabbitMQ as an example to explain how to realize the high availability of the frist MQ.

RabbitMQ has three modes: stand-alone mode, general cluster mode and image cluster mode.

#### Stand alone mode
Stand alone mode, which is demo level, is generally the mode where you start playing locally, and no one uses stand-slone mode for production.

#### Gerneral cluster mode (no high availability)
The common cluster mode means to start multiple RabbitMQ instances on multiple machines, one for each machine. The queue you **create will only be placed on one RabbitMQ instance**, but each instance synchronizes the metadata of the queue (the metadata can be considered as some configuration information of the queue. Through the metadata, you can find the instance where the queue is located). When you consume, in fact, if you connect to another instance, that instance will pull data from the instance where the queue is located.

![mq-7](/images/mq-7.png)

This method is really troublesome and not very good. If **fails to achieve the so-called distributed**, it is a common cluster. Because this causes you to either connect one instance randomly and pull data at a time, or connect the consumption data of the instance hwere the queue is located. The format has **data pulling overhead** and the latter results in **single instance performance bottleneck**.

In addition, if the queue instance goes down, other instances will not be able to pull data from that instance. If you **enable message persistence** and let RabbitMQ land to store messages, the **message will not be lost**. You need to wait for this instance to reconver before you can continue to pull data from the queue.

So it's a bit awkward. There is **no so-called high availability** in **this scheme is mainly to improve the throughput**, that is to say, let multiple nodes in the cluster serve the read and write operations of a queue.

#### Mirror cluster mode (high availability)

This mode is the so-called RabbitMQ high availability mode. Unlike the normal cluster mode, in the image cluster mode, the queue you create, no matter the metadata or the messages in the queue **exist on multiple instances**, that is to say, each RabbitMQ node has a **complete image** of the queue, which contains the whole data of the queue. Then every time you write a message to the queue, you will automatically **synchronize message** to the queue of multiple instances.

![mq-8](/images/mq-8.png)

How can *turn on the image cluster mode**? In fact, RabbitMQ has a very good management console, which is to add a new strategy int the background. This atrategy is **the strategy of image cluster mode**. When specified, data can be synchronized to all nodes, or to a specified number of nodes. When creating a queue again, the application of this strategy will automatically synchronize data to other nodes. It's on.

In this way, the advantage is that any one of your machines is down. It's OK. Other machines (nides) also contain the complete data of the queue. Other consumers can go to other nodes to consume data. The disadvantage is that, first, the performance overhead is too large. Messages need to be synchronized to all machines, resulting in heavy network bandwidth pressure and consumption! Second, playing like this is not distributed, so **there is no scalability**. If a queue is heavily loaded, you add a machine, and the neewly added machine also contains all the data of the queue, and **there is no way to linearly expand** your queue. You think, if the amount of data in this queue is too large for the capacity of this machine, what should we do at this time?

### Kafka's high availability
Kafka's basic understanding of Architecture: it is composed of multiple brokers, each broker is a node; you create a topic; hiwch can be divided into mulutiple partitions, each partition can exist on different brokers, and each partition puts a part of data.

This is **the natural distributed message queue**, that is to say, the data of a topic is **distributed on multiple machines, and each machine puts a part of the data**.

In fact, RabbitMQ is not a distributed message queue, but a traditional message queue. It only provides some clustering and HA (high availability) mechanisms. No matter how you play, the data of a RabbitMQ is placed in a node. Under the mirror cluster, each node also puts the complete data of the queue.

Before Kafka 0.8, there was no HA mechanism. If any broker went down, the partition on that broker would be abandoned, unable to write or read, and no high availability.

For example, let's say that we have created a topic and specified that the number of its partitions is there on three machines. However, if the second machine goes down, 1/3 of the topic's data will be lost, so this is not highly available.

![kafka-before](/images/kafka-before.png)

After Kafka 0.8, HA mechanism is provided, that is, replica replica mechanism. Each partition's data will be synchronized to other machines to form its own multiple replica copies. All replicas will elect a leader, so production and consumption will deal with this leader, and then other replcias ar followers. Ehen writing, the leader will be responsible for synchronizing the data to all the followers. When reading, you can directly read the data on the leader. Can only read and write leaders? It's very simple, **if you can read an write each follower at will, you need to have the problem of care data consistency**. The system is too complex and easy to have problems, Kafka will evenly distribute all replicas of a partition on different amchines, so as to improve fault tolerance.

![kafka-after](/images/kafka-after.png)

In this way, there will be so-called high availability. If a broker goes down, it's OK. The partition on that broker has copies on other machines. If there is a leader of a partition on the down broker, a new leader will be selected from the following **re-election** and you can continue to read and write the new leader. That's what's called high availability.

**When writing data**, the producer writes the leader, then the leader writes the data to the local disk, and then other followers actively pull the data from the leader. Once all the followers have synchronized the data, an ACK will be sent to the leader. After the leader recives the ACK from all the followers, it will return the successful message to the producer. (of course, this is just on of the models, and the behavior can be adjusted appropriately.)

**When consuming**, it will only be read from the leader, but only when a message has been synchronized and successfully returned to ack by all followers, the message will be read by the consumer.

Seeing this, I believe you have roughly understood how Kafka guarantees the high availability mechanism, right? Don't be ignorant. You can draw pictures for the interviewers on the spot. If the interviewer is really a Kafka expert and asks deeply, you can only say that you are sorry. You have not studied deeply.
