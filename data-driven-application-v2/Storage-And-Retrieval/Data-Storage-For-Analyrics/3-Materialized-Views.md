# Materialized Views

A **materialized view** is the actual result of a query, stored on disk, whereas a **virtual view** is essentially a shortcut for writing queries.

When you read from a virtual view, the SQL engine expands it into its underlying query and executes that query against the underlying tables.

When the underlying data changes, a materialized view needs to be updated to reflect those changes. This introduces additional write overhead. However, materialized views can significantly improve read performance when workloads repeatedly execute the same queries.

## Materialized Aggregates

**Materialized aggregates** are a type of materialized view that can be particularly useful in data warehouses.

Data warehouse queries often involve aggregate functions such as `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`. If many queries repeatedly use the same aggregates, processing the raw data every time can be wasteful.

Instead, we can precompute and store these common query results in a **data cube**. A data cube organizes aggregated values across multiple dimensions, creating a collection of aggregates grouped by different combinations of dimensions.

![Data cube illustration](https://github.com/user-attachments/assets/de0e2d7a-243b-479b-b632-b28efc3de195)

A data cube allows queries to retrieve precomputed aggregates rather than repeatedly scanning and processing the underlying data. However, these aggregates must be maintained as the underlying data changes.
