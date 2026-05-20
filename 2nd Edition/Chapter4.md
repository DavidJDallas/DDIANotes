# Chapter 4: Storage and Retrieval

The way the chapter is broadly structured seems to be centered around pedagogical purposes. We start with the most simple example possible, and we develop this. The first example outlines the most simple example of a db that we can possibly think of. We have a get and a set functionality, and all we do is that we have a log data structure, and every time we write we simply add to the end of this log.

This is fine for writes - in fact, it's actually good. It's always O(1) for the writes. But the reads become problematic, without any sort of additional data structures, for the mechanism to find the data is simply to iterate through every piece of data until we find what we want. So, O(n) time.

To have a performative database, we need some additional metadata to help the search engine organise the data in a better way in order to allow the reads to be able to navigate the data much easier. These are indexes.

But it seems like we should be careful here. Because although secondary indexes are just as described here, we can have primary indexes which do literally change the structure of the table itself, as opposed to simply just being metadata on the side.

It's also written:

'Well chosen indexes speed up read queries, but every index consumes additional disk space and slows down writes, sometimes substantially'.

This is also technically true regarding the writes, but I think it risks missing something a bit subtle. It does slow down writes, but if your db access patterns are such that your main write-access is very small writes (for example, single-row writes) then assuming that you have a decent number of rows already in your table, having an index in place will often significantly speed up your performance. For creating new rows, having a clustered index here would speed up the write in that the storage engine would spend far less time finding where to do the insert  (O(n) => O(log n)), and for updating rows, having some sort of secondary index would speed up writes in that you spend far less time actually finding the row in the first place (again, O(n) => O(log n)). The writes themselves will incur more overhead (you also have to update the index to inform it of the write), but you can have substantial performance gains by having indexes in place here. The index can move finding the row from scanning 1,000,000 rows into scanning just 13 rows.

I found myself confused in this chapter at the structure. It's a chapter on storage engines, but much of the focus seems to be on indexes. I think the relationship here is that indexes are a vital part of the search engine performance; it's an internal mechanism of the storage engine. So the better way to think about, I think, is that this chapter is specifically on storage and retrieval, which just are what indexes do. A storage engine is resposible for how data is stored in the database, and how data is retrieved. Indexes are the way that the storage engine does this. 

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

### SSTables, LSM Trees

Hash tables are not commonly implemented mainly for reasons highlighted above - especially the inefficiency of range queries. 

More common to use a structure where you actually sort the data before you store it. SSTables also store key-value pairs, but they're sorted by key, and each key appears only once in the file. We can use only some of the keys in memory, in which case it's called a *sparse* index. 

ASIDE
Cockroach uses this. Cockroach uses LSM-tree indexing, using SSTables via the Pebble search engine. Pebble is inspired heavily by Rocks Db, and Cockroach actually used to use RocksDb storage engine. 

Each block of records can also be compressed.

SSTable format makes reads better, but it' for writes than a simple append-only log. We can't just add the file to the end, because then the table wouldn't be sorted. We solve this via a log-structured approach; this is a hybrid between an append-only log and a sorted file. 

1. Write comes in; write to memory (memtable). Add it to an in-memory ordered map data structure. With these data structures, you can insert keys in any order, look them up efficiently, and read them back in order.

2. When memory gets too big, write to disk. We write the memtable to disk in sorted order as an SSTable file. This new SSTable file is the most recent segment of the db. Stored as a seperate file alongside the other files. Each file has a seperate index. 

3. Reading step. First, try the memtable. If no success, try the most recent file. And so on. If appears nowhere, does not exist.

4. Occasionally merge and compact. This is to combine segment files, discard overwritten or deleted values. 

To delete a key, you append a special deletion record called a tombstone to the data file. When log segments are merged, the tombstone tells the merging process to discard any previous values for the deleted key. 

As described above, we derive from this that more recent data will be much faster to retrieve than older data. To get around this, LSM storage engines often use Bloom filters. 

### Bloom filters

Bloom filters are probabilistic data structures. Extremely space-efficient. They can be used to test whether an element is a member of a set. Importantly, they can tell with 100% confidence if a member is *not* in the set, but only probabilistically if they *are* in the set.
Probabilistic because of the potential of collisions. 

So, the 'yes' is always probabilistic, but the 'no' is always deterministic. 

The way these filters work:

You have an array of 10 boolean values, either true or false.. We build a hash function that will accept any input and hash it to a value between 0 and 10, corresponding to this array.

So, User_1 comes in; we hash this (and therefore produce a deterministic response) and get 3. Therefore, value 3 in the array becomes true.

We now want to check if User_1 exists. So, on the read, we hash User_1 again, and we get 3. Therefore true. BUT we can only take this as probabilistically true because of the risk of collisions. And we know about hashing that as our state space grows, collisions become less likely. So for an array of 10 values, collisions are *very* likely, almost inevitable (1/10 chance).

Our trust in the probabilistic 'true' value can increase as the state space grows. And this is the trade-off: more state-space means more memory required, but the more we can trust it. Less state-space means less memory required (and therefore faster) but the less we can trust it.

Note that it will always, deterministically, be correct if it tells you the member is not in the set. But note the subtlety here: the hash function may not be able to tell you accurately if the member is or isn't in the set (it could be, or it couldn't be). But if it tells you it isn't, you should always fully believe it.

----

False positives are not an issue for LSM based storage engines. If the bloom filter says that a key is not present, we can safely skip that SSTable, since we can be sure it doesn't contain that key. If we get a false positive - it says it's there but isn't, we have to do a bit of extra work but it's not a massive loss; the worst-case scenario is that we do a bit of extra work. 
