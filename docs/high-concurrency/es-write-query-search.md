## Interview questions
How does ES write data? How does ES query data work? What about Lucene at the bottom? Do you understand the inverted index?

## Psychnological analysis of interviewers
Ask this, in fact, the interviewer is to see if you understand some basic principles of ES, because using ES is nothing more than writing data and searching data. If you don't undersatnd what ES is doing when you initiate a write and search request, then you are...

Yes, ES is basically a black box. What else can you do? The only thing you can do is read and write data with ES API. If something goes wrong and you don't know anything, what else can you expect?

## Analysis of interview questions
### ES data writing process
- The client selects a node to send the request. This node is the `coordinating node`.
- `coordinating node` **routes** the document and forwards the request to the corresponding node(with primary shard).
- The `primary shard` on the actual node processes the request and synchronizes the data to the `replica node`.
- If the `primary node` and all `replica nodes` are found to be completed, the response result will be returned to the client.

![es-write](/images/es-write.png)

### ES data reading process
You can query by `doc id`. You will has according to `doc id`, and judge which shard you assigned `doc id` to, and query from that shard.

- The client sends a request to **any node and becomes `coordinate node`.
- `coordinate node` hashes the `doc id` and forwards the request to the corresponding node. This will use the `round-robin` **random polling algorithm**, in the `primary shard` and all its repllicas. Choose one randomly to load balance the read request.
- The node receiving the request returns document to `coordinate node`.
- `coordinate node` returns document to the client.

### ES search data process
The most powerful thing is to do full-text search, that is, for example, you have three pieces of data:
```
Java is really fun
Java is hard to learn
J2EE special cow
```

You search according to the `java` keyword and search for `document` containing `java`. ES will return to you java is really fun, java is hard to learn.

- The client sends a request to a `corrdinate node`.
- The coordinating node forwards the search request to the `primary shard` or `replica shard` corresponding to all **shards**.
- Query phase: Each shard returns its own search results(in fact, some `doc id`) to the coordination node. The coordination node performs data merging, sorting, paging, etc. to produce the final result.
- Fetch phase: The coordination node then pulls the actual `document` data from each node according to `doc id` and finally returns it to the client.

> Write requests are written to the primary shard and then synchronized to all replica shards; read requests can be read from the primary shard or replica shard using a random polling algorithm.

### The underlying principle of writing data

![es-write-detail](/images/es-write-detail.png)

First write to the memory buffer, the data is not searchable in the buffer, at the same time, the data is written to the translog log file.

If the buffer is almost full, or until a certain time, the memory buffer data `refresh` will be added to a new `segment file`, but at this time the data does not directly enter the `segmeng file` disk file, but enters `os cache` first. This process is `refresh`.

Every 1 second, ES writes the data in the buffer to a **new** `segment file`, which produces a new **disk file every second** `segment file`, this `segment file` stores the data written in the buffer in the last 1 second.

However, if there is no data in the buffer at this time, of course, the refresh operation will not be execution. If there is data in the buffer, the refresh operation is executed by default in 1 second, and it is brushed into a new segment file.

In the operating system, the disk file actually has a thing, called `os cache`, that is, the operating system cache, that is, before the data is written to the disk file, it will first enter the `os cache`, first enter the memory cache at the operating system level go with. As long as the data in `buffer` is flushed into `os cache` by the refresh operation, this data can be searched.

Why is ES **quest-real-time**? `NRT`, the full name `near real-time`. The default is to refresh every 1 second, so ES is quasi-real-time because the data written can only be seen after 1 second. You can perform a refresh operation by using ES `restful api` or `java api`, **manual**, which is to manually flash the data in buffer into `os cache` so that the data can be searched immediatey. As long as the data is entered into `os cache`, the buffer will be emptied, because there is no need 10 keep the buffer, and the data has been persisted to disk in translog.

Repeat the above steps, the new data continues to enter buffer and translog, and constantly write `buffer` data to a new `segment file`, every time `refresh` is finished buffer clear, translog is reserved. As this process progresses, the translog will become larger and larger. When the tranlog reaches a certain length, the `commit` operation is triggered.

The first step in the commit operation is to empty the buffer from the existing data `refresh` in the buffer to `os cache`. Then, write a `commit point` to the disk file, which identifies all the `segment file` corresponding to the `commit point`, and forcibly `fsync` all the data in the `os acche` to the disk file. Finally **empty** existing translog log file, restart a translog, the commit operation is complete.

This commit operation is called `flush`. The default `flush` is executed once in 30 minutes by default, but `flush` is also triggered if the translog is too large. The flush operation corresponds to the whole process of commit. We can manually execute the flush operation through ES api and manually flash the data fsync in the os cache to disk.

What is the role of the translog log file? Before you perform the commit operation, the data either stays in the buffer or stays in the os cache. Both the buffer and the os cache are memory. Once the machine dies, the data in the memory is lost. Therefore, the opeartion corresponding to the data needs to be written into a special log file `translog`. Once the machine is down and restarted again, ES will automatically read the data in the translog log file and restore to the memory buffer and os cache. Go in.

Translog is actually written to the os cache first. It is brushed to the disk every 5 seconds by default, so by default, there may be 5 seconds of data that will only stay in the os cache of the buffer or translog file. Hanging up, will **lose** 5 seconds of data. But this performance is better, with up to 5 seconds of data lost. You can also set translog so that each write must be directly `fsync` to disk, but performance will be much worse.

In fact, you are here, if the interviewer does not ask you the problem of losing data, you can give the interviewer a dazzling here, you said, in fact, the first is real-time, the data can be searched after writing for 1 second may lose data. There is 5 seconds of data, stay in the bufferm translog os cacche, segment file os cache, not on the disk, if it is down, it will result in 5 seconds of **data loss**.

**Summary**, the data is first written to the memory buffer, and then every 1s, the data is refreshed to the os cache, and the os cache data can be searched (so we say that ES can be searched from writing to There is a delay of 1s in the middle). Every 5s, the data is written to the translog file (so if the machine is down, the memory data is not available, there will be up to 5s of data loss), the translog is large enough, or the default every 30mins, the commit operation will be triggered, the buffer will be buffered. The data for the zone is flushed to the segment file disk file.

> After the data is written to the segment file, the inverted index is created.

### Delete/updata the underlying principle of data
If it is a delete operation, the commit will generate a `.del` file, which identifies a doc as `deleted` state, then the search will know if the doc has been deleted according to the `.del` file.

If it is an update operation, the original doc is identified as the `deleted` state, and then a new data is written.

Buffer Every refresh, it will produce a `segment file`, so by default it is a `segment file` for 1 second, so there will be more and more `segment file`, and merge will be executed periodically. Each time emrge, multiple `segment file` will be mergerd into one, and the doc identified as `deleted` will be **physically deleted**, and the new `segment file` will be written to disk. This will write a `commit point`, identify all new `segment file`, then open `segment file` for search and delete the old `segment file`.

### Buttom lucene
In a nutshell, Lucence is a jar package that contains a variety of packaged algorithmic code for building inverted indexes. When we develop with Java, we introduced the lucence jar and then developed it based on the lucene api.

With lucene, we can index existing data, and lucene will organize the data structure of the index on the local disk.

### Inverted index
In a search engine, each document has a corresponding document ID, and the document content is represented as a collection of keywords. For example, document 1 is segmented and 20 keywords are extracted, each of which records its occurrence and appearance in the document.

Then, the inverted index is the mapping of the **keyword to the document** ID. Each keyword corresponds to a series of files, and keywords appear in these files.

Give a chesnut.

Have the following documents:

| DocId | Doc |
|---|---|
| 1 | 谷歌地图之父跳槽 Facebook |
| 2 | 谷歌地图之父加盟 Facebook |
| 3 | 谷歌地图创始人拉斯离开谷歌加盟 Facebook |
| 4 | 谷歌地图之父跳槽 Facebook 与 Wave 项目取消有关 |
| 5 | 谷歌地图之父拉斯加盟社交网站 Facebook |

After segmentation of the document, the following **inverted index** is obtained.

| WordId | Word | DocIds |
|---|---|---|
| 1 | 谷歌 | 1,2,3,4,5 |
| 2 | 地图 | 1,2,3,4,5 |
| 3 | 之父 | 1,2,4,5 |
| 4 | 跳槽 | 1,4 |
| 5 | Facebook | 1,2,3,4,5 |
| 6 | 加盟 | 2,3,5 |
| 7 | 创始人 | 3 |
| 8 | 拉斯 | 3,5 |
| 9 | 离开 | 3 |
| 10 | 与 | 4 |
| .. | .. | .. |

In addition, the practical inverted index can also record more information, such as document frquency information, indicating how many documents in the document collection contain a word.

Then, with an inverted index, the search engine can easily respond to user queries. For example, the user enters the query `Facebook`, and the search system searches for the inverted index, and reads the document containing the word, which is the search result provided to the user.

Pay attention to two important details of the inverted index:

- All terms in the inverted index correspond to one or more documents;
- Words in the inverted index **are sorted in ascending order according to lexicographic order**

> The above is just a simple chesnut, not in strict lexicographic order.