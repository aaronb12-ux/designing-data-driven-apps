# Graph Queries in SQL

If we can put graph data in a relational structure, can we also query it using SQL?

We can, but it has some difficulty. Every edge that we traverse in a graph query is effectively a **join with the `edges` table**.

In a relational database, you usually know in advance which joins you need in your query. On the other hand, in a graph query, you may need to traverse a **variable number of edges** before you find the vertex you're looking for. That is, the number of joins is not fixed in advance.

This idea of variable-length traversal paths in a query can be expressed using **recursive common table expressions** with the `WITH RECURSIVE` syntax.

---

## Finding People Who Emigrated from the US to Europe

The below example shows the same query — finding the names of people who emigrated from the US to Europe — expressed in SQL using this technique.

The syntax is clumsy in comparison to Cypher.

```sql
WITH RECURSIVE

  -- in_usa is the set of vertex IDs of all locations within the United States
  in_usa(vertex_id) AS (
      SELECT vertex_id FROM vertices
        WHERE label = 'Location' AND properties->>'name' = 'United States'
    UNION
      SELECT edges.tail_vertex FROM edges
        JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
        WHERE edges.label = 'within'
  ),

  -- in_europe is the set of vertex IDs of all locations within Europe
  in_europe(vertex_id) AS (
      SELECT vertex_id FROM vertices
        WHERE label = 'Location' AND properties->>'name' = 'Europe'
    UNION
      SELECT edges.tail_vertex FROM edges
        JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
        WHERE edges.label = 'within'
  ),

  -- born_in_usa is the set of vertex IDs of all people born in the US
  born_in_usa(vertex_id) AS (
    SELECT edges.tail_vertex FROM edges
      JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
      WHERE edges.label = 'born_in'
  ),

  -- lives_in_europe is the set of vertex IDs of all people living in Europe
  lives_in_europe(vertex_id) AS (
    SELECT edges.tail_vertex FROM edges
      JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
      WHERE edges.label = 'lives_in'
  )

SELECT vertices.properties->>'name'
FROM vertices

-- join to find those people who were both born in the US and live in Europe
JOIN born_in_usa
  ON vertices.vertex_id = born_in_usa.vertex_id

JOIN lives_in_europe
  ON vertices.vertex_id = lives_in_europe.vertex_id;
```

---

## How the Query Works

1. First, find the vertex whose `name` property has the value `United States`, and make it the first element of the set of vertices `in_usa`.

2. Follow all incoming `within` edges from vertices in the set `in_usa` and add them to the same set, until all incoming `within` edges have been visited.

3. Do the same starting with the vertex whose `name` property has the value `Europe`, and build up the set of vertices `in_europe`.

4. For each of the vertices in the set `in_usa`, follow incoming `born_in` edges to find people who were born in some place within the US.

5. Similarly, for each of the vertices in the set `in_europe`, follow incoming `lives_in` edges to find people who live in Europe.

6. Finally, intersect the set of people born in the US with the set of people living in Europe by joining them.

---

## SQL vs. Cypher

The fact that a **4-line Cypher query requires 31 lines of SQL** shows how much of a difference the right choice of **data model** and **query language** can make.



