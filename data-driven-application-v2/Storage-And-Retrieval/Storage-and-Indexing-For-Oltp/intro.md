# Storage and Indexing for OLTP

Many databases internally use a log, which is an append-only data file. Real databases have more issues to deal with, such as handling concurrent writes, reclaiming disk space so that the log doesn't grow forever, and handling partially written records when recovering from a crash.

To efficiently find a value for a particular key in a database, we need a different data structure: an **index**. In this chapter we will look at a range of indexing structures and see how they compare. The idea is to structure the data in a particular way (e.g., sorted by a key) that makes it faster to locate the data you want.

An index is an additional structure that is derived from the primary data.

An important trade-off in storage systems is the use of an index: well-chosen indexes speed up read queries, but every index consumes additional disk space and slows down writes.

For this reason, databases don't usually index everything by default, but require you — the person writing the application or administrating the database — to select indexes manually, using your knowledge of the application's typical query patterns.
