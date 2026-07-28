# The Notions of Consistency and Predicate Locks in a Database System

Iconic paper which was the first to formally propose, define and introduce properly 2PL with various proofs etc. It had apparently been used before, and people were aware of it being used, but this really solidified it and defined it properly. A two-phase lock is two-phase if it never acquires another lock after its first lock. 

Interestingly, Atomicity seems interwoven with what is now 'isolation'. 
'We assume that actions are atomic; that is, if two processes concurrently perform actions, the effect will be as though one of the actions were performed before the other'.
But I think that they don't mean atomic in the sense of all-or-nothing. They mean it more in a Djikstraian sense of a logical unit.

Consistency constraints can't be enforced at an indivdual level, only at the transaction level. Transactions are units of consistency. 

'in general, consistency assertions cannot be enforced before the end of a transaction. In this paper it is assumed that each transaction, when executed alone, transforms a consistent state into a new consistent state; that is, transactions preserve consistency.'

'A particular sequencing of the actions of a set of transactions is called a schedule. A schedule which gives each transaction a consistent view of the state is called a consistent schedule'.

'The principal result is that consistency requires that a transaction must be cosntructed to have a growing and a shrinking phase. During the growing phase it can request new locks. However, once a lock has been released, the transaction cannot request a new one'.

'A phenomenon called phantoms seems to imply that one must lock logical subsets of the db rather than locking individual records present in the db'.

626: 'To insure that each transaction sees a consistent state, a transaction must not request a new lock after releasing some lock'. This result is proved formally. 
