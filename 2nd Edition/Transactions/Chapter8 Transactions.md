# Transactions

In case it's not clear, all Transactions are necessarily implementing ACID. ACID is a specific guarantee that Transactions give you.

The two main things that a transaction prevents against are:
- A failure at some point in your workflow that causes your database operation to fail mid-process, where one of those events in the process is a write.
- Concurrency control.



The first comes under the 'A' of ACID - Atomicity. Could also be called 'all-or-nothing' and have a good similar affect. But essentially all of your operations are bunched into a single unit and succeed or fail together. This allows for consistency guarantees and for you to be able to reason about things much easier. 

The second comes under the 'I' of ACID - Isolation. 

## The History

### The Origin of Transactions
Transactions and the idea of Transactions predate the ACID acronym by about 10 years. I got curious about the origins of this in computing and it seems that the first proto-usage of the term happens in the 1973 paper by Davies and Bjork, who discuss the idea of 'spheres of control'. A sphere enclosed a piece of work whos effects could be:
- Committed
- Backed out
- Isolated from surrounding work
- nested inside another sphere

In 1974-1976 the word Transaction is explicitly defined across a couple of papers which captures the essence of this. System R uses this notion and implements, alongside with SQL. 

Over late 70s into early 80s, the term begins to have a bit more treatment by people attempting to come up with a canonical definition. For example Gray's 1981 paper defines a Transaction as:

'A transaction is a transformation of state which has the properties of atomicity (all or nothing), durability (effects survive failures) and consistency(a correct transformation).' (1981, Abstract).

In this paper Gray does include what we now know as Isolation, but it falls under consistency here. Maybe this is more consistent with other elements of Computer science?

ACID coined in the Harder and Reuter paper and has become the canonical usage now of how we define Transactions. Where ACID are defining properties of a Transaction such that to perform a Transaction is necessarily to adhere to ACID criteria. 

### The Move Away from Transactions


Late 2000s: NoSQL databases come onto the scene and prioritise availability and low latency over consistency. Transactions were a fatality of this. Transactions trade off consistency for slowing things down, and making the db more available. 

But NewSQL stuff flipped that narrative. CRDB, TiDB, Spanner, FoundationDB, YugabyteDb all incorporate Transactions and are highly scalabale databses. 

## What is ACID?

Says its a bit of a gimmick now.

- Atomicity: All the items in your transaction will be written or fail together.
- Consistency: Guarantee that your data is accurrately captured onto disk, basically. Anything else is up to the application level.
- Isolation: Concurrently executing transactions are isolated from each other. 
- Durability: Data is persisted to disk if it returns a successful message. Nowadays maybe also is replicated.

C isn't really a condition. D is pretty basic in terms of whether or not you have it. 

There's a little bit of ambiguity around Atomocity. And the book reflects that. There's a few sections on what this means: 
- Single-object or multi-object Operations. (Combined with how this ties into isolation levels too.)

But really, the bulk of this chapter is for the I in ACID (Isolation). 

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



