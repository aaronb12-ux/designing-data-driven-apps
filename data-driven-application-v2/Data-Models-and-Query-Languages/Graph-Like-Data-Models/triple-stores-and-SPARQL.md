# Triple Stores and SPARQL

The **triple store model** is mostly equivalent to the property graph model, using different words to describe the same ideas. It is nevertheless worth discussing because various tools and languages for triple stores can be valuable additions to your toolbox for building applications.

In a triple store, all information is stored in the form of very simple three-part statements:

```text
(subject, predicate, object)
```

The **subject** of a triple is equivalent to a vertex in a graph. The **object** is one of two things:

- A value of a **primitive datatype**, such as a string or a number.
  - In that case, the predicate and object of the triple are equivalent to the **key and value of a property** on the subject vertex.
  - For example:

    ```text
    (lucy, birthYear, 1989)
    ```

    This is like a vertex `lucy` with properties:

    ```text
    {birthYear: 1989}
    ```

- **Another vertex in the graph.**
  - In that case, the predicate is an **edge** in the graph, the subject is the **tail vertex**, and the object is the **head vertex**.
  - For example:

    ```text
    (lucy, marriedTo, alain)
    ```

    The subject and object, `lucy` and `alain`, are both vertices, and the predicate `marriedTo` is the label of the edge that connects them.

---

## Representing Triples with Turtle

The below example shows the same data from the above section written as triples in a format called **Turtle**, a subset of **Notation3 (N3)**.

```turtle
@prefix : <urn:example:>.

_:lucy     a       :Person.
_:lucy     :name   "Lucy".
_:lucy     :bornIn _:idaho.

_:idaho    a       :Location.
_:idaho    :name   "Idaho".
_:idaho    :type   "state".
_:idaho    :within _:usa.

_:usa      a       :Location.
_:usa      :name   "United States".
_:usa      :type   "country".
_:usa      :within _:namerica.

_:namerica a       :Location.
_:namerica :name   "North America".
_:namerica :type   "continent".
```

Vertices of the graph are written as `_:someName`.

When the predicate represents an **edge**, the object is a vertex, as in:

```turtle
_:idaho :within _:usa.
```

When the predicate is a **property**, the object is a string literal, as in:

```turtle
_:usa :name "United States".
```

Triple stores are another **Semantic Web** technology that has found use outside its original use case. Even if you have no interest in the Semantic Web, triples can be a good internal data model for applications.

---

# The RDF Data Model

The Turtle language we used above is actually a way of encoding data in the **Resource Description Framework (RDF)**, a data model that was designed for the Semantic Web.

RDF can also be encoded in other ways, including, more verbosely, **XML**. Tools like **Apache Jena** can automatically convert between different RDF encodings.

## The Data from the Previous Example Expressed Using RDF/XML Syntax

```xml
<rdf:RDF xmlns="urn:example:"
    xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">

  <Location rdf:nodeID="idaho">
    <name>Idaho</name>
    <type>state</type>
    <within>
      <Location rdf:nodeID="usa">
        <name>United States</name>
        <type>country</type>
        <within>
          <Location rdf:nodeID="namerica">
            <name>North America</name>
            <type>continent</type>
          </Location>
        </within>
      </Location>
    </within>
  </Location>

  <Person rdf:nodeID="lucy">
    <name>Lucy</name>
    <bornIn rdf:nodeID="idaho"/>
  </Person>

</rdf:RDF>
```

The **subject, predicate, and object** of a triple are often URIs.

For example, a predicate might be a URI such as:

```text
<http://my-company.com/namespace#within>
```

or:

```text
<http://my-company.com/namespace#lives_in>
```

rather than just `WITHIN` or `LIVES_IN`.

This is because you should be able to combine your data with someone else's data. If they attach a different meaning to the words `within` or `lives_in`, you won't get a conflict because their predicates are actually:

```text
<http://other.org/foo#within>
```

and:

```text
<http://other.org/foo#lives_in>
```

---

# The SPARQL Query Language

**SPARQL** is a query language for triple stores using the RDF data model. It predates Cypher, and since Cypher's pattern matching is borrowed from SPARQL, they look quite similar.

The same query as before — finding people who have moved from the US to Europe — is similarly concise in SPARQL as it is in Cypher.

## Finding People Who Moved from the US to Europe

```sparql
PREFIX : <urn:example:>

SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn  / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```

The structure is very similar.

The following two expressions are equivalent. Variables start with a question mark (`?`) in SPARQL:

### Cypher

```cypher
(person)-[:BORN_IN]->()-[:WITHIN*0..]->(location)
```

### SPARQL

```sparql
?person :bornIn / :within* ?location.
```

In the following expression, the variable `usa` is bound to any vertex that has a `name` property whose value is the string `United States`.

### Cypher

```cypher
(usa {name:'United States'})
```

### SPARQL

```sparql
?usa :name "United States".
```
