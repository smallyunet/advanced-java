## Interview questions
How to ensure the reliable transmission of messages? Or, how to deal with the problem of message loss?

## Psychological analysis of interviewers
This is for sure. With MQ, there is a bisic principle, that is, there should be **no more data, no less data**, no more data, which is the above-mentioned [repeated consumption and idempotency issues](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md). It can't be less, that is to say, don't lose the data. Then you have to think about it.

If you use MQ to deliber very core messages, such as biling and fee deduction messages, you must ensure that **will never lose the billing messages** in the process of MQ delivery.

## Analysis of interview questions
Data loss may occur in producers, MQ and consumers. Let's analyze it from RabbitMQ and Kafka respectively.

### RabbitMQ
![rabbitmq-message-lose](/images/rabbitmq-message-lose.png)

#### Producer lost data

When the prooducer sends the data to RabbitMQ, the data may be lost in the middle of the way, because network problems and so on are possible.

At this time, you can choose to use the transaction function provided by RabbitMQ, that is, before the producer **sends the data** starts the RabbitMQ transaction `channel.txSelect`, and then sends the message. If the message is no successfully received by RabbitMQ, the producer will receive ana exception message. At this time, you can roll back the transaction `channel.txrollback`, and then try sending the message again. If the message is received, then Can commit transaction `channel.txcommit`.

```java
// Open transaction
channel.txSelect
try {
    // Send message here
} catch (Exception e) {
    channel.txRollback

    // Here's the message again
}

// Submission of affairs
channel.txCommit
```

But the problem is that when the RabbitMQ, transaction mechanism (synchronization) is implemented, the **throughput will be reduced basically because it consumers too much performance**.

So generally speaking, if you want to make sure that the message written to RabbitMQ is not lost, you can turn on the `confirm` mode. After setting the `confirm` mode to the producer, eahc mesage you write will the assigned a unique ID. Then if you write it to RabbitMQ, RabbitMQ willsend you an `ACK` message and tell you that the message is OK. If RabbitMQ fails to process the message, it will call back one of your `NACK` interfaces and tell you that the message has failed to receive. You can try again. And you can use this mechanism to maintain the status of each message ID in memeory. If you haven't received the callback of this message ofr a certain time, you can resend it.

The biggest difference between the transaction mechanism and the `confirm` mechanism is that the **transaction mechanism is synchronous**. After you submit a transaction, you will **block** there, but the `confirm` mechanism is **asynchronous**. After you send a message, you can send the next message, and then RabbitMQ will asychronously callback your interface to inform you that the message has been received.

Therefore, in general, the **confirm mechansim** is used to avoid data loss in the producer segment.

#### RabbitMQ lost data
It means that RabbitMQ lost its own data, you must **enable the persistence** of RabbitMQ, which means that the message will be persisted to disk after being written. Even if RabbitMQ hangs up, the **data stored before will be read automatically after recovery** and general data will not be lost. Unless it is extremely rare that RabbitMQ has not been persisted, it will hang up on its own, and **may cause a small amount of data loss**, but the probability is small.

There are **two steps to set persistence**:

- When creating a queue, set it to persisten.
This ensures that RabbitMQ till persist the metadata of the queue, but it will not persist the data in the queue.
- The second is to set the `deliverymode` of the message to 2 when sending a message.
In the case, RabbitMQ will persist the message to disk.

These two persistence settings must be set at the same time. Even it RabbitMQ is hung and restarted again, it will restart the recovery queue from the disk and recover the data in the queue.

Note that even if you have enabled the persistence mechanism for RabbitMQ, there is a possibility that this message has been written to RabbitMQ, but it has not yet been persisted to the disk. Unfortunately, when RabbitMQ hangs up, a little bit of data in the memory will be lost.

Therefore, persistence can be combined with the `confirm` mechanism of the producer. Only after the message is persisted to the disk will the producer be notified of `ACK`. So even before the message is persisted to the disk, RabbitMQ hangs, the data is lost, and the producer cannot receive `ACK`. You can resend it yourself.

#### Consumer lost data
If RabbitMQ loses data, it's mainly because when you consume, the **process is suspended after it's just consumed, for example, it's embarrassing to restart. RabbitMQ thinks that you consume all the data, and the data is lost.

At this time, you need to use the `ACK` mechanism provided by RabbitMQ. In short, you must turn off the sutomatic `ACK` of RabbitMQ, which can be called through an API, and then `ACK` in the program every time you make sure that your code finishes processing. In this case, if you haven't finished processing, there will be no `ACK`? Then RabbitMQ thinks you haven't finished processing. At this time, RabbitMQ will allocate this consumption to other consumers to process, and the message will not lost.

![rabbitmq-message-lose-solution](/images/rabbitmq-message-lose-solution.png)

### Kafka

#### Consumer lost data
唯一可能导致消费者弄丢数据的情况，就是说，你消费到了这个消息，然后消费者那边**自动提交了 offset**，让 Kafka 以为你已经消费好了这个消息，但其实你才刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

The only situation that may cause the consumer to lose data is that you consume the message, and then the consumer **automatically submits the offset** to Kafka to make him think that you have consumed the message, but in fact, you are just about to process the message. You have not yet processed it, and you hang up. At this time, the message will be lost.

One of the problems encountered in the production environment is that our Kafka consumer writes the data to a memory queue and buffers it first. As a result, sometimes you just write the message to the memory queue, and then the consumer will automatically submit the offset. Then we restart the system at this time, which will result in the loss of data in the memory queue that has not yet been processed.

#### Kafka lost data

A common scenario in this case is that a broker in Kafka goes down and re elects the leader of the parition. Let's think about it. If other followers happen to have some data that is not synchronized at this time, and the leader hangs up, then after a foolower is elected as a leader, there will be some data missing. That's missing some data.

The production environment has also encountered. We also encountered that the leader machine of Kafka was down before. After switching the follower to the leader, we will find that the data is lost.

Therefore, it is fenerally required to set at least four parameters as follows:

- Set the `replication.factor` parameter to topicL this value must be greater then 1. Each partition must have at least 2 copies.
- Set the `min.insync.replicas` parameter on Kafka server: the value must be greater than 1. This requires a leader to sense that at least one follower still keeps in touch with itself and does not fall behind, so as to ensure that the leader has another follower.
- Set `acks=all` on the produce side: This requires that each data must be **written to all replicas before it can be considered as a successful**.
- Set `retries=max` on the producer side (a very large value, meaning umlimited retries): This is **request unlimited retries in case of write failure**, the card is here.

Out production environment is configured according to the above requirements. After this configuration, at least at the Kafka broker end, we can ensure that when the broker where the leader is located fails, and when the leader is switched, the data will not be lost.

#### Will producers lose data?
If `acks=all` is set according to the above idea, it will not be lost. The requirement is that after your leader receives the message and all the followers synchronize to the message, it is considered that this write is successful. If this condition is not met, the producer will automatically try again and again for umlimited times.












