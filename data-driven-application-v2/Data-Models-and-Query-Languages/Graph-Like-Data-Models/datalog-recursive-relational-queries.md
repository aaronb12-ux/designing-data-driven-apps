# Datalog: Recursive Relational Queries
 
Datalog is a much older language than SPARQL or Cypher. It arose from academic research in the 1980s. It is a very expressive language that is especially powerful for complex queries.
 
Many niche databases — such as **Datomic**, **LogicBlox**, **CozoDB**, and **LinkedIn Liquid** — use Datalog as their query language.
 
Datalog is a subset of **Prolog**, a programming language.
 
## Facts
 
The contents of a Datalog database are known as **facts**, and each fact corresponds to a row in a relational table.
 
For example, if we have a `location` table containing locations, with three columns — `ID`, `name`, and `type` — the fact that the US is a country could be written as:
 
```prolog
location(2, "United States", "country").
```
 
where `2` is the ID of the US. In general, the statement `table(val1, val2, ...)` means that `table` contains a row where the first column contains `val1`, the second column contains `val2`, and so on.
 
## Example Data
 
```prolog
location(1, "North America", "continent").
location(2, "United States", "country").
location(3, "Idaho", "state").
 
within(2, 1).    /* US is in North America */
within(3, 2).    /* Idaho is in the US     */
 
person(100, "Lucy").
born_in(100, 3). /* Lucy was born in Idaho */
```
 
## Rules
 
Now that we have defined the data, we can write the same kind of query we've written before in Cypher and SPARQL.
 
Unlike Cypher and SPARQL, which jump right in with `SELECT`, Datalog takes a small extra step: we define **rules** that derive new virtual tables from the underlying facts.
 
These derived tables are like (virtual) SQL views — they are not stored in the database, but can be queried the same way as a table containing stored facts.
 
```prolog
within_recursive(LocID, PlaceName) :- location(LocID, PlaceName, _). /* Rule 1 */
 
within_recursive(LocID, PlaceName) :- within(LocID, ViaID),          /* Rule 2 */
                                       within_recursive(ViaID, PlaceName).
 
migrated(PName, BornIn, LivingIn)  :- person(PersonID, PName),       /* Rule 3 */
                                       born_in(PersonID, BornID),
                                       within_recursive(BornID, BornIn),
                                       lives_in(PersonID, LivingID),
                                       within_recursive(LivingID, LivingIn).
 
us_to_europe(Person) :- migrated(Person, "United States", "Europe"). /* Rule 4 */
/* us_to_europe contains the row "Lucy". */
```
 
In the example above, we define three virtual tables:
 
- `within_recursive(LocID, PlaceName)`
- `migrated(PName, BornIn, LivingIn)` — a virtual table with three columns
- `us_to_europe(Person)`
### How rules work
 
- The content of a virtual table is defined by the part of the rule **after** the `:-` symbol, where we try to find rows that match a certain pattern across the underlying tables.
- A rule **applies** if the system can find a match for all patterns on the right-hand side of the `:-` operator.
- When a rule applies, it's as though the left-hand side of the `:-` was added to the database, with variables replaced by the values they matched.
