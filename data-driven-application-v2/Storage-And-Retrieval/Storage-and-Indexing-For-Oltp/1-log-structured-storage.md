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
