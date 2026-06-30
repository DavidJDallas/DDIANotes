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

- If aysnc replication is used, the new leader may not have received all the writes from the old leader. Most common solution is to discard them (means no durability.) Especially dangerous if other storage systems outside of the db need to use them. 
- Two nodes could both believe they are the leaders. Split brain. If both leaders accept writes, there is no process for resolving conflicts. Data likely to be lost or corrupted. 

### Implementation of Replication Logs

Several replication methods are used in practice. 

#### Statement based replication

Leader logs ever write request that it executes and then sends that statement log to its followers. E.g. ever INSERT, UPDATE, DELETE is forwarded to followers. This can be risky if:
- It's non-deterministic.
- Statements use auto-incrementing columns, or depend on existing data. 
- Statements have side-effects (triggers, SPs, user-defined funcs).

Was used in MySQL before 5.1 VoltsDB uses it, and makes it safe by requiring transactions to be deterministic. 

#### Write-ahead log shipping

In addition to writing the log to disk, the leader also sends it across the network to its followers.

### Problems with replication lag

Being able to tolerate node failures is important, but just 1 part of why you'd want replication. It's also very helpful for scalability and latency. The former because it allows us to process more than 1 node can handle, and the latter because we can potentially place nodes closer to users, geogprahically. Can be called read-scaling architecture.

So: if your application is mainly read-only, replication like this will massively help to scale you upwards. But this approach only really works for asynchronous replication; if you tried to make it synchronous, all replications to the followers would fall apart if you had a single node outage and make the whole system unavailable to write to. But an async system may see outdated information if the follower has fallen behind. This can lead to apparent inconsistencies - two queries to the database at the same time can be different if they hit different nodes. These inconsistencies are temporary. This is known as *eventual consistency*.

#### Reading your own writes

Read-after-write consistency. A guarantee that if the user reloads the page, they will always see any updates they submitted themselves. 

It seems that eventual consistency is the weakest sort of consistency, followed by read-after-write, for replication stuff. 

There are various possible techniques to implement ready-after-write consistency:

- when reading something that the user may have modified, read it from the leader or synchronously updated followers. This does require though that you know who wrote the data. If most things in the application are potentially editable by the user,this approach won't be effective (most things would have to be read from the leader)

More complications arise when one user can access your application from multiple devices. 


#### Monotonic reads

If reading from asynchronous followers, it's possible to see things moving backwards in time. This can happen when the same user makes the same query twice, and hits different nodes on each read. For this to happen, the first read would be updated, and the second read would not be updated.

Guaranteeing against this is called "Monotonic reads". Stronger guarantee than eventual consistency, weaker than strong consistency. 

#### consistent prefix reads

Looks like things move outside of a causal chain. Similar to monotonic in essence but with causality instead of time. E.g. An entity that is only sensible if caused by another entity happens before the causal entity. 


### Solutions for replication lag

There are ways for an application to provide stronger guarantees than the database itself. However, dealing with these kinds of issues in application code is complex and easy to get wrong. 

'The simplest programming model for application developers is to choose a database that provides a strong consistency guarantee for replicas, such as linearisability, and supports ACID transactions. This allows you to mostly ignore the challengs that arise from replication and treat the db as if you just had a single node'.

Interestingly, the counter to this came from the NoSql movement in 2010s, which said that the above model would not work for scalability purposes. But since then, a number of systems - newSQL - have started providing strong consistency and transaction support while also offering fault tolerance high availability and scalability advantages. 


## Multi-leader Replication

The obvious big downside of single-leader replication is that all writes have to go through one node (the leader). And if you can't connect to the leader, you can't connect to the database. 

Section focuses on asynchronous replication. 

### Geographically Distributed Operation

Rarely makes sense to use a multi-leader setup within a single region. You would usually use multi-leader spread over various geographical locations.

Performance: May be better for multi-leader. Or at least has potential to be. 

Tolerance of regional outages: Much easier with multi. If single-leader goes down, have to start failover process. This will solve it (promote a leader from another region) but not ideal. In multi-leader across multi-regions, works fine with one going down.

Tolerance of network problems: Multi-leader can tolerate network problems better.

Consistency: Single-leader wins. Far stronger consistency guarantees. Multi-leader simply can't provide same level of consistency guarantees.

It therefore seems that a key trade-off between single vs multi-leader is trading off performance and availability with consistency. Single-leader can't provide the same promises of performance and uptime, but can provide better consistency. 

Multi-leader is less common. 

Multi-leader replication is a retrofitted feature in many dbs. Consequently there are often subtle config pitfalls and issues. For this reason, multi-leader is often considered dangerous territory. 

### Multi-leader replication topologies

A replication topology describes the communication paths along which writes are propagated from one node to another. 

- Circular
- Star
- All-to-all

All-to-all is most general. All leaders send all their writes to all other leaders. 

### Problems with different topologies

Circular and star: if 1 node fails, can interrupt the flow of replication messages between other nodes. 

Fault tolerance of all-to-all is better. Therefore maybe seems like A2A is more in the spirit of multi-leader. 
A2A: Some network links may be faster than others, meaning that some replication messages may overtake others.


### Sync engines and local-first Software

Multi-leader replication is also appropriate if you have an internet application that needs to continue to work while it's disconnected from the internet. The local device is a leader, can write, then when it comes online it syncs between the replicas of your calendar on all devices/the cloud.

So, architecturally, this is v similar to multi-leader replication between regions. Each device is a region, and the network connection between them is extremely unreliable. 

Live-edit applications like figma, google docs etc will also employ this style of replication. Each web browser tab that has opened the shared file is a replica, and any updates that you make to the file are async replicated to thje devices of the other users.

### Dealing with Conflicting writes

Biggest problem with multi-leader is that concurrent writes on diff leaders can lead to conflicts that need to be resolved. 

#### Conflict avoidance. 

The idea: prevent them from happening in the first place. I.e. at the application level. 

#### Last Write wins

If conflicts can't be avoided, simplest way of resolving is to attach a timestamp to each write and use the value with the most recent timestamp. 

#### Manual Conflict Resolution


## Leaderless Replication

Takes the availability element of multi-leader to its logical extreme. Heavily inspired by Dynamo, and so can sometimes be called dynamo replication. Interestingly, it's noted that Dynamo db is not not dynamo. Dynamo db is single-leader and is Amazon's in-house no-sql db.  

Cassandra, Scylla and Riak all use leaderless, inspired by 

The general idea for leaderless is that writing is massively sped up. You can write to any node. But the reads are slowed down, and configurable depending on how safe you want to be. If you're not too bothered about reading data that is definitely correct, you can configure it for faster reads. If you need correctness, you need to have slower reads since you'll need to verify the read against more nodes. 















