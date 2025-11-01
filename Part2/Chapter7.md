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





Isolation levels vs concurrency control

Isolation levels are what you can guarantee. Serializable is the strongest level, and transactions behave as if they ran one at a time, in some order. There'll be no anomalies, no lost updates, no dirty reads, etc. 

The concurrency control is about the mechanism you use to provide that isolation guarantee. Typically either optimistic concurrency or pessimistic. Pessimistic assumes conflicts will happen and prevents them upfront; optimistic assumes conflicts are rare, detects them at the end. 