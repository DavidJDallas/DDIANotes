# Chapter 10 Consistency and Consensus

We have two competing philosophies of how we obtain consistency in distributed systems.

(1) Eventual Consistency

The fact that a system is replicated is made visible to the application, and application developers are expected to deal with the inconsistencies and conflicts that may arise.

(2) Strong consistency (aka linearisability)

Applications should not have to worry about internal details of replication and the system should behave as if it were a single node. Expectedly, this has a performance cost. Also, some kinds of faults that an EC system can tolerate cause a SC system to go down. 


This chapter delves deeply into the strongly consistent approach, focusing on 3 areas:

- Precisely define linearisability.
- Look at the problem of generating IDs and timestamps.
- Explore how Distributed Systems can achieve linearisability while still remaining fault-tolerant. The answer to this is *consensus algorithms*.

## Linearisability

The idea behind linearisability is this: if you want a replicated database to behave as simply as possible, you should make it behave as if it were a consistent single node database. In other words, make a system appear as if there is only 1 copy of the data. 

The key abstraction behind Transactions are allowing the application developer to be able to assume that, depending on their Tx level, certian race conditions won't occur, they won't have partially written state, and they won't have to contend with various other oddities that non-transactional application code interacting with the database could potentially face. 

The key abstraction behind stronger forms of consistency are that you can assume that a database is a consistent single-node database, even though it is replicated. Remember that replciation is introduced in scaling systems in the first place because it helps for fault tolerance and also allows for faster I/O, with faster reads if you're using single-leader replication (more nodes to read from), and faster reads and writes if you're using leaderless or multi-leader replication (since application can now read and write from >1 nodes).

Linearisability is also known as atomic consistency, strong consistency, immediate consistency, or external consistency. 

The basic idea of linearisability though is to make a system appear as if there is only one copy of the data, and all operations are atomic.

403: 'In a linearisable system, as soon as one client successfully complete a write, all clients reading from the db must be able to see the value just written'. In other words, it's a recency guarantee.



405: 'If reads that are concurrent with a write can return either the old or the new value, then readers could see a value flip back and forth between the old and the new value several times while a write is going on'. That is not linearisability. The values can't k

406: You can also only ever move forward in time with linearisabilit.

There are various weaker levels of consistency models than linearisability. 

#### What makes a System linearisable?

                                        

Lineralisability vs serialisability: Both seem to mean that things can be arranged in a sequential order. But importantly, Txs concern reading and writing multiple objects

Linearisability is a guarantee on reads and writes of a register (an individual object)

### Relying on Linearisability

Areas that it's important

##### Locking and Leader Election

##### Constraints and Uniqueness guarantees
