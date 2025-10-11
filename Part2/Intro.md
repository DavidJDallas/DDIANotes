<h1> Distributed Data</h1>

Part 1 essentially discusses concepts where we assume that we only have 1 machine involved. This part now assumes that we have multiple machines that are involved in storage and retrieval. 

There are many reasons why you may want to use multiple machines:
- Scalability

Spread the load across multiple machines.

Vertical scaling is far simpler. 

But the problem with vertical scaling is that the cost grows faster than linearly. A machine with twice as many CPUs, twice as much RAM, and twice as much disk capacity as another will usually cost significanlty more than twice as much. And due to bottlenecks, a machine twice the size cannot necessarily handle twice the load. 

A shared machine archteicture also tends to offer limited fault-tolerance.

- Fault tolerance/ high availability.

Application can work even if one machine goes down.

- Latency

If users are international, having servers in different locations. 

Horizontal scaling is also called shared-nothing architecture.

'While a distributed shared-nothing architecture has many advantages, it usually also incurs additional complexity for applications and sometimes limits the expressiveness of the data models you can use. In some cases, a single-threaded programme can perform significanlty beter than a cluster with over 100 CPU cores.'

<h2> Replication vs Partioning </h2>

The two common ways to distribute across multiple nodes:

(1) Replication

Keep a copy of the same data on several different nodes, potentially in different locations. Replication provides redundancy: if some nodes are unavailable, the data can still be served from the remaining nodes. Replication can also help improve performance.

(2) Partioning

Splitting a big db into smaller subsets called partitions so tha the diff partitions can be assigned to different nodes (sharding)