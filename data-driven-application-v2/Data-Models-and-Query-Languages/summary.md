# Summary

The relational model remains an important data model for many applications — especially in data warehousing and business analytics, where relational star or snowflake schemas and SQL queries are ubiquitous (present everywhere at the same time). However, there are several alternatives to relational data that have become popular in other domains:

- The document model targets use cases where data comes in self-contained JSON documents and where relationships between one document and another are rare.
- Graph data models go in the opposite direction, targeting use cases where anything is potentially related to everything and where queries potentially need to traverse multiple hops to find the data of interest (a need that can be met using recursive queries in Cypher, SPARQL, or Datalog).
- DataFrames generalize relational data to large numbers of columns, providing a bridge between databases and the multidimensional arrays that form the basis of much machine learning, statistical data analysis, and scientific computing.

One event-based model we discussed is event sourcing, which represents data as an append-only log of immutable events and can be advantageous for modeling activities in complex business domains. An append-only log is good for writing data; in order to support efficient queries, the event log is translated into read-optimized materialized views through CQRS.

One thing that non-relational data models have in common is that they typically don't enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements. However, your application most likely still assumes that data has a certain structure.
