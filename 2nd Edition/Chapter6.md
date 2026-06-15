# Replication

Replication by itself is sufficient if your data fits on one node/device. If it's bigger, you'll need replication plus sharding. 

All the difficulty in replication lies in handling *changes* to replicated data. If your data is read-only, replication is trivial. 

This chapter discusses three families of algorithms:

- Single-leader
- Multi-leader
- Leaderless

Almost all distributed systems use one of the above algorithms. 

There are trade-offs to consider within all of these. For example, whether they use synchronous or asynchronous replication, and how to handle failed replicas. 

Replications of databases is an old topic and the principles haven't changed a lot since the 1970s. The fundamental constraints of networks have remained the same. But concepts such as 'eventual consistency' still cause confusion. 

Back-ups and replicas are seperate issues, and usually work in compliment with each other.

## Single-leader replication

Each node that stores a copy of the db is called a replica. 

### Sync vs Async replication

Discussion of sync vs asycn replication. 
Sync: node waits for the other node to respond that the replication process was successful.
Async: node does not wait. 


Obvious trade-offs. Async is much faster, but less reliable and a node can crash without the other node being aware. 

It's impractical to have fully synchronous replication. Usually, if a db offers synchronous replication, it often means that *one* of the followers is synchronous. This can switch over - if the synchronous follower becomes slow or unresponsive, it can switch to another aysnc follower. This config is sometimes known as semisycnhronous. 

In some systems, a majority (say 3/5) nodes are update synchronously, with the remaining being async. This is an example of a quorum. Majority quoroums are often used in eventually consistent systems or systems that use a consensus protocol for automatic leader election. 

Async replication is widely used.

### Setting up New followers

How to ensure that the new follower has an accurate copy of the leader's data?
Simply copying the data from the leader or the node is not enough since, assuming you don't want to literally lock the database for the duration that this occurrs, you need to account for the newer updates since the replication happens. 

1. Take a consistent snapshot of the leader's db at some point in time. Most databases have this feature.

2. Copy the snapshot to the new follower node.

3. Follower connects to the leader and requests all data changes that have happened since snapshot was taken. Requires that the snapshot is associated with an exact position in the leader's replication log. 

The practical steps vary massively between databases. In some systems, the process is fully automated, whereas in others it can be a somewhat arcane multi-step workflow that needs to be manually performed by an admin. 
