# Graph-Like Data Models

## 1. When Graph Models Are Useful

A graph represents data as:

- Vertices, also called nodes: Entities such as people, locations, or web pages.
- Edges: Relationships connecting those entities.

Examples:

| Graph | Nodes | Edges |
|---|---|---|
| Social network | People | Knows, follows |
| Web | Web pages | Links to |
| Location graph | Cities, states, countries | Located within |

A graph can contain different types of entities and relationships in the same model.

Document models fit naturally when information forms a tree and is usually read together. Graph models become useful when following connections is central to the application.

> Many-to-many relationships alone do not require a graph database. Relational databases can represent them with join tables. The important question is how frequently and deeply you need to traverse those relationships.

## 2. The Property Graph Model

A property graph stores nodes and relationships with properties.

### Nodes

A node has:

- An identifier.
- Labels describing its type.
- Properties stored as key-value pairs.

Example:

```text
Node:
  label: Person
  properties:
    name: "Lucy"
```

### Relationships

A relationship has:

- An identifier.
- A starting node, or tail.
- An ending node, or head.
- A relationship type.
- Properties describing the relationship.

Example:

```text
(Lucy)-[:WORKED_AT {title: "Engineer"}]->(Company)
```

Here, `WORKED_AT` describes the connection, and `title` describes that particular employment relationship.

Graph models are flexible, but that does not mean graph databases cannot enforce constraints. For example, Neo4j supports constraints on property uniqueness, existence, and types. [5]

### Traversal

Traversal means following relationships from one node to another.

```text
Lucy → BORN_IN → Idaho → WITHIN → United States
```

You may also need to follow relationships in the opposite direction—for example, starting from the United States and finding locations within it.

A relational representation could use:

```text
vertices
  vertex_id
  label
  properties

edges
  edge_id
  tail_vertex
  head_vertex
  label
  properties
```

The `edges` table acts like a generalized join table. Indexes on the starting and ending vertex columns support lookups in both directions.

## 3. Cypher: Querying Property Graphs

Cypher expresses graph queries using patterns.

### Basic Syntax

```cypher
(lucy:Person {name: 'Lucy'})-[:BORN_IN]->(idaho:Location)
```

Read this as:

> A person named Lucy has a `BORN_IN` relationship to a location.

- Parentheses represent nodes.
- Brackets represent relationships.
- Arrows show relationship direction.
- `lucy` and `idaho` are query variables.
- `Person` and `Location` are node labels.

The variable names are not automatically stored as properties.

### Creating a Small Graph

```cypher
CREATE
  (usa:Location {name: 'United States'}),
  (idaho:Location {name: 'Idaho'}),
  (europe:Location {name: 'Europe'}),
  (france:Location {name: 'France'}),
  (lucy:Person {name: 'Lucy'}),
  (idaho)-[:WITHIN]->(usa),
  (france)-[:WITHIN]->(europe),
  (lucy)-[:BORN_IN]->(idaho),
  (lucy)-[:LIVES_IN]->(france);
```

### Finding People Born in the US Who Now Live in Europe

```cypher
MATCH
  (person:Person)-[:BORN_IN]->()
    -[:WITHIN*0..]->(:Location {name: 'United States'}),
  (person)-[:LIVES_IN]->()
    -[:WITHIN*0..]->(:Location {name: 'Europe'})
RETURN DISTINCT person.name;
```

The query matches a person whose:

1. Birthplace is the United States or a location within it.
2. Current residence is Europe or a location within it.

`[:WITHIN*0..]` means following zero or more `WITHIN` relationships. Zero allows the starting location itself to match the destination. This variable-length relationship syntax is supported by Cypher. [3]

This finds people matching the birthplace and residence conditions. It does not reconstruct their complete migration history.

### Query Execution

The written pattern does not require the database to start by scanning every person.

It could instead locate the US and Europe, then work backward through relationships. Indexes and the query planner influence how the query executes.

> Describe the relationship pattern you want; do not assume the database will execute it in the order you wrote it.

## 4. Graph Queries in SQL

Graph data can also be stored and queried in a relational database.

For a fixed-length path, joins can connect each step. For a variable-length path, recursive common table expressions can repeatedly follow relationships.

### Example: Find Locations Within the US

Assume the `vertices` and `edges` tables described earlier, with uppercase relationship labels.

```sql
WITH RECURSIVE in_usa(vertex_id) AS (
    -- Start with the United States.
    SELECT vertex_id
    FROM vertices
    WHERE label = 'Location'
      AND properties->>'name' = 'United States'

    UNION

    -- Find locations whose parent is already in the result.
    SELECT e.tail_vertex
    FROM edges AS e
    JOIN in_usa AS parent
      ON e.head_vertex = parent.vertex_id
    WHERE e.label = 'WITHIN'
)
SELECT v.properties->>'name' AS location_name
FROM vertices AS v
JOIN in_usa
  ON v.vertex_id = in_usa.vertex_id;
```

The query:

1. Starts with the US node.
2. Finds locations connected to it by incoming `WITHIN` edges.
3. Repeats for newly found locations.

The result includes the US itself and its nested locations.

Cypher can express this kind of traversal more compactly, but shorter syntax does not prove faster execution or mean SQL is unsuitable.

## 5. Datalog: Facts and Rules

Datalog describes data using facts and derives additional results using rules.

For now, focus on the idea rather than memorizing another query language.

### Facts

Facts resemble rows in tables:

```prolog
within("Idaho", "United States").
within("United States", "North America").
```

### Rules

A rule states what can be concluded when its conditions are true.

```prolog
inside(X, Y) :- within(X, Y).

inside(X, Z) :- within(X, Y), inside(Y, Z).
```

Read `:-` as “if”:

- `X` is inside `Y` if it is directly within `Y`.
- `X` is inside `Z` if it is within some `Y` that is inside `Z`.

These rules let the system derive that Idaho is inside North America.

> The key concept is recursive reasoning: expressing both direct and indirect relationships using rules.

---

# GraphQL: An API Query Language

## 6. GraphQL Is Not a Graph Database

GraphQL lets clients request specific fields from a server-defined API schema. It is different from a database query language such as Cypher or SQL. [2][7]

It can sit in front of relational databases, document databases, graph databases, or other services.

A useful mental model is:

```text
Frontend
   ↓ GraphQL request
API server
   ↓ Database queries or service calls
Databases / internal services
```

### Example: A Chat Application

```graphql
query ChatApp {
  channels {
    name
    recentMessages(latest: 50) {
      content
      sender {
        fullName
      }
      replyTo {
        content
      }
    }
  }
}
```

The client requests:

- Channel names.
- Up to 50 recent messages per channel, according to the API's implementation.
- Each message's content and sender name.
- The original message's content when the message is a reply.

The response follows the requested field structure. Clients can select fields exposed by the schema; they cannot invent arbitrary fields or database queries. [7]

### What GraphQL Does Not Automatically Solve

Selecting fewer response fields does not guarantee efficient database access.

For example, a server might:

1. Fetch a list of messages.
2. Make a separate database query for each message's sender.

This is the N+1 query problem. Batching related lookups is one way to address it. [12]

The server is still responsible for authorization, query limits, and efficient data fetching.

### Recursion and Search

GraphQL can request several explicitly written levels of related data, but it does not provide Cypher-style arbitrary-depth traversal.

Search and filtering are available only through the fields and arguments the API exposes.

> GraphQL determines what a client can request from an API. It does not determine how the server stores or retrieves that information.

---

# Triple Stores, RDF, and SPARQL

## 7. Triple Stores

A triple store represents information as three-part statements:

```text
(subject, predicate, object)
```

Examples:

```text
(lucy, name, "Lucy")
(lucy, bornIn, idaho)
(idaho, within, usa)
```

- Subject: The entity being described.
- Predicate: The attribute or relationship.
- Object: A value or another entity.

These statements can represent both properties and connections.

Property graphs and triple stores both represent graph-shaped data, but they are not identical models.

## 8. RDF and Turtle

RDF, the Resource Description Framework, is a graph data model based on triples. SPARQL is a query language for RDF data. [16]

Turtle is one text format for writing RDF:

```turtle
@prefix ex: <urn:example:>.

ex:lucy ex:name "Lucy".
ex:lucy ex:bornIn ex:idaho.
ex:idaho ex:within ex:usa.
ex:usa ex:name "United States".
```

The prefix expands names:

```text
ex:within → urn:example:within
```

Namespaced identifiers let different datasets distinguish terms that might otherwise share the same short name.

For now, understand the triple structure and the purpose of identifiers. Memorizing RDF/XML is unnecessary.

## 9. SPARQL

SPARQL matches patterns across RDF triples.

For example, this query finds the names of people born in the US or a location within it:

```sparql
PREFIX ex: <urn:example:>

SELECT DISTINCT ?personName
WHERE {
  ?person ex:name ?personName.
  ?person ex:bornIn / ex:within* ?location.
  ?location ex:name "United States".
}
```

- `?person` and `?location` are variables.
- `/` chains relationship steps.
- `*` means zero or more repetitions.

The important connection is that SPARQL and Cypher both let you describe graph patterns, but they target different graph models.

---

# What to Prioritize

## Understand Well

- Nodes, edges, properties, and relationship direction.
- Traversing direct and indirect relationships.
- Why many-to-many relationships do not automatically require a graph database.
- The difference between a graph database query language and GraphQL.
- How a frontend request becomes backend database queries or service calls.
- Why a convenient API can still produce inefficient database access.

## Recognize Without Memorizing

- Cypher: Pattern-based queries for property graphs.
- Recursive SQL: Repeated traversal in relational data.
- Datalog: Facts and rules for deriving relationships.
- RDF: A triple-based data model.
- SPARQL: Queries over RDF data.

## Leave for Later

- Full query syntax for every language.
- RDF/XML serialization.
- Semantic Web history.
- Lists of products supporting each language.
- Detailed graph query optimization.

The main goal is to understand how relationships are represented and queried—not to learn several specialized languages at once.
