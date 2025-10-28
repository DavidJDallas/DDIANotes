<h1> Partitioning </h1>

When the data you're dealing with is too big to fit on one node, you'll need to partition it. This - partioning - is the focus of this chapter. Partioning involves splitting up the data into partions (sometimes called shards). Each partition becomes a small db of its own - in effect - although the db may interact with other partitions. 

The main reason to want to partition data is scalability. Large datasets can be distributed across many disks, and the query load distributed across many processors. 

Structure of the chapter:

- different approaches to partition large datasets and observe how the indexing of data interacts with partioning.
- Rebalancing.
- Overview of how database route requests to the right partition and execute queries. 

Typically, in scaling strategies, we would combine partiotining with replication. 

<h2> Partitioning of Key-Value Data </h2>

The end goal of partioning is to spread the data evenly across all nodes. This, naturally, is hard to get right. It's easy to end up with either hot-spots (too much activity in one node) or skewed (too much data in one node).

Splitting this randomly would do this somewhat effectively in theory, but then we have no way of knowing which node to query.

<h4> Key-range partition</h4>
If we assume we have a key for each row, which we always use to query by (obvs not so for many SQL queries) we can partition by key range. If we do this, within each partition we can keep keys in sorted order. The downside of key-range partioning is that certain access patterns can lead to hot-spots. 

<h4> Hash-key partition </h4>

Because of risk of skew and hot-spots, many databases use a hash function to determine the partition of a key. Why would a hash functiuon change anything? A good hash function will take skewed data and uniformyl distribute it. Doesn't need to be cryptographically strong either. This technique is good at distributing keys fairly among the partitions. 

BUT, with this way, we lose the ability to do efficient range queries. 

You can always end up with an edge case in these examples where one particular key can create hot-spots and/or skews. Imagine 1 user who gets huge amounts of traffic/data - even hashing won't prevent this since hashing is deterministic. 

<h2> Rebalancing </h2>

Things will change in the db that can lead to data and requests needing to move from one node to another. Regardless of partioning scheme, rebalancing is usually expected to meet some minimum requirements:

- After rebalancing, the load should be shared fairly between the nodes in the cluster.
- While rebalancing is happening, the database should continue accepting reads and writes
- No more data than necessary should be moved between nodes to make rebalancing fast and to minimise the network and disk I/O load. 

<h4> Strategies to Rebalance </h4>