## Interview questions
How to solve the problem of message queue delay and expiration? What should I do when the message queue is full? There are millions of messages standing in the backlog for several hours. How to solve it?

## Psynological analysis of interviewers
In fact, the essence of the question is to say that there may be something wrong with your consumption end and you will not consume; or the consumption speed is extremely slow. Then it's a problem. Maybe the disk of your message queue cluster is almost full and nobody is consuming it. What should I do at this time? Or the whole thing has been overstocked for serveral hours. What do you do at this time? Or you have a long backlog, so for example, RabbitMQ has set the message expiration time and then it doesn't work.

So in this case, in fact, it's quite common online. It's usually not available. One is a big case. Generally, for example, the consumer needs to write MySQL after each consumption. As a result, MySQL hangs up and the consumer hangs there. It doesn't move. Or something goes wrong with the consumer, which causes the consumption speed to be extremely slow.

## Analysis of interview questions
About this matter, let's sort it out one by one. Let's assume a scenario. Now we have a consumer failure, and then a large number or messages are backing in MQ. Now there are accidents and panic.

### A large number of messages have been backlog in MQ for serveral hours
Tens of millions of pieces of data have been backlog in MQ for seven or eight hours, from more than 4 p.m. to more than 11 p.m. This is a real scene we have encountered. It's really an online failure. At this time, otherwise, it's to fix the problem of consumer, let it recover the consumption speed, and then wait for a few hours for the consumption to be completed. This can't be said in the interview.

One conumer per second is 1000, three consumers per second is 3000, and one minute is 180000. So if you have a backing of millions to tens of millons of data, even if the consumer recovers, it will take about an hour to recover.

At this time, the capacity can only be expanded temporarily. The specific operation steps and ideas are as follows:
- Fix the problem of the consumer first, make sure it recovers the consumption speed, and then stop all the existing consumers.
- Create a new topic, partition is 10 times of the original, and temporarily establish the original 10 times of the number of queues.
- Then write a temporary consumer program to distribute data. This program is deployed to consume the backlog data. After **consumption, do not do time-consuming processing**, directly and evenly poll and write the temporarily established 10 times number of queues.
- Then temporarily requistion 10 times of machines to deploy consumers, and each batch of consumers consumes a temporary queue of data. This is equivalent to temporarily expanding the queue resources and consumer resources by 10 times, and consuming data at a normal speed of 10 times.
- After the fast consumption of the backlog data, **needs to restore the previously deployed architecture**, and **re** uses the original consumer machine to consume messages.

### mq 中的消息过期失效了
假设你用的是 RabbitMQ，RabbtiMQ 是可以设置过期时间的，也就是 TTL。如果消息在 queue 中积压超过一定的时间就会被 RabbitMQ 给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在 mq 里，而是**大量的数据会直接搞丢**。

这个情况下，就不是说要增加 consumer 消费积压的消息，因为实际上没啥积压，而是丢了大量的消息。我们可以采取一个方案，就是**批量重导**，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入 mq 里面去，把白天丢的数据给他补回来。也只能是这样了。

假设 1 万个订单积压在 mq 里面，没有处理，其中 1000 个订单都丢了，你只能手动写程序把那 1000 个订单给查出来，手动发到 mq 里去再补一次。

### mq 都快写满了
如果消息积压在 mq 里，你很长时间都没有处理掉，此时导致 mq 都快写满了，咋办？这个还有别的办法吗？没有，谁让你第一个方案执行的太慢了，你临时写程序，接入数据来消费，**消费一个丢弃一个，都不要了**，快速消费掉所有的消息。然后走第二个方案，到了晚上再补数据吧。
