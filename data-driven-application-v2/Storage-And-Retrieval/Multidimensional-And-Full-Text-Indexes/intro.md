
# Multidimensional and Full-Text Indexes

What if we want to search using more than one attribute? A single-column index may not be sufficient to efficiently perform the search.

## Types of Multicolumn Indexes

### Concatenated Index

- Combines several fields into a single key by appending one column's value to another.
- The resulting key is stored in a regular index, such as a B-Tree.
- The order of the columns matters because the index is sorted by the combined key. It is most effective when queries filter by the leading columns.

### Multidimensional Indexes

- Allow queries to search across multiple columns or dimensions simultaneously.
- They are particularly useful for multidimensional data, such as geospatial coordinates.

For example, consider a query that searches for restaurants within a specific geographic area:

```sql
SELECT *
FROM restaurants
WHERE latitude > 51.4946 AND latitude < 51.5079
  AND longitude > -0.1162 AND longitude < -0.1004;
```

This query searches within a rectangular geographic region defined by latitude and longitude ranges.

We could also use a concatenated index by transforming the two-dimensional location into a single value and then indexing that value using a regular B-Tree. However, this approach does not always handle multidimensional range queries efficiently.

Specialized multidimensional indexes, such as **R-trees**, are designed to handle searches across multiple dimensions more efficiently.
