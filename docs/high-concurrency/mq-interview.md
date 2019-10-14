## Message Queue Interview Scenario

**Interviewer**：Hello.

**Candidate**：Hello.

（The interviewer saw that on your resume, well, there's a bright spot. You used 'MQ' in your project, for example, you used 'ActiveMQ'.）

**Interviewer**：Have you ever uesd message queues in your system? (The interviewer opened the interview in a casual tone.)

**Candidate**：Used (You feel nothing at this time.)

**Interviewer**：So how do you use message queues in your projects?

**Candidate**：blah, blah, blah, "Our system a sends a message to the queue and other systems consume sonething. For example to 'ActiveMQ' every time a new order is placed, and an inventory system int the backgroud is responsible for getting the information and updating the inventory."

(Some suudents will enter a misunderstanding here, that is, you just know and answer how you this message queue to do something with this message queue.)

**Interviewer**：So why do you use message queues? If your order system does not send a message to 'MQ', but directly calls an interface of the inventory system, then the call succeeds directly and the inventory is updated

**Candidate**：Well, yes. (For a moment, why? I haven't thought about it very carefully. The boss let it go.) Then you talked nonsense.

(The interviewer listened to you for a moment, then listened to your nonsense, and began to feel something about it. He doubt that you have thought about it before.)

**Interviewer**：So what are the advantages and disadvantages of using message queues?

(The interviewer is thinking at this point that you haven't thought much about why 'MQ' should be used in the project. So I'll ask you a little simpler. I'll ask if you've considered the advantages and disadvantages of 'MQ' before using it in the message queue.)

**Candidate**：This... It's true I haven't thought much about this problem in the ordinary time. Nonsense.

(The interviewer already feeld that you can't do it. He doesn't think much about it at ordinary times.)

**Interviewer**：`Kafka`、`ActiveMQ`、`RabbitMQ`、`RocketMQ` 都有什么区别？

(The interviewer asks you this question, that is to say, bypass the more empty topic and see directly whether you know about all kinds of 'MQ' middleware, whether you have done your homework and whether you have done any research.)

**Candidate**：We used ActiveMQ, so nothing else. The difference is not very clear...

(The interviewer feels that you're just using it blindly, and that you don't have any thinking at all. You don't think you can do it.)

**Interviewer**：So how do you ensure high availability of message queues?

**Candidate**：This... I usually simply go API calls, not very clear how the message queue is deployes...

**Interviewer**：How to ensure that news is not re-consumed? How to ensure that consumption is idempotent?

**Candidate**：What? ('MQ' is not just writing-consuming, where are so many problems?)

**Interviewer**：How to ensure the reliable transmisiion of messages? What if the message is lost?

**Candidate**：We haven't lost much information...

**Interviewer**：How to ensure the sequence of messages?

**Candidate**：Sequentially? What do you mean? Why do I want to ensure the sequence of message? Isn't it in order?

**Interviewer**：How to solve the problem of delay and expiration of message queue? What should I do when the message queue is full? The are millions of news backing for hours, how can we solve it?

**Candidate**：No, I haven't encountered these problems at ordinary times. It's just simple to use and know some functions of 'MQ'.

**Interviewer**：If you were asked to write a message queue, how would you design the architecture? Tell me your thoughts.

**Candidate**：... I'd better go.

---

This is actually an interview style of the interviewer, that is to say, the interviewer's questions are not divergent, but spread out slowly from a small point. For example, the interviewer may talk to you about high-concurrent topics, such as caching, 'MQ', etc. On this topic, **from shallow to deep, step by step**.

In fact, the above is a very typical process of technical investigation about message queue. A good interviewer must start from a point you have done, and then conduct in-depth investigation layer by layer, one by one, until the technical point is thoroughly explored and ased to the bottom.
