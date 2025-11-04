<h1> Consistency and Consensus </h1>

321: 'The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees'. 

Most databases will offer some sort of eventual consistency. But this is often not enough. 

The chapter covers three main areas:

(1) Examining one of the strongest consistency models in common use: linearizability. Examination of pros and cons.
(2) Examine issues of ordering events in a distributed system (ordering guarantees).
(3) Explore how to atomically commit a distributed transaction, which leads on to the consensus problem.

<h2> Linearisability </h2>

Aka atomic consistency, strong consistency, immediate consistency, external consistency. The basic idea of linearisability is to make a system <i>appear</i> as if there were only one copy of the data, and all operations on it are atomic. 
In a linearisable system, as soon as one client successfully completes a write, all clients reading from the database must be able to see he value just written. 

this is quite different to serialisability:
- Serialisability : an isolation property of tranasctions - a guarantee that the transactions will be performed as if they are running sequentially. About multiple objects
- linearisability : recency guarantee on reads and writes of a reigster (an individual object).

Both combined are referred to as strict serialisability, or strong one-copy serialisability. Some db do do this, such as Google Spanner, FoundationDb. But Cockroach does acheive SS for transactions that touch overlapping key-ranges. 

<h3> Implementing linearisability </h3>

The most common approach to making a system fault-tolerant is to use replication. But out of these, the only way to guarantee linearisability is through consensus algorithsm. 

<h3> Costs of linearisability </h3>

Performance (latency and throughput), and hardware complexity sometimes. Always slower for writes. 

He discusses the CAP theorem, and says its unhelpful. 

336: If application requires linearisability, and some replicas are disconnected due to a network problem, then some replicas cannot process requests while they are disconnected - must either wait until the network problem is fixed, or return an error.

If application doesn'\t require linearisability, then it can be written in a way that each replica can process requests independently, even if disconnected from other replicas. In this case, replication can remain available in the face of a network problem.

When the CAP theorem is formally defined, is very narrow in scope. It only considers 1 consistency model (linearisability) and one kind oif fault (network partitions) or nodes that are alive but disconnectede. Thus doesn't have practical value for designing systems. 