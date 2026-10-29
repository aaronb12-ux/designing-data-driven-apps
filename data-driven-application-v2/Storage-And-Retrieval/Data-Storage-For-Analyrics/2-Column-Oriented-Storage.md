
# Column-Oriented Storage

Data warehouses often use a relational schema with a large **fact table** containing foreign-key references to dimension tables.

When fact tables contain trillions of rows and petabytes of data, storing and querying them efficiently becomes challenging. Dimension tables are usually much smaller and more manageable, so we will focus on storing fact tables.

Analytical queries often scan a large number of rows but access only a few columns. For example, the following query calculates the total quantity of products sold, grouped by weekday and product category, for a particular year and selected categories:

```sql
SELECT
  dim_date.weekday,
  dim_product.category,
  SUM(fact_sales.quantity) AS quantity_sold
FROM fact_sales
  JOIN dim_date ON fact_sales.date_key = dim_date.date_key
  JOIN dim_product ON fact_sales.product_sk = dim_product.product_sk
WHERE
  dim_date.year = 2024 AND
  dim_product.category IN ('Fresh fruit', 'Candy')
GROUP BY
  dim_date.weekday,
  dim_product.category;
```

In most OLTP databases, storage is laid out in a **row-oriented** fashion: all values from one row of a table are stored together.

Document databases are similar: an entire document is typically stored as one contiguous sequence of bytes.

To process the query above, a row-oriented storage engine may need to read many rows from disk, even though the query uses only a few columns from the fact table. Indexes on `fact_sales.date_key` and `fact_sales.product_sk` may help locate relevant rows, but the database may still need to read and process many columns that the query doesn't use. This can be inefficient for analytical queries that scan large datasets.

**Column-oriented (or columnar) storage** addresses this problem by storing the values of each column together instead of storing all the values of each row together.

Because each column is stored separately, a query can read only the columns it needs, reducing disk I/O and the amount of data that must be processed.

![Column-oriented storage illustration](https://github.com/user-attachments/assets/474616a4-057b-434c-b73a-40d8825cee56)

Columnar storage engines typically divide data into blocks containing many rows. Within each block, the values from each column are stored separately. The system preserves the correspondence between values so that it can reconstruct rows when needed.

A range query can read only the required columns from the relevant blocks. Some systems can also skip blocks that cannot contain matching values.

Columnar storage is widely used in modern analytical databases.

## Column Compression

We can further reduce disk I/O and network bandwidth requirements by compressing data. Column-oriented storage is particularly well suited to compression because values within a column often have similar types or repeated values.

One technique used in data warehouses is **bitmap encoding**:

![Bitmap encoding illustration](https://github.com/user-attachments/assets/67f0daab-1d29-4abe-989e-8b73cb7efa3e)

For a column with `n` distinct values, we can create `n` separate bitmaps, one for each distinct value. Each bitmap contains one bit per row: the bit is `1` if the row contains that value and `0` otherwise.

For example, a bitmap for the value `CA` would have a `1` wherever the column contains `CA` and a `0` elsewhere.

These bitmaps can be sparse, meaning they contain many zeros. Storing every bit individually may therefore be inefficient.

**Run-length encoding** compresses consecutive sequences of identical bits by recording the bit value and the length of the run rather than storing every bit individually.

Techniques such as **Roaring Bitmaps** use different representations depending on the data's characteristics, choosing representations that efficiently store the bitmap. This can make bitmap indexes compact and efficient to query.

Bitmap indexes are well suited to common data warehouse queries:

- `WHERE product_sk IN (31, 68, 69)`
  - Retrieve the bitmaps for `product_sk = 31`, `product_sk = 68`, and `product_sk = 69`, then calculate their bitwise OR. The resulting bitmap identifies rows matching any of the three values.

- `WHERE product_sk = 30 AND store_sk = 3`
  - Retrieve the bitmaps for `product_sk = 30` and `store_sk = 3`, then calculate their bitwise AND. The resulting bitmap identifies rows matching both conditions.
  - This works because the bitmaps use a consistent row ordering: the *k*-th bit in each bitmap corresponds to the same row.

## Sort Order in Column Storage

Although columnar storage keeps each column's values together, the database must preserve the correspondence between values belonging to the same row.

Columnar databases can choose a sort order based on common query patterns. For example, if queries frequently filter by date ranges, sorting by `date_key` can make those queries more efficient.

Sorting groups identical or similar values together. This can improve compression because repeated values appear in long sequences, such as `CA, CA, CA, CA, NY, NY, NY`.

Sorting can also improve range queries. If data is sorted by date, the database can use block-level metadata to skip blocks whose date ranges do not overlap the requested range.

The sort order is a trade-off: choosing a sort key that benefits common queries can improve performance, but queries that filter by other columns may not benefit as much.

## Writing to Column-Oriented Storage

In data warehouses, a log-structured approach that batches many writes is often more efficient than modifying data directly in the middle of a large columnar dataset. Direct modifications could require rewriting substantial amounts of data.

One approach is to first write incoming records to a **row-oriented, in-memory store**. Once enough data accumulates, the system converts it into columnar format and merges it with existing data, writing new files in bulk.

This approach allows the system to handle incoming writes efficiently while preserving the benefits of columnar storage for analytical queries.

