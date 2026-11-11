# DataFrames, Matrices, and Arrays

These structures are common in analytics, data processing, and machine learning rather than transactional application storage.

## 1. DataFrames

A DataFrame organizes data into rows and columns. Common tools include Python's pandas library, R, and Apache Spark.

DataFrames support operations similar to SQL:

- Filtering rows.
- Selecting and transforming columns.
- Joining datasets.
- Grouping rows and calculating aggregates.

They are useful for cleaning, exploring, and transforming data.

> Think of a DataFrame as a table you manipulate through code—not necessarily a database table.

## 2. Transforming Tabular Data into a Matrix

A matrix is a two-dimensional array. Many machine-learning algorithms work with numerical matrices.

For example, a table of movie ratings might contain:

```text
user_id | movie_id | rating
```

It can be reshaped into a matrix where:

- Each row represents a user.
- Each column represents a movie.
- Each cell contains that user's rating for that movie.

<img width="355" height="167" alt="Movie ratings transformed from a table into a user-by-movie matrix" src="https://github.com/user-attachments/assets/98c289fa-a5b5-426e-b459-eb35ce2b9d45" />

Most users have not rated most movies, so many combinations have no recorded rating. This produces a sparse structure.

Sparse array representations avoid storing every cell explicitly. However, a missing rating is not necessarily the same thing as a rating of zero.

The original ratings can still be stored efficiently as relational rows. Reshaping them into a matrix serves a different purpose: preparing the data for numerical computation.

> The storage format and the computation format do not have to be the same.

## 3. Multidimensional Arrays

A matrix has two dimensions; arrays can have more.

Multidimensional arrays are useful for scientific data such as geospatial measurements and medical images. Specialized systems such as TileDB support storing and querying array-shaped data.

## What to Remember

- DataFrame: A table-like structure for processing and analyzing data.
- Matrix: A two-dimensional array used in numerical computation.
- Multidimensional array: An array with additional dimensions.
- Sparse representation: Stores sparse data without explicitly storing every cell.

For backend and data-flow work, focus on the transformation:

```text
Stored records → DataFrame transformations → Analysis or numerical input
```

You do not need to learn the underlying linear algebra until your work requires it.
