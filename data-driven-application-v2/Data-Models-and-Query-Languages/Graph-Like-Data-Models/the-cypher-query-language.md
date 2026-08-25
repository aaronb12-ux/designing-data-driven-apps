# The Cypher Query Language

Cypher is a query language for property graphs, originally created for the **Neo4j** graph database and later developed into an open standard as **openCypher**. Besides Neo4j, Cypher is supported by **Memgraph, KuzuDB, Amazon Neptune, Apache AGE**, and others.

## Creating a Graph with Cypher

The below example shows the Cypher query to insert the below **Figure 3.6** into a graph database.

Each vertex is given a symbolic name, like `usa` or `idaho`. That name is not stored in the database but used only internally within the query to create edges between the vertices, using an arrow notation:

```cypher
(idaho)-[:WITHIN]->(usa)
```

This creates an edge labeled `WITHIN`, with `idaho` as the **tail node** and `usa` as the **head node**.

```cypher
CREATE
  (namerica :Location {name:'North America',  type:'continent'}),
  (usa      :Location {name:'United States',  type:'country'  }),
  (idaho    :Location {name:'Idaho',          type:'state'    }),
  (lucy     :Person   {name:'Lucy' }),
  (idaho) -[:WITHIN]-> (usa) -[:WITHIN]-> (namerica),
  (lucy)  -[:BORN_IN]-> (idaho)
```

---

## Finding People Who Emigrated from the US to Europe

Suppose we want to find the names of all the people who emigrated from the **United States to Europe**. We can do that by finding all the vertices that have:

- A `BORN_IN` edge to a location within the US
- A `LIVES_IN` edge to a location within Europe

Then, we return the `name` property of each of those vertices.

The below shows how to express that query in Cypher. The same arrow notation is used in a `MATCH` clause to find patterns in the graph:

```cypher
(person)-[:BORN_IN]->()
```

This matches any two vertices that are related by an edge labeled `BORN_IN`.

- The **tail vertex** of that edge is bound to the variable `person`.
- The **head vertex** is left unnamed.

### A Cypher Query to Find People Who Emigrated from the US to Europe

```cypher
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (:Location {name:'Europe'})
RETURN person.name
```

### Reading the Query

The query can be read as follows:

Find any vertex (call it `person`) that meets **both** of the following conditions:

1. `person` has an outgoing `BORN_IN` edge to a vertex.
   - From that vertex, you can follow a chain of outgoing `WITHIN` edges until eventually you reach a vertex of type `Location`, whose `name` property is equal to `United States`.

2. That same `person` vertex also has an outgoing `LIVES_IN` edge.
   - Following that edge, and then a chain of outgoing `WITHIN` edges, you eventually reach a vertex of type `Location`, whose `name` property is equal to `Europe`.

For such a `person` vertex, return the `name` property.

---

## Query Execution

There are several possible ways of executing the query.

### Approach 1: Start with the People

The description given here suggests that you start by scanning all the people in the database, examining each person's **birthplace** and **residence**, and returning only those people who meet the criteria.

### Approach 2: Start with the Locations

But equivalently, you could start with the two `Location` vertices and work backward.

If there is an index on the `name` property, you can efficiently find the two vertices representing:

- The **US**
- **Europe**

Then you can proceed to find all locations (states, regions, cities, etc.) in the US and Europe, respectively, by following all incoming `WITHIN` edges.

Finally, you can look for people who can be found through an incoming `BORN_IN` or `LIVES_IN` edge at one of the location vertices.
