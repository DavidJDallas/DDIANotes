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

403: 'In a linearisable system, as soon as one client successfully complete a write, all clients reading from the db must be able to see the value just written'. In other words, it's a recency guarantee.

Lineralisability vs serialisability: Both seem to mean that things can be arranged in a sequential order. But importantly.