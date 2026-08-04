## Graph-Like Data Models

If your application has mostly one-to-many relationships (tree-structured data) and few other relationships between records, the document model is appropriate.

But what if many-to-many relationships are very common in your data? As connections within your data become more complex, it becomes more natural to start modeling data as a graph.

Some examples of data modeled as a graph:
- **Social Graph:** vertices are people, edges indicate which people know each other.
- **Web Graph:** vertices are web pages, and edges indicate HTML links to other pages.

Graphs are not limited to such homogeneous data. An equally powerful use of graphs is to provide a consistent way of storing completely different types of objects in a single database.

Graphs provide several different, but related, ways of structuring and querying data. In this section we will discuss the property graph model and triple-store model. These models are fairly similar in what they can express, and some graph databases support both.

We will also look at four query languages for graphs (Cypher, SPARQL, Datalog, and GraphQL), as well as SQL support for querying graphs.

<img width="421" height="289" alt="Screenshot 2026-08-04 at 12 35 31 AM" src="https://github.com/user-attachments/assets/1fe875df-bec5-4774-8d47-006378d509ea" />
