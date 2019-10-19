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

#### 消费端弄丢了数据
RabbitMQ 如果丢失了数据，主要是因为你消费的时候，**刚消费到，还没处理，结果进程挂了**，比如重启了，那么就尴尬了，RabbitMQ 认为你都消费了，这数据就丢了。

这个时候得用 RabbitMQ 提供的 `ack` 机制，简单来说，就是你必须关闭 RabbitMQ 的自动 `ack`，可以通过一个 api 来调用就行，然后每次你自己代码里确保处理完的时候，再在程序里 `ack` 一把。这样的话，如果你还没处理完，不就没有 `ack` 了？那 RabbitMQ 就认为你还没处理完，这个时候 RabbitMQ 会把这个消费分配给别的 consumer 去处理，消息是不会丢的。

![rabbitmq-message-lose-solution](/images/rabbitmq-message-lose-solution.png)

### Kafka

#### 消费端弄丢了数据
唯一可能导致消费者弄丢数据的情况，就是说，你消费到了这个消息，然后消费者那边**自动提交了 offset**，让 Kafka 以为你已经消费好了这个消息，但其实你才刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

这不是跟 RabbitMQ 差不多吗，大家都知道 Kafka 会自动提交 offset，那么只要**关闭自动提交** offset，在处理完之后自己手动提交 offset，就可以保证数据不会丢。但是此时确实还是**可能会有重复消费**，比如你刚处理完，还没提交 offset，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。

生产环境碰到的一个问题，就是说我们的 Kafka 消费者消费到了数据之后是写到一个内存的 queue 里先缓冲一下，结果有的时候，你刚把消息写入内存 queue，然后消费者会自动提交 offset。然后此时我们重启了系统，就会导致内存 queue 里还没来得及处理的数据就丢失了。

#### Kafka 弄丢了数据

这块比较常见的一个场景，就是 Kafka 某个 broker 宕机，然后重新选举 partition 的 leader。大家想想，要是此时其他的 follower 刚好还有些数据没有同步，结果此时 leader 挂了，然后选举某个 follower 成 leader 之后，不就少了一些数据？这就丢了一些数据啊。

生产环境也遇到过，我们也是，之前 Kafka 的 leader 机器宕机了，将 follower 切换为 leader 之后，就会发现说这个数据就丢了。

所以此时一般是要求起码设置如下 4 个参数：

- 给 topic 设置 `replication.factor` 参数：这个值必须大于 1，要求每个 partition 必须有至少 2 个副本。
- 在 Kafka 服务端设置 `min.insync.replicas` 参数：这个值必须大于 1，这个是要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队，这样才能确保 leader 挂了还有一个 follower 吧。
- 在 producer 端设置 `acks=all`：这个是要求每条数据，必须是**写入所有 replica 之后，才能认为是写成功了**。
- 在 producer 端设置 `retries=MAX`（很大很大很大的一个值，无限次重试的意思）：这个是**要求一旦写入失败，就无限重试**，卡在这里了。

我们生产环境就是按照上述要求配置的，这样配置之后，至少在 Kafka broker 端就可以保证在 leader 所在 broker 发生故障，进行 leader 切换时，数据不会丢失。

#### 生产者会不会弄丢数据？
如果按照上述的思路设置了 `acks=all`，一定不会丢，要求是，你的 leader 接收到消息，所有的 follower 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。
