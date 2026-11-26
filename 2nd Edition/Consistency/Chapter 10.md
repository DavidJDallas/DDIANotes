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


To recap: Linearisability concerns single operations on single objects. It's a guarantee about the order of things across distributed systems. 

- Serialisability concerns > 1 operations on database Transactions.
- Guarantees that for N number of Transactions running concurrently on N number of rows, the database will handle it (ignoring implementation details) as if those Transactions were run one after the other. 
- There is no guarantee in serialisability alone that your Transactions would run in any particular order, or that the first Transaction would go first, etc. 

So: Serialisability does not make guarantees about precise ordering between the N number of transactions running concurrently (what will get processed first), or about how your application will read the writes. 

- Linearisability doesn't make any guarantees about how your Transactions will be run, or that Transactions won't interleave and cause race conditions. 


- It seems that this murkiness comes from the research around them being done on 2 different communuties: The database community creating their own language, phrasing and concepts, and the Distributed Systems community creating their own language, phrasing, and concepts.
- The database person cares about what happens in the database, and cares about correctness and performance within this system.
- The DS person cares about what happens when different nodes fail, and how to preserve correctness in this case.
- The language they both develop is different. 


|                         | Databases                                     | Distributed systems                                          |
|-------------------------|-----------------------------------------------|--------------------------------------------------------------|
| Unit of work            | Transaction (many operations)                 | Single operation                                             |
| Failure model           | One machine crashes and recovers from disk    | Partial failure: some nodes or links fail while others continue |
| What "correct" means    | Invariants are preserved                      | Replicas agree; what observers see is coherent               |
| Time                    | Some serial order, not necessarily real time  | Real time, or causality                                      |
| Main cost worried about | Throughput, contention                        | Availability, latency, coordination                          |
| How guarantees are defined | By the anomalies they forbid (dirty read, write skew) | As properties of histories that must hold (a total order exists...) |


| Word        | Database meaning                                              | Distributed-systems / concurrency meaning                          |
|-------------|---------------------------------------------------------------|--------------------------------------------------------------------|
| Consistency | ACID "C": application invariants hold                         | Replicas agree, reads aren't stale (CAP, consistency models)       |
| Atomic      | ACID "A": all-or-nothing on failure                           | Indivisible to observers; takes effect at one instant (atomic registers) |
| Partition   | Sharding: splitting data across nodes                         | Network partition: nodes that can't communicate                    |
| Serializable | Strongest standard isolation level                           | In etcd: a cheaper, possibly stale read mode                       |
| Commit      | Transaction is durable and visible                            | Raft: log entry is held by a majority of replicas                  |
| Snapshot    | Snapshot isolation                                            | Consistent snapshot of global state (Chandy–Lamport)               |


For more on this , [Kemme and Alonso](https://www.comp.nus.edu.sg/~vldb2010/proceedings/files/papers/TY02.pdf)


## Implementing Linearisability

Klepmann walks through each type of replication to assess if can be made linearisable:

#### Single-leader

- As long as you perform all writes on the leader, it's likely to be linearisable. *But* this assumes that you know who the leader is. Quite possible for a node that's not the leader to think that it is, and this would likely violate linearisability. 

- Sharding a single-leader db would not affect linearisability. But cross-shard Txs are a different matter.

#### Multi-leader 

- Multi-leader is *not linearisable*, because they concurrently process writes on multiple nodes and asyncrhonously replicate them to other nodes. So, they can produce writes which require resolution.

#### Leaderless 

- *Probably* not linearisable.

- Leaderless replication. Some claim it is, but doesn't seem to be true. Even with quorums, nonlinearisable behaviour is possible due to things like network delays. It may be the case, unavoidably, that a write that was submitted by the client earlier in time isn't processed properly until after another write submitted later. 

### Consensus algorithms

- Consensus algorithms are a service that the above can use in order to achieve consensus amongst the nodes as to determining things like who the leader is, who is in the cluster, or who holds a lock, etc. 
- But they also use replication themselves, and are therefore can be looked at from this angle.

Likely linearisable. Some consensus algorithms are essentially single-leader replication with automatic leader election and failover. Carefully designed to prevent split-brain, allowing them to implement linearisable storage safely. 

## The Cost of Linearisability

- Network partitions: The network within each region is working, but the regions can't talk to each other.
- For single-leader, this will fall over unless the leader is in the same region as all the replicas. 
- For multi-leader, each region can continue operating normally, assuming that the leaders are split across regions. 

- But more broadly, this failure issue doesn't need specifically a network partition - it will have the same symptoms if you have any sort of network failure. 

Here is the trade-off:

- If you require linearisability, and some replicas get disconnected for network reasons, the replicas will be temporarily unable to process requests. Sometimes know as 'CP' or 'Consistency under network partition'.
- If you *don't* require linearisability, you can process requests independently, even if disconnected. In this case, the application can remain available under network issues, but its behaviour is not linearisable. Known as AP, 'availability under partition'.

Klepmann is critical of CAP (as formally defined?) because:
- Considers only 1 consistency model (linearisability)
- Considers only 1 kind of fault (network partitions, which are quite rare)

There have been efforts to generalise CAP.


#### Linearisability and Network Delays


- Very few systems are linearisable in practice.
- Often this is done for performance reasons, not for fault tolerance. 
- This seems like a hard limit, not just an optimisation problem. E.g. Attiya and Welch prove that if you want linearisability, the response time of read and write requests is at least proportional to the uncertainty of delays in the network. 


## Id generators and logical clocks

There may be use cases in your application where you want an id generator that auto-increments to show a clear increase and identify each new item in a clear order.

Single node, in-memory generators are easy. But
- They're not fault-tolerant
- They're slow if you want to create a record in another region, as you potentially have to make a roundtrip to the other side of the planet just to get an ID.
- That single node could become a bottleneck if you have high write throughput. 

### Alternatives for Id generators

- Sharded Id assignment
- Preallocated blocks of Ids
- Random UUIDs
- Wall-clock timestamp made unique

All generate Ids that are unique, but they have much weaker ordering guarantees for Ids than the single-node autoincrementing scheme. 

- Wall-clock time can provide at best an approximate ordering. 

Klepmann presents 'logical clocks' as a solution here. 
### Logical Clocks

These are common in distributed systems. A logical clock 'is an algorithm that counts the events that have occurred'. It won't tell you what time it is, but you can compare timestamps between two events from a logical clock and it will tell you which one is earlier. 

Requirements for a logical clock:

- Timestamps are compact (a few bytes in size) and unique
- You can compare any 2 timestamps and determine which one is earlier (i.e. totally ordered)
- The order of timestamps is consistent with causality

#### Lamport Timestamps


Lamport clocks don't provide linearisability but do provide a total ordering. So, they are objectively orderded, but may not be subjectively correct i.e up to date, and therefore may look out-of-order for individual clients. 

- Lamport timestamp is simply a pair of (counter, node Id). 2 Nodes may sometimes have the same counter value, but by including the node ID in the timestamp, each timestamp is made unique. 

### Hybrid Logical Clocks

Limitations to LTs.

- No direct relationship to physical time and so you can't use them to find e.g. all messages posted on a particular date.
- If 2 nodes never communicate, one node's counter increment will never be reflected in the other one's counter. As a result, events generated around the same time on different nodes could have v different counter values.

Hybrid logical clocks combine the advantages of a physical time-of-day clock with the ordering guarantees of Lamport clocks. 

- Counts seconds or microseconds, and when 1 node sees a timestamp from another node that is greater than its clock value, moves its own local value forward to match the other node's timestamp. 

- Every time a timestamp from a hybrid logical clock is generated, it's also incremented, which ensures that the clock moves forward monotonically even if the underlying physical clock jumps backward.

## Consensus

426: 'The standard formulation of consensus involves getting multiple nodes to agree on a single value.'

### FLP result

From the paper "Impossibility of Distributed Consensus with One Faulty Process". 

- In a fully async system, no deterministic algorithm can guarantee consensus if even 1 process might crash.

Conditions:

- Asynchronous network. (i) Messages must always be eventually delivered, but no requirement on how long they take. (ii) no shared clock. (iii) no limit on how slowly they run.
- Crash failures. At most one process can stop without warning. No lies or byzantine behaviour, just a crash. 
- Consensus: Every process starts with a value, and the algorithm must meet three conditions: (i) Agreement amongst nodes (ii) Validity: decided value was one of the starting values, (iii) Termination: every non-fault process eventually decides.

FLP provdes that no deterministic protocol can meet all 3 conditions in every possible execution. 

The basic problem: *You can't tell a crashed process from a slow one*. Wait for a process and it might be dead - you wait forever. Move on without it - may be alive, and holding information that decides the outcome.


FLP doesn't say consensus is impossible in practice, or that it usually fails. It says there's no guarantee that it terminates under the worst-case timing.

Remember too that FLP assumes 3 conditions. The real systems, with proper proofs, work around it by dropping one of FLP's assumptions.

- Partial synchrony. Assume the system eventually behaves synchronously. E.g. Paxos and Raft. 
- Failure detectors: add an oracle thats allowed to be wrong for a while. 
- Randomisation: Use coin-flips. Termination then happens with probability 1 rather than in every execution.

In an async system with failures, you can guarantee safety, but you have to give up guaranteed liveness. 

### The Many Faces of Consensus

We can express consensus in several ways. E.g.

- Single-value consensus.
- Constructing an append-only log also requires consensus. Usually formalised as a total order broadcast. With a log you can implement state machine replication, leader-based replication, event sourcing, etc.
- An atomic fetch-and-add (atomic increment) operation also is equivalent to consensus.
- Atomic commitment of a multidatabase or multishard Tx is equivalent to consensus, in that it requires all participants agree on whether to commit or abort the Tx.


#### Single-Value Consensus

Getting all nodes to agree on a single value. 1 or more nodes may propose values, and the consensus algorithm decides on one of those values. 
So, a consensus algorithm must have the following properties:

- Uniform agreement. No 2 nodes decide differently.
- Integrity. After a node has decided one value, it cannot change its mind by deciding another value. 
- Validity: If a node decides value v, then value v was proposed by a node.
- Termination: Every node that does not crash eventually decides a value. 

Termination is a liveness property, whereas others are a safety property. 

#### Compare and Set as Consensus

