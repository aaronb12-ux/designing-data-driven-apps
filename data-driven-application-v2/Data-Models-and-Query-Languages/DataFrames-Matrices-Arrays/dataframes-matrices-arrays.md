# DataFrames, Matrices, and Arrays

The data models we have seen so far in this chapter are generally used for both transaction processing and analytic purposes. There are also a few data models that you are likely to encounter in an analytical or scientific context but that rarely feature in OLTP systems, including DataFrames and multidimensional arrays of numbers, such as matrices.

## DataFrames

- The DataFrame data model is supported by the R language, the Pandas library for Python, Apache Spark, ArcticDB, Dask, and other systems.
- DataFrames are a popular tool for data scientists preparing data for training ML models, but they are also widely used for data exploration, statistical data analysis, data visualization, and similar purposes.
- A DataFrame supports relational-like operators that perform bulk operations on its contents, such as filtering the rows based on a condition, grouping rows by some columns, and aggregating other columns.
- A DataFrame is generally manipulated through a series of commands that modify its structure and content.

## Transforming to a Matrix

A common use of DataFrames is to transform data from a relational-like representation into a matrix or multidimensional array representation, which is the form in which many ML algorithms expect their input.

A simple example of such a transformation is shown below. On the left is a relational table, and on the right the data has been transformed into a matrix where each column is a movie and each row is a user.

<img width="355" height="167" alt="Screenshot 2026-09-28 at 3 17 24 AM" src="https://github.com/user-attachments/assets/98c289fa-a5b5-426e-b459-eb35ce2b9d45" />

- The matrix is sparse, which means there is no data for many user-movie combinations.
- The matrix may have thousands of columns and would therefore not fit well in a relational database, but DataFrames and libraries that offer sparse arrays can handle such data easily.
- Once the data is in the form of a matrix of numbers, it is amenable to linear algebra operations, which form the basis of many ML algorithms.

## Other Uses

- Some databases, such as TileDB, specialize in storing large multidimensional arrays of numbers. They are called **array databases** and are most commonly used for scientific datasets such as geospatial measurements and medical imaging.
- DataFrames are also used in the financial industry for representing time-series data, such as the prices of assets and trades over time.
- DataFrames have been added to batch processing frameworks such as Spark and Flink as well.
