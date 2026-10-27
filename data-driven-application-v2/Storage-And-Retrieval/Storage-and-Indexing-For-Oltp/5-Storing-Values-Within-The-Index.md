# Storing Values Within the Index

Normally, we query indexes using a key, but an index can also store additional data depending on its type.

## Clustered Index

- If the actual data (row, document, or vertex) is stored directly within the index structure, it is called a **clustered index**.
- When you look up a key in a clustered index, the database finds the key and immediately returns the entire row.

## Heap File

- Alternatively, the index value can be a reference to the actual data, such as the primary key of the row or a direct reference to its location on disk.
- When rows are stored separately from the index, the location where they are stored is known as a **heap file**. A heap file stores data in no particular order.
- PostgreSQL uses the heap file approach.

## Covering Index

- A middle ground is a **covering index**.
- It stores some of a table's columns within the index, in addition to storing the full row in the heap or in a primary-key clustered index.
