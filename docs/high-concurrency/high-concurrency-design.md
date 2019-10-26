## Interview questions
How to design a high concurrency system?

## Psychnological analysis of interviewers
To tell you the truth, if the interviewer asks you this question, you must try your best. Why? Because you don't see what is said in the JD recruitment of many companies at present, experience with high concurrency is preferred.

If you do have read talent and practical learning, and you work too high concurrent systen in the Internet company, then you do take the offer basically like taking things out of the bag, no problem. The interviewer would never ask you that, or he would be stupid.

Suppose you work in a well-known e-comerce company for too many concurrent systems, with hundreds of millions of users, billions of traffic a day, and tens of thousand or even hundreds of thousands of concurrent systems in peak period. Then people will ask you system architecture carefully. What is your system architecture? How is it deployed? how many machines are deployes? How is cache used? How does MQ work? How does the database work? It's about digging deep into how you can hundle high concurrency.

Because those who do too much concurrency must know that the system architecture that is separated from the business is all on paper. When the complex business scenario and high concurrency, the system architecture must not be so simple. It can be done with redis and MQ? Of course not. The real system architecure will be many time more complex than this simple so-called "high concurrency architecture" after matching with business.

If an interviewer asks you a question, how to design a high concurrency system? I'm sorry, but **must be because you didn't actually do too much concurrent system**. The interviewer can't stand out from your resume, so the will ask you how to design a high concurrency system. In fact, the essence is to see if you have studied and accumulated certain knowledge.

Of course, the best way is to recruit a real overemployed and concurrent buddy, but the number of such buddies is scarce, which is not easy to recruit. So maybe it's better to recruit a guy who has studied than a guy who can't do anything.

So this time you have to do a personal show, show all your knowledge about high concurrency!


## Analysis of interview questions
In fact, the so-called high concurrency, if you want ot undersatnd thisi problem, you have to start from the root of high concurrency, why there is high concurrency? Why is high concurrency so awesome?

I'd like to make it simple. At he beginning, the system was connected to the database, but when the database was supported to two or three thousand per second, it was almost over. That's why it's said that many companies, at the beginning of their work, have relatively low technology, resulting in rapid business development, sometimes the system can't bear the pressure.

Sure. Why not? If your database is carrying 5000/8000 or even tens of thousands of concurrency per second in an instant, it will surely go down, because mysql, for example, can't carry such a high concurrency at all.

So why is there a high incidence of bull force? Because more and more people are using the Internet now, many apps, websites and systems are carrying high concurrent requests, which may be thousands of concurrent requests per second in the peak period, which is normal. If it's something like double eleven, it's possible to have hundreds of thousands of concurrent per second.

So how to play with such a high concurrency and such a complex business? What's really powerful is someone who plays with high concurrency architecture in a complex business system, but you don't, so I'll tell you how to answer this question.

It can be divided into the following six points:

- System split
- Caching
- MQ
- Sub warehouse and sub table
- Separation of reading and writing
- ElasticSearch

![high-concurrency-system-design](/images/high-concurrency-system-design.png)

### System resolution
Divide a system into serveral subsystems and use Dubbo to do it. Then each system connects to a database, which is a database. Now, multiple databases can also carry high concurrency.

### Cache
Cache, cache must be used. Most of the high concurrency scenarios are ** Read more, write less**, so you can write a copy in both the database and the cache, and then read a lot of cache. After all, it's easy for redis to have tens of thousands of concurrent machines. So you can consider how to use cache to resist high concurrency in **Read scenarios that host major requests in your project**.

### MQ
MQ, must use MQ. You may still have high concurrent write scenarios, such as dozens of frequent database operations in a bussiness operation. It's orazy to add, delete, add, delete, and change. If you use redis to carry write, it's not good. People cache data, and the data will be LRU at any time. The data format is very simple, and there is no transaction support. So you have to use MySQL to use mysql. What do you do? Use MQ. A large number of write requests are poured into MQ. Queue up and play slowly. The system behind **writes slowly after consuming** and controls it within the MySQL bearing range. So you have to consider how to use MQ to write asynchronously and improve the concurrency in your project in the scenario that carries complex write business logic. It is OK for MQ single machine to resist tens of thousands of concurrency, which has been specially mentioned before.

### Sub library table
In the final database level, the requirement of high concurrency is inevitable. OK, then split a database into multiple databases and multiple databases to carry higher concurrency. Then, split a table **into multiple tables** and keep the amount of data in each table a little less, so as to improve the performance of SQL running.

### Read/write separation
Read write separation. This means that most of the time, the database may also be read more and write less. It is unnecessary to focus all requests on one database. You can build a master-slave architecture. **master data write** in, **slave database read** out, and make a read-write separation. **When there is too much reading traffic, you can **add more slave databases**.

### ElasticSearch
Elastic search, or ES for short. ES is distributed and can be expanded at will. Distributed nature can support high concurrency, because it can be expanded and machines can carry higher concurrency at any time. Then some relatibely simple operations of query and statistics class can be considered to be hosted by ES, and some operations of full-text search class can also be considered to be hosted by ES.

The above six points are basically some things that high concurrency system must do. You can consider carefully combining the knowledge mentioned before. At that time, you can systematically elaboare this part, and then what problems should be paid attention to in each part. As mentioned before, you can elaborate, indicating  that you have accumulated a little bit on this part.

To tell you the truth, after all, what you are really good at is not to understan d some technologites, or what a high concurrency system should look like. In fact, in a real complex business system, high concurrency is dozens to hundreds of times more complex than the points mentioned above. You nedd to consider: which needs to be divided into different databases and tables, which doesn't need to be divided into different databases and tables,how to join single database and single table with different databases and tables, which data should be put in the cache, which data can carry the high concurrency request. You need to complete the analysis of a complex business system, and then gradually add the transformation of the high concurrency system architecture. This process is incomparable. Complex, once done once, and done, you will be very popualr in this market.

In fact, what most companies really value is not that you have some basic architecture knowledge related to high concurrency, some technologies in the architecture, RocketMQ, Kafka, redis, Elasticsearch, and high concurrency. You oknow, it can only be a second-class talent. For a complex distributed system with hundreds of thousands of lines of code, step by step architecture, design and parctice of high concurrency architecture, this experience is invaluable. 

















