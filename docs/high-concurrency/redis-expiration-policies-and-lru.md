## Interview questions
What are the redis expliration policies? What are the memory elimination mechanisms? Handwritten LRU code implementation?

## Psychnological analysis of interviewers
If you don't even know the question, you will be embarrassed when you come up, and you can't answer it. When you write the code online, you will take it for granted that the data written into redis will exist, and the system will cause various bugs. Who will be responsible for it?

There are two common problems:
- Why is the data written to redis gone?

Some students may encounter that redis in the production envirnoment often throws away some data and writes it in. After a while, it may be gone. My God, classmate, you ask this question to explain redis, you are useless. Redis is the cache, you give it storage right?

Howling cache? Use memory as a cache. Is memory infinite? Memory is very valuable and limited. Disks are cheap and large. Maybe a machine has dozens of G's of memory, but there can be a few T of hard disk space. Redis is based on memory for high-performance, high-concurrency read and write operations.

Since the memory is limited, for example, redis can only use 10G. If you write 20G data in it, what will it do? Of course, 10G of data will be destroyed, and then 10G of data will be retained. What data did you kill? What data is retained? Of course, it is to kill the data that is not commonly used, and keep the commonly used data.

- The data is clearly out of date, how is it still taking up memory?

This is determined by redis' expiration policy.

## Analysis of interview questions
### Redis expiration policy
The redis expiration plicy is: **Delete regularly + lazy delete**.

The so-called **periodic deletion** means that redis defaults to randomly extracting some keys with expiration time every 100ms, checking whether it expires, and deleting if it expires.

Assuming 10w keys are placed in redis, the expiration time is set. You check 10w keys every few every few hundred milliseconds, then redis is basically dead, cpu load will be very high, and it will be consumed in your check expiration key. Up. Note that instead of traversing all the keys that set the expiration time every 100ms, it is a performance **disaster**. In fact, redis is randomly selected every 100ms some keys to check and delete.

But the problem is that regular deletions can cause a lot of expired keys to be deleted and not deleted. So it lazy. That is to say, when you get a key, redis will check if the key expires if the expiration time is set. If it expires, it will be deleted at this time and will not return anything to you.

> When you get the key, if the key has expired at this time, it will be deleted and nothing will be returned.

But in fact, this is stil a problem. If you delete a lot of expired keys on a regular basis, and then you don't check it in time, you will not take the lazy deletion. What happens at this time? If a large number of expired keys are piled up in memory, causing the redis memory block to run out, what?

The answer is: **Take the memory elimination mechanism**.

### Memory elimination mechanism
The redis memory elimination mechanism has the following：
- noeviction: When the memory is not enough to accommodate the newly written data, the new write operation will report an error. This is generally not used, it is really disgusting.
- **allkeys-lru**: When there is not enough memory to hold the newly written data, remove the least recently used key in the **keyspace** (this is the most commonly used).
- allkeys-random: When the memory is not enough to accommodate the newly written data, in the **keyspace**, randomly remove a key, this is fenerally no one to use, for random ,it must be the least recently used key get rid of it.
- volatile-lru: When there is not enough memory to accommodate the newly written data, remove the least recently used key in the **keyspace with** set expiration time (this is generally not appropriate).
- volatile-random: When the memory is not enough to hold the newly written data, **randomly removes a key in the key space** where the expiration time is set.
- volatile-ttl：When there is not enough memory to accommodate the newly written data, in the key space **where the expiration time is set**, the key with the **earlier expiration time** is preferentially removed.

### Handwriting an LRU algorithm
You can handwrite the original LRU algorithm on the spot, the amount of code is too large, it seems not realistic.

I don't want to create my own LRU from the ground up, but at least I need to know how to implement a Java version of LRU using the existing JDK data structure.

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;

    /**
     * How much data can be cached in
     *
     * @param cacheSize Cache size
     */
    public LRUCache(int cacheSize) {
        // True means that the linkedHashMap is sorted in order of access, with the most recent access in the header and the oldest in the tail.
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // When the amount of data in the map is greater than the specified number of caches, the oldest data is automatically deleted.
        return size() > CACHE_SIZE;
    }
}
```