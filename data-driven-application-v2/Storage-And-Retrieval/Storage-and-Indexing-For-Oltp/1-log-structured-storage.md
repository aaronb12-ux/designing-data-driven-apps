# Log-Structured Storage

Assume that we have an append-only file and want to speed up reads. One way to do this is by keeping a hash map in memory that maps every key to the byte offset where the most recent value for that key can be found.

<img width="536" height="306" alt="Screenshot 2026-10-04 at 12 53 32 AM" src="https://github.com/user-attachments/assets/1c6acc84-df6a-4ea2-9984-55b36cad943a" />

This approach is faster, but it still suffers from several problems:

- **Wasted disk space:** You never free up disk space occupied by old log entries that have been overwritten. If you keep writing to the database, you might eventually run out of disk space.
- **Slow restarts:** The hash map is not persisted, so you have to rebuild it when you restart the database. For example, you could scan the entire log to find the latest byte offset for each key. This makes restarts slow when there is a lot of data.
- **Memory limitations:** The hash table must fit in memory. In principle, you could maintain a hash table on disk, but it is difficult to make an on-disk hash map perform well.
- **Expensive random access:** An on-disk hash map requires a lot of random-access I/O and is expensive to grow when it becomes full.
- **Hash collisions:** Collisions require additional logic to handle.

## The SSTable

In practice, hash tables are not used very often for database indexes. Instead, it is much more common to store data in a structure that is **sorted by key**.

One example is a **Sorted Strings Table (SSTable)**. This file format also stores key-value pairs, but it ensures that:

- Key-value pairs are sorted by key.
- Each key appears only once in the file.

<img width="598" height="340" alt="Screenshot 2026-10-04 at 1 10 49 AM" src="https://github.com/user-attachments/assets/c9c51876-32ee-401d-a106-8dc297e7631d" />

### Sparse Indexing

You can group the key-value pairs within an SSTable into blocks of a few kilobytes and store the first key of each block in an index. This type of index, which stores only some of the keys, is called a **sparse index**.

For example, in the figure above:

- The first key of one block is `handbag`.
- The first key of the next block is `handsome`.

Suppose we are looking for the key `handiwork`, which does not appear in the sparse index.

Because the keys are sorted, we know that `handiwork` must appear between `handbag` and `handsome`. Therefore, we can:

1. Look up the offset for `handbag` in the index.
2. Seek to that offset in the file.
3. Scan the file from that point until we find `handiwork`.

This allows us to locate keys efficiently without storing an index entry for every key.

### Constructing and Merging SSTables

SSTable writes can be difficult because we need to keep all the strings in sorted order without rewriting the entire file every time. We cannot just do a linear scan on the disk file to insert a new key, because inserting data into the middle of a file requires shifting all the subsequent data on the disk. That operation is incredibly slow.

Instead, we can use **Red-Black Trees** or **Skip Lists** that are stored in memory to keep incoming data sorted on-the-fly without performance penalties. This in-memory data structure is called a *memtable*.

Once this table gets bigger than a set threshold, we write it out to the disk in sorted order as an SSTable file.

When we want to search for keys, we first check the memtable and the most recent on-disk segment. If it's not there, we check the next-oldest segment until we find the key or reach the oldest segment.

Eventually, we merge segments, which works like the **mergesort** algorithm. In the figure below, we essentially merge all the unique keys in each segment into one.

<img width="344" height="273" alt="Screenshot 2026-10-05 at 1 38 57 AM" src="https://github.com/user-attachments/assets/343db503-655d-4897-b2a0-68f10323365b" />

### Bloom Filters

Bloom filters speed up reads in LSM storage. They are fast and approximate ways of checking whether a particular key appears in a particular SSTable.

#### How It Works

Each key is hashed using a function that produces a set of numbers, which are then interpreted as indexes into an array of bits (the Bloom filter array). We set the bits corresponding to those indexes to `1` and leave the rest as `0`.

The bitmap is stored as part of the SSTable, along with the sparse index of keys.

If we want to know whether a key appears in the SSTable, we compute the same hash of that key and check the bits at those indexes.

- If **all the bits are `1`**, the key *may* exist.
- If **any bit is `0`**, the key definitely does not exist.

These checks can be made extremely quickly using the bitwise operations that all CPUs support.

<img width="465" height="251" alt="Screenshot 2026-10-06 at 1 27 15 AM" src="https://github.com/user-attachments/assets/133bb50a-391c-43ef-b4dd-056ff3373ee6" />

There is a possibility of **false positives** occurring, where a key that does not exist has its bits map to all `1`s. The probability of false positives depends on the number of keys, the number of bits set per key, and the total number of bits in the Bloom filter.

Though this typically is not a problem because if the Bloom filter says that a key is not present, we can safely skip that SSTable since we can be sure that it does not contain the key.

If the Bloom filter says the key is present, we have to consult the sparse index and decode the block of key-value pairs to check whether the key is really there.




