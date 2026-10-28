# Cloud Data Warehouses

Unlike traditional data warehouses, cloud data warehouses can take advantage of scalable cloud infrastructure, such as object storage and serverless computing platforms.

Many cloud warehouses support automatic log ingestion and integrate easily with data processing frameworks such as Google Cloud Dataflow and AWS Kinesis. They are also more elastic because they decouple query computation from the storage layer.

Data is persisted in object storage rather than on local disks, making it easy to adjust storage capacity and compute resources independently.

The following components, which used to be integrated into a single system, are now implemented separately:

- **Query Engine:** Parses SQL queries, optimizes them into execution plans, and executes them against the data.
- **Storage Format:** Determines how table rows are encoded as bytes in a file, which is typically stored in object storage or a distributed file system.
- **Table Format:** Specifies which files constitute a table and defines the table's schema.
- **Data Catalog:** Defines which tables are contained in a database and is used to create, rename, and drop tables.
