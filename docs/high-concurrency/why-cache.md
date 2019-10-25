## Interview questions
How is caching used in projects? What are the consequences of improper use of cache?

## Psychnological analysis of interviewers
This question, Internet companies must ask, if a person even cache is not clear, it is really more embarrassing.

When it comes to caching, the first question is to ask where your project uses caching first. Why use is? Can't you? What are the possible adverse consequences if it is used?

This is to see if you have any thinking behind the cache. If you are stupid and can't give the interviewer a reasonable answer, the interviewer must have a bad impression on you. If you think too little, you know how to do the work.

## Analysis of interview questions
### How is caching used in projects?
This needs to be combined with the business of your own project.

### Why cache?
Cache is mainly used for two purposes: **high performance** and **high concurrnecy**.

#### High performance
Suppose that in such a scenario, you have an operation, a request comes, and you operate MySQL in all kinds of disorderly ways. It takes 600ms to find a result in half a day. But this result may not change in the next few hours, or it can also be changed without immediate feedback to users. So what to do at this time?

Cache, the result of the query of the tossed 600ms. Throw it into the cache. A key corresponds to a value. Next time someone checks it, don't go away from the MySQL tossed 600ms. Directly from the cache, find a value through a key, and do it in 2ms, 300 times better performance.

That is to say, for some results that need complex operation time-concuming to be found, and it is determined that they will not change much later, but there are many read requests, then it is better to directly put the results of the query in the cache and directly read the cache later.

#### High concurrence
MySQL is such a heavy database. At the end of the day, it is not designed to let you play with high concurrency. Although you can play with it, its natural support is not good. It's easy to alarm when MySQL is supported to `2000qps`.

So if you have a system with 10000 requests coming in second during the peak the peak period, a single MySQL machine will definity die. You can only cache at this time. Put a lot of data in the cache instead of MySQL. The cache function is simple. To put it bluntly, it is a `key value` type operation. The single machine supports tens of thousands of concurrency in a second esaily, and supports high concurrency so easy. The concurrent capacity of a single machine is dozens of times that of MySQL.

> Cache is memory driven, which naturally supports high concurrency.
### What are the bad consequences after using cache?
Common cache problems are as follows:
- [Cache and database write inconsistency](/docs/high-concurrency/redis-consistence.md)
- [Cache avalanche, cache penetration](/docs/high-concurrency/redis-caching-avalanche-and-caching-penetration.md)
- [Cache concurrent contention](/docs/high-concurrency/redis-cas.md)

I'll elaborate later.