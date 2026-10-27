# Comparing B-Trees and LSM-Trees

## Introduction

LSM-Trees are better suited for write-heavy applications, whereas B-Trees are often faster for reads.

## Read Performance

### B-Trees

- Looking up a key involves reading one page at each level.
- Reads have predictable performance because B-Trees are wide rather than tall.
- Range queries are simple and fast, as they can make use of the sorted structure of the tree.

### LSM Storage Engines

- Reads have to check several SSTables at different stages of compaction, though Bloom filters help reduce the number of disk I/O operations required.
- Range queries are fast as well, but they need to scan all the segments in parallel and combine the results. Bloom filters don't help with range queries because we would have to compute the hash of every single key within the range, which is impractical.
- If data cannot be written to the disk fast enough when a memtable fills up, many storage engines, like RocksDB, apply **backpressure**: they suspend reads and writes until the memtable has been written to the disk.

Both options work, but it depends on the details of the storage engine and workload.

For read throughput, modern SSDs (solid-state drives) can perform many independent read requests in parallel. Both LSM-Trees and B-Trees are able to provide high read throughput, but storage engines need to be carefully designed to take advantage of this parallelism.

## Sequential Versus Random Writes

The pattern of many small, scattered writes (found in B-Trees) is called **random writes**.

The pattern of fewer large writes (as found in LSM-Trees) is called **sequential writes**.

Disks generally have higher sequential write throughput than random write throughput, which means that a log-structured storage engine can generally handle higher write throughput on the same hardware than a B-Tree.

## Write Amplification

With any type of storage engine, one write request can result in multiple I/O operations. For an LSM-Tree, the value is written to the log for durability, then again when the memtable is written to the disk, and again every time the key-value pair is part of a compaction.

A B-Tree index must write every piece of data at least twice: to the WAL and to the tree page itself.

If you take the total number of bytes written to disk in a workload and divide that by the number of bytes you would have to write if you simply wrote an append-only log with no index, you get the **write amplification**.

Write amplification is a problem in both LSM-Trees and B-Trees. Which is better depends on various factors, such as the length of your keys and values and how often you overwrite existing keys versus insert new ones.

For typical workloads, LSM-Trees tend to have lower write amplification because they don't have to write entire pages, and they can compress chunks of the SSTable. This is another factor that makes LSM storage engines well suited for write-heavy workloads.

A storage engine with lower write amplification will wear out the SSD less quickly.

## Disk Space Usage

B-Trees can become fragmented over time, meaning the database file may contain a lot of pages that are no longer used by the B-Tree. Databases need a background process that moves pages around or reclaims unused space, such as the `VACUUM` process in PostgreSQL.

Fragmentation is less of a problem in LSM-Trees, since the compaction process periodically rewrites the data files anyway, and SSTables don't have pages with unused space.
