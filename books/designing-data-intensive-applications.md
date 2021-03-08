## Introduction

Behind the rapid changes in technology, there are enduring principles that remain true, no matter which version of a particular tool you are using. If you understand those principles, you’re in a position to see where each tool fits in, how to make good use of it, and how to avoid its pitfalls. 

## Chapter 1: Reliable, Scalable, and Maintainable Applications

The three most important concerns for any data intensive applications are Reliability, Scalability, and Maintainability.

### Reliability
It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. Reliability is about building reliable systems from unreliable parts.

The [Netflix Chaos Monkey approach](http://techblog.netflix.com/2011/07/netflix-simian-army.html) is an example of deliberately inducing faults to ensure that the fault-tolerance machinery is continually exercised and tested, which can increase the confidence that faults will be handled correctly when they occur naturally.

Hardware Faults — Hard disks are reported as having a mean time to failure (MTTF) of about 10 to 50 years. Thus, on a storage cluster with 10,000 disks, we should expect on average one disk to die per day. Hardware errors are usually random and sometimes there could be weak correlation like rack temperature.

Software Faults — Cascading failures. Kernel bugs etc.

Human Error — Wrong configurations. Confusing design. Setup proper telemetry to deal with this.

### Scalability

<ins>Load and Performance</ins>

Load parameters, depend on the architecture of your system, could be requests per second, the ratio of reads to writes in a DB, number of simultaneously active users in a chat room, or the hit rate on a cache. 

For example, Twitter’s scaling challenge is not primarily due to tweet volume, but due to fan-out—each user follows many people, and each user is followed by many people. 

As for Performance, throughput is often relevant in case of batch processing systems like Hadoop. In online systems, **response time** is a measure of performance. We need to think of response time not as a single number, but as a distribution of values that you can measure. Because response time is also affected by a lot of random factors like a context switch to a background process, the loss of a network packet and TCP retransmission, a garbage collection pause, a page faultforcing a read from disk, mechanical vibrations in the server rack etc.

It’s common to consider average response time but that doesn’t tell you how many people experienced that delay. Talking in terms of percentile is a better idea. For example, take your list of response times and sort it from fastest to slowest, then the median is the halfway point. Median is also known as the 50th percentile (p50). To figure out your outliers are, look at higher percentiles: p95, p99, and p999 (tail latencies)

So a SLA expectations may have metrics like — p50 of less than 200 ms and a p99 of under 1s with 99.9% uptime.

It's not sufficient to measure the response time at the API level. It should be measured at the client side. Some requests maybe fast but they might be help up in queue because the server is processing the slower request (_head-of-line blocking_). Also, during testing, if the load testing client waits for a request to complete before sending new one, that behaviour has the effect of artificially keeping the queues shorter in the test than they would in reality, which skews the measurement.

<ins>Percentiles in Practice</ins>

When several backend calls are needed to serve a request, it takes just a single slow backend request to slow down the entire end-user request. So, even a very small percentage of backend calls can increase the chances of getting a slow call (_tail latency amplification_).

As for capturing the response time, you can keep a rolling window of response times of requests in the last 10 minutes. Every minute, calculate the median and various percentiles over the values in that window and plot those metrics on a graph.

Algorithms for calculating a good approximation of percentiles at minimal CPU and memory cost, are _forward decay_, _t-digest_, or _HdrHistogram_. Beware! Averaging percentiles, e.g., to reduce the time resolution or to combine data from several machines, is mathematically meaningless. The right way of aggregating response time data is to add the histograms.

The architecture of highly scalable systems is application specific. It depends on the volume of reads, the volume of writes, the volume of data to store, the complexity of the data, the response time requirements, and the access patterns.

### Maintainability

The three pillars of maintainable systems are — _Operability_, _Simplicity_, and _Evolvability_.

Good operations can often work around the limitations of bad (or incomplete) software, but good software cannot run reliably with bad operations. Operability means having good visibility into the system’s health, and having effective ways of managing it.

A software project mired in complexity is sometimes described as a [_big ball of mud_](http://www.laputan.org/pub/foote/mud.pdf)

One of the best tools for removing accidental complexity is _Abstraction_. A good abstraction can hide a great deal of implementation detail behind a clean, simple-to-understand façade.

## Chapter 2: Data Models and Query Languages

It's likely that in the foreseeable future, relational databases will continue to be used alongside a broad variety of nonrelational datastores — an idea that is sometimes called _polyglot persistence_.

The disconnect between the the RDBMS models and the application Objects is sometimes called an _impedance mismatch_.

The _hierarchical model_ of 1970s was very similar to present day _Document Data model_. It worked well for one-to-many relationships, but  many-to-many relationships were difficult, and it didn’t support joins. Developers had to decide whether to duplicate (denormalize) data or to manually resolve references from one record to another.

Various solutions were proposed to solve the limitations of the hierarchical model. The two most prominent were the relational model (which became SQL, and took over the world) and the network model (which initially had a large following but eventually faded into obscurity).

The main arguments in favor of the document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to the data structures used by the application. The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.

Which data model leads to simpler application code? It depends. For highly interconnected data, the document model is awkward, the relational model is acceptable, and **Graph-Like Data Models** are the most natural.

### Schema-on-read Vs Schema-on-write 

The term _schemaless_ often used for document databases is misleading. A more accurate term is **schema-on-read** (the structure of the data is implicit, and only interpreted when the data is read), in contrast with **schema-on-write** (the traditional approach of RDBMS, where the schema is explicit and the DB ensures all written data conforms to it).

_Schema-on-read_ is similar to dynamic (runtime) type checking in programming languages, whereas _schema-on-write_ is similar to static (compile-time) type checking. Just like the never ending debate about the merits of static and dynamic type checking, there's no right or wrong answer to the contentious topic of enforcement of schemas in database.

When the format of the data is changed, in case of RDBMS, a schema migration is required whereas for document model, the application code often handles the change. Sometimes, RDBMS can take the document model approach. Running the UPDATE statement on a large table is likely to be slow on any database, since every row needs to be rewritten. If that is not acceptable, the application can alter the schema and backfilling of data can happen during read time, like it would with a document database.

The _schema-on-read_ approach is advantageous in cases when there are many different types of objects, and it is not practical to put each type of object in its own table. Or, when the structure of data is dependent on external systems which can change anytime.

In case of document database, it's recommended that documents are kept fairly small and updates which increase the size of the document are avoided because if the endoded size of a document increases during update, the whole document has to be rewritten.

It seems that relational and document databases are becoming more similar over time. Today, many RDBMS offer the ability to index and query inside JSON documents. Similary, some Document databases support relational-like joins in its query language.

### Declararive Vs Imperative

An imperative language tells the computer to perform certain operations in a certain order. E.g. stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.

Whereas, in a declarative query language (like SQL), we just specify the pattern of the data we want — conditions (where clause) and transformation (sorting, grouping, and aggregation) — but not _how_ to achieve that goal. It's query optimizer's job to decide which indexes and which join methods to use, and in which order to execute various parts of the query.

A declarative query language is more concise and easier to work with than an imperative API. It also hides implementation details of the database engine, so that performance improvements can be introduced without requiring any changes to queries.

Finally, with declarative languages parallel execution can be leveraged. Today, CPUs are getting faster by adding more cores, not by running at significantly higher clock speeds than before. Imperative code is very hard to parallelize across multiple cores and multiple machines, because it specifies instructions that must be performed in a particular order. Declarative languages have a better chance of getting faster in parallel execution because they specify only the pattern of the results, not the algorithm that is used to determine the results. The database is free to use a parallel implementation of the query language.

In a web browser, using declarative CSS styling is much better than DOM manipulation using JavaScript. Similarly, in databases, declarative query languages like SQL turned out to be much better than imperative query APIs (CODASYL).

MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. 

The relational model can handle simple cases of many-to-many relationships, but as the connections within your data become more complex, it becomes more natural to start modeling your data as a graph. A graph consists of two kinds of objects: _vertices_ (also known as nodes or entities) and _edges_ (also known as relationships or arcs). 

Typical examples of data that can be modeled as a graph — 

**Social graphs** - Vertices are people, and edges indicate which people know each other.<br>
**The web graph** — Vertices are web pages, and edges indicate HTML links to other pages.<br>
**Road or rail networks** — Vertices are junctions, and edges represent the roads or railway lines between them.

Among several different ways of structuring and querying data in graphs, we have the **property graph model** (Neo4j, Titan, and InfiniteGraph) and the **triple-store model** (Datomic and AllegroGraph).

_Cypher_, _SPARQL_, and _Datalog_ are examples of declarative query language for Graphs and _Gremlin_ and _Pregel_ use imperative query language.

!["graph model using relational db"](https://user-images.githubusercontent.com/26899066/80941761-e9178b80-8e00-11ea-8c22-fa5bb2859167.png)<br>
👆 *Representing a property graph using a relational schema*

Graphs are good for evolvability: as you add features to your application, a graph can easily be extended to accommodate changes in your application’s data structures.

In a **triple-store** graph model, all information is stored in the form of very simple three-part statements: (subject, predicate, object). For example, in the triple (Jim, likes, bananas), Jim is the subject, likes is the predicate (verb), and bananas is the object.

### Semantic Web and RDF

The semantic web is fundamentally a simple and reasonable idea: websites already publish information as text and pictures for humans to read, so why don’t they also publish information as machine-readable data for computers to read? The Resource Description Framework (RDF) was intended as a mechanism for different websites to publish data in a consistent format, allowing data from different websites to be automatically combined into a web of data—a kind of internet-wide **database of everything**

SPARQL is a query language for triple-stores using the RDF data model. SPARQL is a nice query language—even if the semantic web never happens, it can be a powerful tool for applications to use internally.

Datalog is a much older language than SPARQL or Cypher. It provides the foundation that later query languages build upon. Datalog’s data model is similar to the triple-store model, generalized a bit. Instead of writing a triple as _(subject, predicate, object)_, we write it as _predicate(subject, object)_.

The Datalog approach requires a different kind of thinking and it’s less convenient for simple one-off queries, but it can cope better if your data is complex.

### Summary

So we see that the nonrelational **NoSQL** datastores have diverged in two main directions:
- Document databases for cases where relationships between one document and another are rare.
- Graph databases go in the opposite direction, i.e., they are good for cases where anything is potentially related to everything.

One thing that document and graph databases have in common is that they typically don’t enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements.

All three models (document, relational, and graph) are widely used today, and each is good in its respective domain.

## Chapter 3: Storage and Retrieval

In order to tune a storage engine to perform well on your kind of workload, you need to have a rough idea of what the storage engine is doing under the hood.

[Checkout the example](https://github.com/anshulkhare7/ShellUtils/tree/master/simpledb) of a very simple database using two bash function.

To efficiently find the value for a particular key in the database, we need a different data structure: an index. The general idea behind indexing structures is to keep additional metadata on the side, which acts as signpost and helps you to locate the data you want.

### Hash Indexes

Let’s say our data storage consists only of appending to a file. Then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file — the location at which the value can be found. As the data grows, our append only log file may eventually run out of space. To address it, we can break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform _compaction_ on these segments. **Compaction** means throwing away duplicate keys in the log, and keeping only the most recent update for each key.

Moreover, since compaction often makes segments much smaller (assuming that a key is overwritten several times on average within one segment, we can also merge several segments together at the same time as performing the compaction. Each segment now has its own in-memory hash table, mapping keys to file offsets. 

Appending and segment merging are sequential write operations, which are generally much faster than random writes, especially on magnetic spinning-disk hard drives. To some extent sequential writes are also preferable on flash-based solid state drives (SSDs) 

To delete a record, you append a special record (called _tombstone_) and while merging/compaction process, that record can be dropped. 

To recover from crash, checksums can be used allowing corrputed parts to be ignored.

This hash table based index has limitations like limited in memory space. 

### SSTables and LSM-Trees

In Sorted String Tables, the keys in a segment are sorted. This makes merging and compaction more efficient. Sorted order also means that the in-memory index can be sparse. One key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly.

Maintaining a sorted structure on disk is possible, but maintaining it in memory is much easier. There are plenty of well-known tree data structures that you can use, such as red-black trees or AVL trees. With these data structures, you can insert keys in any order and read them back in sorted order.

Storage engines that are based on this principle of merging and compacting sorted files are often called LSM (_Log Structured Merge-Tree_) storage engines.

The basic idea of LSM-trees is — keeping a cascade of SSTables that are merged in the background. Even when the dataset is much bigger than the available memory it continues to work well. Since data is stored in sorted order, you can efficiently perform range queries (scanning all keys above some minimum and up to some maximum), and because the disk writes are sequential the LSM-tree can support remarkably high write throughput.

### B-Trees

The most widely used indexing structure isn't LSM Tree but the B-tree. B-trees break the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time. The page contains several keys and references to child pages. Each child is responsible for a continuous range of keys, and the keys between the references indicate where the boundaries between those ranges lie. The number of references to child pages in one page of the B-tree is called the _branching factor_.

A B-tree with **n** keys always has a depth of O(log n). Most databases can fit into a B-tree that is three or four levels deep, so you don’t need to follow many page references to find the page you are looking for. (A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.)

For crash resilience,  B-tree implementations include an additional data structure on disk: a write-ahead log (WAL or a redo log).

### Comparing LSM Trees and B-Trees

As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. 

In both types of index structures, one DB write results in multiple disk writes over the course of the database’s lifetime. This is known as _write amplification_. 

LSM-trees are typically able to sustain higher write throughput than B- trees, partly because they sometimes have lower write amplification and also on magnetic drives sequential writes (which LSM tree leverages) are faster than random writes.

A downside of log-structured storage is that the compaction process can sometimes interfere with the performance of ongoing reads and writes. Especially when database becomes bigger.

In new datastores, log-structured indexes are becoming increasingly popular. There is no quick and easy rule for determining which type of storage engine is better for your use case, so it is worth testing empirically.

### Other Indexing Structures

A _clustered index_ is where the indexed row is stored along with the index key. For example, in MySQL’s InnoDB storage engine, the primary key of a table is always a clustered index, and secondary indexes refer to the primary key (rather than a heap file location).

A compromise between a clustered index (storing all row data within the index) and a nonclustered index (storing only references to the data within the index) is known as a _covering index_ or _index with included columns_, which stores some of a table’s columns within the index. This allows some queries to be answered by using the index alone.

_Fuzzy Indexes_ allow you to search using **similar** keys, such as misspelled words.

Counterintuitively, the performance advantage of in-memory databases is not due to the fact that they don’t need to read from disk. Even a disk-based storage engine may never need to read from disk if you have enough memory, because the operating system caches recently used disk blocks in memory anyway. Rather, they can be faster because they can avoid the overheads of encoding in-memory data structures in a form that can be written to disk.

An in-memory database architecture could be extended to support datasets larger than the available memory, without bringing back the over‐ heads of a disk-centric architecture. The so-called **anti-caching** approach works by evicting the least recently used data from memory to disk when there is not enough memory, and loading it back into memory when it is accessed again in the future.

### Transaction Processing or Analytics?

The data model of a data warehouse is most commonly relational, because SQL is generally a good fit for analytic queries. However, the indexing algorithms discussed so far in this chapter work well for OLTP, but are not very good at answering analytic queries.

Some databases, such as Microsoft SQL Server and SAP HANA, have support for transaction processing and data warehousing in the same product. However, they are increasingly becoming two separate storage and query engines, which happen to be accessible through a common SQL interface.

Data warehouse vendors such as Teradata, Vertica, SAP HANA, and ParAccel typically sell their systems under expensive commercial licenses. Amazon RedShift is a hosted version of ParAccel. More recently, a plethora of open source SQL-on-Hadoop projects have emerged; they are young but aiming to compete with commercial data warehouse systems. These include Apache Hive, Spark SQL, Cloudera Impala, Facebook Presto, Apache Tajo, and Apache Drill. Some of them are based on ideas from Google’s Dremel.

Most OLAP systems store the data in form of **star schema** (aka _dimensional modeling_). In this scheme, a **fact_table** at the center is surrounded by **dimension_tables**. Even date and time are often represented using dimension tables, because this allows additional information about dates (such as public holidays) to be encoded, allowing queries to differentiate between sales on holidays and non-holidays.

A variation of this template is known as the **snowflake schema**, where dimensions are further broken down into subdimensions. For example, there could be separate tables for brands and product categories, and each row in the **dim_product** table could reference the brand and category as foreign keys, rather than storing them as strings in the **dim_product** table. 

### Column-Oriented Storage

Analytic workloads are very different from OLTP. OLAP which requires sequentially scanning across a large number of rows, indexes are much less relevant. Instead it becomes important to encode data very compactly, to minimize the amount of data that the query needs to read from disk. 

The idea behind column-oriented storage is simple: don’t store all the values from one row together, but store all the values from each column together instead. If each column is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work. In each column file the rows are in the same order. 

Column oriented storage are well-suited for compression. Depending on the data in the column, different compression techniques can be used. For example, _bitmap encoding_. Operators, such as the bitwise AND and OR, can be designed to operate on such chunks of compressed column data directly. This technique is known as _vectorized processing_.

In a column store, it doesn’t necessarily matter in which order the rows are stored. It’s easiest to store them in the order in which they were inserted, since then inserting a new row just means appending to each of the column files. However, we can choose to impose an order, (like SSTables), and use that as an indexing mechanism.

Another advantage of sorted order is that it can help with compression of columns. If the primary sort column does not have many distinct values, then after sorting, it will have long sequences where the same value is repeated many times in a row. A simple run-length encoding, could compress that column down to a few KBs even if the table has billions of rows.

A clever extension of this idea is store the same data sorted in several different ways. Data needs to be replicated to multiple machines anyway, so that you don’t lose data if one machine fails. You might as well store that redundant data sorted in different ways so that when you’re processing a query, you can use the version that best fits the query pattern.

### Aggregation: Data Cubes and Materialized Views

In a relational data model, a virtual view is just a shortcut for writing queries. When you read from a virtual view, the SQL engine expands it into the view’s underlying query on the fly and then processes the expanded query. A materialized view is an actual copy of the query results, written to disk.

A common special case of a materialized view is known as a data cube or OLAP cube. It is a grid of aggregates grouped by different dimensions. Certain queries become very fast because they have effectively been precomputed. The disadvantage is that a data cube doesn’t have the same flexibility as querying the raw data. 

## Chapter 4: Encoding and Evolution


