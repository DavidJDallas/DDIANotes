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


#### Defining what it means for the Transaction to lock something
(2a/2b) T holds e (locks e) at step i if some earlier step J <= i locked e, and no step between j and i unlocked it. 

#### Defining well-formed
(3a/3b) A transaction is well formed if:
(i) if the action on an entity is a lock, then the entity is not already locked.
(ii) If the action on an entity is not a lock, then the entity must already be locked.
(iii) The final action on the entity is always an unlock. 


#### Defining a Schedule
(4) A schedule must match the order of the Transaction exactly, and be the same length. A transaction can itself not be re-ordered or re-arranged, and no additional steps can be added. 

Note though that we can interleave schedules, and have any number of other transactions steps wedged between them. The definition of the schedule just preserves relative order, not contiguity. 
Note that we don't have atomocity yet in the sense of an indivisible unit - this is something that is achieved by serialisability. 


Naturally, non-serial schedules run the risk of giving a transaction an inconsistent view of the state. To find the equivalency, we find the DEP.

#### Defining DEP, the measuring instrument
(5a/5b)
DEP is the measuring instrument and can be modelled via arrows. It's a dependency relation between the elements in your schedule. Importantly, DEP is calculated by looking at the same entity as it passes through each transaction, and taking the total of all entities as they move through the Transactions.

Take entity e. e is modified by T1, T2, T3. 
So: T1 -> T2 -> T3.
You DEP *is just* T1->T2->T3. That's it. 

This says nothing about correctness, whether it's right, or any sort of value judgement like that. You are just measuring. 

The normative statements come by comparing this relation to other schedules.

Next, we need to introduce equivalence relations. Equivalence relations are defined over the whole collections of all entities that a given group of Transactions touch, taken together. 

Take entity e1, e2, e3. They're all modified by T1, T2, T3, T4.

e1: T1->T2-> T3
e2: T3-> T4 
e3: T2->T3->T4->T1.

Note: This schedule ends up cycling, so it's inconsistent. 

A schedule is *equivalent* IFF your collected DEP relations, see above, are equal to another schedule's DEP.

Then, a schedule is defined as *consistent* IFF it's equivalent to a serial schedule. 


#### A signature of Consistency
(6) Derives a signature of consistency. Basically means you can look for a structural property to label a schedule as consistent, rather than have to do a horribly inefficient search to see if the two match. 

The reasoning is - from any serial schedule you can derive that the dependency relation if T1 -> T2 for e1, then it can never be T2->T1 for any other entity in that dependency relation. 
6b comes from the same observation. And is that we can't have cyclicy in serial schedules. Therefore, no consistent schedules can be cylic, or:

consistent schedules must be acyclic. 

#### Locking in Schedules
(2) Defines what it means for a transaction to hold a lock on e. This - (7) - says the same thing about a position in a schedule. Same shape as (2).

A schedule S is legal if for all k, if S(K) = (T, a, e) and e is locked by T through step k, then e is not locked by any other transaction through step k. 

'Legal schedules observe the lock protocol that a transaction attempting to lock an already locked entity must wait'.

Not every legal schedule is consistent.

'if legality is to insure consistency in all contexts, then it is necessary that each transaction lock each entity before otherwise acting on it and that the transaction ultimately unlock each such locked entity'

####

If every transaction locks what it touches and never takes a new lock after releasing one, then any schedule the lock manager permits is guaranteed to be equivalent to running them one at a time - and if either discipline is dropped, some other transaction exists that will break it. 

In other words, we need guarantees of both legality and consistency in order to have proper serialisability isolation. 

(8a): Consistency requires that transactions be well-formed. Unless all transactions are well-formed, it is possible to construct a legal but inconsistent schedule. 


In a sentence, we can summarise the proof:
If every transaction locks what it touches and never grabs a new lock once it has let one go, then any schedule a lock manager will actually permit is guaranteed to be equivalent to running them one at a time. \





