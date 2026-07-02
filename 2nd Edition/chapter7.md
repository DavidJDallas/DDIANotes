# Sharding

When there's (i) too much data, or (ii) too much write thoroughput that a single node can't handle it, sharding is a good solution. 

REgarding (ii), I presume this would only be an issue for single-leader replication strategies. Since of course multi-leader and leaderless would not have built up pressure on 1 node for writes.

Sharding is usually combined with replcation. 
