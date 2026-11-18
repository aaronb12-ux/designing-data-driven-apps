# Storage and Indexing for OLTP

A storage engine manages how a database stores, retrieves, and updates data.

For backend work, the main questions are:

- How does it locate a record without scanning everything?
- What work happens when a record changes?
- How does it recover after a crash?
- What trade-offs does an index introduce?

An index provides an additional way to locate data efficiently. It speeds up suitable reads, but consumes storage and adds maintenance work to writes.

> Choose indexes around query patterns—not simply because a column exists.

## 1. Starting Point: An Append-Only Log

A simple key-value store could append each update to a file:

```text
user:42 → "Alex"
user:18 → "Sam"
user:42 → "Alexander"
```

The newest entry for a key represents its current value.

To avoid scanning the file for every lookup, maintain an in-memory hash map:

```text
Key → File offset of its latest value
```

<img width="536" height="306" alt="Append-only log with an in-memory hash index pointing to current values" src="https://github.com/user-attachments/assets/1c6acc84-df6a-4ea2-9984-55b36cad943a" />

### Main Limitations

- Old versions consume disk space until reclaimed.
- Rebuilding the index after a restart can be expensive unless recovery metadata is saved.
- An index entry for every key can require substantial RAM.
- Hash indexes do not naturally support ordered scans or range queries.

This design can be useful. Sorted structures offer a different set of trade-offs.

---

# Log-Structured Storage and LSM Trees

## 2. SSTables: Sorted Files on Disk

An SSTable, or Sorted String Table, stores entries sorted by key.

In the simplified model, each file contains one current entry per key. Real engines may retain multiple versions for snapshots and concurrent transactions.

SSTables are normally immutable: Updates go into new structures instead of modifying existing files.

<img width="598" height="340" alt="Sorted key-value entries with a sparse index" src="https://github.com/user-attachments/assets/c9c51876-32ee-401d-a106-8dc297e7631d" />

### Sparse Indexes

Because the entries are sorted, the index does not need to contain every key.

It can record the first key and file offset of each block:

```text
handbag  → Block 1
handsome → Block 2
```

To find `handiwork`:

1. Find the block whose range could contain it.
2. Read that block.
3. Search within it.

The index is smaller because it identifies blocks rather than every individual record.

## 3. The LSM Write Path

An LSM tree, or Log-Structured Merge Tree, combines an in-memory write buffer with sorted files on disk.

A typical write path is:

```text
Write
  ↓
Append to write-ahead log
  ↓
Update in-memory memtable
  ↓
Flush memtable into an SSTable
  ↓
Compact SSTables in the background
```

### Memtable

The memtable holds recent writes in a searchable, typically sorted structure, such as a skip list or balanced tree.

When it reaches a threshold, it is flushed into an SSTable.

### Write-Ahead Log

The memtable is in RAM, so a crash could erase it.

A write-ahead log, or WAL, records updates so unflushed state can be reconstructed after a restart. RocksDB normally writes updates to both its WAL and memtable. [119]

The durability of an acknowledged write depends on when the log is synchronized to persistent storage—not merely whether the application wrote to an operating-system buffer. [121]

> The WAL protects recent writes. SSTables organize persisted data for efficient access.

## 4. The LSM Read Path

A lookup checks recent in-memory data and the on-disk files that could contain the key.

The engine must return the newest version visible to the read, not an older copy from another file.

A simplified search order is:

```text
Memtables → Recent SSTables → Older SSTables
```

Actual file selection depends on the compaction strategy and metadata.

### Bloom Filters

A Bloom filter helps determine whether a file could contain a key:

- Definitely absent: Skip the file.
- Possibly present: Search the file to confirm.

Bloom filters can produce false positives but, when correctly maintained, not false negatives.

<img width="465" height="251" alt="Bloom filter using hashed positions in a bit array" src="https://github.com/user-attachments/assets/133bb50a-391c-43ef-b4dd-056ff3373ee6" />

The useful idea is:

> A small approximate structure can avoid an expensive disk read for a key that is not there.

Ordinary whole-key Bloom filters mainly help point lookups. They do not directly answer arbitrary range queries.

## 5. Compaction

Multiple SSTables can contain different versions of the same key.

Compaction merges sorted files and removes versions that are no longer needed.

<img width="344" height="273" alt="Sorted files merged during compaction" src="https://github.com/user-attachments/assets/343db503-655d-4897-b2a0-68f10323365b" />

Compaction also helps reclaim deleted data. A deletion is commonly recorded as a tombstone so an older value is not accidentally returned. The marker and older values can be removed when it is safe.

Compaction is ongoing maintenance—not just something triggered when the disk is nearly full.

### Strategies to Recognize

| Strategy | Basic approach | Typical trade-off |
|---|---|---|
| Size-tiered | Merge similarly sized sorted runs. | Often less rewriting, but more files or runs to search. |
| Leveled | Organize files into levels with controlled key-range overlap. | Often better read and space efficiency, but more rewriting. |

These are tendencies, not guarantees. Compaction policy, value size, and workload can substantially change the result. [128]

### Backpressure

If flushing or compaction cannot keep up, the engine may slow or stop incoming writes.

RocksDB calls these write stalls. Reads are not necessarily suspended, although competing background I/O can affect their performance. [125][131]

---

# B-Trees

## 6. Pages and Tree Navigation

B-trees organize sorted keys into fixed-size pages.

Many database indexes use a B+ tree variant, where leaf pages contain data entries or references to rows.

<img width="353" height="230" alt="B-tree with internal pages directing searches to leaf pages" src="https://github.com/user-attachments/assets/4e94e278-61d0-48e5-927f-5e5e60fe512c" />

To find a key:

1. Start at the root page.
2. Choose the child covering the key's range.
3. Repeat until reaching the appropriate leaf.

Each page can reference many children. This branching factor keeps the tree shallow.

A balanced tree has search depth proportional to `log(n)`, although actual disk reads depend on which pages are cached.

### Updates and Splits

Unlike immutable SSTables, traditional B-tree engines modify existing pages, usually in an in-memory buffer first.

When a page becomes too full:

1. Split its contents across pages.
2. Update the parent references.
3. Split the parent too if necessary.

<img width="451" height="334" alt="A full B-tree page split with updated parent references" src="https://github.com/user-attachments/assets/efbf2dbc-1af3-4a52-85ee-f081010de2c6" />

The exact split rules depend on the implementation. You do not need to memorize them to understand the storage trade-off.

## 7. Crash Recovery

A split may modify several pages. A crash halfway through must not leave the database permanently inconsistent.

Many B-tree engines use a WAL:

```text
Record recovery information in WAL
                 ↓
Make the required WAL durable
                 ↓
Persist modified data pages
```

The database can use the log to recover after a crash.

Modified pages can remain buffered in memory before being written to disk. Durability does not require immediately flushing every changed data page if the required recovery log is safely persisted.

> Both B-tree and LSM engines can use a WAL. A WAL is a recovery mechanism, not a defining feature of only one index structure.

---

# Comparing B-Trees and LSM Trees

## 8. Main Trade-Offs

| Concern | B-tree | LSM tree |
|---|---|---|
| Updates | Modify pages. | Buffer writes, flush sorted files, then compact. |
| Point reads | Navigate a tree. | May check multiple files; filters reduce unnecessary reads. |
| Range reads | Traverse ordered leaf entries. | Merge ordered results from relevant structures. |
| Background work | Page flushing and other maintenance. | Flushing and compaction. |
| Performance risks | Page I/O, contention, splits. | Compaction interference, read amplification, write stalls. |

LSM engines are often attractive for write-heavy workloads. B-trees often provide straightforward, predictable lookup paths.

Neither is universally faster. Caching, hardware, compression, concurrency, and data distribution matter.

### Sequential vs. Random Writes

LSM designs batch updates into larger sequential file writes.

Traditional B-tree designs can produce more scattered page writes, although buffering and logging reduce their cost.

Sequential access is especially valuable on hard disks. SSDs handle random access much better, but write volume and I/O patterns still matter.

## 9. Read, Write, and Space Amplification

These terms describe extra work or storage beyond the application's logical data.

| Term | Meaning |
|---|---|
| Read amplification | Reading more data or checking more locations to answer a request. |
| Write amplification | Writing more bytes to storage than the application's logical update size. |
| Space amplification | Using more storage than needed for one compact copy of the live data. |

For example, an LSM update may be written to:

1. The WAL.
2. An SSTable during a flush.
3. Additional SSTables during compaction.

A B-tree engine may write WAL records and entire modified pages.

Neither structure always has lower write amplification. RocksDB's compaction choices explicitly trade off write amplification against read amplification. [128]

Space also has trade-offs:

- B-tree pages can contain free space and become fragmented.
- LSM files can retain obsolete versions and need temporary space during compaction.

---

# Indexes You Use as a Backend Engineer

## 10. Primary and Secondary Indexes

A primary key uniquely identifies a row.

A secondary index provides another lookup path, such as finding users by email or posts by author.

Indexed values need not be unique:

```text
author_id → Matching post identifiers
```

The engine can represent duplicates using lists of row references or separate entries distinguished by row identity.

Every additional index consumes storage and adds maintenance work when relevant data changes.

## 11. Multicolumn Indexes

A multicolumn, or composite, index contains more than one key column.

For a timeline query:

```sql
SELECT id, content, created_at
FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```

A suitable candidate is:

```sql
CREATE INDEX posts_author_created_idx
ON posts (author_id, created_at DESC);
```

The index groups entries by author, then orders that author's entries by creation time.

Column order matters. Multicolumn B-tree indexes are generally most effective when the query constrains the leading columns. [133]

> An index on `(author_id, created_at)` is not equivalent to one on `(created_at, author_id)`.

## 12. Where the Actual Row Lives

### Clustered Storage

In a clustered index organization, the leaf level stores the row data itself.

A lookup reaches the row through that structure.

Terminology varies by database, so check what “clustered” means in the system you are using.

### Heap Storage

A heap stores table rows separately from indexes, without maintaining an index-key ordering.

An index entry points to the row's location.

PostgreSQL uses this arrangement: Its indexes are separate from the table heap, including indexes supporting primary keys. [134]

### Covering Index

An index covers a query when it contains all the column values that query needs.

For example:

```sql
CREATE INDEX users_email_cover_idx
ON users (email)
INCLUDE (display_name);
```

This can support:

```sql
SELECT display_name
FROM users
WHERE email = 'alex@example.com';
```

In PostgreSQL, an index-only scan may avoid fetching the table row, but visibility checks can still require heap access. [134]

Covering is relative to a query—not a requirement to store the entire row.

---

# Embedded and In-Memory Databases

## 13. Embedded Storage Engines

An embedded database runs inside the application's process rather than as a separate database server.

```text
Application process
  ├─ Application code
  └─ Embedded storage engine → Local files
```

“Embedded” describes deployment, not the underlying index structure or durability.

## 14. In-Memory Databases

An in-memory database keeps its main working data in RAM.

Benefits include fast access and convenient support for structures such as sets and sorted sets.

Trade-offs include:

- RAM cost and capacity limits.
- Recovery and restart behavior.
- Persistence configuration.

In-memory does not automatically mean non-durable. Redis supports snapshots, an append-only file, both, or no persistence. The configuration determines the potential data loss and recovery trade-offs. [135]

---

# What to Prioritize

## Understand Well

- Why indexes speed up reads but add write and storage costs.
- The LSM path: WAL → Memtable → SSTable → Compaction.
- The B-tree path: Navigate pages → Modify buffered pages → Persist safely.
- Why Bloom filters help avoid unnecessary point-lookup reads.
- Why compaction can affect latency and write capacity.
- How index column order relates to actual queries.
- The difference between a heap, clustered storage, and a covering index.

## Recognize Without Memorizing

- Sparse indexes.
- Tombstones.
- Leveled vs. size-tiered compaction.
- Read, write, and space amplification.
- Page splits and branching factor.

Leave byte layouts, exact split algorithms, and storage-engine tuning parameters for when your work requires them.

The central lesson:

> Storage engines move work between reads, writes, memory, and background maintenance. Indexes are useful when those trade-offs match the application's access patterns.
