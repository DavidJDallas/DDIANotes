<h1> Storage and Retrieval </h1>

<h2> Overview and Introduction </h2>

This is essentially a chapter on storage engines, and on what they do, how they work under the hood, etc. We can split up storage engines into log-structured storage engines, and page-structured storage engines. 

There is a great deal of discussion on indexes in this chapter, and how the indexes used in databases work. We can think of this as explaining what happens 'under the hood' of indexes, when you - the developer - create indexes in your database. Different databases will utilise different data structures and algorithms to power their indexing, and this chapter explains the whats and hows of these in order for the developer to gain a deeper understanding. 

Storage engines can be broadly split into page-structured storage engines, and log-structured storage engines. 

This chapter feels a bit tangential: it sets out to discuss storage engines, and then discusses indexes in huge depth. The way I understand this seeming tension is that storage engine's main functionality involves the use of indexes - indexes are the most important feature to explore in storage engines. 

He introduces a nice pedagogical story at the start of this to showcase an issue. If we have a database where all we do is just add a row to the end of the log every time we write, this will have pretty good write performance. It's simple - just add to the end. But reads become problematic - we look at O(n) read time, which as the size grows becomes very problematic. To have a performative database, we need some additional metadata which becomes updated every time we write, so that the write updates the metadata to help the search engine organise the data in a better way in order to allow the reads to be able to navigate the data much easier. We call these indexes. 

But we should be careful here. For many examples, the index is the metadata on the side. But for certain indexes, the index creates the structure of at least the tables in the database. For example, clustered indexes and primary indexes fundamentally arrange the particular table around the specific property. It's not just a pointer mechanism stored on the side. 

The chapter progresses not historically, but as a development of complexity in terms of the data structures and algorithms that we introduce to impose order onto the database; we move from more simple (hash indexes) into more complex. 

<h2> Indexes Under the Hood </h2>

Indexes will work very differently depending on the database, or storage mechanism, we choose. (I use the term storage mechanism to allow for discussion of indexing in storage mechanisms like caching systems (e.g. Redis) which are certainly not databases but would use indexing and are valuable to discuss.)

Klepmann discusses several types of data structures that power indexes across different databases.

<h3> Hash Indexes </h3>

Pros
- Very quick, very simple. 
- Amazing for exact key lookups (O(1)).

Cons
- Not efficient for queries using ranges of values.
- Hash map needs to be stored in memory (NOT on disk)

<h3> SSTables and LSM-Trees </h3>

Building on from hash indexes - this solves the problem in hash indexes of the inefficiency for queries over ranges of values. Each file segment is now in sorted order. 

The general conceptual idea is: save up a bunch of records in memory, and then write to a file segment in sorted order. In batches. 

Optimised for write-operations. Actually sort-of inverts the classical trade off that we see in indexes: faster reads for more write overheads. This maximises write-speed, at the expense of read-speed.

- uses a bloom filter, a way to make hash collisions far less likely (^7 power less likely, I think).

Used by databases like Cassandra

<h3> B-Trees </h3>

Most traditional relational databases (row-oriented) use some version of this.