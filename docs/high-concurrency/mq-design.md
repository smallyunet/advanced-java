## Interview questions
If you are asked to write a message queue, how to design the architecture? Say what you think.

## Psychological analysis of interviewers
In fact, when it comes to this question, the interviewers usually have to examine two parts:

- Do you have a deeper understanding of the principle of a message queue, or grasp the architecture principle of a message queue as a whole?
- Take a look at your design ability and give you a common system, that is, message queueing system. See if you can grasp the overall architecture design from the overall perspective and give some key points.

To be honest, when asking similar questions, most people are basically confused, because they never think about similar questions at ordinary times. Similar questions, for example, what would you do if you were asked to design a Spring framwork? What would you do if you were asked to design a Dubbo framework? What would you do if you were asked to design a Mybatis framework?

## Analysis of interview questions
In fact, to answer such questions, let's be clear. Don't ask you to look at the source code of that technology, at least you need to know the basic principle, core components and basic architecture of that technology, and then refer to some open-source technologies to explain the idea of designing a system.

For example, for this message queuing system, let's consider it from the following perspectives:

- First of all, this MQ needs to support scalabillty, that is, if you need to expand the capacity quickly, you can increase the throughput and capacity. What's the matter? To design a distributed system, please refer to the design concept of Kafka, broker -> topic -> partition. Each partition puts a machine and stores some data. If the resources are not enough now, simply add a partition to the topic, then do data migration and add machines, can't you store more data and provide higher throughput?

- Secondly, you have to consider whether the data of this MQ should land on the disk? That's for sure. You need to drop the disk to ensure that the data will not be lost if the process hangs. How do I land the disk? Sequential write, so there is no addressing overhead of random read and write on disk. The performance of sequential read and write on disk is very high, which is Kafka's idea.

- Thirdly, do you think about the availability of your MQ? For this matter, please refer to Kafka's high availability guarantee mechanism explained in the previous usability section. Multiple copies -> leader & follower -> broker can provide external services by re electing the leader.

- Can data 0 be lost? Yes, refer to the Kafka data zero loss scheme we mentioned earlier.

MQ must be very complicated. The interviewer asks you this question, which is actually an open question. He just wants to see if you have the thinking and ability of overall conseption and design from the perspective of architecture. It's true that this problem can wipe out a large number of people, because most people usually don't think about these things.
