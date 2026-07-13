# A History and Evaluation of System R

Relational model was proposed by E.F Codd in 1970. They have 2 important properties:

(1) All information is represented by data values, never by any sort of "connections" which are visible to the user. 
(2) The system supports a very high-level language in which users can frame requests for data without specifying algorithms for processing the requests. 


System R is an experimental system constructed at the San Jose IBM research lab to demonstrate that a relational db system can incorporate the high performance and complete function required for everyday production use. 

System R was the first Database to use SQL. The two were developed hand in hand by the same group of researchers.
SQL was designed as the high-level human readable UI for System R.

Key goals for it were

(i) To provide a high-level, non-navigational user interface for maximum user productivity and data independence. (This turned out to be SQL(!))
(ii) To support different types of database use including programmed transactions, ad hoc queries, and report generation.
(iii) To support a rapidly changing db environment, in which tables, views, indexes, transactions and other objects could easily be added to and removed from the db without stopping the system. 
(iv) To support a population of many concurrent users, with mechanisms to protect the integrity of the db in a concurrent-update environment.
(v) To provide a means of recovering the contents of the database to a consistent state after a failure of hardware or software.
I.e. A and I. 

Phase 0 involved development of SQL, and happened 1974-1975. 
