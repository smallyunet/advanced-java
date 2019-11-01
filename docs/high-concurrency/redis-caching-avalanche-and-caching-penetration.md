## Interview questions
What is the avalanche, penetration and breakdown of redis? What happens when redis crashes? How should the system deal with this situation? How to deal with redis penetration?

## Psychnological analysis of interviewers
In face, it's necessary to ask about cache, because the avalanche and penetrationo of cache are the two biggest problems of cache, or they don't appear. Once they appear, they are fatal problems, so the interviewer will ask you.

## Analysis of interview questions
### Cache avalanche
For system A, suppose that there are 5000 requests per second at the peak of each day, and the cache can handle 4000 requests per second at the peak of each day, but the cache machine unexpectedly goes down completely. The cache is hung. At this time, 5000 requests fall into the database in one second. The database will not be able to carry them. It will give an alarm and then hang up. At this time, if no special scheme is adopted to deal with this fault, DBA is in a hurry to restart the database, but the database is immediately killed by the new traffic.

This is the cache avalanche.

![redis-caching-avalanche](/images/redis-caching-avalanche.png)

About three years ago, a well-known Internet company in China suffered from an avalanche and background system crash due to a cache accident. The accident lasted from the afternoon of the same day to 3-4 a.m. In the evening, and the company lost tens of millions.

The solution to the problem is as follows.
- In advance: redis is highly available, master-slave + sentinel, redis cluster, to avoid total crash.
- In the process: loca echcache + hystrix current limit & degradation to avoid MySQL being killed.
- Afterwards: redis is persistent. Once it is restarted, it will automatically load data from the disk and quickly recover the cached data.

![redis-caching-avalanche-solution](/images/redis-caching-avalanche-solution.png)

The user sends a request. After receiving the request, system a checks the local echcahe cache first, and then redis if it is not found. If neither ehcache nor redis exist, check the database again, and write the results in the database to ehcache and redis.

The current limiting component can set the requests per second, how many can pass the component, and what about the remaining failed requests? **Go down**! You can return some default values, or friendly hints, or blank values.

Benefits:
- The database will never die. The current limiting component ensures that only a few requests can pass each second.
- As long as the database is immortal, that is to say, for users, 2/5 of the requests can be processed.
- As long as 2/5 of the requests can be processed, it means that you system is not dead. For users, it may be that you can't swipe the page after clicking several times, but you can swipe it one after clicking several times.

### Cache penetration
For system A, suppose 5000 requests per second, and 4000 of them are malicious attacks by hackers.

The 4000 attacks sent by hackers can't be found in the cache. Every time you go to the database, you can't find them either.

Here's a chestnut. The database ID starts from 1. As a result, all the request IDS sent by hackers are negative. In this way, there will not by any in the cache. Every time a request is "**cached in nothing**", it queries the database directly. The cache penetration of thiss malicious attack scenario will directly kill the database.

![redis-caching-penetration](/images/redis-caching-penetration.png)

The solution is very simple. Every time system a fails to find it in the database, it writes a null value to the cache, such as 'set-999 unknown'. Then set an expiration time, so that the next time the same key is accessed, the data can be fetched directly from the cache before the cache expires.

### Cache breakdown
Cache breakdown means that a key is very hot and frequently accessed. In the case of centralized high concurrent access, when the key fails, a large number of requests break through the cache and directly request the database, just like a hole in a barrier.

The solution is also simple: you can set the hotspot data to never expire; or you can implement the mutual exclusive lock based on redis or zookeeper, wait for the first request to build the cache, then release the lock, and then other requests can access the data through the key.
















