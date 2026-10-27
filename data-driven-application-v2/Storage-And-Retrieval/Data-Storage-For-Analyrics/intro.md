# Data Storage for Analytics

The data model of a data warehouse is most commonly relational because SQL is generally a good fit for analytics queries. Many graphical data analysis tools generate SQL queries, visualize the results, and allow analysts to explore the data.

Data warehouses and relational OLTP databases look similar because they both have a SQL query interface. However, their internals can differ because they are optimized for very different query patterns.

**Hybrid transactional and analytical processing (HTAP)** databases increasingly use separate storage and query engines for transactional and analytical workloads, accessible through a common SQL interface.
