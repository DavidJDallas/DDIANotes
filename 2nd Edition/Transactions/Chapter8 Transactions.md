# Chapter 8: Transactions

When writing to a db, loads of things can go wrong. Transactions prevent the fallout of a lot of this. It 'allows the application to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead' (p278).

The first of these - allowing the developer to ignore certain error scenarios - comes under the 'A' of ACID - Atomicity. Could also be called 'all-or-nothing' and have a good similar effect. But essentially all of your operations are bunched into a single unit and succeed or fail together. This allows for consistency guarantees and for you to be able to reason about things much easier. It won't prevent the problem (a failure at some point in the system) but it can stop the negative consequences of it.

The second of these - allowing the developer to ignore at least some concurrency issues - comes under the 'I' of ACID - Isolation. As will be seen, the 'at least some' qualifier is needed because this depends on what isolation guarantee the database gives us. We can have much weaker forms of isolation that just prevent no dirty reads and no dirty writes (e.g Read-committed isolation) all the way through to serialisable isolation where the guarantee is strongest and allows the developer to operate on the assumption that they will run *as if* the transactions were being run one after the other.

As always, how strong you want your isolation guarantee trades off with performance - weaker isolation guarantees generally means more potential to be performant, but with reduced consistency guarantees. And vice versa.

## What is ACID?

- **Atomicity:** All the items in your transaction will be written or fail together.
- **Consistency:** Guarantee that your data respects the invariants of your application code. Kleppmann points out that really this is down to the application.
- **Isolation:** Concurrently executing transactions are isolated from each other.
- **Durability:** Data is persisted to disk if the transaction is successful. Nowadays maybe also is replicated.

C isn't really a condition. D is pretty basic in terms of whether or not you have it.

There's a little bit of ambiguity around Atomicity. And the book reflects that. There's a few sections on what this means:

- Single-object or multi-object Operations. (Combined with how this ties into isolation levels too.)

But really, the bulk of this chapter is for the I in ACID (Isolation).

## History

### The Origin of Transactions

- **1973:** Davies and Bjork papers. 'Spheres of Control'. Applied to general computing, not specifically databases. A sphere enclosed a piece of work whose effects could be: (i) committed, (ii) backed out, (iii) isolated from surrounding work, (iv) nested inside another sphere.
- **1974 (but not actually in print until 1976):** Idea taken specifically to databases and the word 'Transaction' is given to it. (Eswaran et al paper)
- **1975:** System R started to be developed. Created and implemented SQL and Transactions more as we know them in a relational database. In a separate talk Kleppmann says that the typical types of isolation:
  - Serialisable, repeatable reads, read committed, read uncommitted

  Are all coined in this db also.
  - Snapshot Isolation seems to come later (1995 paper)
- **1981:** Gray paper (Gray worked on System R and involved a lot in the IBM research around this). Attempt to define a Transaction more clearly:

  > 'A transaction is a transformation of state which has the properties of atomicity (all or nothing), durability (effects survive failures) and consistency (a correct transformation).'

  So, missing the 'I', but this is implicitly included in consistency.
- **1983:** Harder and Reuter paper where canonical definition of a Transaction is given via ACID acronym.
- **1995:** Snapshot Isolation via Berenson, Gray et al paper: <https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf>

### The Move Away from Transactions

- **Late 2000s:** NoSQL databases come onto the scene and prioritise availability and low latency over consistency. Transactions were a fatality of this. Transactions trade off consistency for slowing things down, and making the db more available. With rise of distributed computing Transactions were often what got cut.
- **2006:** Google's BigTable
- **2007:** Amazon's Dynamo (not to be confused with DynamoDB)(This is also the reference that Kleppmann gives in Replication chapter on what popularised leaderless replication).
- **2008:** BASE paper (Basically available, Soft State, Eventual Consistency)

Then: 'NewSQL'. Transactional implementations introduced into new distributed systems stuff. Seen as bridging the best of both worlds, to a degree.

- **2012:** Google's Spanner paper
- **2014:** Cockroach DB started, released in 2017 (I think). Inspired heavily by Spanner.

But NewSQL stuff flipped that narrative. CRDB, TiDB, Spanner, FoundationDB, YugabyteDB all incorporate Transactions and are highly scalable databases.

## Single-Object and Multi-Object Operations

Multi-object transactions require some way of determining which read and write operations belong to the same transaction. In Relational DBs, this is typically done based on the client's TCP connection to the db server.

But many non-relational databases don't have this ability. (Surely this isn't why they don't allow for MO transactions?)

### Single-Object Writes

A and I also apply when a single object is being changed. Example of writing a large JSON object. Storage engines almost universally aim to provide atomicity and isolation the level of a single object. Therefore the suggestion is that satisfying this criterion does not qualify for saying you support Transactions.

### Need for Multi-Object Transactions

Could we do without Multi-object writes? And just be safer and do everything single-object like NoSQL often does? List given of v plausible use cases where this wouldn't work:

- Row in 1 table has a FK reference to a row in another table.
- DBs with secondary indexes, indexes need to be updated every time you change a value. Without transaction isolation, possible for a record to appear in one index but not in the other.

### Handling Errors and Aborts

There are some ways it can go wrong when we simply retry an aborted transaction.

- The retry mechanism is only worth doing if you're facing a transient error.
- If the transaction actually succeeded, but the network was interrupted while the server tried to acknowledge the successful commit to the client, then retrying the transaction causes it to be performed twice unless you have an additional application-level de-dupe mechanism in place.
- If the transaction has side effects, you don't want to retry without some sort of safety mechanism in place.

## Weak Isolation Levels

The weakest isolation level is Read Committed. This provides a guarantee of no dirty reads, and no dirty writes.

On a step-up from Read Committed, we have Repeatable Reads and Snapshot Isolation. These are basically indistinguishable for a user, but have very different implementation details. Repeatable reads utilises locks more, and was implemented in the original System R database. Snapshot Isolation came in 1995, and uses more sophisticated techniques like MVCC.

### Read Committed

Makes two guarantees:

- When reading from the db, you will see only the data that has been committed (no dirty reads)
- When writing to the database, you will overwrite only data that has been committed. (No dirty writes).

The most common way to prevent *dirty writes* is to use row-level locks. When a transaction wants to modify a particular row, it must first acquire a lock on that row. It then holds that lock until the transaction is committed or aborted.

To prevent dirty reads, we could implement the same locking mechanism. But this is usually overkill for performance-wise since it blocks reads more frequently than is required. More common:

> 'For every row that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the row are simply given the old value' (p293).

Some dbs support an even weaker level: Read uncommitted. Prevents dirty writes but doesn't prevent dirty reads. This can provide better perf (expected). However, says that it can reduce the probability of lost updates.

Trying to get clear on this last bit. A few possible explanations jump out. But first, we should be clear that fundamentally a lost update is a read-modify-write race condition where two transactions will read the same value simultaneously, both update it, but then only the last to successfully commit their transaction will get the update. The first one to commit will have their update lost, hence the name.

1. By avoiding dirty reads you will get more up-to-date data from your reads. You therefore reduce the risk of stale data and rely on the fact that most transactions will not fail, therefore increasing your odds of avoiding a lost-update but increasing the risk overall of other types of inconsistent data issues.

### Snapshot Isolation and Repeatable Reads

In addition to the guarantees that Read Committed gives you, these Isolation levels also guarantee you against *read skew*. Read skew is a race condition that will be fixed if you re-read, but is a temporary bad read.

Snapshot isolation is the most common solution to this problem. The idea is that each transaction reads from a consistent snapshot of the database - it sees all the data that was committed to the database at the start of the transaction.

- To implement, typically uses write locks to prevent dirty writes. This blocks other writes.
- But reads don't need locks.
- Readers never block writers and writers never block readers.
- Generally implemented using a generalisation of the mechanism for preventing dirty reads, but instead of just using 2 versions of each row, the db must potentially keep many versions. Hence MVCC - Multi-version concurrency control.

So, we can say that MVCC is an implementation technique to achieve snapshot isolation.

### Preventing Lost Updates

![](./taxonomy.png)

This next section follows the general core: Snapshot Isolation is quite good, and prevents certain race conditions. And does so in a more efficient way than traditional approaches of taking out locks. It also - as we see - laid the foundations of serialisable snapshot isolation that was published in 2008, which most modern serialisability is built upon.

But it nonetheless leaves the developer using it open to various race conditions.

- Lost Updates. Bad, but preventable relatively fine nowadays.
- Write skews. Worse, but can be prevented.
- Phantoms, particularly write skews that involve phantoms. Worse still, and even harder to prevent without serialisability.

#### Lost Update

Take two transactions T1 and T2. Both handle the shared state of user1's balance.

![](./LostUpdate.png)

- (1) T1 reads from the database - user1's balance is 7000.
- (2) T2 reads from the database - user1's balance is 7000.
- (3) T1 updates their value to the database - user1's balance is 7,400.
- (4) T2 updates their value to the database - user1's balance is 7,600.

But we now finish this process with an incorrect balance for user1. It should be 8,000, instead it's 7,600. The first update has been lost.

No individual step is wrong. Yet clearly something has gone wrong. *The problem only exists at the level of the schedule*; it only exists when taking all the steps together as a whole. And it's not correct because the observable outcomes of running these transactions is not the same as if we ran them serially, i.e. ran them one after the other.

A bit more formally: ≥ 2 transactions each (a) read object S and (b) write S, where the value written is derived from the value read, and at least one transaction's write lands between another's read and that other's write.

- It's not seen as damning, because it's both more easily preventable, and SSI automatically prevents it.

##### Atomic Write Operations

- The bit that does the damage for LUs is the gap between the read and the write. In this gap, another Transaction can commit to the db and make the value you read from stale. So, writing directly removes this gap, and solves the issue.
- Instead of doing a read-modify-write, you just do a write directly. E.g. `UPDATE` syntax, no reads.
- ORM frameworks can make it easy to accidentally write code that perform unsafe read-modify-write cycles.

ORMs often require this - or at least make it the more straightforward option. You need to first bring the data into memory, then you update the data in memory, and then write back to the database. I know some ORMs do have options to do straight writes (`UPDATE`s), but often more of a niche feature.

##### Explicit Locking

- use the application to explicitly lock objects that are going to be updated.
- Carries risks of (i) you can always forgot to implement this if it's your policy, (ii) is a blocking operation and can cause deadlocks.

##### Automatically Detecting Lost Updates

Alternatively, we can allow them to happen in parallel, and let the db do the work. Lots of databases at the Snapshot Isolation level detect for this automatically.

##### Conditional Writes

Some databases allow for conditional writes - an operation that prevents lost updates by allowing an update to happen only if the value has not changed since you last read it.

- Sometimes called optimistic locking.

#### Conflict Resolution and Replication

If your db is replicated, preventing LUs takes on another dimension. Because the dbs have copies of the data on multiple nodes and data can potentially be modified concurrently on different nodes, additional steps need to be taken.

Generally, techniques based on locks or conditional writes don't apply in the context of multi-leader or leaderless, since they allow several writes to happen concurrently.

Instead, a common approach is to allow concurrent writes to create several conflicting versions, and to use application code or specialised data structures to resolve and merge these versions after the fact.

### Write Skews and Phantoms

A more subtle class of race conditions between concurrent writes. And nastier than LUs because not straightforward to fix.

We can think of Write Skews as a generalisation of the Lost-Update problem.

![](./write-skew.png)

So, we imagine two transactions, T1, T2, reading this at the same time. Both read 2, and so both proceed down this branch. Both update, but after both T1 and T2 have committed, then there are no doctors on call. 

Whereas LUs require >=2 Transactions to (a) read the same state, (b) update the same state that they read from, Write Skews hold (a) but inverts (b) as a condition, and also add a third condition - (c) - that the two differing writes, taken together, violate some invariant that would not have been violated had the transactions run serially.

- write skews are particularly difficult because they violate these invariants which can't simply be blocked by things like atomic writes. The atomic write would have an invariant in it which is still violated because it depends on data that is rendered stale by an incoming write from another Transaction in your set of Ts.

- With preventing write skew, the ways to prevent it are more limited vs LUs. Atomic single-object operations don't help, as multiple objects are involved.
- Automatic detection of LUs that you find in some implementation of SI doesn't help. To automatically prevent, this requires true serialisation.
- If you don't use serialisability, the 2nd best option is probably to explicitly lock the rows that the transaction depends on.

#### Phantoms Causing Write Skews


A phantom is a phenomenon where whilst you are reading from a specific range in your database that matches a certain criteria, the specific range is mutated (updated/deleted/inserted to) which will not have been taken into account when you have initially made the read that determines what items need to be updated. The data you have read and will go on to make decisions informed by, has now changed.

Snapshot Isolation avoids phantoms in read-only queries, but not for transactions that involve writes.

You can materialise conflicts, but serialisability is far preferable.

## Serialisability

3 different ways to do serialisability:

- Literally execute transactions one after the other
- 2PL. First established technique, 1976 (?), established formally in 'Notions of Consistency and Predicate Locks in a database system'. In the Pessimistic CC family.
- Optimistic concurrency control, e.g. serialisable snapshot isolation. First proposed 2008 'Serializable Isolation for Snapshot Databases' (Cahill et al). This takes Snapshot isolation (1995) and makes it serialisable

### Actual Serial Execution

May seem obvious, but only been > 2000s that this has been do-able, given that we need single-threaded loops for executing the transactions. RAM became cheap enough to keep the entire active dataset in memory, and designers realised that OLTP transactions are usually short and make only a small number of reads and writes.

- Implemented in VoltDB, H-Store, Redis, Datomic.

#### Encapsulating Transactions in Stored Procedures

Solves the problem of back-and-forth over a network - you just have all the stuff you need on the db side, and run it via the SP.

#### Sharding

Executing all transactions serially makes concurrency control much simpler, but it limits the transaction throughput of the database to the speed of a single CPU core on a single machine

#### Summary of Serial Execution

Serial execution of transactions has become viable due to recent computing. Has certain constraints:

- Every transaction must be small and fast, because it only takes 1 slow transaction to stall all transaction processing.
- Most appropriate when the active dataset can fit in memory.
- Write throughput must be low enough to be handled on a single CPU core, or else transactions need to be sharded without requiring cross-shard co-ordination.
- Cross-shard transactions are possible, but their throughput is hard to scale.

### Two-Phase Locking (2PL)

2PL was the only algorithm used for serialisability in databases for around 30 years. First formalised and defined properly in 'The Notions of Consistency and Predicate Locks in a Database System' (1976), but seems to be used before.

Readers block writers, but not readers. Writers block both readers and writers. 

#### Implementation of 2PL

Implemented by having a lock on each object on the database. Lock can be either shared mode or exclusive mode. Then:

![](./lock-flow.png)

If a lock is taken and a T tries to access a locked object, they typically wait, not abort. Deadlocks occur when all Ts involved are waiting on other Ts and nobody can proceed; system picks a victim. More frequent when high contention.

#### Performance

Generally bad, because you basically stop transactions running concurrently if they touch the same object. Step better than running them serially, but still quite slow. 
However, as will be seen, can be comparable to an OCC algorithm if lots of contentions.

#### Predicate Locks

You can't just take out locks on individual objects and prevent serialisability. This will not prevent phantoms. To solve this, we need a predicate lock. 

Importantly, a predicate lock applies even to object that don't yet exist in the database, but that might be added in the future.

#### Index-range locks

Naturally, predicate locks don't perform well. Consequently, most databases using 2PL implement index-range locking (sometimes known as next-key locking). This is a simplified approximation of predicate locking. 

We try and find an index that is either equal to or greater than the predicate range - it must at least contain all of it. Then we lock this index

Pros

- Faster, because we're locking something that already exists. We don't need a new subsystem to take out the lock, we utilise existing ones.
- You're also already doing an index traversal, so this just piggy-backs on work you're already doing, doesn't require new work like the predicate lock.

Cons

- You can get false positives: since the lock is on a superset of the predicate, it's very possible that it will block other readers/writers from things that don't actually need to be blocking.

If index-range locks are implemented, and there's no suitable index, it fails back to whole-table locks. 

### Serialisable Snapshot Isolation

Provides full serialisability with only a small performance penalty compare to snapshot isolation. 
First described in 2008.

Optimistic Concurrency Control technique, in contrast to the pessimistic techniques we've seen above. 

#### Optimistic vs Pessimistic Concurrency Control

- Optimistic: Assume everything will be fine, let things proceed, and then check at the end if conflicts have happened. If they have, abort.
- Pessimistic: Assume that if there's any overlap, this will be bad, and so don't let it happen in the first place. Block via a lock.

Concurrent operations are allowed on potentially dangerous scenarios, and then when the T wants to commit, the db checks whether anything bad happened (i.e. whether isolation was violated). If so, the db will abort.

Optimistic concurrency control more broadly that performs badly if high contention. Not specific to SSI. But tends to perform better than pessimistic if contention is not too high.

*Why?*

- OCC contentions are more costly than PCC contentions: 
- an OCC contention involves the storage engine effectively doing the processesing of N number of Transactions, and then at commit time, aborting N-1 of them (worst case scenario). A PCC contention just makes all Ts wait, if problematic (will also abort under certain conditions). So OCC pays in more CPU, I/O; PCC pays mainly in time waiting (still doing some work, like lock manager mechanisms).

- Importantly, for practical purposes, PCC will usually be faster in these circumstances. Say we have N number of Transactions, all contending. Then, PCC will be T x N.
OCC is, best case scenario, T x N, more likely T x N(N-(something))

- OCC can easily take many times longer, since each one will have to be retried by the application. If the application enables an automatic retry, then this contention-and-abort cycle could go on for a while.

- This is exemplified greater when Transactions are longer running - we exert a lot of work on the system only for it to then be aborted. So all of the above is multiplied in effects. With the above example, we can easily imagine wasting huge amounts of resources on abort-retry cycles. 


#### What is SSI

- Based on snapshot isolation algorithm, but adds on top of it an algorithm for detecting serialisation conflicts among reads and writes and determining which transactions to abort. 
- I.e., all reads within a transaction are made from a consistent snapshot of the database.

- High-level idea is that we let N number of transactions run concurrently with one another, keep records of everything that they touch, and then if they violate certain constraints which could *potentially* cause a race condition, you abort at least 1 of them. 


#### How does it Work?

We saw above that 2PL is solved by taking out locks in a strategic way. Intuitively quite straightforward as a solution. SSI is more permissive, but still allows for false positives and will never be a 1-1 mapping between times aborted and race-conditions prevented. 


Two cases set out regarding how to avoid: 
- Detecting reads of a stale MVCC object version (uncommitted write ocured before the read)
- Detecting writes that affect prior reads (the write occurs after the read)

MVCC: when a transaction reads from a consistent snapshot in an MVCC db, it ignores the writes that were made by any other transactions that hadn't yet committed at the time the snapshot was taken. 

- w-w is first write wins. This it just inherits from SI, nothing new to SSI. The new stuff is how it handles the r-w race conditions. 

###### Detecting reads of a stale MVCC object

- Db tracks when a transction ignores another transaction's write because of MVCC visibility rules. When the T wants to commit, the db checks whether any of the ignored writes have now been committed. If so, abort.
- We need to wait until committing because if the T was read-only it would be fine.

##### Detecting writes that affect prior reads

Here, we consider another transaction modifying data after it has been read. 


## Distributed Transactions

If your db uses single-leader replication, the transaction execution happens only on the leader. The followers simply apply the log of writes that were committed by transactions on the leader.


Concurrency control for distributed transactions are broadly similar to those for single-node concurrency control. 2PL works on a distributed setting. For SSI there distributed serialisabiliy checkers. 

But achieving atomicity in a distributed transaction is a whole new challenge. The chapter focuses on this. 

In single-node transactions, commitmment crucially depends on the order in which data is duraby written to disk. First will be the data, then the committ record. Whether or not the transaction succeeds depends on if the commit record succeeds. 

In a distributed setting, this isn't so straight-forward. When a T wants to commit, it's not sufficient to just send a commit request to all the nodes and independently commit the transaction on each one - the commit could succeed on some nodes and fail on others. 

A better approach is to ensure that the nodes involved in a T either all commit or all abort and to prevent a mixture of the two. To achieve this is known as the atomic commitment problem. 

### Two-Phase Commit


Uses a new component that doesn't normally appear in single-node transactions: a co-ordinator/transaction manager. 

- write data
- prepare (ask all participants if they are ready to commit)
- commit

It may seem that this can still lead to problems. For example, the commit and prepare phases can still be lost?

The process is broken down in a bit more detail to set out why this works:

1. When the application wants to begin a distributed transaction, it requests a Tx ID from the co-ordinator. Globally unique.
2. Application begins a single-node Tx on each of the participants and attaches the globally unique TxId to the single-node Tx. All reads and writes done here. 
3. When ready to commit,the co-ordinator sends a prepare request to all participants, tagged with global TxId. *If any request fails or times out*, co-ordinator sends an abort request for that TxId to all participants. 
4. When a participant receives a prepare request, makes sure it can definitely commit the Tx under all circumstances. By replying yes, the node *promises* to commit the Tx without error if requested. It does not yet commit. In other words, it promises that there are no reasons it can't commit.
5. When co-ordinator receives all responses, makes a definitive decision to commit or abort. It writes that decision to its transaction log on disk so that it knows which way to proceed if there's a crash. This is the *commit point*.
6. Once the co-ordinator\'s decision has been written to disk - the commit point - commit or abort request is sent to all participants. If *this* request fails or times-out, it must replay indefinitely until it succeeds; there's no going back. It can't now refuse. 

It seems that the key point here is that all the participant agrees to is that there are no *non-transient* reasons that it can't commit. The usual reasons that a Tx may fails of transient problems are discarded, since the participant will jsut keep retrying and therefore achieve eventual consistency. It seems a bit confusing here though, since presumably it's theoretically possible for the nodes to be inconsistent for several minutes or longer and therefore the only guarantee is eventual consistency. 

Therefore 2 points of no returns:
(a) when a participant votes yes, it promises that it will definitely be able to commit later.
(b) once the co-ordinator decides, the decision is irrevocable. 

##### Co-ordinator failure

Once we reach the prepare requests stage, it becomes more tricky if the co-ordinator crashes. Before, the participant can safely abort the Tx. After, the participant must wait to hear back from the co-ordinator. The participant must wait. This is called 'in doubt' or 'uncertain'. 

The only way for 2PC to complete is by waiting for the co-ordinator to recover. This is blocking, and can cause problems when it happens indefinitely.

We have a 3PC, which is non-blocking. But trickier.

Says best way to solve the problem above from 2PC is to replace the single-node co-ordinator with a fault-tolerant consensus protocol. Done in chapter 10.

### Distributed Transactions across different systems

Many cloud services don't implement 2PC because of the operational problems that they bring (performance, promising more than they can guarantee, etc). 

There are 2 types of distributed Txs that are often conflated:

- Database-internal distributed transactions

Some distributed databases (i.e. ones that use replication and sharding in their standard config) support internal Txs across their nodes. 

- Heteregenous distributed transactions. 

Participants are >=2 technologies communicating with each other.

Db internal Txs don't have to be compatible with any other system, so they can use any protocol and apply optimisations specific to that particular technology. Heteregenous are more challenging. 

- Exactly-once semantics.

##### XA Transactions

A standard for implementing 2PC across heteregenous technologies. Introduced in 1991. 
- Not a network protocol; it's an API for interfacing with a transaction co-ordinator. 
- Assumes that your application uses a network driver or client library to communicate with the participant databses or messaging services. 

##### Holding Locks while in doubt

The problem with the Tx being stuck in doubt is to do with locking. Db Txs usually require row-level exclusive locks on any rows they modify, to prevent dirty writes. Any if you want serialisable isolation using 2PL, it's much worse. 
The db can't release those locks until the Tx commits or aborts.  This could be a long time. 

##### Recovering from Co-ordinator failure

Whilst in theory if the co-ordinator crashes, it should be able to just re-read from the recovery log, sometimes orphaned transactions do occur. The only way out is for an admin to decide whether to abort or commit. Resolving this requires a lot of manual effort, and it most likely needs to be done under stress and time pressure during a serious production outage. 

Many XA implementations have an emergency escape hatch called heuristic decisions: allowing a participant to unilaterally decide to abort or commit an in-doubt transaction without a definitive decision from the co-ordinator. This would break atomicity.

##### Exactly-once message processesing, revisited

You don't actually need distributed Txs to achieve exactly-once semantics. An alternative approach is as follows:

- Assume every message has a unique Id, and in the db you have a table of message IDs that have been processesed. When you start processesing a message from the broker, you begin a new Tx on the db and check the message ID. If same message Id is already existing, you know that it's been processesed, so ack the message to the broker and drop.
- If the messageId is *not* in the db, add it to the table. Then process the message. When you finish processesing the message, commit the Tx on the db.
- Once db Tx successfully commits, ack the message to the broker.
- Once the message has been successfully acked to the broker, you know that it won't try processesing the same message again, so delete the message ID from the db (in a sep Tx).

Note that this wouldn't, I don't think, work exactly for something like an outbox pattern. In an outbox pattern, you'd have something like:

- Commit data needed for the side-effect to an outbox table, within the same tx that goes to your regular table that also needs persisted data. This guarantees atomocity and consistency across the two sources.
- The side-effect service will regularly poll the outbox table and take the instructions and info from it. The outbox table is the source of truth. The side-effect service will just retry if it fails, and is safe.