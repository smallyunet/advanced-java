## Interview questions
How many ways does Redis persist? What are the advantages and disadvantages of different persistence mechanisms? How is the specific underlying layer of the persistence mechanism implemented?

## Psychnological analysis of interviewers
If redis just caches the data in memory, if redis goes down and restarts, all the data in memory will be lost. You have to use redis's persistence mechanism, white writing data to memory, asynchronously slowly write data to disk files for persistence.

If redis goes down and restarts, it will automatically load some previously persisted data from the disk. A little data may be lost, but at least not all data will be lost.

This is actually the same, for some problems that may be encountered in the production environment of redis, that is, if redis is suspended and restarted, the data in memory will be lost? Can you restore the data when restarting?

## Analysis of interview questions
Persistence is mainly used for disaster recovery and data recovery,. It can also be classified as a highly available link. For example, if you redis hangs up completely, then redis will be unavailable. All you have to do is make redis available Become available as soon as possible.

Restart redis and let it provide external services as soon as possible. If no data backup is done, then redis is started and unavailable. The data is gone.

It is possible to say that a large number of requests came over, all the caches could not be hit, and no data could be found in redis. At this time, it was dead and a **cache avalanche** problem occurred. If all requests are not hit by redis, they will go to the data source such as mysql database, all of a sudden mysql will accept high concurrency, and then it will hang up...

If you make redis persistent, and your backup and recovery solutions are enterprise-level, then even if your redis fails, you can quickly restore it by backing up your data, and provide external services immediately once it is restored.

### Two ways to persist redis
- RDB：RDB persistence mechanism is to perform **periodic** persistence on the data in redis.
- AOF：The AOF mechanism writes a log for each write command, and writes it to a log file in the append-only mode. When redis restarts, it can be restarted by **playback** the write instruction in the AOF log. Build the entire data set.

Through RDB or AOF, you can persist the data in redis memory to disk, and then back up the data to other places, such as cloud services such as Alibaba Cloud.

If redis hangs, the memory on the server and the data on the disk are lost. You can copy the previous data from the cloud service, put it in the specified directory, and then restart redis. Redis will automatically persis the data files Data to recover data in memory and continue to provide external services.

If you use both RDB and AOF persistence mechanisms, when redis restarts, **AOF** will be used to reconstruct the data, because the **data in AOF is more complete**.

#### Advantages and disadvantages of RDB
- RDB will generate multiple data files, each of which represents the redis data at a certain moment. This method of multiple data files is **very suitable for cold standby**. This complete data file can be Sent to some remote secure storage, such as Amazon's S3 cloud service, which can be Alibaba Cloud's ODPS distributed storage in China, and regularly backs up data in redis with a predetermined backup strategy.
- The impact of RDB on the read an write services provided by redis to the outside is very small. Redis **maintains high performance**, because the redis main process only needs to fork a child process, and let the cchild process perform disk IO operations for RDB persistence.
- 相对于 AOF 持久化机制来说，直接基于 RDB 数据文件来重启和恢复 redis 进程，更加快速。

- 如果想要在 redis 故障时，尽可能少的丢失数据，那么 RDB 没有 AOF 好。一般来说，RDB 数据快照文件，都是每隔 5 分钟，或者更长时间生成一次，这个时候就得接受一旦 redis 进程宕机，那么会丢失最近 5 分钟的数据。
- RDB 每次在 fork 子进程来执行 RDB 快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。

#### AOF advantages and disadvantages
- AOF 可以更好的保护数据不丢失，一般 AOF 会每隔 1 秒，通过一个后台线程执行一次`fsync`操作，最多丢失 1 秒钟的数据。
- AOF 日志文件以 `append-only` 模式写入，所以没有任何磁盘寻址的开销，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。
- AOF 日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。因为在 `rewrite` log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。在创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件 ready 的时候，再交换新老日志文件即可。
- AOF 日志文件的命令通过非常可读的方式进行记录，这个特性非常**适合做灾难性的误删除的紧急恢复**。比如某人不小心用 `flushall` 命令清空了所有数据，只要这个时候后台 `rewrite` 还没有发生，那么就可以立即拷贝 AOF 文件，将最后一条 `flushall` 命令给删了，然后再将该 `AOF` 文件放回去，就可以通过恢复机制，自动恢复所有数据。
- 对于同一份数据来说，AOF 日志文件通常比 RDB 数据快照文件更大。
- AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒 `fsync` 一次日志文件，当然，每秒一次 `fsync`，性能也还是很高的。（如果实时写入，那么 QPS 会大降，redis 性能会大大降低）
- 以前 AOF 发生过 bug，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。所以说，类似 AOF 这种较为复杂的基于命令日志 / merge / 回放的方式，比基于 RDB 每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有 bug。不过 AOF 就是为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是**基于当时内存中的数据进行指令的重新构建**，这样健壮性会好很多。

### How to choose RDB and AOF
- Don't just use RDB, because that will cause you to lose a lot of data;
- Don't just use AOF, because there are two problems: First, you can perform cold standby through AOF, and the recovery speed is faster without cold standby with RDB; Second, RDB simply and crudely generates data snapshots each time, which is more robust Can avoid the bug of AOF, a complicated backup and recovery mechanism;
- Redis supports the simultaneous activation of two persistence methods. We can use both AOF and RDB persistence mechanisms, and use AOF to ensure that data is not lost, as the fist choice for data recovery. Use RDB to do cold backup at different levels You can also use RDB for fast data revoery when AOF files are lost or damaged and unavailable.
