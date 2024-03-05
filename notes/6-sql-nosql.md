# SQL AND NO-SQL DATABASE

## SQL Database

It is a single machine database by design, meaning it has no partitioning guaranteeing availability and consistency both at a time.

Table is collection of logical data points. Column represents attributes of tha data/entity and rows the actual data.

1. Data is going to follow a fixed schema.
2. SQL assumes that data is going to be normalized. The more we normalize more we need to use JOINS.
3. All transaction have ACID properties.

>ACID
>* Atomic - 0% or 100% of a transaction will be done.
>* Consistency - DB state remains consistent before and after transactions. 
>* Isolation - No one transaction interferes another transactions. All translations take place in complete isolation.
>* Durable - Data one committed is never going to be lost.

#### Pros of SQL:

If your underlying data has fixed schema, SQL allows to store data in normalized manner:
1. Storage space is not wasted—storing only fixed attributes
2. Allows performing JOINs relatively optimally knowing the JOINs are expensive.
3. Provides logN row access.

#### Shortcoming of SQL:

If your underlying data does not follow fixed schema, then a lots of cons emerge:

SQL is built with the following assumptions:
* SQL is a single machine DB
  * Thus provides feature of joins (normalization)
  * ACID guarantees

A single machine cannot store data more than 4TB.

**How to store a huge amount of data?**

SQL to support huge data by sharding—adding more machines.
Usually, SQL does not come inbuilt sharding.

For SQL to support the huge data by sharding meaning, rather than employing one machine, we will need to employee more than one machines.
If we need to run SQL as a sharded database as a cluster, we'll need to write the wrapper for consistent hashing logic, run the reverse proxy etc.

ACID guarantees were provided out of a single box. Transaction on data which sits on a single machine only such transaction will give ACID guarantee.
Intra shard SQL transaction will give you ACID but Inter shard will not provide ACID.

Transaction on one shard should not affect another to provide ACID guarantee. In case of inter shard transaction we might need costly locks across shards to avoid interference of transactions.

In case of sharded SQL, the best way would be to assume that every shard behaves like an independent SQL database.

If JOINs have become super costly (joins across shards are indeed costly) so we need to compromise normalization. Instead we need to replicate data.
This is known as de-normalization.

| SQL                                                                                 | No SQL                                                  |
|-------------------------------------------------------------------------------------|---------------------------------------------------------|
| Structured Data                                                                     | Semi-structured, no structured data                     |
| Out of box, it is a single machine DB                                               | out of box are cluster database                         |
| For single machine we normalize our data, if in cluser then we need to de-normalize | De-normalization needs to be done as running as cluster |
| Normalized - ACID guarantees, demoanlized - No ACID guarantees                      | De-normalized hence no ACID guarantees                   |

SQL to NoSQL is just a spectrum, depending on how we use it we define how structured or non-structured it is.
Use SQL for SQL use-cases and NoSQL for noSQL use-cases.

## No SQL Database

Anything that is not a SQL is a NoSQL.

* Using semi-structured or no structured data
* By design, it will be a horizontally sharded database

### Types of No SQL Database
1. `Key-value No SQL database` - Redis, Memcached

Think of hashmap at scale. Each shard will store key-value pairs.

How to decide the sharding key? The key of the data itself will be sharding key.

Example, storing Aadhaar number (key) and (values) name, pincode, gender etc. This key also acts as sharding key which help us find the machine where the data is stored.
Whenever a request comes up we take the key and locate the machine where the values are stored. 
The value can be anything, there is no limit on it.

How to store the values with no limit? - in future lectures

It can come up as a great cache since we have key values.

It will follow the same concepts of sharding and replication.

The key-values or data is store din primary memory or secondary memeory depends on the database you use example, Redis stores everything in the cache.
Redis stores in secondary memory only for persistence. 

2. `Document DB` - MongoDB, CouchDB, ElasticSearch

It Is just like a key-value store but a mature version of it since it has more power.

It stores data as documents. Every document has document_id and value document example json document. It looks similar to key-value, key is document_id and value is a json document.

Example - Amazon Product DB
```
107: {name: cat toy, price: 100, color: Blue + orange, category: Pets Toy, Pets, Toys, seller: xyz}
220: {name: lenevo laptop, price: 80000, category: computers, laptop, lenevo, hdd: 500GB, RAM: 8GB...}
```

Sharding key is going to be document_id.

We can create indexes, query documents not only based on the document_id but also based on document attributes.
Meaning, we can query the documents based on the price attributes even though many documents might not have price attributes.

In key-value databases given the values provided are just one value we cannot query inside the values. 

How will something like this be handled in reality?
 - On top of key-value pairs, the document db allows creating indexes to make the queries faster.

create replicas and de-normalize data to make operations faster.

They allow queries and also indexes to fasten these queries on internal attributes of documents.

Documents are arranged in collections, tags, metadata, directories. The more grouping info we provide the faster the queries become.

3. `Wide Column Database/ Column Family Databases` - HBase, Cassandra

This is closest to NoSQL database to SQL DB.

We can have multiple column families and each family can have multiple columns.

```
Example facebook messenger:

Column Familes {columns}

Messages {receiver_id, message, isOpened}

# things we show on page of a conversations list
Conversations {last_msg-timestamp, read, last_msg}

```

The column families are static/ fixed, but what we store inside column families can vary.

A wide column DB has column families defined.
All column families is a collection of variable columns not fixed.
We can get the value of any column based on Row Key, this key also acts as the sharding key
