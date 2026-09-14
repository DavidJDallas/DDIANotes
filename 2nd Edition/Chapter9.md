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