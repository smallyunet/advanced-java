## Interview questions
What data types are there for Redis? In which scenarios are they suitable for use?

## Psychnological analysis of interviewers
Unless the interviewer feels that you are looking at your resume, it is a relatively junior classmate who has worked for less than 3 years. There may be no in-depth study of the technology, and the interviewer will ask such questions. Otherwise, the interviewer really didn't want ask more questions during the valuable interview time.

In fact, there are two main reasons for asking this question:
- See if you have a comprehensive understanding of what features redis has, how to use it in general, what to use in the scene, I am afraid that you will not have the simplesst KV operation.
- See how you have played redis in actual projects.

If you don't answer well, don't say a few data types, and don't say any scenes. When you are finished, the interviewer will definitely not be impressed with you. I think you usually make a simple set and get.

## Analysis of interview questions
Redis mainly has the following data types:
- string
- hash
- list
- set
- sorted set

### string
This is the simplest type, the normal set and get, to do a simple KV cache.
```bash
set college szu
```

### hash
This is a kind of structure similar to map. This is generally used to cache structured data, such as an object(provided that **this object does not nest other objects**) in the redis, and then read and write the cache each time. At the time, you can operate a field in the hash.

```bash
hset person name bingo
hset person age 20
hset person id 1
hget person name
```

```json
person = {
    "name": "bingo",
    "age": 20,
    "id": 1
}
```

### list
List is an ordered list, this can play a lot of tricks.

For example, you can store some list-type data structures through lists, similar to fan lists, article comment lists, and the like.

For example, you can use the lrange command to read elements in a closed interval, and you can implement paged queries based on list. This is a great feature. Based on redis, you can implement simple high-performance paging, which can be similar to microblogging. Thing, high performance, go one page at a time.
```bash
# O start position, 01 end position, end position is -1, indicating the last position of the list, that is, view all.
lrange mylist 0 -1
```

For example, you can make a simple message queue, smash it in from the list header and get it out of the list tail.
```bash
lpush mylist 1
lpush mylist 2
lpush mylist 3 4 5

# 1
rpop mylist
```

### set
Set is an unordered collection that is automatically deduplicated.

Directly based on the set to throw the data in the system to be heavy, automatically give it a heavy weight, if you need to quickly deemphasize some data, you can of course de-duplicate based on the HashSet in jvm memory, but if is one of your systems deployed on multiple machines? It is necessary to perform global set deduplication based on redis.

You can play the intersection, union, and difference set based on set. For example, you can divide the fan list of two people into one, and see who their two friends are?  Right.

Put two big V fans in two sets and intersect the two sets.
```bash
#-------Operate a set-------
# Add element
sadd mySet 1

# View all elements
smembers mySet

# Determine if a value is included
sismember mySet 3

# Delete some elements
srem mySet 1
srem mySet 2 4

# View the number of elements
scard mySet

# Randomly delete an element
spop mySet

#-------Operation multiple sets-------
# Move a set element to another set
smove yourSet mySet 2

# Find the intersection of two sets
sinter yourSet mySet

# Find the union of two sets
sunion yourSet mySet

# Find elements in yourSet instead of mySet
sdiff yourSet mySet
```

### sorted set
A sorted set is a sorted set that is de-duplicated but can be sorted, given a score when written in, and sutomatically sorted by score.
```bash
zadd board 85 zhangsan
zadd board 72 lisi
zadd board 96 wangwu
zadd board 63 zhaoliu

# Get the top three uses (the default is ascending, so rev needs to be descended)
zrevrange board 0 3

# Get the ranking of a user
zrank board zhaoliu
```