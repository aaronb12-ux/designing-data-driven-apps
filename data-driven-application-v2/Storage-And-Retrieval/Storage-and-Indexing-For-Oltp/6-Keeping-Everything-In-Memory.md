# Keeping Everything in Memory

The data structures discussed so far in this chapter have all been answers to the limitations of disks.

Disks are more difficult to work with than main memory. With both magnetic disks and SSDs, data needs to be laid out carefully to achieve good read and write performance.

Disks have two significant advantages:

- **Durability:** Data persists even when the machine is powered off.
- **Lower cost per gigabyte:** Disk storage is much cheaper than RAM.

Many databases are not very large, so it is feasible to keep them entirely in memory, potentially distributed across several machines. This has led to the development of **in-memory databases**.

Writing to disk also has operational advantages:

- Files can easily be backed up.
- Data can be inspected directly.
- External utilities can analyze the data.

Another interesting use case for in-memory databases is providing data models that are difficult to implement efficiently with disk-based indexes.

For example, **Redis** provides a database-like interface to various in-memory data structures, such as:

- Priority queues
- Sets

Because Redis keeps data in memory, its implementation can be comparatively simple.

**Key idea:** In-memory databases trade some of the cost and durability advantages of disk storage for faster access and simpler implementations of certain data structures.
