# Chapter 4: Storage and Retrieval

The way the chapter is broadly structured seems to be centered around pedagogical purposes. We start with the most simple example possible, and we develop this. The first example outlines the most simple example of a db that we can possibly think of. We have a get and a set functionality, and all we do is that we have a log data structure, and every time we write we simply add to the end of this log.

This is fine for writes - in fact, it's actually good. It's always O(1) for the writes. But the reads become problematic, without any sort of additional data structures, for the mechanism to find the data is simply to iterate through every piece of data until we find what we want. So, O(n) time. 

To have a performative database, we need some additional metadata to help the search engine organise the data in a better way in order to allow the reads to be able to navigate the data much easier. These are indexes. 

But it seems like we should be careful here. Because although secondary indexes are just as described here, we can have primary indexes which do literally change the structure of the table itself, as opposed to simply just being metadata on the side. 

It's also written:

'Well chosen indexes speed up read queries, but every index consumes additional disk space and slows down writes, sometimes substantially'. 

This is also technically true regarding the writes, but I think it risks missing something a bit subtle. It does slow down writes, but if your db access patterns are such that your main write-access is very small writes (for example, single-row writes) then assuming that you have a decent number of rows already in your table, having an index in place will often significantly speed up your performance. For creating new rows, having a clustered index here would speed up the write in that the storage engine would spend far less time finding where to do the insert  (O(n) => O(log n)), and for updating rows, having some sort of secondary index would speed up writes in that you spend far less time actually finding the row in the first place (again, O(n) => O(log n)). The writes themselves will incur more overhead (you also have to update the index to inform it of the write), but you can have substantial performance gains by having indexes in place here. The index can move finding the row from scanning 1,000,000 rows into scanning just 13 rows.

## log-structured storage

Starts by looking at log-structured storage, before moving onto B-tree structured storage. B-tree storage is the norm 

### Hash maps

First, we look at hash maps for indexes.

Pros
- Very quick, very simple
- Amazing for exact key lookup (O(1))

Cons
- Not efficient for queries using ranges of values. This is because hashing functions are literally designed to distribute the data they hash as randomly as possible to avoid collisions. So it's unordered and therefore hard (impossible?) to infer groupings. 
- Hash maps need to be stored in memory (not on disk).

### SStables, LSM Trees

Hash tables are not commonly implemented mainly for reasons highlighted above - especially the inefficiency of range queries. 

More common to use a structure where you sort by key. SSTables are one such example. 

Interestingly, Cockroach uses this. Cockroach uses LSM-tree indexing, using SSTables via the Pebble search engine. Pebble is inspired heavily by Rocks Db, and Cockroach actually used to use RocksDb storage engine. 

Sparse: index that stores only some of the keys.

Each block of records can also be compressed.

SSTable format makes reads better, but it's trickier for writes than a simple append-only log. We can't just add the file to the end, because then the table wouldn't be sorted. We solve this via a log-structured approach; this is a hybrid between an append-only log and a sorted file. 
