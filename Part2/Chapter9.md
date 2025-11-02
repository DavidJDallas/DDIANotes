<h1> Consistency and Consensus </h1>

321: 'The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees'. 

The chapter covers three main areas:

(1) Examining one of the strongest consistency models in common use: linearizability. Examination of pros and cons.
(2) Examine issues of ordering events in a distributed system (ordering guarantees).
(3) Explore how to atomically commit a distributed transaction, which leads on to the consensus problem.