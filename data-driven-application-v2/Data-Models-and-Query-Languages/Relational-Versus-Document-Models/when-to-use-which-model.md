# When To Use Which Model

The document data models provide schema flexibility, better performance due to locality, and — for some applications — a structure that is closer to the object model used by the application.

The relational model counters by providing better support for joins and many-to-one and many-to-many relationships.

## Schema Flexibility in the Document Model

Most document databases, and the JSON support in relational databases, do not enforce any schema on the data in documents.

When all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure.

## Data Locality For Reads and Writes

A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB's BSON).

If your application often needs to access the entire document, this storage locality has a performance advantage. If data is split across multiple tables, then multiple index lookups are required to retrieve it all, which may require more disk seeks and take more time.
