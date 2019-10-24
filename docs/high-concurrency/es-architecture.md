## Interview questions
Can you explain the principle of distributed archtecture of ES?

## Psychological analysis of interviewers
In search, Lucene is the most popular search library. A few years ago, the industry generally asked, do you know Lucene? Do you know the principle of inverted index? Now it has been out for a long time, because now many projects directly use the Luncene based distributed search engine, ElasticSearch, or ES for short.

Now distrubuted search has basically become the standard configuration of Java System in most Internet industries, especially the popular one is ES. In the post few years, when ES was not popualr, people generally used Solr. But in the past two years, most enterprises and projects began to turn to ES.

So Internet interview will definitely talk with you about distributed search engine, and also about es. If you really don't know, then you are really out.

If the interviewer asks you the first question, he will ask you generally: can you introduce the distributed architecture design of ES? Take a look at your basic understanding of distrubuted search engine architecture.

## Analysis of interview questions
The idea of ElasticSearch is distributed search engine. The bottom layer is actually based on Lucene. The core idea is to start multiple ES process instances on multiple machines to form an ES cluster.

The **basic unit of storing data in ES is index**. For example, if you want to store some order data in ES now, you should create an  index `order_idx` in ES, and all order data will be written into this index. An idex is almost the same as a table in mysql.

```
index -> type -> mapping -> document -> field。
```

Well. for a more straightforward introduction, I'll make an analogy here. But remember, don't draw an equal sign. Analogy is only for the convenience of understanding.

Index is equivalent to a table in MySQL. Type cannot be compared with MySQL. An index can have multiple types. Each type field is similar, but there are some slight differences. Let's say there is an index, the order index, which is used to store order data. For example, you can create tables in MySQL. Some orders are orders for physical goods, such as a dress and a pair of shoes. Some orders are orders for virtual goods, such as game point cards, and recharge the phone charges. Most of the fields of the two orders are the same, but a few of them may be slightly different.

Therefore, two types will be created in the order index, one is the physical goods order type, and the other is the virtual goods order type. Most of the fields of these two types are the same, and a few of them are different.

In many cases, there may be only one type in an index, but it's true that if there are multiple types in an index(**Note**, `mapping types` has been completely removed in elasticsearch 7. X, please refer to [official documents](https://github.com/elastic/elasticsearch/blob/6.5/docs/reference/mapping/removal_of_types.asciidoc) for details.) You can think that index is a category table, and each type representes a table in MySQL. Each type has a mapping. If you think a type is a specific table, index represents a type of multiple types. Mapping is the **table structure definition** of this type. When you create a table in mysql, you must define the table structure, which fields there are and what type each field is. In fact, you write a piece of data in a type in the index, which is called a document. A document represents a row in a table in MySQL. Each document has multiple fields, and each field represents the value of a field in the document.

![es-index-type-mapping-document-field](/images/es-index-type-mapping-document-field.png)

You make an index, which can be split into multiple `shards`. Each shard stores part of the data. It's good to split multiple Shards. First, **supports horizontal expansion**. For example, if you data volume is 3T, there shards, and each shard has 1T data. If the current data volume is increased to 4T, how to expand it is very simple. Rebuild an index with four shards and import the data into it. Second, **improves performance**. The data is distributed in multiple shards, i.e. multiple servers. Some operations will be executed in parallel and distributed on multiple machines, which improves throughput and performance.

Then there are actually multiple backups of the shard data, that is to say, each shard has a `primary shard`, which is responsible for writing data, but there are several `reokuca shards`. After the `primary shard` writes data, it will synchronize the data to several other `replica shards`.

![es-cluster](/images/es-cluster.png)

Through this replica scheme, each shard's data has multiple backups. If one machine goes down, it doesn't matter. There are other data copies on other machines. High availability.

In ES cluster, multiple nodes will automatically select a node as the master node. The master node is actually responsible for some management work, such as maintaining index metadata, switching primary shard and replica shard identities, etc. If the master node goes down, a new node will be selected as the master node.

If the non master node is down, the master node will transfer the identity of the primary shard on that down node to the replica shard on other machines. Then, if you fix the down machine and restart it, the master node will control to allocate the missing replica shard, synchronize the subsequent modified data and so on, so that the cluster can return to normal.

To put it more simple, if a non master node goes down. Then the primary shard on this node is gone. Well, the master will switch the replica shard corresponding to the primary shard (on other machines) to the primary shard. If the down machine is repaired, the repaired node is no longer a primary shard, but a replica shar.

In fact, the above is the basic architecture design of elasticsearch as a distributed search engine.