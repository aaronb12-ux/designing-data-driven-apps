
# Column-Oriented Storage

Data warehouses often use a relational schema with a large fact table that contains foreign-key references to dimension tables.

If you have trillions of rows and petabytes of data in your fact tables, storing and querying them efficiently becomes challenging. Dimension tables are usually much smaller and more manageable, so we will focus on storing fact tables.

Fact tables exist in data warehouses, and a typical query accesses only a few columns from them. In the example below, the query accesses a large number of rows but needs only a few columns. Is there a better way to execute this query efficiently?

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

In most OLTP databases, storage is laid out in a **row-oriented** fashion: all values from one row of a table are stored next to one another.

Document databases are similar: an entire document is typically stored as one contiguous sequence of bytes.

To process the query above, you may have indexes on `fact_sales.date_key` and/or `fact_sales.product_sk`. A row-oriented storage engine may still need to load entire rows from disk into memory, parse them, and filter out those that don't meet the conditions. This can take a long time.

**Column-oriented (or columnar) storage** is simple: instead of storing all the values from one row together, it stores all the values from each column together.

If each column is stored separately, a query needs to read and parse only the columns it uses, which can save a lot of work.

![Column-oriented storage illustration](https://github.com/user-attachments/assets/474616a4-057b-434c-b73a-40d8825cee56)

Columnar storage engines don't store an entire column as one unit. Instead, they divide the table into blocks containing thousands or millions of rows and store each column's values separately within each block.

A range query needs to load only the columns it needs from the relevant blocks.

Columnar storage is used in almost all modern analytics databases.

## Column Compression

We can further reduce disk throughput and network bandwidth requirements by compressing data. Fortunately, column-oriented storage often lends itself very well to compression.

One technique that is effective in data warehouses is **bitmap encoding**:






