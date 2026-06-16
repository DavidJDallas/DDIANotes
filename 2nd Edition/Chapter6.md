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

Brief section on using object storage as a backup. Benefits are the cheapness, and the ability to store massive amounts of data straightforwardly.

Drawbacks are that there's no real 'brain' behind it. We can't use it like a normal db that would have a search engine, etc.

### Handling Note Outages

Our goal of running a good distributed system is to be able to keep the system going despite node outages. Node outages will be somewhat inevitable - either through failures, or we have to perform updates.

So the question is: how do we handle node outages? How do we ensure the system continues to run smoothly when this happens, and how do we get the nodes back up?

The answer will differ depending on what node fails, the leader of the follower.

#### The Follower Node

The follower knows from its log where the last piece of data it processed was. So the follower can just connect to the leader again and request all data from where it last got to before it went down. 

This is conceptually very simple. But it can be a bit more tricky practically in terms of performance. If the db has a high write thoroughput or the the node has been down a while, it hits performance. 

There's also a general question about the leader deleting its log of writes. Generally it can do this after all followers have it, but if one follower is down for ages, does it still delete? Trade-offs both side.

#### The leader node

Trickier. To do this:

- Follower needs to be promoted to leader.
- Clients need to be reconfigured to send their writes to the new leader
- Other followers need to start consuming data changes from the new leader.

This is called failover. It can happen manually or automatically. If automatic, it would usually happen as follows:
(1) Determining that the leader has failed.
(2) Choosing a new leader
(3) Reconfiguring the system use the new leader.

Failover is full of things that can go wrong. 
