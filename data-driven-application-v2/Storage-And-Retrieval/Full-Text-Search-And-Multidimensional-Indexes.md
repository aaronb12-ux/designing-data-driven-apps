# Full-Text Search and Multidimensional Indexes

Different query patterns benefit from different index structures:

- Composite indexes: Search using an ordered combination of fields.
- Spatial indexes: Search locations or regions.
- Inverted indexes: Find documents containing particular terms.

## 1. Full-Text Search

Full-text search finds documents based on terms within their text, rather than requiring an exact match for the entire field.

For example, searching for `database` could find articles that mention databases in their titles or bodies.

Depending on configuration, search can also support word forms, synonyms, and typo tolerance. These are additional capabilities—not automatic properties of every full-text index.

### Text Analysis

Before indexing, text is commonly processed into searchable tokens:

```text
"Learning About Databases"
           ↓
["learning", "about", "databases"]
```

Analysis may include lowercasing, reducing words to a common form, or filtering common words. Search queries need compatible analysis so their tokens match the indexed terms. [140][141]

## 2. Inverted Indexes

An inverted index maps each term to the documents containing it:

```text
database → [document_1, document_3]
index    → [document_1, document_2]
```

These lists are called posting lists. They can also contain term frequencies and positions within documents. [139]

This reverses the usual direction:

```text
Document → Its words
```

into:

```text
Word → Matching documents
```

The engine can find candidates without scanning every document.

For multiple terms:

- AND: Find documents appearing in both posting lists.
- OR: Find documents appearing in either list.

Positions help support phrase matching. Relevance scoring determines the order of results rather than treating every matching document as equally useful. [137][146]

> The key concept is “term → matching documents.” You do not need to learn ranking formulas yet.

## 3. Composite Indexes: Ordered Combinations

A composite, or multicolumn, index stores a combination of column values in a defined order.

For example:

```sql
CREATE INDEX restaurants_city_rating_idx
ON restaurants (city, rating);
```

Think of the ordering as:

```text
First by city, then by rating within each city.
```

This is not necessarily literal string concatenation. It is an ordered combination of typed values.

Column order matters: Multicolumn B-tree indexes are generally most effective when queries constrain their leading columns. [133]

## 4. Spatial Indexes: Searching an Area

Some queries constrain multiple dimensions independently.

For example:

```sql
SELECT *
FROM restaurants
WHERE latitude > 51.4946
  AND latitude < 51.5079
  AND longitude > -0.1162
  AND longitude < -0.1004;
```

This asks for restaurants inside a latitude-longitude rectangle.

An ordinary B-tree index on `(latitude, longitude)` orders primarily by latitude. It does not directly group every possible two-dimensional rectangle into one contiguous range.

Spatial structures such as R-trees organize regions and bounding boxes to support these access patterns.

A spatial index can narrow the candidate set; the database may still need to check the exact spatial condition.

> Filtering on two columns does not automatically require a spatial index. What matters is the query pattern and whether a conventional index narrows the search sufficiently.

## What to Remember

| Query pattern | Index concept |
|---|---|
| Find posts by author, ordered by time. | Composite B-tree index. |
| Find restaurants inside an area. | Spatial index. |
| Find articles containing particular words. | Inverted index. |

All three avoid scanning everything, but they organize data differently.

Choose the index around what the query needs to find—not just the number or type of columns involved.
