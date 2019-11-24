## Interview questions
In the case of a large amount of data (billions of levels), how can a query efficiency improve?

## Psychnological analysis of interviewers
This question is definitely to be asked. To put it bluntly, it is so see if you have actually done es, because? In fact, es performance is not as good as you think. Many times the amount of data is large, especially when there are hundreds of millions of data, you may find it hard to find a search, how to run a `5 ~ 10s`, pitted. The first time you searched, it was `5 ~ 10s`, but it was faster, maybe hundreds of milliseconds.

You are very embarrassed, each user's first visit will be slower, compare cards? So if you haven't played es, or you are playing demo, you are asked how easy it is to show that you are not good at es.

## Analysis of interview questions
To be honest, es performance optimization is not a silver bullet, what do you mean? That is, **don't expect to adjust a parameter by hand, you can handle all the slow performance scenarios **. Maybe there is a scene where you can change the parameters, or adjust the syntax, you can do it, but definitely not all scenarios can be like this.

### Performance optimization killer——filesystem cache
The data you write to es is actually written to the disk file. **When querying, the operating system will automatically cache the data in the disk file to `filesystem cache`.

![es-search-process](/images/es-search-process.png)

The search engine of es relies heavily on the underlying `filesystem cache`. If you give more memory to the `filesystem cache`, try to make the memory contain all the `idx segment file` index data files, then you will basically search for it. It is memory, and the performance will be very high.

How big can the performance gap be? Many of our previous tests and pressure tests, if the disk is generally a second, the search performance is absolutely second level, 1 second, 5 seconds, 10 seconds. But if you go `filesystem cache`, it is pure memory, then the performance is generally an order of magnitude higher than the disk, basically milliseconds, ranging from a few milliseconds to hundreds of milliseconds.

Here is a real case. A company es node has 3 machines, each machine looks a lot of memory, 64G, the total memory is `64 * 3 = 192G`. Each machine gives es jvm heap `32G`, then the rest is reserved for `filesystem cache` which is `32G` for each machine. The total `systemsystem cache` for the `filesystem cache ` is `32 * 3 = 96G` memory. At this time, the index data file on the entire disk occupies a total of `1T`, then the data amount of each machine is `300G`. Is this performance good? `filesystem cache` memory is only 100G, one tenth of the data can be put in memory, the other is on the disk , and then you perform the search operation, most of the operations are to go to the disk, the performance is definitely poor.

After all, you have to make es perform better. In the best case, your machine's memory hold at least half of your total data.

According to our own production environment practice experience, in the best case, only a small amount of data is stored in es, which is the index you want to search for, if the memory is reserved for `filesystem cache` 100G, then you control the index data within `100G`, so that you data is almost all searched for memory, the performance is very high, generally within 1 second.

For example, you now have a row of data, `id, name, age...` 30 fields. But if you search now, you only need to search according to the three fields `id, name, and age`. If you stupidly write all the fields of a row of data into es, it will lead to the saying that `90%` of the data is not used for searching. The result is that it takes up the space of `filesystem cache` on the es machine, and the amount of data of a single piece of data. The larger, the less data that `filesystem cache` can cache. In fact, just write **a few fields to be retrieved in es**, for example, write es `id, name, age` three fields, then you can put other field data in mysql/hbase, we generally recommend using `es+hbase` for such an architecture.

The characteristic of hbase is that **is suitable for online storage of massive data**, that is, hbase can write massive data, but do not do complicated search, do some simple operations based on id or range query...Search from es according to name and age, the result may be 20 `doc id`, then according to `doc id` to hbase to query the **complete data corresponding each `doc id`** give it back and return it to the front end.

The data written to es is preferably less than or equal to, or is slightly larger than the memory capacity of the filesystem cache of es. Then you can take 20ms to retrieve from es, and then go to hbase according to the id returned by es, check 20 data, it may take 30ms, maybe you play so, 1T data will put, every time The query is 5~10s, and the performance may be very high now, each query is 50ms.

### Data warming up
If you say that even if you follow the above scheme, the amount of data written by each machine in the es cluster is more than double the `filesystem cache`. For example, if you write 60G data to a machine, the result is `filesystem cache` is 30G, and there are still 30 G data left on the disk.

In fact, you can do **data warm-up**.

For example, for Weibo, you can put some big V, usually a lot of people's data, you can get a system in advance in the background, every other time, you own background system to search for hot data, brush to in the filesystem cache, when the user actually looks at the hot data, they search directly from the memory, very quickly.

Or e-commerce, you can view some of the most commonly viewed items, such as iphone 8, hot data ahead of the background to create a program, every 1 minute to actively visit once, brush to `filesystem cache`.

For those data that you think are hot and often have access, it is best to do a special cache preheating subsystem, that is, to access the hot data at regular intervals, let the data enter the `filesystem Go inside the cache `. This way, the next time someone else visits, the performance will be much better.

### Hot and cold separation
ES can do horizontal splitting similar to mysql, that is, write a large number of data with little access and low frequency, write an index separately, and then write a separate index of hot data that is accessed frequently. It is best to write **cold data to an index and then write the hot data to another index**, which will ensure that the hot data is left in the `filesystem os cache` as soon as it is warmed up. **Don't let cold data be washed out**.

You see, suppose you have 6 machines, 2 indexes, one for cold data, one for exothermic data, and 3 shards for each index, 3 machines radiate data index, and 3 machines put cold data index. Then, if you spend a lot of time accessing the hot data index, the hot data may account for 10% of the total data. At this time, the amount of data is very small, almost all of them are kept in the `filesystem cache`, and the hot data can be ensured. The access performance is very high. But for cold data, it is in other indexes, not on the same machine as the hot data index, and everyone has nothing to do with each other. If someone accesses cold data, a large amount of data may be on the disk. At this time, performance is almost the same, 10% of people access cold data, and 90% of people access hot data. It doesn't matter.

### Document model design

对于 MySQL，我们经常有一些复杂的关联查询。在 es 里该怎么玩儿，es 里面的复杂的关联查询尽量别用，一旦用了性能一般都不太好。

最好是先在 Java 系统里就完成关联，将关联好的数据直接写入 es 中。搜索的时候，就不需要利用 es 的搜索语法来完成 join 之类的关联搜索了。

document 模型设计是非常重要的，很多操作，不要在搜索的时候才想去执行各种复杂的乱七八糟的操作。es 能支持的操作就那么多，不要考虑用 es 做一些它不好操作的事情。如果真的有那种操作，尽量在 document 模型设计的时候，写入的时候就完成。另外对于一些太复杂的操作，比如 join/nested/parent-child 搜索都要尽量避免，性能都很差的。

### Paging performance optimization
es 的分页是较坑的，为啥呢？举个例子吧，假如你每页是 10 条数据，你现在要查询第 100 页，实际上是会把每个 shard 上存储的前 1000 条数据都查到一个协调节点上，如果你有个 5 个 shard，那么就有 5000 条数据，接着协调节点对这 5000 条数据进行一些合并、处理，再获取到最终第 100 页的 10 条数据。

分布式的，你要查第 100 页的 10 条数据，不可能说从 5 个 shard，每个 shard 就查 2 条数据，最后到协调节点合并成 10 条数据吧？你**必须**得从每个 shard 都查 1000 条数据过来，然后根据你的需求进行排序、筛选等等操作，最后再次分页，拿到里面第 100 页的数据。你翻页的时候，翻的越深，每个 shard 返回的数据就越多，而且协调节点处理的时间越长，非常坑爹。所以用 es 做分页的时候，你会发现越翻到后面，就越是慢。

我们之前也是遇到过这个问题，用 es 作分页，前几页就几十毫秒，翻到 10 页或者几十页的时候，基本上就要 5~10 秒才能查出来一页数据了。

有什么解决方案吗？
#### Deep paging is not allowed (default depth paging performance is poor)
跟产品经理说，你系统不允许翻那么深的页，默认翻的越深，性能就越差。

#### Similar to the recommended products in the app, it is continuous pulled down one page at a time
类似于微博中，下拉刷微博，刷出来一页一页的，你可以用 `scroll api`，关于如何使用，自行上网搜索。

scroll 会一次性给你生成**所有数据的一个快照**，然后每次滑动向后翻页就是通过**游标** `scroll_id` 移动，获取下一页下一页这样子，性能会比上面说的那种分页性能要高很多很多，基本上都是毫秒级的。

但是，唯一的一点就是，这个适合于那种类似微博下拉翻页的，**不能随意跳到任何一页的场景**。也就是说，你不能先进入第 10 页，然后去第 120 页，然后又回到第 58 页，不能随意乱跳页。所以现在很多产品，都是不允许你随意翻页的，app，也有一些网站，做的就是你只能往下拉，一页一页的翻。

初始化时必须指定 `scroll` 参数，告诉 es 要保存此次搜索的上下文多长时间。你需要确保用户不会持续不断翻页翻几个小时，否则可能因为超时而失败。

除了用 `scroll api`，你也可以用 `search_after` 来做，`search_after` 的思想是使用前一页的结果来帮助检索下一页的数据，显然，这种方式也不允许你随意翻页，你只能一页页往后翻。初始化时，需要使用一个唯一值的字段作为 sort 字段。