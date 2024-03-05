# NoSQL Internals

## How to navigate to an entry in NoSQL databases?
Assuming data is stored in key-value pairs.
### Iteration 1:
* 101 : Mohsin
* 102 : Rahul
* 103 : Akash

Search Operation: O(n)

Read Operation: O(n)

Write Operation: 

### Iteration 2:
Write Operation - Write at the end of the file write the new entry. O(1)

Update Operation - To update an entry simply insert at the end of the file. O(1)

Read Operation - Start reading from the end and stop at the first key match. O(n) (greater than O(n) because we'll have multiple duplicate entry for keys - n is the number of unique records)

Delete is same as update.

|Writes|Updates| Reads                            |Storage|
|------|-------|----------------------------------|-------|
|Great|Great| Super Bad                        | Super Bad|
|O(1)|O(1)| O(N * avg. number of duplicates) |O(N * avg. number of duplicates)|

-----------------------------
### How data is read?

Main memory - Primary memory / RAM

Secondary Memory - Hard disk

We cannot load 1GB or 100Gb file at once in the main memory. Main memory can load only a chunk of data of limited size.

Given that the file which stores the database will keep on becoming bigger and bigger within no time it will become big enough that it cannot be loaded
to main memory in one chunk.

If at a time main memory load 100MB file at max then we can divide secondary memory into 100MB file size chunks. Instead of storing one bigger file in secondary memory, its better to divide and store in secondary memory.

--------------------------------------
### Iteration 3 / Iteration 4

Writing in secondary memory is very slow, writing in primary is very fast.

We are only writing, updating at the last chunk (EOF) then in that case keep the latest chunk in the primary memory and not in secondary memory.

What if without the updates made to secondary memory there is power cut or system restarts, in that case all the updates will be lost. Keep a copy of chunk in both primary and secondary memory (backup file) and keep syncing them.

The chunk in main memory is called memTable (map or dictionary). If we have map in primary memory then we can update the entry in O(1), meaning there is no need for duplicate entries in the chunk cause we can do the update in O(1) complexity.

In way that both secondary and primary memory are in sync, then even if primary memory is volatile we are able to recreate it with the backup.

Suppose we are updating primary memory, and it is becoming near 100MB size, then dump the map to secondary memory as file/chunk and then safely delete backup from secondary memory and then clear the memTable in primary memory.

Given we are updating data in primary memory, we will never have duplicate key / entry in one chunk. We will have duplicate between files but not in a file.

How about using sorted map as memTable?

This will ensure that data is always sorted within a chunk/file.

The chunk / file we stored on secondary memory is called SSTable (sorted string table).

> Outcomes:
> 
> All the chunk files stored on the secondary memory are
> * without duplicates inside the same file
> * They are stored on keys
> * All of them are approx 100MB in size.

> Notes:
> * Duplicates can still exist between SSTables.
> * Only within a SSTable keys are stored not between teo SSTables.

#### How reads will happen?
What is the present value of 107 or the key?

1. Check if memTable contains the key? - We can read and reply in O(1) for sorted hashmap it is O(logN) worst case 
2. If data is not present in memTable - 
    * Look into the latest SSTable and see if key is present there. (We will need to bring SSTable to primary memory). If it is present then return the value.
    * If not present then penultimate SSTable (goto previous SSTable)

Note: Step 2 might have to be performed recursively.

|Writes|Updates| Reads                                                                       | Storage utilization              |
|------|-------|-----------------------------------------------------------------------------|----------------------------------|
|Great|Great| Best case: Great, Avg case (first few SSTables): Good, Worst case: Very Bad | Bad                              |

### Compaction

The load on database keeps on increasing and decreasing, that means we can do some compaction in the disk in parallel. 

Bring 2 consecutive SSTables in primary memory and compare these 2 and memTable and generate output after merging.
1. If memTable has the latest value or entry for the key - don't write to output
2. If both SSTables have entry for a key - take the latest one and insert in output
3. If a key is present in either of the SSTables then include in output.

|Writes|Updates| Reads                                                                                                               | Storage utilization           |
|------|-------|---------------------------------------------------------------------------------------------------------------------|-------------------------------|
|Great|Great| Best case: Great, Avg case (first few SSTables): Good, Worst case: Slightly improved (reduced size with compaction) |                               |

### Indexing - Using Binary search to make Read way better

What binary search can't be applied directly?

Binary search can be applied directly as the elements are not of same/ equal size.

File size - 100MB, max key-value pair size will be 100MB it cannot be > 100MB.
Divide the file in logical blocks of 64KB size.

How many blocks = 100MB/64Kb = 1600 blocks

We can choose the block size we want - 64kB, 128 and so on

Every key-value will never occupy two blocks, we will ensure that each key-value pair takes only 1 block.

Each block can accommodate more than one entry as long as it has space left. Once an entry cannot be accommodated it is written to new block.  

Binary search on the first elements of the blocks. [101, 111, 400, 900, 1100, 1300]
We will read on 64kB of data, reducing it from 100MB to 64kB.

> Bigger the block size less the read optimization, smaller the block size the better the read optimization will we have.

1. check the key in memTable. If found return
2. If not in memTable, goto last SSTable (read only 64kB of data, 1 block and not all blocks of 100MB data) which is approx 1600 times faster.

TODO reads: 

Check Bloom Filter - tell if key is present in db in O(1), if present go ahead and read else no need.

LSM Trees