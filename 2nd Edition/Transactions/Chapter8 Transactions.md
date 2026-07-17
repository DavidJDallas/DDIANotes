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

1975: System R started to be developed. First database to implement Transactions, create and implement SQL (!)

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

A and I also apply when a single object is being changed. Example of writing a large json object. Storage engines almost universally aim to provide atomicity and isolation the level of a single object. 

### Need for MO transactions

Could we do without Multi-object writes? And just be safer and do everything single-object like NoSQL often does? List given of v plausible use cases where this wouldn't work:

- Row in 1 table has a FK reference to a row in another table.
- Dbs with secondary indexes, indexes need to be updated every time you change a value. Without transaction isolation, possible for a record to appear in one index but not in the other. 

### Handling Errors and Aborts

There are some ways it can go wrong:

- IF the transaction actually succeeded, but the network was interruped while the server tried to ack the successful commit to the client, then retrying the transaction causes it to be performed twice unless you have an additional application-level dedupe mechanism in place. 



