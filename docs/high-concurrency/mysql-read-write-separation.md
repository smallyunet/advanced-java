## Interviewe questions
Do you do MySQL read-write separation? How to realize the read-write separation of MySQL? What is the principle of MySQL master-slave replication? How to solve the delay problem of MySQL master-slave synchronization?

## Psynological analysis of interviewers
At this stage of high concurrency, it is necessary to separate reading from writing. Because in fact, most internet companies, some websites, or apps, actually read more than write less. So in this case, it is to write a master database, but eh master database hangs multiple slave databases, and then reads from multiple slave databases. Can't it support higher read concurrent pressure?

## Analysis of interview questions
### How to realize the read-write separation of MySQL?
In fact, it's very simple. It's based on the master-slave replication architecture. In short, it's to build a master database and hang multiple slave databases. Then we just write the master database and the master database will automatically synchronize the data to the slave database.

### What is the principle of MySQL master-slave replication?
The master database writes the changes to the binlog log and then after the slave database is connected to the master database, there is an IO thread in the slave database, which copies the binlog of the master database to its local location and writes it to a relay log. then there is a SQL thread in the database that will read binlog from relay log, and then execute the content of binlog log, that is, execute SQL again lcoally, so as to ensure that the data of itself and the main database are the same.

![mysql-master-slave](/images/mysql-master-slave.png)

There is a very important point here, that is, the process of synchronizing master database data from slave database is serialization, that is to say, parallel operations on master database will be executed serially on slave database. So this is a very important point. Because of the characteristics of copying from the master database and executing SQL serially from the salve database, the data from the slave database will be slower than the master database in the high concurrency scenario, which is **delayed**. Therefore, it often occurs that the data just writen to the main database may not be read. It will take tens of milliseconds, or even hundreds of milliseconds to read.

And there is another problem here, that is, if the main database suddenly goes down, and then the data is not synchronized to the slave database, some data may not exist in the slave database, and some data may be lost.

So MySQL actually has two mechanisms in this block, one is **semi synchronous replication** to solve the problem of data loss in the main database, the other is **parallel replication** to solve the problem of master-slave synchronization delay.

The so-called **semi synchronous replication**, also known as `semi sync` replication, means that after the master database writes the binlog log, it will **force** to synchronize the data to the slave database immediately. After the slave database writes the log to its local relay log, it will return an ACK to the amster database, and the master database will not consider the write operation completed until it receives an ACK from **at least one slave database**.

The so-called **parallel replication** refers to operning multiple threads from the database, reading the logs of different databases in the delay log in parallel, and then **replaying the logs of different databases in parallel**, which is library level parallelism.

### MySQL master slave synchronization delay problem(essence)
It's a small production accident that front line did deal with the online bugs caused by the master-slace synchronization delay.

This is the scene. One student wrote code logic like this. Insert a piece of data, find it out, and then update the data. At the peak of production environment, write and develop to 2000/s, at this time, the master-slave replication delay is about tens of milliseconds. Online, we will find that there are always some data every day. We expect to update some important data status, but we don't update it at the peak. Users give feedback to customer service, and customer service will give us feedback.

Through MySQL command:
```sql
show status
```
If you look at `seconds_behind_master`, you can see that the data copied from the master database is serveral ms behind.

In general, if the master-slave delay is serious, there are the following solutions:
- Sub database, a main database is divided into multiple main databases, and the write concurrency of each main database is reduced by serveral times. At this time, the master-slave delay can be ignored.
- Open the parallel replicationo supported by mysql, and multiple libraries can be copied in parallel. If we say that the write concurrency of a certain library is very high, and the write concurrency of a single library is developed to 2000/s, parallel replication is still meaningless.
- Rewrite the code, write the code students, to be careful, insert data at once query may not find.
- If it does exist, you must insert it first, query it immediately upon request, and then perform some operations in reverse immediately, and set the direct connection main database **for this query**. **Do not recommand** this method. If you do this, the meaning of separation between reading and writing will be lost.
