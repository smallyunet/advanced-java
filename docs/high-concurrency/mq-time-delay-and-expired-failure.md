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

### Message in MQ expired
Suppose you use RabbitMQ, which can set the expiration time, that is TTL. If the message backlog in the queue exceeds a certain time, it will be cleared by RabbitMQ, and the data will be lost. Then this is the second pit. This is not to say that a large amount of data will be overstocked in MQ, but **a large amount of data will be directly lost**.

In this case, it doesn't mean to increase the message backlog of consumer consumption, because there is no backlog in fact, but a large number of messages are lost. We can take a plan, that is mass retransmission. We have done similar things on the front line. When there is a large backlog, we discard the data directly, and then after the peak period, for example, when everyone drinks coffee and stays up late until 12 p.m., users go to bed. At this time, we will start to write a program, write a temporary program to find out the lost data bit by bit, and then refill the MQ to make up for the data lost in the daytime. This is the only way.

Suppose 10000 orders are backlog in MQ and have not been processed, and 1000 of them have been lost. You can only manually write a program to find out the 1000 orders and manually send them to MQ to make up again.

### MQ is almost full
If the message is stuck in MQ and you haven't dealt with it for a long time, then the MQ is almost full, what shoud you do? Is there any other way to do this? No, who asked you to slow down the implementation of the first scheme? You write programs temporarily, access data to consume, **consume one, discard one, and do not**, and consume all messages quickly. Then go to the second plan, and fill in the data in the evening.