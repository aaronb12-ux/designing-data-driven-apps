# Property Graph Model

In a **property graph** (also called a **labeled property graph**) model, each vertex consists of the following:

### Vertex

- A **unique identifier**
- A **label** (string) describing the type of object the vertex represents
- A set of **outgoing edges**
- A set of **incoming edges**
- A collection of **properties** (key-value pairs)

### Edge

Each edge consists of the following:

- A **unique identifier**
- The vertex at which the edge starts (**tail vertex**)
- The vertex at which the edge ends (**head vertex**)
- A **label** describing the relationship between the two vertices
- A collection of **properties** (key-value pairs)

## Relational Representation

You can think of a graph store as consisting of two relational tables:

1. A table for **vertices**
2. A table for **edges**

### Representing a Property Graph with a Relational Schema

<img width="467" height="298" alt="Property graph relational schema" src="https://github.com/user-attachments/assets/cc132bf1-a72c-470c-af54-7023276d7ad0" />

## Important Aspects of the Model

- Any vertex can have an edge connecting it with **any other vertex**.
- There is **no schema restricting which kinds of things can or cannot be associated**.

### Graph Traversal

Given any vertex, you can efficiently find both its **incoming** and **outgoing** edges. This allows you to traverse the graph—following a path through a chain of vertices—in both directions:

- **Forward** through outgoing edges
- **Backward** through incoming edges

This is why the relational representation has indexes on both the `tail_vertex` and `head_vertex` columns.

### Labels and Relationships

By using different labels for different kinds of **vertices** and **relationships**, you can store several kinds of information in a single graph while still maintaining a clean data model.

### The Edges Table

The `edges` table is similar to a **many-to-many associative (join) table**, but generalized to allow many different types of relationships to be stored in the same table.

There may also be indexes on **labels** and **properties**, allowing vertices or edges with certain characteristics to be found efficiently.
