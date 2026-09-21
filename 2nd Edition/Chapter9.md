# The Trouble with Distributed Systems

345: 'If you want your system to be reliable in the presence of faults, you have to radically change your mindset and focus on what could go wrong, even though it may be unlikely.  

## Faults and Partial Failures

There is no fundamental reason that software on a single computer should be flaky. Single computers are designed to either work, or fail. We prefer a computer to crash completely rather than returning a wrong result, because wrong results are difficult and confusing to deal with. 
But it does happen that computers can be non-deterministic. SEe: https://research.google/pubs/cores-that-dont-count/

When writing software that runs on several computers, connected by a network, the situation is fundamentally different. Faults occur much more frequently. 

PArtial failure: Some nodes of the system are working fine, others aren't. 

This non-determinism and possibility of partial failures is what makes distributed system hard to work with.

I think there are parts of single systems that are very similar to distributed systems, and some that are fundmaentally different, creating a difference in-kind. 

First, single systems are, in a way, distributed. We take many parts of the computer, and tie them together via increasing levels of software, and present the engineering task (successful, largely) of writing it so that when we get a failure, we crash rather than produce determinism. It seems then that there's a way in which, partially due to reduced complexity, we have just pulled of a better engineering feat on the same type of system here. 

But there's also a way in which the two are fundamentally different.
(1) DSs will almost always optimise to carry on, rather than crash, like the computers. It would be a terrible engineering decision from, say, Google, to choose to crash their whole system because one node has gone down. Therefore the choice is not one of crash vs keep going and risk indeterminism/unpredictabiulity, it's crash vs keep going. In other words, we've tried to engineer the risk out of it and write algorithms to deal with partial failure as totally normal. 
(2) Single nodes share state in a simple way that DSs simply can't. Clocks, power supplies, etc. 

'Shared fate'.

(2) causes 1.

## Unreliable Networks

The internet and most internal networks in datacenters are asynchronous packet networks. Here, one node can send a message (a packet) to another node, but the network gives no guarantees as to when it will arrive or whether it will arrive at all. If you send a request and expect a response, many things could go wrong:

- Request may have been lost.
- Request may be waiting in a queue.
- Remote node may have failed.
- Remote node may have temporarily stopped responding.
- Remote node may have processed your request, but response has been lost on the network. 

The sender can't even tell whether the packet was delievered, and the only option is for the recipient to send a response mssage, which may in turn be lost or delayed. In short, if you send a request to another node and don't receive a response, it's impossible to tell why. 

Usual way to handle this is a timeout. 

## The Limitations of TCP

Network packets have a max size of generally a few kb. But many applications need to send messages that are > than this. Usually, TCP is used for this. This establishes a connection that breaks large data streams into individual packets and puts them back together again on the receiving side. 

Described as reliable because it:
- detects and retransmits dropped packets
- Detects re-ordered packets and puts them back in the correct order.
- Detects packet corruption by using a simple checksum.

Also figures out how fast it can send data so that it's transferred as quickly as possible, but without overloading the network or the receiving node.

The only way to be sure that a request was successful is to receive a positive response from the application itself. 

Nevertheless, TCP is v useful because it provides a convenient way of sending and receiving messages that are too big to fit in 1 packet. 

## Network Faults in Practice

Whenever any communication happens over a network, it may fail. There's no getting around it. 

The term network partition is often used when one part of the network is cut off from others. 

## Fault Detection

Many systems need to automatically detect faulty nodes. E.g.:

- Load balancer needs to stop sending requests to a faulty node/dead node.
- If single-leader replication, need to initiate failover. 

But it's hard to tell whether a node is working or not, due to the above point that the only way to be sure that a request was successful is to receieve a positive response from the application itself. There are however specific circumstances where you may get feedback:

- If you can reach the machine where the node is running, but no process listening (because it crashes), the OS will close or refuse TCP connections by sending an RST or FIN reply.
- If a node process crashed but the node's OS is still running, a script can notify other nodes about the crash so that another node can quickyl take over. 

352: 'If something has gone wrong, you may get an error response at some level of the stack, but in general you have to assume that you will get no response at all.Since the node could actually be alive, you need to work out a balance between false positives and false negatives. Too short a timeout causes alive nodes to be incorrectly suspected to be dead, and too long a timeout causes unecessary delays waiting for dead nodes. 

### Network Congestion and Queuing

The variability of packet delays on computer network is most often due to queuing.

Sometimes people prefer the UDP protocol, compared to TCP. UDP is faster, and doesn't re-transmit lost packets or perform flow control. Good choice when delayed data is wortless (streaming a call, streaming a show).

Queueing delays have an especially wide range when a system is close to its max capacity. 

- Instead of using constant timeouts, systems can continually measure response times and their variability, and automatically adjust timeouts according to the observed response time distribution. Phi Accrual failure detector is one way of doing this. TCP retransmission timeouts work similarly. 

## Synchronous vs Asychronous Networks

Compare datacentres to the traditional fixed-line telephone network. The latter is extremely reliable, delayed audio frames and dropped calls are very rare. Why can't we do that in computer networks?

Calls establish a circuit, and a fixed amount of bandwidth is allocated for the call along the entire route. Remains in place until the call ends. Each side is guarntee to be able to send exactly 16 bits of audio data for every 250 microscends. 

This is a synchronous network - the space has already been reserved for it and doesn't need to queue. We call this a bounded delay.

TCP packets in their connection opportunistically use whatever network bandwidth is available. You can give TCP a variable sized block of data and it will try to transfer it in the shortest time possible. Etherner and IP are packed-switched protocols, which suffer from queueing and thus *unbounded delays* in the network. 

This is because they are optimised for bursty traffic. Requesting a web page, sending an email, or transferring a file doesn't have any particular bandwidth requirement, we just need it ASAP. If you wanted to transfer a filee over a circuit, you'd need to guess a bandwidth allocation, and if you guess too low or too high it's problematic. 

There have been some attempts to build hybrid networks that support both circuit switching and packet switching. ATM was a competitot to Ethernet in 1980s, but didn't take off. And there are various QoS mechanisms around. But these are not enabled in multi-tenant datacentres and public clouds, or when communicating via the internet. 


## Unreliable Clocks

- Applications depend on clocks in various ways. 
- Durations: Has this request timed out? What's the 99th percentile response time?
- Points-in-time: When does this cache entry expire? What is the timestamp on this error message?

Main takeaway from this section: When we work on Distributed Systems, and use clocks, it's hard to reliably keep these clocks in-sync. 

We can do it, to a greater or lesser extent, though. There's a sliding trade-off between the more effort we want to spend on this, vs accepting the inaccurracy and building systems that work-around this. 

Most of the chapter is spent accepting that we can't trust clock-syncing, and exploring what this means for a DS. 


### Monotonic vs Time-of-Day clocks (Overview of the terms)

Most computers now have at least a (1) time of day clock, (2) monotonic clock.

#### Time of Day

'Normal' clock behaviour - gives you the time. 

- Usually synced with NTP, which means that a timestamp on one machine means same as another machine. 
- BUT they have various oddities. Inc jumping about in time. 
- Can also jump due to start and end of DST.

#### Monotonic

Suitable for measuring a duration. More like a stopwatch.

Issues:
- On a server with multiple CPU sockets, may be a seperate timer per CPU. May not be synced with other CPUs and dangerous to assume they will be. 

But, *usually fine in a DS to use a monotonic clock for measuring elapsed time*. Why? Because doesn't assume any syncing between different nodes' clock, and not sensistive to slight inaccuracies.

### Clock syncing and Accuracy

*ToD clock sycning is not as reliable as you'd hope.*

Expanding on this:
- Quartz clock in a typical computer drifts, depending on the temperature of the machine. approx 17 seconds if you re-sync once a day.
- If a computer clocks drifts too much from an NTP server, may refuse to sync, or be forcibly reset. 
- If a node is accidentally firewalled from NTP servers, misconfig may go unnoticed for some time. Seems like this does happen.
- NTP syncing is only as good as the network delay (congested network makes syncing worse)
- Leap seconds crash many large systems. (Leap seconds retired from 2035)

*BUT*

- If we *really* want to do it, we can.
- E.g. The MiFID II European regulation for financial institutions requires all high-frequency trading funds to sync their clocks to within 100 microseconds of UTC. 
- You'd achieve that with special hardware (GPS receivers/atomic clocks), Precision Time Protocol (PTP), careful deployment and monitoring. 

### Relying on Synced clocks

This sub-section, the rest that follows, assumes that (a) we're not going to do PTP, and that (b) we're going to use ToD clocks.

In short, we need to learn to deal with incorrect ToD clocks. 

- Part of the issue is that they're easy to miss that they're wrong. 
- Node goes down - obvious. Clock drift - often not obvious until way too late. Silent data corruption, etc.

- If you use software that requires synced clocks between nodes, it needs to be monitored, and declared dead quickly if it drifts too far.

##### Timestamps for Ordering Events

Conditions for this problem:

(1) > 1 close-together writes happening to the same key. (Close enough that the gap is smaller than the clock-skew)

(2) The writes are timestamped by different clocks (i.e. accepted on different nodes, or stamped by different clients).

(3) The system decides the order by comparing those timestamps.

- This is a particularly problematic issue for databases with multi-leader (e.g. couch db) and leaderless (e.g. Cassandra) replication, because (3) is the default option/common choice.

- Single-leader replication dbs can still fall into this, but it's rarer. 

**TL;DR**

- Where we have two different writes happening close enough together, multiple nodes, with slightly out-of-sync clocks, then a write that comes after another write can actually be before it, according to the clock, even if it logically follows it. 

- Can be prevented by ensuring that when a value is overwritten, the new value always has a higher timestamp than the overwritten value, even if that timestap is ahead of the writer's clock. BUT this incurs the cost of an additional read. (According to Claude: this is what CRDB uses to prevent this issue.)

- Even though it might be tempting to resolve conflicts by keeping the most "recent" value and discarding others, it's important to be aware that the definition of "recent" depends on a local time-of-day clock, which could be incorrect. 


##### Clock Readings with a Confidence Interval

- Because of uncertainties and delays, it doesn't make sense to think of a clock reading as a point in time. It's more like a range of times, within a confidence interval. E.g. a system may be 95% confident that the time is now between 10.3 and 10.5 seconds. 

- Presented as a useful way to have a base-line to rely on for your clocks, that (presumably) isn't as complex as implementing PTP (precise-time protocol, above).

- Most systems though don't expose this uncertainty. Google Spanner does, though.

##### Synchronised clocks for global snapshots

- MVCC allows read-only transactions to see a snapshot of the db, a consistent state a particular point in time, without locking and interfering with read/write transactions. 
- Requires a monotonically increasing tx Id. Fine for single node computers. 
- But when distributed, this is difficult to generate, because we need to co-ordinate. The txId must reflect causality: If TB reads or overrwrites a value that was previously written by A, then B must have a higher TxId than A. Otherwise the snapshot wouldn't be consistent. 


**Spanner does it as follows**
- Use the clock's confidence interval as reported by the TrueTimeAPI. 
- If the two CIs don't overlap, i.e. the latest time of the early interval is earlier than the earliest time of the later interval, then one definitely happened after the other. Only if the intervals overlap are we unsure. 

### Process Pauses

- Another example of dangerous clock use in a DS. 
- Imagine a db with a single leader per shard: only the leader is allowed to accept writes. 
- How does a node know that it's still leader (that it hasn't been declared dead by the others) and that it may safely accept writes? Options are :

##### Obtain a lease

Like a lock with a timeout. When a node obtains a lease, it knows that it's the leader for a certain amount of time, until the lease expires. To remain leader, the node must periodically renew the lease before it expires. 

However, it will likely rely on a synced clock. It will need to check expiry times for when the lease expires. If the clocks are out of sync, it'll start to do strange things. You'll also need to do a comparision check for the difference in time remaining. But what if there's a pause in the system here? And this is a legitimate thing to assume.

- Contention among threads accessing a shared resource, such as a lock or queue, can cause threads to spend a lot of time waiting. 
- When the OS context-switches to another thread or when the hypervisor switches to a different VM (when running in a VM), the currently running thread can be paused at any arbitrary point in the code. 

So, in short, a continuation of the central message of we're-only-safe-if-plan-pessimistically: 

**A node in a DS must assume that its execution can be paused for a significant length of time at any point, even in the middle of a function.**

##### Providing Response Time Guarantees

But, those reasons for pausing can be eliminated if we try hard enough. There exist real-time systems, systems that control the movements of physical objects like cars, aircraft, etc. Here, the software must respond by a specified deadline, and failure to meet that deadline may cause a failure of the entire system. 

- Real-time is a technical term within Embedded systems Software.
- Providing real-time guarantees in a system requires support from all levels of the software stack. 
- Requires a large amount of additional work and severely restricts the range of programming languages, libraries, and tools that can be used. 
- For most server-side data processesing systems, real-time guarantees are not economical or appropriate. 

##### Limiting the GC

