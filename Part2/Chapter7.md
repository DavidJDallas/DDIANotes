<h1> Transactions </h1>

In software engineering, things can and do wrong. Sometimes catastrophically. Transactions make things safer, and whilst they don't prevent failures, they prevent partial failures or partial success. 

'With transactions, error handling becomes much simpler for an application, because it doesn't need to worry about partial failure'. 

Transactions were the main casualty of the NoSql movement into default replication and partioning. Many NoSQL databases abandonded transactions, or redefined it to describe a much weaker set of guarantees. 

<h2>The Meaning of ACID </h2>

- Atomicity
- Consistency
- Isolation
- Durability