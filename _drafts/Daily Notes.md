---
typora-root-url: ../../blog
---



# Daily Notes

[TOC]





# News reporting 

[Crypto company Anchorage raises $80 million after getting federal banking charter](https://techcrunch.com/2021/02/25/crypto-company-anchorage-raises-80-million-after-getting-federal-banking-charter/)



# AI Index Research 

Ref: https://aiindex.stanford.edu/report/

Chapter 1 Research: 

- Number of AI journel grew 34.5%, larger than last yr's 19.6% 
- China srupassed US in AI **journal citations** (but US still with more AI conference papers) 

Chatper 2; Technical performance:

- Generative everything (like DeepFake)
- NLP needs some new evaluation metrics. The existing NLP evaluatin metric no longer effective to measure models (they are doing so well - with human level performance) 

Chapter 3: Economy

- US recorded drop in AI jobs (300k in 2020, vs 325k in 2019) 
- Asia sees more increase in Jobs posted 
- More money (9% more money)  put into smaller number of AI startups (5% more companies)

Chapter 4: AI Education 

- 23% of CS PhD are AI focused (survey based result)
- 82% of foreign AI students stayed in the US

# Command lIne tricks 

- The use of `-I{}` to do string subs 

```
cat hosts | xargs -I{} ssh root@{} hostname
```

- `pgrep`, `pkill` :
  - Search/kill with process (or arguments with `-f` ): 
- `cut`, `paste`, `join` 
  - For text file manipulation 



# Wormhole 

Problem:

- Polling for each app is not realistic as App needs to determine the polling wait time etc
- Updates interposed on writes require custom data stores => FB has so many and error prone for each data source to write to a customized data store 

Solution HIgh level:

- Publisher reads txn logs by producers (MySQL, ZippyDB) 
- Encapsulate data in a wormhole update 



Reliability 

- [publisher failure] Multiple-copy reliable delivery: subscribers could specify multiple publisher 
- [subscriber failure] Stateful position in txn logs most recently ACKed 

Single-Copy Reliable Delivery 

- TCP realiability 
- periodicaly datamarker (30 seconds)



Update:

- Flow: combinationf of position + datasource for stream of data 
- [stable case] One reader for reading from same positions for multiple flows 
- [recovery case] Cluster flows so that a single reader could serve multiple flows => a Caravan! 
- Shared TCP connection for multiple flows to the same subscriber 



Load Balancing:

- Among subscribers:
  1. randomly selection
  2. ZooKeeper based mechanism for dynamic redirecting





Evaluation:

Latency:

- 50k updates over hours between pub and sub
- pub and sub in same data centers
- 99.5% updates delivered under 100ms
- long tail of 5 seconds 



IO vs latency tradefoff: 

- 20GB to be sent over with 10 different datamarkers (0, 2, 4, ... 18)
- more caravans allowed will reduce latency but with higher amplification factor 
- 6x read amplication could reduce as 40% of latency



Throughput:

- subs max 600k updaates/sec without latency penalty (100ms)
- pub 350MB/sec without parallelism support 



Comparison with others

Google's Thialfi:

- different workload: Google mostly user apps, but FB's internal services => I/O effi requirement higher for FB
- Thialfi only sending versions (for invaliudations most of the time) 





Question:

- What if more than once update during recovery is not allowed? 
- Is there still the limit on publisher paralleism 



# 公司

小码教育： 战略投资



# Memory Ordering 

- we as programmers expect program order
- processor aggresively reorders instructions to hide memory latency 
- orer of order R/W on different cache blocsk work on uni-processor, but not on multi-procesor 
  - easy for Uni-processor to have atomic memory accesses, in program order
- how does out-of-order work
  - processor fetches instructin in-order
  - issue out-of-order (preserving )

- issueing in-order in multi-processor could result in order-of-order reads (due to caching and interconnect delay)



- One can use syncrhonization points to allow regions where re-ordeing is allowed 
  - a properly synchronized program should yield the same result on a Sequentially consistent machine (eeven though the underlying hardware is not SC)
- Release consistency and Weak ordering ???



# Syncrhonization 

How does cache coherence handle load/stores sequence? 

- Test and Test-and-Set

- need to lock the cache line 
- This could be optimized with a `Test adnd Test-and-Set` optimization
  - Basically wrap the atomic xchg with a while loop 
- But unlocking would genreate a lots of traffic on the cache lock still 
  - So use something like a backoff 



- Ticket lock 
  - good 
  - more space
  - still traffic on reading the next_serving 

- Queue-based or List-based locks
  - generally good for high contention



LL/SC comparing to CMPXCHG/XCHG

- LL/SC could detect A->B->A problem 

# 社区

需求：

1. 邻里关系
   1. 拼单
2. 周边服务（家政、保姆、蔬菜）
3. 物业管理
   1. 报修





学生校内社交需求难以被高效满足：学生们现在主要使用的社交软件包括QQ、微信，都是建立在熟人社交的基础上。熟人社交App的一些功能特性，比如只有验证后的好友才能进行互动、转发的信息内容限制在已经通过的好友圈子里，都大大限制了社交圈子的拓展。然而，作为校园内的学生，却总是会参与到各种各样的社交场合中，并且有社交需求向外拓展自己熟人的圈子。比如上课作为学生主要的一个校园活动，现有的微信、QQ等社交App并不能很好地满足学生寻求课友的机会；又比如学校里总会有各种各样的校园活动，而现有的社交平台并不能够很好地让学生寻找到志同道合、趣味相投的小伙伴。校内信息渠道低效学校场景下每天产生的信息流是巨大的，比如就业信息、课程反馈、转专业咨询、校内趣闻等等。这些信息对于身处于学校这个紧密社群内的学生来说，也是校园生活重要的一个元素。然而，现有的主流社交平台（微信、QQ、快手、抖音等）都不能成为这些信息的有效载体。一些仅有的官方网站、公众号，又局限于本身形式的单一和互动性差的限制。其次，社群恐慌心态（指一种由患得患失所产生持续性的焦虑，得上这种症的人总会感到别人在自己不在时经历了什么非常有意义的事情）是校园内学生或多或少会面临的。这也使得学生对这些身边的校园信息会有更大的关注度。线上线下场景融合困难在校园内会经常发生各式各样的线下活动（球赛、年级大会、汇演、比赛）。这些线下的活动目前并没有很好地结合线上的场景。而线上线下场景的结合会带来更多有趣的应用场景。



竟品：

- 叮咚小区：挂了
  - 门槛设置100人上限？满足一定人数才可激活？

# J1 豁免

https://www.1point3acres.com/bbs/forum.php?mod=viewthread&tid=573315&highlight=j1

https://instant.1point3acres.com/thread/662977

https://www.1point3acres.com/bbs/thread-633686-1-1.html

https://zhuanlan.zhihu.com/p/23264190

# Dario's Post 

How Dario approach works?

1. Study qualitatively and quantitativley like a Doctor / Historian 
2. Form a high level archetypical POV as hypothesis 
3. Put them into algorithms for testing 



Dario thinks there are two things in the current status-quo:

1. the static: what exists 
2. the dynamic: timeless and universal forces that produce changes 

One should be focusing on 2 rather 1. (Or at least he is)



Having the state and the central goverment fight over relative powers is a marker for entering the Stage 6 (civil war)





# Snowflake talk 

### Snowflow data storage architecture:

- A table is partitioned into multiple files
- Each file contains multiple rows 
- A meta data object is generated for each file, including statistics (range) for each column of tuples stored in the file 
- File is not modifiable! 



An insert into the table:

- would create a new file (and meta object) for those rows inserted 



A delete/update of a row: 

- would create a new file reflecting the updated and deleted row in the file 
-  would create a new meta object reflecting that the old file containing the row is not deleted. 
- this is a rare workload pattern for OLAP actually, so this is not optimized



A select:

- would scan the meta objects
- Construct and find a set of files need to scan based on the select predicate
- This is the dominant workload, so this is optimized heavily 



### Reclustering 

The select performs best if a file contains a small range of values (like the files do not overlap with each other in range, identical and adjacent values are stored in a file) 

- Therfore a select could skip those files not meeting the predicate. 



background reclustering task based on depth(overlapping number of files at specific range )

reclusting with levels 



# TiDB



### Question

1. https://pingcap.com/blog-cn/tidb-source-code-reading-11/ Index nested loop join: why the inner worker doesn't do probing?





#### New DB Architecture

https://pingcap.com/blog-cn/new-ideas-for-designing-cloud-native-database/

- Still compute and storage seperation 
- But compute nodes with some states (like cache) 
  - -- This might result in more complex recovery mechanism as the compute node goes done 
  - ++ good for OLTP workload 
  - ++ shared most of the good things of "shared-everything" design 





### Code components:

https://pingcap.com/blog-cn/tidb-source-code-reading-2/



### TiDB Architecture

![image-20210215110410017](/img/in-post/image-20210215110410017.png)

Differences with the NoisePage:

- Use of MySQL protocol (See `server.go` as entry reference)
- There is the session context? (What's this for?)
- There is the local/distributed executor 



Lifetime of a SQL 

- `Session.Execute at session.go` : entry point for the execution 
- Parser parses to AST 
- Compile the AST node 
- Optimizer to generate `ExecStmt` (`executor/adapter.go`)
- executor encapsulated as a `recordSet`: executor + stmt + start_ts 

![image-20210215135221731](/img/in-post/image-20210215135221731.png)



###### Example: INSERT query 

<img src="/img/in-post/image-20210215142359608.png" alt="image-20210215142359608" style="zoom:50%;" />





### Parsing in TiDB 

ref: https://pingcap.com/blog-cn/tidb-source-code-reading-5/

Source code => Lex (define patterns) => Tokens

Tokens => Yacc(with grammer) => AST 



### Optimizing in TiDB 

Ref: https://pingcap.com/blog-cn/tidb-source-code-reading-6/#optimizing

Some functons:

- `dagPhysicsOptimize`:  logical to physical ocnversion 
- `convert2PhysicalPlan`: recursive does the optimzation 
- `Task` : interface that is the unit of physicalPlan to be compared and chosen 



#### Rule-based logical rules:

1. Column pruning 
   - `PruneColumns` interface implemented by all(?) operators 
2. Min-max elimination (`min_max_eliinate.go`)
   - from `scan + aggregation` to `scan + sort + limit` 
   - if there is a sorted index, this is good 
   - `NULL` value needs to be handled properly 
3. Projection elimination 
   - When: child operator producing all fields in projection 
   - When: a parent operator is using a smaller set of fields 
4. Predicate push down 
   - Try to convert outter join to inner join (so that there is no need to add NULL joined rows)
   - Push down predicate on each inner join table 
   - Certain operators could not have predicate push down (predicate push down does not conform to commutativity)
     - select then limit VS limit then select (predicate on select in this case could not be pushed down to limit) 
5. GroupBy elimination:
   - if the group by column(s) are unique, then it could be eliminated 
6. Outer join elimination:
   - outer join could be eliminated if (1 AND (2.1 OR 2.2)). 
     1. JoinOp's parent operator only uses outer plan's columns 
     2. one of the below:
        1. join key is unique in the inner plan 
        2. JoinOp's parent operator will do distinct operationon the outer plan's columns 
   - **Intuition is: if the resulting outer join's output rows are all unique, there is no point of using join right?** 
7. Nested query expansion/elimination:
   - OK, [this shit](https://pingcap.com/blog-cn/tidb-source-code-reading-21/#%E5%AD%90%E6%9F%A5%E8%AF%A2%E4%BC%98%E5%8C%96--%E5%8E%BB%E7%9B%B8%E5%85%B3) is rather complicated. I am not ready to dive into the details yet





##### IMPORTANT OP: Propogate special property to all the nodes

This is one step during optimization. 

**Unique property:** The property of some columns being Primary key indexed or unique key indexed could be propogate to all nodes to be utilized:

- Projection: if `a,b,c` being projected, and `a` is a unique indexed key, then `a` 's unique property should be propogated up (only the child nodes are aware of this)

**MaxOneRow**: only need to output one 



### Index based range filtering:

**Single column indexed (e.g., column `a`)**

- AND clauses: remove those not covered by the index 

  - `a > 1 and a < 5 and b>2` => `a > 1 and a < 5`

- OR: if any clause could not be used (e.g. with point filter on one of the columns), the entire expression could not be used. 

  - `a = 1 or b = 2` => NO 
  - `a  > 10 or (a < 2 and b = 1)` => `a > 10 or a < 2` 

  

**Multiple column indexed (`a, b, c` ):**

- AND: only if all preceding clauses are point search, the next column will be included:
  - `a > 1 and b = 1` => `a > 1` 
  - `a in (1, 2, 3) and b > 1` => `a in (1, 2,3) and b > 1`
- OR: if any of the clause (those ANDs) could not be used entirely, the entire OR chain could not be used 
  - `a > 1 and b =1 OR a in (1, 2, 3) and b > 1` => NO 



### DDL in TiDB 

- A DDL job queue for to-be-handled DDL queries, a history queue for jobs done 
  - Add index special job queue is also used to provide concurrent adding index 
- Only the owner in a cluster will be handling it. Non-owner only queues it at a DDL job queue
- DDL changes going through phases. (as per Google's F1?)













### Questions

1. What is the session context for 
2. why Hash parition only supports integers, but key based parition supports Blob and Text as well. 
3. 





# Apache Pinot 

By @kishoreBytes

Requirement:

User facing => low QPS 





##### Segments

![image-20210215165331047](/img/in-post/image-20210215165331047.png)

- segment has indepdent plan 



# Paul Graham post 

http://paulgraham.com/worked.html



Working on things less prestigious

> It's not that unprestigious types of work are good per se. But when you find yourself drawn to some kind of work despite its current lack of prestige, it's a sign both that there's something real to be discovered there, and that you have the right kind of motives. Impure motives are a big danger for the ambitious. If anything is going to lead you astray, it will be the desire to impress people. So while working on things that aren't prestigious doesn't guarantee you're on the right track, it at least guarantees you're not on the most common type of wrong one.





# TiKV 

Things to explore further: 

- Coprocessor
- Placement Driver 



### LSM Tree 

- https://github.com/google/leveldb/blob/master/doc/impl.md



### TiKV and RockDB

https://tikv.github.io/deep-dive-tikv/key-value-engine/rocksdb.html

- Use of prefix bloom filter: 
  - false positive on keys with the same prefix 
    - MVCC in TiKV and each version consits of key (for a row) + timestamp (version info). So a PBF tells u which data region MIGHT contain the key 



### Isolation level of TiKV

It uses SI (therefore it is subject to write skew anomaly): https://tikv.github.io/deep-dive-tikv/distributed-transaction/isolation-level.html#snapshot-isolation

>As a concrete example, imagine V1 and V2 are two balances held by a single person, James. The bank will allow either V1 or V2 to run a deficit, provided the total held in both is never negative (i.e. V1 + V2 ≥ 0). Both balances are currently $100. James initiates two transactions concurrently, T1 withdrawing $200 from V1, and T2 withdrawing $200 from V2.
>
>If the database guaranteed serializable transactions, the simplest way of coding T1 is to deduct $200 from V1, and then verify that V1 + V2 ≥ 0 still holds, aborting if not. T2 similarly deducts $200 from V2 and then verifies V1 + V2 ≥ 0. Since the transactions must serialize, either T1 happens first, leaving V1 = -$100, V2 = $100, and preventing T2 from succeeding (since V1 + (V2 - $200) would be -$200), or T2 happens first and similarly prevents T1 from committing.
>
>If the database is under snapshot isolation (MVCC), however, T1 and T2 operate on private snapshots of the database: each deducts $200 from an account, and then verifies that the new total is zero, using the other account value that held when the snapshot was taken. Since neither update conflicts, both commit successfully, leaving V1 = V2 = -$100, and V1 + V2 = -$200.



### Tmestamp Oracle 

- a central service that gave out timestamps in increasing order 
- the highest range of tiemstamps is persisted to the disk so recovery will continue the timestamps from that 
- requests batched to save RPC overhead (at the cost of higher txn latency) 
- Lock -> request commit timestamp -> Write, and request start timestamp -> Read, these happens before relationship make sure that a lower timestamp will guarantee READ COMMITTED 



### Service Layer 

Jobs: 

- RPC service 
- Store id => address mapping 
- TIKV communication 



### Storage Layer 

https://pingcap.com/blog-cn/tikv-source-code-reading-11/

- stores transaction and MVCC stuff 
- execution engines uses function such as `async_read` and `async_write` as API into the storage engines 
- Has raw and transactional APIs 
- There are BTree engine, RocksDBEngine (this seems to be the one used)



### Distributed Transaction 

- Cross regional transaction provided at TiDB, TiKV only suports single region transaction (client, `TiDB's store/tikv` needs to handle multiple regions )

##### The `Lock` mutation type: 

- it is used for ensure some keys not to be changed during tranx (like `SELECT .. FOR UPDATE` )

##### The `Rollback` mutation type:

- Rollback's commit_ts is the same as start_ts, so that a prewrite from the same transaction would be visible, and the prewrite will fail on write conflict.
- Rollback will happen if a txn's request timeout, and rollbacked by other transactions. 

##### Remaining  Locks resolution 

- If a lock has timeout, check the owner's status. If the owner committed, obtain its `commit_ts` to clear its other secondary locks. If the owner not committed yet, roll it back 
- lock resolution comes with the lite version and the full version:
  - lite: only clears those keys specified 
  - full: all locks in the same region with `start_ts` equal to a certain `ts`
- TiKV uses latches to lock all the keys 



##### Sequential Forward Scan

https://pingcap.com/blog-cn/tikv-source-code-reading-13/

- Keeps the Write and Lock cursor. 
- Not sure why the lock cursor check is this way....



### Coprocessor 

https://pingcap.com/blog-cn/tikv-source-code-reading-14/

##### Motivation

Push physical plan to TiKV (Storage) so that computation could be done closer to data 





#### Follwer Read 

https://pingcap.com/blog-cn/follower-read-the-new-features-of-tidb/

##### Problems:

All read requests were served by the leader, lower thoroughput by overloading the leader 



##### Solution:

Have the follower request the leader for committed index (aka, is this read request committed?), and wait until the request is committed and then apply it (returning results to the client). 

So this helps by reducing load on the leader (assuming handling request consumes more resources than handling that committed index check) 



##### Issues



# Database of Future 

#### Some large paradigm shifts:

- More in-memory DB of terabytes 

- More active-active replication 
- More NoSQL support from RDBMS
- Other data processing framework (Spark?)

> In effect, the OLTP marketplace is now becoming a main memory DBMS marketplace. Again, traditional disk-based row stores are just not competitive. To work well, new solutions are needed for concurrency control, crash recovery, and multi-threading, and I expect OLTP architectures to evolve over the next few years.
>
> My current best guess is that nobody will use traditional two phase locking. Techniques based on timestamp ordering or multiple versions are likely to prevail. The third paper in this section discusses Hekaton, which implements a state-of-the art MVCC scheme.
>
> Crash recovery must also be dealt with. In general, the solution proposed is replication, and on-line failover, which was pioneered by Tandem two decades ago. The traditional wisdom is to write a log, move the log over the network, and then roll forward at the backup site. This active-passive architecture has been shown in [[6](http://www.redbook.io/ch4-newdbms.html#ref-commandlogging)] to be a factor of 3 inferior to an active-active scheme where the transactions is simply run at each replica. If one runs an active-active scheme, then one must ensure that transactions are run in the same order at each replica. Unfortunately, MVCC does not do this. This has led to interest in deterministic concurrency control schemes, which are likely to be wildly faster in an end-to-end system that MVCC.
>
> --- [Redbook](http://www.redbook.io/ch4-newdbms.html)



#### Spark 

- Potential an interesting product (that is likely to last longer than MapReduce)
- But still lacking as a SQL/Data warehouse product 
- And lacks persistence (at 2016) 

>The original argument for Spark is that it is a faster version of Map-Reduce. It is a main memory platform with a fast message passing interface. Hence, it should not suffer from the performance problems of Map-Reduce when used for distributed applications. However, according to Spark’s lead author Matei Zaharia, more than 70% of the Spark accesses are through SparkSQL. In effect, Spark is being used as a SQL engine, not as a distributed applications platform! In this context Spark has an identity problem. If it is a SQL platform, then it needs some mechanism for persistence, indexing, sharing of main memory between users, meta data catalogs, etc. to be competitive in the SQL/data warehouse space. It seems likely that Spark will turn into a data warehouse platform, following Hadoop along the same path.
>
>On the other hand, 30% of Spark accesses are not to SparkSQL and are primarily from Scala. Presumably this is a distributed computing load. In this context, Spark is a reasonable distributed computing platform. However, there are a few issues to consider. First, the average data scientist does a mixture of data management and analytics. Higher performance comes from tightly coupling the two. In Spark there is no such coupling, since Spark’s data formats are not necessarily common across these two tasks. Second, Spark is main memory-only (at least for now). Scalability requirements will presumably get this fixed over time. As such, it will be interesting to see how Spark evolves off into the future.
>
>--- Michael Stonebraker 2015



Weak Isolation:

- It is hard but it is rare to see anomalies under weak consistency model. 



# Optimizer

Ch7: Query Opt, http://www.redbook.io/ch7-queryoptimization.html



#### The two OGs in the query optimizer world: 

- System R (DP search) 
- The Volcano/Cascade  (Top down), which are notable in 2 things:
  - Extensible (parameterizable with multiple input languages an dexecution targets, making it handy for different execution engines and data models)
  - Top-down goal oritend search strategy (not neccessirily better than bottom-up though, still debatable) 



#### Progressive Query optimizing: 

Key insight: why not optimize the query while executing it? 

- Intra-operator: based on query execution stats, stopped/started operator. 
- Inter-operator: used current operator's stats to generate operators in the future. 







# Percolator

#### Motivation 

- continously transforming a large repo of documents as new documents arrive (indexing all the pages) 
- Alternative: batch-based indexing using MapReduce framework
  - Slow: proportional to the size of all the docs 
- Alternative: traditional DBMS
  - Not scalable: petabytes of data in 



#### High level features :

- Observers: user-speicifed column changes callbacks 
- ACID transactions(multiple-row) over a random-access repo 



#### Design

- Each node consists of : 
  - percolator worker 
  - Bigtable tablet server
  - GFS
- Uses timestamp oracle and lighweight lock service 
- No central transaction management service 



### Locking: 

- stores lock in special in-meemory columns 

##### Writing multiple rows: 

- update 1st: (ACID transaction to single row providide by Bigtable)
  - add entry to the data column 
  - add primary lock to the lock column 
- update another row:
  - add entry to the data column 
  - add secondary lock to the lock column 
- Commit:
  - add to the write column update 1 for commit
  - reomve the primary lock 
  - (Optional?) add to the write column of update 2 and remove the secondary lock 



#### Evaluation:

- **Reduces High latency:**  incremental so no more batching updates necessary. (50% lower average age of docs in Google search results => more recent indexed results )
- **NOT good for large computation**: those are better for MapReduce framework, good for incremental small changes





# Postgres Internals 

Resources: 

- Internal tour PDF: https://www.postgresql.org/files/developer/tour.pdf





# Great Ideas in CS

#### Caching and Lease 

Problems to solve: 

- costly to check everytime



Real applications: 

- TiKV uses LeaseRead so that the Raft leader in a region doesn't need to check other peers while serving the Read  
- Databases optimizers use it to cache compiled query plan (so new queries don't have to go through parsing\binding\rewriting again)



#### Bloom filter

Problems to solve:

- quicker and more condensed way of finding out if something exists (or rather does not exist) 



Real Apps: 

- Bit index in data warehousing 



#### Decoupling 

Why: 

- better reusuablity 



How:

Break big things into smaller pieces 



Example:

In databases: 

- Division of query plan generation and security  checking 



#### Partitioning

Some entities contain multiple components on which operations could be parallel but have to be serial because they are treate as one. 

Example: 

- Timestamp splitting: multiple timestamps for different fields 
- Thread-local objects. 





# Information intake

Andrew Bosworth: https://fb.workplace.com/groups/ads.pages.fyi/permalink/1515588601823083/





# Questions to ask 

Daily Routine Reading List:

