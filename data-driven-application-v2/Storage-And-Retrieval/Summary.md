# Chapter Summary: Storage and Retrieval

This chapter examined how databases store data and retrieve it efficiently, focusing on the different storage-engine designs used for transactional and analytical workloads.

## OLTP vs. OLAP

Database workloads are commonly divided into two broad categories:

- **OLTP (Online Transaction Processing):** Handles operational transactions, such as creating, updating, or retrieving individual records.
- **OLAP (Online Analytical Processing):** Supports analytical queries that examine large amounts of data to identify patterns, trends, and aggregates.

## OLTP: Transactional Workloads

OLTP storage engines are optimized for a high volume of requests. Each request typically reads or writes a small number of records and requires a low-latency response.

- **Access patterns:** Records are commonly retrieved through a primary key or secondary index.
- **Indexes:** Many indexes maintain an ordered mapping from keys to records, allowing efficient lookups and, where supported, range queries.

### Two Common OLTP Storage-Engine Designs

#### 1. Log-Structured Storage

- Writes are appended to files rather than overwriting existing data in place.
- When records are updated or deleted, newer entries supersede older ones. Obsolete data is eventually removed through compaction.
- This approach can provide high write throughput and is used by storage engines such as LSM-trees and systems including RocksDB, Cassandra, and ScyllaDB.

#### 2. B-Trees and Update-in-Place Storage

- The database organizes data into fixed-size pages that can be read, modified, and written back to disk.
- B-trees (commonly B+ trees) keep keys ordered and support efficient lookups, inserts, deletes, and range scans.
- B-trees are used by most major relational databases and many non-relational databases.
- B-trees can provide strong read performance, particularly for workloads with frequent lookups and range queries. However, whether they outperform log-structured engines depends on the workload, hardware, and implementation.

## OLAP: Analytical Workloads

OLAP systems, such as data warehouses, are optimized for complex read queries that scan and aggregate large numbers of records.

- **Column-oriented storage:** Values from the same column are stored together rather than storing every row together.
- **Compression:** Columnar data can often be compressed effectively because values in a column tend to have similar types and patterns.
- **Reduced disk I/O:** A query can read only the columns it needs, avoiding the cost of scanning unrelated fields.
- These properties make column-oriented storage well suited to analytical queries that process large datasets.

## Indexes for Specialized Queries

We also examined indexes designed to search across multiple conditions or dimensions, beyond ordinary key-based lookups.

- **Multidimensional indexes:** Structures such as R-trees organize data across multiple dimensions. For example, a geographic query can use latitude and longitude to find points within a region on a map.
- **Full-text search indexes:** Inverted indexes map terms to the documents or records in which they appear. This makes it efficient to search for documents containing one or more keywords.

## Key Takeaway

There is no universally optimal storage engine. OLTP systems prioritize efficient, low-latency access to individual records, while OLAP systems prioritize scanning and aggregating large datasets. The storage layout and indexing strategy should match the workload.
