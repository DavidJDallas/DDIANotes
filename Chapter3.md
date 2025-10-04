<h1> Storage and Retrieval </h1>

<h2> Overview and Introduction </h2>

This is essentially a chapter on storage engines, and on what they do, how they work under the hood, etc. We can split up storage engines into log-structured storage engines, and page-structured storage engines. 

There is a great deal of discussion on indexes in this chapter, and how the indexes used in databases work. We can think of this as explaining what happens 'under the hood' of indexes, when you - the developer - create indexes in your database. Different databases will utilise different data structures and algorithms to power their indexing, and this chapter explains the whats and hows of these in order for the developer to gain a deeper understanding. 

Storage engines can be broadly split into page-structured storage engines, and log-structured storage engines. 

<h2> Indexes Under the Hood </h2>

Indexes will work very differently depending on the database, or storage mechanism, we choose. (I use the term storage mechanism to allow for discussion of indexing in straoge mechanisms like caching systems (e.g. Redis) which are certainly not databases but would use indexing and are valuable to discuss.)

Klepmann discusses several types of data structures that power indexes across different databases.

<h3> Hash Indexes </h3>

Very quick, very simple. Not efficient for queries using ranges of values, but amazing for exact key lookups (O(1) for exact key lookups)

<h3> SSTables and LSM-Trees </h3>

Optimised for write-operations. Actually sort-of inverts the classical trade off that we see in indexes: faster reads for more write overheads. This maximises write-speed, at the expense of read-speed.

Used by databases like Cassandra

<h3> B-Trees </h3>

Most traditional relational databases (row-oriented) use some version of this.