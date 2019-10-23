## Lucene and ES' past life and present life
Lucene is the most advanced and powerful search library. If you directly develop based on Lucene, it's very complex. Even if you write some simple functions, you have to write a lot of Java code. You need to understand the principle in depth.

Based on Lucance, Elastic Search hides the complexity of Lucene and provides a simple and easy-to-use restful API/Java API interface (in addition to API interfaces of other languages).

- Distributed document storage engine
- Distributed search engine and analysis engine
- Distributed, supporting PB level data

## Core soncept of ES
### Near Realtime
Near real time has two meanings:

- There is a small delay (about 1s) from writing data to data being searched.
- Search and analysis based on ES can reach second level.

### Cluster
A cluster contains multiple nodes. Each node belongs to a cluster that is determined by a configuration. For small and medium-sized applications, a cluster is normal at the beginning.

### Node
Node is a node the cluster, and the node also has a name, which is randomly assigned by default. The default node will join a cluster named `Elastic Search`. If you start a bunch of nodes directly, they will automatically form an elastic search cluster. Of course, a node can also form an elastic search cluster.

### Document & field
Document is the smallest data unit in ES. A document can be a customer data, a product clasification data, and an order data, which is usually represented by JSON data structure. Each type under index can store multiple documents. There are multiple fields in a document, and each field is a data field.

```json
{
    "product_id": "1",
    "product_name": "iPhone X",
    "product_desc": "IPhone",
    "category_id": "2",
    "category_name": "Electronic product"
}
```

### Index
Index contains a lot of document data with similar structure, such as commodity index. An index contains many documents, and an index represents a similar or the same kind of document.

### Type
Type: there can be one or more types in each index. Type is a logical classification of index. For example, there are multiple types under commodity index: daily chemical commodity type, electrical commodity type and fresh commodity type. The fields of documents under each type may be different.

### shard
A single machine can not store a large amount of data. ES can split the data in an index into multiple shards, which are stored on multiple servers. With shard, you can scale horizoritaily, store more data, distribute search and analysis operations to multiple servers, and improve throughput and performance. Each shard is a Lucene index.

### replica
Any server may fail or go down at any time, and shards may be lost, so multiple replica copies can be created for each shard. Replica can provide backup service in case of shard failure to ensure data is not lost. Multiple replicas can also improve the throughput and performance of search operations. Primary shard (set at one time during index building, can't be modified, default is 5), replica shard (modify quantity at any time, default is 1), default is 10 shards per index, 5 primary shards, 5 replica shards, the minimum highly available configuration, which is 2 servers.

Let's just say that shard is divided into primary shard and replica shard. The primary shard is generally referred to as shard, and the replica shard is generally referred to as replica.

![es-cluster-0](/images/es-cluster-0.png)

## ES core concept VS DB core concept
| es | db |
|---|---|
| index | Data base |
| type | Data sheet |
| docuemnt | One raw of data |

The above is a simple analogy.