## Interview questions
What are the concurrent competition problems of redis? How to solve this problem? Do you know CAS scheme of redis transaction?

## Psychnological analysis of interviewers
This is also a very common online problem, that is **multiple clients write** a key at the same time, maybe the data that should have arrived first comes later, resulting in the wrong data version; or multiple clients get a key at the same time, modify the value and then write back, as long as the sequence is wrong, the data is wrong.

Moreover, redis has its own optimistic lock scheme of CAS class to solve this problem naturally.

## Analysis of interview questions
At a certain time, multiple system instances update a key. Distributed locks can be implemented based on zookeeper. Each system obtains a distributed lock through zookeeper to ensure that only one system instance can operate a key at the same time, and no one else is allowed to read or write.

![zookeeper-distributed-lock](/images/zookeeper-distributed-lock.png)

All the data you want to write to the cache are found in MySQL. You have to write to MySQL. When you write to MySQL, you must save a time stamp. When you check from mysql, you can also find the time stamp.

Before **writing, first judge** whether the current value timestamp is never than the value timestamp in the cache. If so, it can be written. Otherwise, the old data cannot be used to cover the new data.