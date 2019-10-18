## Distributed system interview cannon
Some students, before that, mainly engaged in traditional industries or outsourcing projects, have always been in small companies with relatibely simple technology. They have a common problem, that is, they haven't done a lot of distributed systems. Nowadays, Internet companies generally do distributed systems. They don't do the underlying distributed system, distributed storage system Hadoop HDFS, distributed computing system Hadoop MapReduce/Spark, and distributed straming computing system storm.

Distributed business system is to divide a large system originally developed in Java into **multiple sub-systems**. Multiple sub-systems call each other to form a large system as a while. Suppose you have made an OA system, which contains permission module, employee module, leave module, financial module, and a project, which contains a bunch of modules, which will call each other and deploy one machine. Now, if you take this system apart, there are four systems: permission system, employee system, leave system and financial system, four projects, which are deployed on four machines respectively. When a request is completed, the employee system, the permission system, the leave system and the financial system are called. The four systems have completed part of the work respectibely. After the last four systems have completed the work, the request is considered to have been completed.

![simple-distributed-system-oa](/images/simple-distributed-system-oa.png)

> Spring cloud has been rising and popular in recent years. It's just popular, but it hasn't been popularized yet. At present, Dubbo is popularized, so here we mainly talk about Dubbo.

The interviewer may ask you the following questions.
### Why split the system?
- Why split the system? How to split the system? Can I split it without Dubbo? What's the difference between Dubbo and Thrift?
### Distributed service framework
- How does Dubbo work? Can I continue to communicate if the registration center is hung up?
- What serialization protocols does Dubbo support? What about Hessian's data structure? Does PB know? Why is PB the most efficient?
- What are the Dubbo load balancing strategies and high availability strategies? What about dynamic agent strategy?
- What si Dubbo's SPI thought?
- How to manage, demote, retry and timeout based on Dubbo?
- How to design the idempotence of distributed service interface (such as not repeated deduction)?
- How to ensure the order of distributed service interface requests?
- Hwo to design a Dubbo like RPC framework?

### Distributed lock
- How to design distributed locks with redis? Can we use ZK to design distributed locks? Which is the most efficient way to implement there two distributed locks?

### Distributed transaction
- Do you understand distributed transactions? How do you solve the problem of distributed transaction? What should TCC do if the network fails to connect? How to ensure the consistency of XA?

### Distributed session
- How to implement distributed seesion in cluster deployment?
