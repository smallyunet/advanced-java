## Message Queue Interview Scenario

**Interviewer**：Hello

**Candidate**：Hello

（The interviewer saw that on your resume, well, there's a bright spot. You used 'MQ' in your project, for example, you used 'ActiveMQ'.）

**Interviewer**：Have you ever uesd message queues in your system? (The interviewer opened the interview in a casual tone)

**Candidate**：Used (I feel nothing at this time.)

**Interviewer**：So how do you use message queues in your projects?

**Candidate**：blah, blah, blah, "Our system a sends a message to the queue and other systems consume sonething. For example to 'ActiveMQ' every time a new order is placed, and an inventory system int the backgroud is responsible for getting the information and updating the inventory."

(Some suudents will enter a misunderstanding here, that is, you just know and answer how you this message queue to do something with this message queue.)

**Interviewer**：So why do you use message queues? If your order system does not send a message to 'MQ', but directly calls an interface of the inventory system, then the call succeeds directly and the inventory is updated

**Candidate**：Well, yes. (FOr a moment, why? I haven't thought about it very carefully. The boss let it go.) Then you talked nonsense.

(The interviewer listened to you for a moment, then listened to your nonsense, and began to feel something about it. He doubt that you have thought about it before.)

**Interviewer**：So what are the advantages and disadvantages of using message queues?

(The interviewer is thinking at this point that you haven't thought much about why 'MQ' should be used in the project. So I'll ask you a little simpler. I'll ask if you've considered the advantages and disadvantages of 'MQ' before using it in the message queue.)

**Candidate**：This... It's true I haven't thought much about this problem in the ordinary time. Nonsense.

(The interviewer already feeld that your buddies can't do it. He doesn't think much about it at ordinary times.)

**Interviewer**：`Kafka`、`ActiveMQ`、`RabbitMQ`、`RocketMQ` 都有什么区别？

（面试官问你这个问题，就是说，绕过比较虚的话题，直接看看你对各种 `MQ` 中间件是否了解，是否做过功课，是否做过调研）

**Candidate**：我们就用过 `ActiveMQ`，所以别的没用过。。。区别，也不太清楚。。。

（面试官此时更是觉得你这哥儿们平时就是瞎用，根本就没什么思考，觉得不行）

**Interviewer**：那你们是如何保证消息队列的高可用啊？

**Candidate**：这个。。。我平时就是简单走 API 调用一下，不太清楚消息队列怎么部署的。。。

**Interviewer**：如何保证消息不被重复消费啊？如何保证消费的时候是幂等的啊？

**Candidate**：啥？（`MQ` 不就是写入&消费就可以了，哪来这么多问题）

**Interviewer**：如何保证消息的可靠性传输啊？要是消息丢失了怎么办啊？

**Candidate**：我们没怎么丢过消息啊。。。

**Interviewer**：那如何保证消息的顺序性？

**Candidate**：顺序性？什么意思？我为什么要保证消息的顺序性？它不是本来就有顺序吗？

**Interviewer**：如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？

**Candidate**：不是，我这平时没遇到过这些问题啊，就是简单用用，知道 `MQ` 的一些功能。

**Interviewer**：如果让你写一个消息队列，该如何进行架构设计啊？说一下你的思路。

**Candidate**：。。。。。我还是走吧。。。。

---

这其实是面试官的一种面试风格，就是说面试官的问题不是发散的，而是从一个小点慢慢铺开。比如说面试官可能会跟你聊聊高并发话题，就这个话题里面跟你聊聊缓存、`MQ` 等等东西，**由浅入深，一步步深挖**。

其实上面是一个非常典型的关于消息队列的技术考察过程，好的面试官一定是从你做过的某一个点切入，然后层层展开深入考察，一个接一个问，直到把这个技术点刨根问底，问到最底层。