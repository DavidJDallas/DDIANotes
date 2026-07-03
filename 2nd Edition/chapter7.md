# Sharding

When there's (i) too much data, or (ii) too much write thoroughput that a single node can't handle it, sharding is a good solution. 

REgarding (ii), I presume this would only be an issue for single-leader replication strategies. Since of course multi-leader and leaderless would not have built up pressure on 1 node for writes.

Sharding is usually combined with replcation. So, copies of each shard are stored on multiple nodes. 

## Pros and Cons of Sharding

- Primary reason for sharding is scalability. If the volume of data or the write throughput has become too much for a single node, allows us to spread that data and those writes across multiple nodes. 

- Replication is useful at both small and large scale; it enables fault tolerance and offline operations. Sharding, though, is a *heavyweight solution that is mostly relevant at large scale.* Often best to avoid sharding , and stick with a single-sharded db if possible. This is largely because sharding adds complexity. 

- Typically have to decide which records to put in which shard by choosing a partition key. All records with the same partition key are placed in the same shard. If you get this wrong, you have inefficient searches across all shards. Difficult to change.

- Often works well for key-value data, where you can easily shard the key. Harder with relational data, where you may want to search by a secondary index, join records that might be distributed across shards. 

- Another problem: a write may need to update related records in several shards. Ensuring consistency across multiple shards requires a distributed transaction, which are usually much slower. 

## Sharding for Multitenancy

Sometimes sharding is used to implement Multitenancy systems. Either each tenant is given a seperate shard, or multiple small tenants may be grouped together into a larger shard. 

Has several advantages:
- REsource isolation
- Permission isolation
- Cell-based architecture
- Per-tenant backup and restore
- REgulatory compliance
- Data residence
- Gradual schema rollout

Main challengs are:
- Assumes that each individual tenant is small enough to fit on a single node.
- If you have many small tenants, creating a seperate shard for each may be too much overhead. 


## Sharding of Key-value data

The goal of sharding is to spread the data and query load evenly across nodes. If this happens, then we have a linear scaling of nodes vs workload (10x as many nodes = 10x workload.)

If it's not fair, and some nodes are taking more work, we say it's *skewed*. This obviously makes sharding much less effective. 

A shard with disproportionaley high load is called a hot shard or hot spot. If 1 key has a particularly high load, we call it a *hot key*.

To do this properly - to split the dataset into shards - we need an algorithm that takes as input the partition key of a record, and tells us which shard contains that record. 

### Sharding by Key-range

We can assign a contigous range of partition keys to each shard. E.g. like the volumes of a paper encyclopedia. So, keys 1-10 belong in shard 1, 11-20 in shard 2 etc. Need not be evenly spaced keys, since data may not be evenly spaced. 

- shard boundaries might be chosen manually by an admin, or the db can choose them automatically. 

- Within each shard, keys are stored in sorted order. Makes range scans easy.

- Downside: you can easily get a hot shard if there are a lot of writes to nearby keys.

#### Rebalancing key-range sharded data

With databases that manage shared boundaries automatically, a shard split is typically triggered by the shard reaching a configured size. In some systems, the write thoroughput being above a certain threshold does it.

### Sharding by hash of key

Key-range sharding is useful if you want records with nearby (but different) partition keys to be grouped into the same shard. If you don't care if they're near each other, a common approach is then to has the partition keys before mapping it to a shard. 

To get this to correspond to an actual shard, we can do Hash modulos. We use the modulo operator by say, 10, on it, and it returns  a number between 1 and 10. The problem with this is that if the number of nodes N changes, most of the keys have to be moved from one node to another. This is costly. It's easy to compute, but requires lots of rebalancing, typically. 

A simple but widely used solution is to create many more shards than there are nodes and assign several shards to each node. In this model, only entire shards are moved between nodes. So still some cost, but not as bad. This approach used in Citus (sharding layer for PostgreSQL), Riak, Elasticsearch, and Couchbase. Works as long as you have a good estimate of how many shards you will eventually need. If this is wrong, you need an expensive resharding operation.

If the required number can't be predicted in advance, it's better to use a scheme in which the number of shards can adapt easily to the workload.
We can combine key-range sharding with a hash function so that each shard contains a range of hash values rather than a range of keys. 


### Skewed workloads and releiving hot spots

### Operations: Automatic vs manual Rebalancing

## Request Routing

## Sharding and Secondary Indexes





























