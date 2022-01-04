---
typora-root-url: ../../blog
---



# Daily Notes

[TOC]



## Memory Ordering 

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



## Syncrhonization 

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



# J1 豁免

https://www.1point3acres.com/bbs/forum.php?mod=viewthread&tid=573315&highlight=j1

https://instant.1point3acres.com/thread/662977

https://www.1point3acres.com/bbs/thread-633686-1-1.html

https://zhuanlan.zhihu.com/p/23264190



Statement of Reason 

To whom it may concern,
This letter is to request a waiver for the 2-year home residence requirement based on the 'No Objection Letter' that will be obtained from my home country (P.R. China). The reasons I would like to apply for a waiver are the followings:
I came to the U.S. as a J-1 exchange scholar in 2017 during my overseas exchange program at my college, the National University of Singapore. As a J-1 holder, I was fortunate to work for a Silicon Valley-based start-up Naya Health, and learned a great deal about entrepreneurship and technology. 

After that, in 2019, as an F-1 student, I started my Master of Science in Computer Science at Carnegie Mellon University and will be graduating in December 2020. I have obtained my Post-completion OPT and will start my practical training as a Production Engineer at Facebook Inc. Facebook, as my employer, will sponsor me during the optional practical training period, helping me acquire industrial experience.

I will apply for the No Objection Letter and my embassy will directly provide you with the `No Objection Letter` issued for me.
I appreciate your support and assistance in waiving my two-year home country residence requirement.
Sincerely,

XU Chen

12/01/2020



06/18/2016 - 08/15/2016: F-1 status (N0017000058), Stanford Unviersity summer program

08/15/2019 - Now: F-1 status (N0030638519), Carnegie Mellon University Master of Science in Computer Science



尊敬的驻美总领馆：

按照《J-1签证豁免申请办法（修订稿）》的要求，特此提出豁免本人的J-1两年回国服务要求的申请。本人于2019年5月获新加坡国立大学计算机学士文凭，2019年8月自费赴美学习至今。本人在2017年1月至2017年12月参加了新加坡国立大学海外交流项目，以J1访问学者身份在美国硅谷创业公司Naya Health实习。
本人即将于2020年12月从美国卡内基梅隆大学毕业，已申请Post OPT并获批，将入职位于加利福尼亚州的Facebook工作深造。本人在海外留学期间一直是官方留学生学生联合会成员（新加坡学联，卡内基学联），受到了领事馆的不少照顾和指点。尤其是在疫情期间，收到来自外交部的抗疫资源，很是感动。本人希望能够在回国长期发展之前，在美国的科技公司深造一段时间，为之后回国的发展和贡献打下一个更加坚实的基础，特此提出J-1豁免申请。

附：留学时间及经历
2015年8月  - 2016年6月：就读新加坡国立大学
2016年6月 - 2016年8月：F-1签证，自费赴美斯坦福大学暑期学校项目
2016年8月  - 2017年1月：就读新加坡国立大学
2017年1月- 2017年12月：J-1签证，以访问学者身份自费赴美Naya Health带薪实习，并在斯坦福大学就读。
2017年12月- 2018年1月： 空闲在家（浙江杭州）
2018年1月- 2018年5月：就读新加坡国立大学
2018年5月- 2018年8月：北京微软亚洲研究院实习
2018年8月- 2019年5月：就读新加坡国立大学
2019年8月- 2019年11月：F-1签证，自费赴美卡内基梅隆大学就读计算机硕士。
2019年11月- 2020年1月：空闲在家（浙江杭州）
2020年1月- 至今：就读卡内基梅隆大学

当前所持EAD 的有效时间为：2022年1月10日

 



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
  - Extensible (parameterizable with multiple input languages and execution targets, making it handy for different execution engines and data models)
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







# Linux Code 

## Processes 

#### Design:

- “The kernel stores the list of processes in a circular doubly linked list called the task list”
- Some architecture stores the `current task` pointer in one of ther registers while some stored it at the top of the kernel stack (growing down). 



#### Scheduling

##### Why assigning absolute timeslices to nice value (priority) is troublesome?

- HIgh switching cost <= running multiple low priority jobs <= they would context switch at a high frequency 
- From relative to absolute <= increment in 1 of nice value might result in half of the time slices
- Aboslute timeslices depend on the timer tick (which could be in the range of 10 - 1 `ms`) => too much variation 



##### So comes the CFS:

> “CFS is based on a simple concept: Model process scheduling as if the system had an ideal, perfectly multitasking processor. In such a system, each process would receive 1/n of the processor’s time, where n is the number of runnable processes, and we’d schedule them for infinitely small durations, so that in any measurable period we’d have run all n processes for the same amount of time”
>
> Excerpt From: Robert Love. “Linux Kernel Development, 3/e.” Apple Books. https://books.apple.com/us/book/linux-kernel-development-3-e/id401112485

- nice value will change the relative weight a process owns (lower nice value => higher priority => higher proportion thant `1/n` )



# Work Tips 

### How to 1 on 1: 

##### A couple of good phrases:

- If you cannot handle the truth, don't ask me to be honest 



##### Steps:

1. make it safe for them:

   - Be honest in really wanting the feedbacks 
   - Give a reason 
   - Tell them you already have an issue 
   - Thank in advance

   > Rethink my 1:1 with my Intern Mentor: 
   >
   > Hey man, thanks for taking you time to mentor me (4)! I really appreciate the fact that you set up such frequent meetings with me. And your feedback means a great deal(4) 
   >
   > You know, as an intern, I believe you are definitely seeing more things that I see, and I might not be aware of many of bad things I should not be doing, or things I should be doing more often. So I really want to take this time to hear your thoughts on my first 2 weeks. (1,2?)
   >
   > I see many other interns posting things on the intern group, should i be more active and visible in terms of my work, just like other interns who did that? Like, in general, how should I improve my communication? (3)

2. Make it easy:

   - Give options

   - Ask about emotions (hmm, maybe hard on this one?)

   - Ask for letter grade

   - Ask to expand on feedbacks 

     >So should I be doing more of A, B , C? Do you think if posting things more frequently is better? Like what you think would be a good frequency or occasion to do so? (1)
     >
     >I am not sure how others perceive the posting, I am afraid they would see it as you know, over self-promoting, what do you feel when you see such posts in general? (2). 
     >
     >One of the feedbacks I had was that I should probably reach out to more teammates more frequently, what do you think? Like what would be a good frequency? Because I feel not sure if I should reach out to them if there is no obvious agenda. 

3. Make it rewarding: 

   - Thank them (duhhh) 
   - Apply to action 
   - Attribute change/improvement to their feedback 









# 购物

羽绒服

- https://thenorthface.world.tmall.com/category-1068419405.htm?spm=a312a.7700824.w4011-14964131723.57.2d671280jILPW2&catName=%D3%F0%C8%DE%B7%FE%2F%BC%D0%C3%DE&parentCatId=1068418495&parentCatName=%C4%D0%C9%CF%D7%B0&catId=1068419405&search=y&orderType=price_desc
- https://detail.tmall.com/item.htm?spm=a312a.7700824.w4011-14711919654.101.5ac31410AFjLSj&id=600945498227&rn=4f8256ec3bdb60a10802f1eb05f30c1a&abbucket=4&sku_properties=1627207:3232483









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



#### Prefetching

Why: improves latency 

Example: 

- Bufferpool prefetching in DB 
- instructions prefetching in CPU
- page prefetching in OS disk I/O 





# Intresting companies





# Questions to ask 

Daily Routine Reading List:

