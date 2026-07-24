# Transactions

When writing to a db, loads of things can go wrong. Transactions prevent the fallout of a lot of this. It 'allows the application to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead' (p278).


The first of these - allowing the developer to ignore certain error scenarios - comes under the 'A' of ACID - Atomicity. Could also be called 'all-or-nothing' and have a good similar affect. But essentially all of your operations are bunched into a single unit and succeed or fail together. This allows for consistency guarantees and for you to be able to reason about things much easier. It won't prevent the problem (a failure at some point in the system) but it can stop the negative consequences of it. 

The second of these - allowing the developer to ignore at least some concurrency issues - comes under the 'I' of ACID - Isolation. As will be seen, the 'at least some' qualifier is needed because this depends on what isolation guarantee the database gives us. We can have much weaker forms of isolation that just prevent no dirty reads and no dirty writes (e.g Read-committed isolation) all the way through to serialisable isolation where the guarantee is strongest and allows the developer to operate on the assumption that they will run *as if* the transactions were being run one after the other.

As always, how strong you want your isolation guarantee trades off with performance - weaker isolation guarantees generally means more potential to be performative, but with reduced consistency guarantees. And vice versa. 


## What is ACID?


- Atomicity: All the items in your transaction will be written or fail together.
- Consistency: Guarantee that your data respects the invariants of your application code. Klepmann points out that really this is down to the application.  
- Isolation: Concurrently executing transactions are isolated from each other. 
- Durability: Data is persisted to disk if the transaction is successful. Nowadays maybe also is replicated.

C isn't really a condition. D is pretty basic in terms of whether or not you have it. 

There's a little bit of ambiguity around Atomocity. And the book reflects that. There's a few sections on what this means: 
- Single-object or multi-object Operations. (Combined with how this ties into isolation levels too.)

But really, the bulk of this chapter is for the I in ACID (Isolation). 


## The History

### The Origin of Transactions

1973: Davies and Bjork. 'Spheres of Control'. Applied to general computing, not specifically databases. A sphere enclosed a piece of works whose effects could be: (i) committed, (ii) backed out, (iii) isolated from surrounding work, (iv) nested inside another sphere.

1974 (but not actually in print until 1976): Idea taken specifically to databases and the word 'Transaction' is given to it. (Eswaran et al paper)

1975: System R started to be developed. First database to implement Transactions, create and implement SQL (!). In a sep talk Kleppmann says that the typical types of isolation:
- Serialisable, repeatable reads, snapshot isolation, read committed, read uncommitted
Are all coined in this db also. 

1981: Gray paper (Gray worked on System R and involved a lot in the IBM research around this). Attempt to define a Transaction more clearly: 'A transaction is a transformation of state which has the properties of atomicity (all or nothing), durability (effects survive failures) and consistency (a correct transformation).'
So, missing the 'I', but this is implicitly included in consistency.

1983: Harder and Reuter paper where canonical definition of a Transaction is given via ACID acronym. ACID intended to provide necessary and jointly sufficient conditions for defining what a Transaction is. 


### The Move Away from Transactions


Late 2000s: NoSQL databases come onto the scene and prioritise availability and low latency over consistency. Transactions were a fatality of this. Transactions trade off consistency for slowing things down, and making the db more available. With rise of distributed computing Transactions were often what got cut.

2006: Google's BigTable
2007: Amazon's Dynamo (not to be confused with dynamoDb)(This is also the reference that Klepmann gives in Replication chapter on what pioneered leaderless replication).
2008: BASE paper (Basically available, Soft State, Eventual Consistency)

Then: 'NewSQL'. Transactional implemenations introduced into new distributed systems stuff. Seen as bridging the best of both worlds, to a degree. 
2012: Google's Spanner paper
2014: Cockroach DB

But NewSQL stuff flipped that narrative. CRDB, TiDB, Spanner, FoundationDB, YugabyteDb all incorporate Transactions and are highly scalabale databses. 


## Single Object and Multi-Object Operations

Multi-object transactions require some way of determining which read and write operations belong to the same transaction. In Relational DBs, this is typically done based on the client's TCP connection to the db server. 

But many non-relational databases don't have this ability. (Surely this isn't why they don't allow for MO transactions?)

### SO writes

A and I also apply when a single object is being changed. Example of writing a large json object. Storage engines almost universally aim to provide atomicity and isolation the level of a single object. Therefore the suggestion is that satisfying this criterion does not qualify for saying you support Transactions. 

### Need for MO transactions

Could we do without Multi-object writes? And just be safer and do everything single-object like NoSQL often does? List given of v plausible use cases where this wouldn't work:

- Row in 1 table has a FK reference to a row in another table.
- Dbs with secondary indexes, indexes need to be updated every time you change a value. Without transaction isolation, possible for a record to appear in one index but not in the other. 

### Handling Errors and Aborts

There are some ways it can go wrong when we simply re-try an aborted transaction.

- The re-try mechanism is only worth doing if you're facing a transitive error.

- IF the transaction actually succeeded, but the network was interrupted while the server tried to acknowledge the successful commit to the client, then retrying the transaction causes it to be performed twice unless you have an additional application-level de-dupe mechanism in place. 

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
'For every row that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the row are simply given the old value' (p293).

Some dbs support an even weaker level: Read uncommitted. Prevents dirty writes but doesn't prevent dirty reads. This can provide better perf (expected). However, says that it can reduce the probability of lost updates. 
Trying to get clear on this last bit. A few possible explanations jump out. But first, we should be clear that fundamentally a lost update is a read-modify-write race condition where two transactions will read the same value simultaneously, both update it, but then only the last to successfully commit their transaction will get the update. The first one to commit will have their update lost, hence the name.
(1) By avoiding dirty reads you will get more up-to-date data from your reads. You therefore reduce the risk of stale data and rely on the fact that most transactions will not fail, therefore increasing your odds of avoiding a lost-update but increasing the risk overall of other types of inconsistent data issues. 

### Snapshot Isolation and Repeatable Reads

In addition to the guarantees that Read Committed gives you, these Isolation levels also guarantee you against *read skew*. Read skew is a race condition that will be fixed if you re-read, but is a temporary bad read. 

Snapshot isolation is the most common solution to this problem. The idea is that each transaction reads from a consistent snapshot of the database - it sees all the data that was committed to the database at the start of the transaction. 

- To implement, typically uses write locks to prevent dirty writes. This blocks other writes. 
- But reads don't need locks. 
- Readers never block writers and writers never block readers. 
- Generally implemented using a generalisation of the mechanism for preventing dirty reads, but instead of just using 2 versions of each row, the db must potentially keep many versions. Hence MVCC - Multi-version concurrency control. 

So, we can say that MVCC is an implementation technique to achieve snapshot isolation. 

### Preventing Lost Updates

MVCC is a commonly use implementation technique for dbs and often used to implement SSI. 
