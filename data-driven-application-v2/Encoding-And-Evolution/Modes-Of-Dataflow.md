# Modes of Dataflow

We will discuss the most common ways data flows between processes:

- Through databases
- Through service calls
- Through workflow engines
- Through asynchronous messages

## Dataflow Through Databases

In a database, the process that writes data encodes it, while the process that reads the data decodes it.

Think of storing something in a database as **sending a message to your future self**.

It is common for multiple processes to access the same database simultaneously. These processes might be:

- Different applications or services.
- Multiple instances of the same service running in parallel for scalability or fault tolerance.

As a result, some processes may be running newer versions of the code while others are still running older versions.

This means a value in the database could be written by a newer version of the code and subsequently read by an older version.

Therefore, **forward compatibility is particularly important when using databases**.

### Different Values Written at Different Times

Rewriting (migrating) existing data into a new schema is possible, but it can be expensive when dealing with large datasets.

Some storage engines handle schema changes without requiring an immediate rewrite of all existing data:

- **LSM-tree storage engines:** Data is rewritten into the latest format during compaction.
- **Relational databases:** Many allow simple schema changes, such as adding a new column with a `NULL` default value, without immediately rewriting existing rows.

When an old row is read, the database can supply `NULL` for any columns that are missing from the data stored on disk.

Schema evolution therefore allows the database to appear as though all records use a single, consistent schema, even though the underlying storage may contain records written using different historical versions of the schema.

More complex schema changes may require rewriting data at the application level. Examples include:

- Changing a single-valued attribute into a multivalued attribute.
- Moving data from an existing table into a separate table.

### Archival Storage

It is common to take snapshots of a database for backup purposes or to load data into a data warehouse.

It is useful to encode data consistently across different schema versions because snapshots taken at different points in time may contain data written using different historical schemas.

Since a database dump is typically written in one operation and remains immutable afterward, formats such as **Avro Object Container Files** are a good fit.

They allow records to be stored together with the schema needed to interpret them, making them suitable for large, immutable data dumps.
