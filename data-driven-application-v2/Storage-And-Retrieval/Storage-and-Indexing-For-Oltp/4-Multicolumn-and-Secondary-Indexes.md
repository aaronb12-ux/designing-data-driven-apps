# Multicolumn and Secondary Indexes

So far, we have only discussed key-value indexes, which are like primary-key indexes in the relational model. A primary key uniquely identifies one row in a relational table, one document in a document database, or one vertex in a graph database.

It's common to have **secondary indexes**. A secondary index can easily be constructed from a key-value index. In a secondary index, the indexed values are not necessarily unique, meaning there might be multiple rows, documents, or vertices associated with the same index entry.

This can be handled in two ways:
- Store a list of matching row identifiers for each index entry.
- Make each entry unique by appending a row identifier to it.

Both storage engines with in-place updates, like B-Trees, and log-structured storage engines can be used to implement an index.
