# A Foundational Look at Race Conditions and Concurrency

Schedule: The actual sequence of low-level operations, from all transactions, in the order the database really executed them. 


The entire apparatus exists to answer 1 question: can this unfamiliar thing be shown equivalent to a famililar thing? Serialisability isn't inherently good, it's a reduction target. 



The core foundation of concurrency control: concurrent execution is correct iff every observable result - final state and ever value read - matches what some serial order could have produced. 
Originally this was defined to be 'no outcomes', instead of 'no observable results', whereby observable results are a union of the final state and values read. There's an important difference here. There are a whole class of concurrency issues that do produce incorrect final states in this example (write-writes, read-modify-write, etc). But there are clear concurrency problems whereby the state is absolutely fine (the actual state of the database has no problem), but the observable result - the value that the programme reads - is wrong. For example, a read skew, or a dirty read. Here, what's wrong is the value read, not the final state. Thus we need the 'observable result' qualification, with the further clarification.  

This is important. There are many things that can go wrong when playing two operations sequentially, especially if you get them in the wrong order. Two operations that aren't commutative will produce different outcomes depending on the order they run in; and 1 can be wrong whilst the other is correct. If your concurrency issue can be reached by playing the states sequentially, then it is still correct as far as the concurrency control is 'correct'. All we try and do in concurrency control is eliminate the class of problems that are emergent problems in virtue of being run concurrently - there is absolutely no attempt to try and eliminate problems that sequential execution could produce. 

State: what's in the db. Cells with values in them. 
Observed value: what a transaction read out and carried away. Not in the db, in the handler's local variable. 
