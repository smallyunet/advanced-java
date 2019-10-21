## Interview questions
How to ensure the order of messages?

## Psychological analysis of interviewers
In fact, this is also a topic that must be asked when using MQ. First, do you know the order? Second, do you have any way to ensure that the messages are in order? This is a common problem in production system.

## Analysis of interview quesions
For example, we have done a MySQL `binlog` ysnchronization system before, but the pressure is still bery great. The daily synchronization data should reach up to 100 million, that is to say, the data is synchronized from one MySQL database to another intact (MySQL -> MySQL). A common point is that for example, a big data tem needs to synchronize a MySQL database to perform various complex operations on the company's business system data.

You add, delete, and change a piece of data in mysql, corresponding to three `binlog` logs. Then therse three `binlog` logs are sent to MQ, and then consumed for executiong in turn. At least you need to ensure that people come in order, right? Otherwise, it would have been: add, modify and delete; you have changed the order to delete, modify and add. Isn't it all wrong? 

Originally, when this data was synchronized, it should be deleted at last, as a result, you got the order wrong, and finally the data was preserved, so the data synchronization went wrong.

Let's first look at two scenes where the order will be disordered:
- **RabbitMQ**: One queue, multiple consumers. For example, the producer sends three pieces of data to RabbitMQ in the order of data1/data2/data3, and presses in a memory queue of RabbitMQ. Three consumers consume on of these three data from MQ. As a result, consumer 2 performs the operation first, stores data2 in the database, and then data1/data3. It's not obviously messy.

![rabbitmq-order-01](/images/rabbitmq-order-01.png)

- **Kafka**: For example, we built a topic with three partitions. when the producer writes, he can actually specify a key. For exmaple, if we specify an order ID as the key, the data related to this order will be distributed to the same partition, and the data in this partition must be in order.    
When consumers extract data from the partition, there must be order. Here, the order is OK, there is no confusion. Then, we may have multiple threads in the consumer to process messages concurrently. Because if the consumer consumes processing in a single thread, and the processing is relatively time-consuming, for example, it takes tens of MS to process a message, then only tens of messages can be processed in on second, which is too low throughput. If multiple threads run at the same time, the order may be disordered.

![kafka-order-01](/images/kafka-order-01.png)

### Soluiton
#### RabbitMQ
Splitting multiple queues, one consumer for each queue, is just a few more queues, which is really a trouble point, or just one queue but corresponding to one consumer, and then the consumer uses the memory queue to queue internally, and then distributes it to different workers at the bottom layer for processing.
![rabbitmq-order-02](/images/rabbitmq-order-02.png)

#### Kafka
- One topic, one partition, one consumer, internal single thread consumption. The throughput of single thread is too low. Generally, this is not used.
- Write N memory queues, and all data with the same key will go to the same memory queue, then for N threads, each thread will consume one memory queue respectively, so as to ensure the order.

![kafka-order-02](/images/kafka-order-02.png)