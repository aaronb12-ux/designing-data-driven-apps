## Object-Relational Mapping

Object-relational mapping (ORM) frameworks like ActiveRecord and Hibernate reduce the amount of boilerplate code required for this translation layer for converting between the objects in the application code and the database model of tables, rows, and columns.

**Here are some commonly cited problems with ORMs:**

* ORMs are complex and cannot completely hide the differences between the two models, so developers still end up having to think about the relational and the object representations of the data
* Many ORMs work only with relational OLTP databases. Organizations with diverse data systems such as search engines, graph databases, and NoSQL systems might find ORM support lacking
* Some ORMs automatically generate relational schemas, but these might be awkward for users who are directly accessing the relational data, and they might be inefficient on the underlying database. Customizing the ORM's schema and query generation can be complex and negate the benefit of using the ORM in the first place

**Here are some advantages of ORMs:**

* For data that is well suited to a relational model, some kind of translation between the persistent relational and in-memory object representation is inevitable, and ORMs reduce the amount of boilerplate code required for this translation
* Some ORMs help with caching the results of database queries, which can help reduce the load on the database
* ORMs can also help with managing schema migrations and other administrative activities
