# Transactions

In software engineering, things can and do wrong. Sometimes catastrophically. Transactions make things safer, and whilst they don't prevent failures, they prevent partial failures or partial success. 

'With transactions, error handling becomes much simpler for an application, because it doesn't need to worry about partial failure'. 

Transactions were the main casualty of the NoSql movement into default replication and partioning. Many NoSQL databases abandonded transactions, or redefined it to describe a much weaker set of guarantees. 

## The Meaning of ACID

- Atomicity
- Consistency
- Isolation
- Durability

### Atomicity

In short: guarantee of all-or-nothing. A transaction can crash halfway through and we can be confident that nothing has been committed. Similarly, if a transaction completes we know everything inside it has been committed.

### Consistency

Points out that this is heavily to do with the application code, and that Hellerstein has remarked that the C in ACID was put in just to make the acronym work. Consistency in short is put good data in, get good data saved. The db will be in a good state based on the input of the transaction.

### Isolation

Concurrently executing transactions are isolated from each other. They are many different degrees of isolation: the strongest is serializable isolation (transactions will run as if running sequentially), and weakness goes quite far down to not much isolation guarantees at all.

#### Durability

Once a transaction has been committed, any data it has written will not be forgotten.

There's most to discuss around Atomicity and Isolation. Consistency and Durability are weaker and more straightforward.

## Single object and multi-object operations

Storage engines almost universally aim to apply atomicity and isolation on the level of a single object. So, most single object submits will have some sort of ACID guarantee. But:

'A transaction is usually understood as a mechanism for grouping multiple operations on multiple objects into one unit of execution' (p230).

'Many distributed datastores have abandoned multi-object transactions because they are difficult to implement across partitions, and they can get in the way in some scenarios where very high availability or performance is required'. (p231)

There are edge cases where the A in ACId will not be true. E.g. transaction is committed, but network fails while server tries to acknolwedge submission to the client.

## Weak Isolation Levels 

Concurrency bugs (race conditions) don't arise in simultaneous reads. They only arise when writes are involved, whether that be one trying to write at the same time as another tries to read, or one trying to write at the same time as the other trying to write. 

Race condition bugs are non-deterministic - they sometimes occurr, sometimes don't, and are dependent on various external factors to the code/db (multiple attempts toa ccess the data simultaneously).

Not from DDIA is a brief taxonomy:

- write-read issue (dirty read)
- read-write issue (non-repeatable read)
- read-write (set-based): Phantom read
- write-write (lost update)

Note: we can see avoiding use of mutable states in your appliation code as a way to entirely avoid race conditions.

Serialisable isolation has a performance cost that many databases simply don't want to pay the price for. Weaker isolation levels are not just a theoretical problem - major problems ahve arisen from it. This is further interesting because many databses are technically ACID - which consulstants will advise to use for fintech, and areas where this sort of safety is paramount - and yet an ACID database, if using weaker isolation, can still absolutely allow serious bugs like this. 

### Different types of Weak Isolation

#### Read Committed 

The two guarantees from this are <b>No dirty reads </b> and <b> No Dirty Writes. </b>

I.e., when reading from the database, you will only see data that has been committed. And when writing to the database, you will only overwrite data that has been committed. 

It may seem that everything that needs to be done here for ACID compliance has been done. But there are many ways you can still have concurrency bugs. 

#### Snapshot Isolation and Repeatable Read

Snapshot isolation: the transaction sees a consistent, static, snapshot of the database as it existed at the moment the transaction began. 

MVCC is the underlying mechanism that makes snapshot isolation possible and efficient.

#### Serialisability

This is the strongest guarantee, but it naturally comes with trade offs (usually speed).

#### Actual Serial Execution 

Here, we literally sequentially run transactions. Only really introduced in 2007, when RAM became cheap enough for entire dataset to be kept in memory, and database designers realised that OLTP transactions are usually short and only make a small number fo reads and writes. 

Implemented in VoltDb and H-store, Redis, and Datomic.

Certain requirements for this type of execution:

- Every transaction must be small and fast.
- Limited to use cases where the active dataset can fit in memory.
- Write thoroughput must be low enough to be handle on a single CPU core.
- Cross-partitioned transactions possible, but hard limit to which they can be used.

##### Two Phase Locking: 2PL

Dominant way of ding it for 30 years. 

In 2PL, 'writers don't just block writers; they also block readers and vice versa. Snapshot isolation has the mantra readers never block writers, and writers never block readers...(2pl) protects against all the race conditions discussed earlier, including lost updates and write skew'. 

Pessimistic concurrency.

#### Serialisable snapshot-isolation

'serializable snapshot isolation is an optimistic concurrency control technique. Optimistic in this context means that instead of blocking if something potentially dangerous happens, transactions continue anyway, in the hope that everything will turn out all right' (p261)

Obviously with snapshot isolation, the assumption made at the start of the transaction may not be true - things may have changed. 

There are two cases to consider when we ask the question: how does the database know if a query result might have changed?

(1) Detecting reads of a stale MVCC object version (uncommitted write occured before the read)
(2) Detecing writes that affect prior reads (the write occurs after the read)

Isolation levels vs concurrency control

Isolation levels are what you can guarantee. Serializable is the strongest level, and transactions behave as if they ran one at a time, in some order. There'll be no anomalies, no lost updates, no dirty reads, etc.

The concurrency control is about the mechanism you use to provide that isolation guarantee. Typically either optimistic concurrency or pessimistic. Pessimistic assumes conflicts will happen and prevents them upfront; optimistic assumes conflicts are rare, detects them at the end.