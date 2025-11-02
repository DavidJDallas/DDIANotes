<h1> Transactions </h1>

In software engineering, things can and do wrong. Sometimes catastrophically. Transactions make things safer, and whilst they don't prevent failures, they prevent partial failures or partial success. 

'With transactions, error handling becomes much simpler for an application, because it doesn't need to worry about partial failure'. 

Transactions were the main casualty of the NoSql movement into default replication and partioning. Many NoSQL databases abandonded transactions, or redefined it to describe a much weaker set of guarantees. 

<h2>The Meaning of ACID </h2>

- Atomicity
- Consistency
- Isolation
- Durability


<h4> Atomicity </h4>

In short: guarantee of all-or-nothing. A transaction can crash halfway through and we can be confident that nothing has been committed. Similarly, if a transaction completes we know everything inside it has been committed.

<h4> Consistency </h4>

Points out that this is heavily to do with the application code, and that Hellerstein has remarked that the C in ACID was put in just to make the acronym work. Consistency in short is put good data in, get good data saved. The db will be in a good state based on the input of the transaction. 

<h4> Isolation </h4>

Concurrently executing transactions are isolated from each other. They are many different degrees of isolation: the strongest is serializable isolation (transactions will run as if running sequentially), and weakness goes quite far down to not much isolation guarantees at all. 

<h4> Durability </h4>

Once a transaction has been committed, any data it has written will not be forgotten. 



There's most to discuss around Atomicity and Isolation. Consistency and Durability are weaker and more straightforward. 

<h2> Single object and multi-object operations </h2>

Storage engines almost universally aim to apply atomicity and isolation on the level of a single object. So, most single object submits will have some sort of ACID guarantee. But:

'A transaction is usually understood as a mechanism for grouping multiple operations on multiple objects into one unit of execution' (p230).

'Many distributed datastores have abandoned multi-object transactions because they are difficult to implement across partitions, and they can get in the way in some scenarios where very high availability or performance is required'. (p231)

There are edge cases where the A in ACId will not be true. E.g. transaction is committed, but network fails while server tries to acknolwedge submission to the client. 

<h2> Weak Isolation Levels </h2>

Concurrency bugs (race conditions) don't arise in simultaneous reads. They only arise when writes are involved, whether that be one trying to write at the same time as another tries to read, or one trying to write at the same time as the other trying to write. 

Race condition bugs are non-deterministic - they sometimes occurr, sometimes don't, and are dependent on various external factors to the code/db (multiple attempts toa ccess the data simultaneously).

Not from DDIA is a brief taxonomy:
- write-read issue (dirty read)
- read-write issue (non-repeatable read)
- read-write (set-based): Phantom read
- write-write (lost update): 

Note: we can see avoiding use of mutable states in your appliation code as a way to entirely avoid race conditions.

Serialisable isolation has a performance cost that many databases simply don't want to pay the price for. Weaker isolation levels are not just a theoretical problem - major problems ahve arisen from it. This is further interesting because many databses are technically ACID - which consulstants will advise to use for fintech, and areas where this sort of safety is paramount - and yet an ACID database, if using weaker isolation, can still absolutely allow serious bugs like this. 

<h3> Different types of Weak Isolation </h3>

<h4> Read Committed </h4>

The two guarantees from this are <b>No dirty reads </b> and <b> No Dirty Writes. </b>

I.e., when reading from the database, you will only see data that has been committed. And when writing to the database, you will only overwrite data that has been committed. 

It may seem that everything that needs to be done here for ACID compliance has been done. But there are many ways you can still have concurrency bugs. 

<h4> Snapshot Isolation and Repeatable Read </h4>

Snapshot isolation: the transaction sees a consistent, static, snapshot of the database as it existed at the moment the transaction began. 

<h4> Serialisability </h4>

This is the strongest guarantee, but it naturally comes with trade offs (usually speed).

<h5> Actual Serial Execution </h4>

Here, we literally sequentially run transactions. Only really introduced in 2007, when RAM became cheap enough for entire dataset to be kept in memory, and database designers realised that OLTP transactions are usually short and only make a small number fo reads and writes. 

Implemented in VoltDb and H-store, Redis, and Datomic.

Certain requirements for this type of execution:

- Every transaction must be small and fast.
- Limited to use cases where the active dataset can fit in memory.
- Write thoroughput must be low enough to be handle on a single CPU core.
- Cross-partitioned transactions possible, but hard limit to which they can be used.

<h5> Two Phase Locking: 2PL </h5>

Dominant way of ding it for 30 years. 

In 2PL, 'writers don't just block writers; they also block readers and vice versa. Snapshot isolation has the mantra readers never block writers, and writers never block readers...(2pl) protects against all the race conditions discussed earlier, including lost updates and write skew'. 

Pessimistic concurrency.

<h5> Serialisable snapshot-isolation</h5>

'serializable snapshot isolation is an optimistic concurrency control technique. Optimistic in this context means that instead of blocking if something potentially dangerous happens, transactions continue anyway, in the hope that everything will turn out all right' (p261)


Isolation levels vs concurrency control

Isolation levels are what you can guarantee. Serializable is the strongest level, and transactions behave as if they ran one at a time, in some order. There'll be no anomalies, no lost updates, no dirty reads, etc. 

The concurrency control is about the mechanism you use to provide that isolation guarantee. Typically either optimistic concurrency or pessimistic. Pessimistic assumes conflicts will happen and prevents them upfront; optimistic assumes conflicts are rare, detects them at the end. 