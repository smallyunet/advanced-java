## Interview questions
How to ensure that Redis is highly concurrent and highly available? Can Redis' master-slave replication principle be introduced? Can Redis's sentinel principle be introduced?

## Psychnological analysis of interviewers
In fact, asking this question is mainly to test you, how high can the redis single machine carry? If the single machine can't help but expand, more concurrency? Will redis hang? Since redis will hang, how can I guarantee that redis is highly available?

In fact, it is aimed at some problems that you must consider in the project. If you have not considered it, then you really think too little about the problems in the production system.

## Analysis of interview questions
If you use redis caching technology, you must consider how to use redis add multiple machines, to ensure that redis is high concurrency, and how to make redis guarantee that you are not hanged and die directly, that is, redis is highly available.

Because this section has more content, it will be divided into two sections for explanation.
- [Redis master-slave architecture](/docs/high-concurrency/redis-master-slave.md)
- [Redis based on sentinel to achieve high availability](/docs/high-concurrency/redis-sentinel.md)

Redis implementation **high concurrency** mainly rely on **master-slave architecture**, on master and more slaves, in general, many projects are actually enough, single master used to write data, tens of thousands of QPS, single use To query the data, multiple slave instances can provide QPS of 10w per second.

If you want to accommodate a large amount of data while achieving high concurrency, you need a redis cluster. After using the redis cluster, you can provide hundreds of thousands of reads and writes per second.

Redis is highly available. If it is a master-slave architecture deployment, then you can implement it with a sentinel. Any instance can be used to switch between active and standby.