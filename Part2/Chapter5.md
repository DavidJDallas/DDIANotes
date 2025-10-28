<h1> Replication </h1>

To do replication, your dataset needs to be small enough so that each machine can hold a copy of the entire dataset. If not, then you need to do sharding/partioning. Thus, replication has more constraints than sharding and has specific conditions under which you can do it.

If the data that you're replicating is read-only, then replication is easy. You just copy the data to every node, and done. *All of the difficulty in replication lies in handling changes to the data (writes)*. This is therefore what the chapter is about. 

Discussion of popular algorithms for replicating changes between nodes - single leader, multi leader, leaderless. Almost all distributed databases use one of these 3 approaches. 

<h2>Leaders and Followers</h2>

Each node that stores a copy of the db is called a replica. When we have multiple, we have to ask the question: how do we nesure that all the data ends up on all the replicas?

Most common solution: leader-based replication (active-passive/master-slave).
One replica denoted the leader, others denoted the followers. 

Whenever reader writes new data, also sends the data changes to all followers. Each follower takes the log from the leader and updates local copies of the db accordingly, in same order. 

When a client wants to read from the db, it can query either leader of followers. However, writes are only accepted on the leader. 

<h3> Synchronous vs Async replcation </h3>

