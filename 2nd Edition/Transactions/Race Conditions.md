# A Foundational Look at Race Conditions and Concurrency

Schedule: The actual sequence of low-level operations, from all transactions, in the order the database really executed them. 


The entire apparatus exists to answer 1 question: can this unfamiliar thing be shown equivalent to a famililar thing? Serialisability isn't inherently good, it's a reduction target. 



The core foundation of concurrency control: concurrent execution is correct iff every observable result - final state and every value read - matches what some serial order could have produced. 
Originally this was defined to be 'no outcomes', instead of 'no observable results', whereby observable results are a union of the final state and values read. There's an important difference here. There are a whole class of concurrency issues that do produce incorrect final states in this example (write-writes, read-modify-write, etc). But there are clear concurrency problems whereby the state is absolutely fine (the actual state of the database has no problem), but the observable result - the value that the programme reads - is wrong. For example, a read skew, or a dirty read. Here, what's wrong is the value read, not the final state. Thus we need the 'observable result' qualification, with the further clarification.  

This is important. There are many things that can go wrong when playing two operations sequentially, especially if you get them in the wrong order. Two operations that aren't commutative will produce different outcomes depending on the order they run in; and 1 can be wrong whilst the other is correct. If your concurrency issue can be reached by playing the states sequentially, then it is still correct as far as the concurrency control is 'correct'. All we try and do in concurrency control is eliminate the class of problems that are emergent problems in virtue of being run concurrently - there is absolutely no attempt to try and eliminate problems that sequential execution could produce. 

State: what's in the db. Cells with values in them. 
Observed value: what a transaction read out and carried away. Not in the db, in the handler's local variable.


It's worth also making this distinction: the state can be fixed afterwards and modified and rectified. But the observed value, once it's seen, has been seen, and sent out into the world, cannot be rectified, really. Once the observed value obtained via a concurrency problem has been sent out via an email, or an API response, it's bad, and isn't a straightforward case of fixing.


A scenario: A colleague proposes: "Let's not slow anything down with locking. We'll log every read and write, and run a job overnight that checks whether each day's execution matched some serial order. If it didn't, we'll alert and fix it."

This is a variation of optimistic concurrency control, but crucially it fails because of being done overnight. If it were to work, it would need to be done at commit time. Let's first sketch out something that absolutely kills it, before we can correct the colleague's idea and explain how this is the foundation of optimistic concurrency control. 

Let's assume a case where there's a concurrency correctness violation on the read. Therefore, the read will read a skewed value that does not represent any actual state on the database, and it 'escapes', returned to the application. This read tells the user they have £3,000 in their account. The user then sends £500 to their savings account - same application, same database, same table. This transaction is totally legal, and does not violate any sort of concurrency correctness. Therefore, even when the reconcilation is done overnight, this escaped value has caused a major incorrectness violation that cannot be rendered correct again by reconcilation. 

To be able to implement what the colleague is suggesting, we need to perform the checks *on commit*. It would work like follows:
(i) Start the transaction, T1. No locks taken, just log all the bits you need to log.
(ii) If a read, T2, comes in in the meantime that touches any state that T1 touches, the read, T2, will read a snapshot of the database from before T2 started. 
(iii) Once the transaction finishes, it does the checks - has anything happened while this transaction has run that would be impossible had the transactions ran serially? IF yes, abort the transaction. If no, commit.


- A transaction is a programme, not a list of operations. The system only ever sees the trace so far. 

The system must make irrevocable local decisions with only the past available, about executions that aren't finished.

But the system does have, and this is enough, it can see which transactions have touched every object and in what way, and the order they arrived. The essence of CC is to claim that this is sufficient.
