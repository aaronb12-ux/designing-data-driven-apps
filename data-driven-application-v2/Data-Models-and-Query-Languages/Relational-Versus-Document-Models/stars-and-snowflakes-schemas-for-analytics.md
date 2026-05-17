# Stars and Snowflakes: Schemas for Analytics
 
## Overview
 
Data warehouses are usually relational, and there are a few widely used conventions for the structure of tables in a data warehouse, including a star schema, a snowflake schema, dimensional modeling, and one big table (OBT). These structures are optimized for the needs of business analysts. ETL processes translate data from operational systems into the selected schema.
 
## Star Schema
 
The below figure shows an example of a star schema that might be found in the data warehouse of a grocery retailer. At the center of the schema is a so-called fact table (in this example, it is called `fact_sales`). Each row of the fact table represents an event that occurred at a particular time (here, each row represents a customer's purchase of a product).
 
If we were analyzing website traffic rather than retail sales, each row might represent a page view or click by a user.
 
<img width="527" height="454" alt="Screenshot 2026-05-17 at 1 49 02 PM" src="https://github.com/user-attachments/assets/df55c198-34fc-4b0a-83b1-cc6c52f78dae" />
### Fact Tables
 
Facts are captured as individual events, because this allows maximum flexibility of analysis later. This means that the fact table can become extremely large. A big enterprise may have many petabytes of transaction history in its data warehouse, mostly represented as fact tables.
 
Some columns in the fact table are attributes, such as the price at which the product was sold and the cost of buying it from the supplier. Other columns in the fact table are foreign-key references to other tables, called dimension tables.
 
As each row in the fact table represents an event, the dimensions represent the who, what, where, when, how, and why of the event.
 
In the above example, each row in the `fact_sales` table uses a foreign key to indicate which product was sold in that particular transaction.
 
### Why "Star" Schema?
 
The name star schema comes from the fact that when the table relationships are visualized, the fact table is in the middle, surrounded by its dimension tables; the connections to these tables are like the rays of a star.
 
## Snowflake Schema
 
A variation of this template is the snowflake schema, where dimensions are further broken into subdimensions. For example, there could be separate tables for brands and product categories, and each row in the `dim_product` table could reference the brand and category as foreign keys, rather than storing them as strings in the `dim_product` table. 
 
Snowflake schemas are more normalized than star schemas, but star schemas are often preferred because they are simpler for analysts to work with.
 
## Dimension Tables
 
In a typical data warehouse, tables are quite wide and can have hundreds of columns. 
 
Dimension tables can also be wide as they include all the metadata that may be relevant for analysis. For example, the `dim_store` table may include details of which services are offered at each store, whether it has an in-store bakery, the square footage, the date when the store first opened, when it was last remodeled, and how far it is from the nearest highway.
 
## Relationships
 
A star or snowflake schema consists mostly of many-to-one relationships (e.g., many sales occur for one particular product, in one particular store) represented as the fact table having foreign keys into dimension tables, or dimensions into subdimensions.
 
If a customer buys several different products at once, that multi-item transaction is not represented explicitly; instead, the fact table has a separate row for each product purchased, and those facts all just happen to have the same customer ID, store ID, and timestamp.
 
## One Big Table (OBT)
 
Some data warehouse schemas take denormalization even further and leave out the dimension tables entirely, folding the information in the dimensions into denormalized columns in the fact table instead (essentially precomputing the join between the fact table and the dimension tables).
 
This approach is called one big table (OBT) and while it requires more storage space, it sometimes enables faster queries.
 
---
 
## Summary
 
| Schema Type | Normalization | Complexity | Query Performance | Storage |
|-------------|---------------|------------|-------------------|---------|
| Star Schema | Denormalized | Simple | Fast | Moderate |
| Snowflake Schema | Normalized | Complex | Moderate | Efficient |
