# The Notions of Consistency and Predicate Locks in a Database System

Iconic paper which was the first to formally propose, define and introduce properly 2PL with various proofs etc. It had apparently been used before, and people were aware of it being used, but this really solidified it and defined it properly. A two-phase lock is two-phase if it never acquires another lock after its first lock. 


1. Concurrency is worth wanting, but is dangerous. Risks of conflict.
2. Definition of what 'went wrong' means. A schedule is fine if it could have been serial. This is formalised as DEP, the dependency relation. Correctness is defined as being indistinguishable from serial execution. 

3. Locks alone don't get you there. The failure lives in the relationship between 2 entities, rather than in either one. 

4. Two disciplines on transactions close the gap. 
(i) well formed: lock before you touch, release what you locked. 
(ii) Two-phase: never acquire after your first release. 

Both are checkable by reading a single transaction in isolation. You don't need any knowledge of anything else that is running. 
Formally proved that well formed + two-phase transactions produce only serialisable schedules, no matter how the scheduler interlaves them. Violate either, and you can be broken. 

5. But none of that works on a real database. You can have phantoms if you're locking over names that are fixed in advance, since no values can be written which match your criteria mid-transaction, that will affect your outcome. 

6. The fix is to lock predicates, not rows. 


In 1 line: Correctness under concurrency means indistinguishability from serial execution. 2PL buys that with only local checks, and predicate locks extend it to a world where you address data by description rather than by name. 

Interestingly, Atomicity seems interwoven with what is now 'isolation'. 
'We assume that actions are atomic; that is, if two processes concurrently perform actions, the effect will be as though one of the actions were performed before the other'.
But I think that they don't mean atomic in the sense of all-or-nothing. They mean it more in a Djikstraian sense of a logical unit.

Consistency constraints can't be enforced at an individual level, only at the transaction level. This is because it simply can't be the case to have instantaneous transmission, which is what would be required to preserve consistency over the application (e.g. total balance = 100, transfers from and to equaling 100.) Transactions are units of consistency. 

'in general, consistency assertions cannot be enforced before the end of a transaction. In this paper it is assumed that each transaction, when executed alone, transforms a consistent state into a new consistent state; that is, transactions preserve consistency.'

'A particular sequencing of the actions of a set of transactions is called a schedule. A schedule which gives each transaction a consistent view of the state is called a consistent schedule'.

'The principal result is that consistency requires that a transaction must be cosntructed to have a growing and a shrinking phase. During the growing phase it can request new locks. However, once a lock has been released, the transaction cannot request a new one'.

'A phenomenon called phantoms seems to imply that one must lock logical subsets of the db rather than locking individual records present in the db'.

Definition of locking:

'If transaction T1 attempts to lock entity e1 which is already locked by Transaction T2 then either T1 must wait for T2 to unlock e1 or T1 must preempt e1 from T2. If T1 waits and then T2 attempts to lock an entity e2 locked by T1, then T2 must wait or pre-empty. If both T1 and T2 wait, then deadlock arises.'

Pre-empt means that it will forcibly take the lock by aborting the holder. In modern terms, pre-emption is victim selection. 
Wait or pre-empt is exhaustive: either you block, or somebody dies. 

625: The argument:

(1) Whilst the problem of temporary inconsistency is inherent, conflict is not. We want to avoid conflict. 
(2) One solution is to partition entities into disjoint classes and then only allow for concurrency with Transactions that don't touch each other. But, usually impossible to examine a transaction and decide exactly which subset of the state it will be. *why*?
(3) Instead, and this is the topic of the paper, they opt for dynamic locking. Transactions lock entities for several reasons. 
- prevent conflict with other transactions (lock out changes made by other transactions)
- temporarily suspend consistency assertions on the locked entities.
- Reproducibility of reads. 
- Recovery and transaction backup. One backup procedure for a transaction T is to undo all of its updates as recorded in the log. Then all entities locked by T may be unlocked and T may be reset to its initial state.
(4) Crucially - a transaction must not request a new lock after releasing some lock. 

626: 'To insure that each transaction sees a consistent state, a transaction must not request a new lock after releasing some lock'. This result is proved formally. 

The proof, in plain english, and summarised:
First, we have definitions and assumptions:
(1a) Temporary inconsistencies. After the first step of T1 or T2, A !== B. 
(1b) Conflict. 

(2a/2b) T holds e at step i if some earlier step J <= i locked e, and no step between j and i unlocked it. 

(3a/3b) A transaction is well formed if:
(i) if the action on an entity is a lock, then the entity is not already locked.
(ii) If the action on an entity is not a lock, then the entity must already be locked.
(iii) The final action on the entity is always an unlock. 

(5a/5b)
DEP is the tool that spots the difference. Sometimes it will disallow harmless scheduling; it's delibaretly conservative. It's semantics blind. 

In a sentence, we can summarise the proof:
If every transaction locks what it touches and never grabs a new lock once it has let one go, then any schedule a lock manager will actually permit is guaranteed to be equivalent to running them one at a time. \





