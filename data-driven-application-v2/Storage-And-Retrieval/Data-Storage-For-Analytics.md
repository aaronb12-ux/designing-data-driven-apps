# Data Storage for Analytics

Analytical systems answer questions across large datasets, such as:

- How much revenue did each product category generate?
- How did usage change over the last year?
- Which regions had the highest sales?

Data warehouses commonly expose relational tables and SQL, but their storage and execution strategies differ from transactional databases.

| Workload | Typical access pattern |
|---|---|
| OLTP | Read or update a small number of records per request. |
| Analytics / OLAP | Scan many records, select a few columns, and calculate aggregates. |

> A shared SQL interface does not mean two databases organize or process data the same way.

## 1. Cloud Warehouses: Separate Storage and Compute

Many cloud warehouses separate persistent storage from the compute resources that execute queries.

```text
SQL query
    ↓
Compute workers
    ↓
Shared persistent storage
```

This lets storage and compute capacity scale independently. BigQuery, for example, separates the two and distributes query processing across workers. [153]

A common architecture stores durable data remotely and may use local storage or memory as a cache. Not every warehouse uses identical storage mechanisms.

### Components to Recognize

These distinctions are especially useful in open data-lake and lakehouse architectures:

| Component | Responsibility |
|---|---|
| Query engine | Plans and executes queries. |
| File format | Defines how values are encoded within files, such as Parquet. |
| Table format | Tracks the files and metadata representing a table, such as Iceberg. |
| Catalog | Helps locate tables and their current metadata. |

For example, an engine can use a catalog to locate an Iceberg table, read its metadata, and then access the underlying Parquet files. [152]

> A file format describes file contents. A table format manages a table that may span many files.

## 2. Column-Oriented Storage

### Row-Oriented Layout

Values from the same row are stored together:

```text
Row 1: date, product, store, quantity, price
Row 2: date, product, store, quantity, price
```

This is convenient when an operation needs most of a particular record.

### Column-Oriented Layout

Values from the same column are stored together:

```text
date:     [...]
product:  [...]
store:    [...]
quantity: [...]
price:    [...]
```

An analytical query can read the columns it needs without reading all the others. This reduces I/O for queries that scan many rows but use only a subset of fields. [153]

![Column-oriented storage illustration](https://github.com/user-attachments/assets/474616a4-057b-434c-b73a-40d8825cee56)

### Example: Sales by Weekday and Category

```sql
SELECT
    d.weekday,
    p.category,
    SUM(s.quantity) AS quantity_sold
FROM fact_sales AS s
JOIN dim_date AS d
    ON s.date_key = d.date_key
JOIN dim_product AS p
    ON s.product_sk = p.product_sk
WHERE d.year = 2024
  AND p.category IN ('Fresh fruit', 'Candy')
GROUP BY d.weekday, p.category;
```

From the fact table, the query needs only:

- `date_key`
- `product_sk`
- `quantity`

It does not need unrelated fields, even if the table contains hundreds of columns.

### Preserving Rows

Columnar formats typically group rows into blocks or row groups, then store each column's values together within the group.

The positions still correspond:

```text
The third date, third product, and third quantity
belong to the same row.
```

Columns are not independently reordered, because that would break the records.

## 3. Compression and Skipping Data

Columnar layouts put similar values together, making them good candidates for compression.

For example:

```text
CA, CA, CA, CA, NY, NY, NY
```

Run-length encoding can represent this as:

```text
CA × 4
NY × 3
```

Sorting related values together can improve compression.

### Block Skipping

Systems can also keep metadata about blocks, such as minimum and maximum dates.

For a query requesting one month, blocks entirely outside that month can be skipped.

Sorting or clustering by a frequently filtered field makes this metadata more useful.

> Reading fewer columns and skipping irrelevant blocks are separate optimizations that can work together.

The best sort order depends on common queries. Sorting by date may help date filters more than filters on unrelated columns.

## 4. Bitmap Indexes

A bitmap represents matching rows using bits:

- `1`: The row matches.
- `0`: The row does not match.

For a region column:

```text
Rows:  CA  NY  CA  TX

CA:    1   0   1   0
NY:    0   1   0   0
TX:    0   0   0   1
```

![Bitmap encoding illustration](https://github.com/user-attachments/assets/67f0daab-1d29-4abe-989e-8b73cb7efa3e)

Bitwise operations combine conditions:

- OR finds rows matching either condition.
- AND finds rows matching both conditions.

For example:

```text
product = 30 bitmap
          AND
store = 3 bitmap
          ↓
Rows matching both
```

This requires the bit positions to refer to the same rows.

Bitmaps can be compressed. Run-length encoding helps with long repeated runs; formats such as Roaring use different representations depending on the data.

Bitmap indexes are one analytical technique—not a requirement of columnar storage, and not always the best representation for every column.

## 5. Writing to Columnar Storage

Columnar organization favors bulk processing, but incoming application records usually arrive one at a time.

A common approach is:

```text
Incoming records
       ↓
Write buffer or staging area
       ↓
Batch into columnar files
       ↓
Compact or reorganize files as needed
```

Batching avoids repeatedly rewriting large compressed structures for individual changes.

Exact update mechanisms vary. Some systems write replacement files or record changes separately before later consolidation. For example, managed Iceberg tables can create new files for writes and perform background compaction. [164]

> Analytics storage moves some work into batching and background maintenance to make large reads efficient.

## 6. Virtual Views vs. Materialized Views

| Type | What is stored | Main trade-off |
|---|---|---|
| Virtual view | A query definition. | Reuses query logic, but does not inherently store its results. |
| Materialized view | Persisted results derived from a query. | Faster repeated reads, but requires storage and refresh work. |

For example, repeatedly scanning raw sales to calculate daily totals may be wasteful.

A materialized view can store those totals ahead of time. BigQuery supports materialized views for recurring aggregation patterns to reduce repeated computation and scanning. [155]

Refresh behavior depends on the system:

- Some views update incrementally.
- Some refresh periodically or on demand.
- Results may lag behind source data unless the system compensates for recent changes.

Always check the freshness guarantee, not just the read speed.

### Materialized Aggregates and Data Cubes

A materialized aggregate stores a summary such as:

```text
date + category → total quantity sold
```

A data cube organizes aggregates across combinations of dimensions, such as date, category, and region.

![Data cube illustration](https://github.com/user-attachments/assets/de0e2d7a-243b-479b-b632-b28efc3de195)

The benefit is avoiding repeated scans of raw events.

The cost is maintaining summaries and storing extra data. Precomputing more dimension combinations can increase that cost substantially.

You do not need to materialize every possible combination—focus on summaries that recurring queries actually use.

## What to Remember

- OLTP commonly accesses a few records; analytics commonly scans many records and a few columns.
- Columnar storage reduces unnecessary column reads and supports effective compression.
- Sort order and block metadata can eliminate irrelevant reads.
- Bitmap indexes combine matching row sets efficiently.
- Batching and background maintenance help manage columnar writes.
- Materialized views trade refresh work and storage for faster repeated queries.
- File formats, table formats, catalogs, and query engines solve different problems.

For backend and data-flow work, understand this path:

```text
Application events or records
            ↓
Ingestion and transformation
            ↓
Analytical tables
            ↓
SQL queries and materialized summaries
            ↓
Reports, dashboards, or downstream applications
```
