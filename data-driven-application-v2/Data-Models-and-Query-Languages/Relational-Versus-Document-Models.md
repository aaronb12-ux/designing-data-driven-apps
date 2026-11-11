# Relational and Document Data Models

## 1. The Core Difference

The relational model organizes data into tables containing rows and columns. Related records can live in separate tables and be connected through keys and joins. SQL is the language commonly used to query relational databases.

The document model represents data as documents, usually using JSON-like structures. A document can contain nested objects and arrays, making it convenient to store related information together.

NoSQL is an umbrella term, not a single data model. Document databases are one category of NoSQL database.

The main design question is:

> Which information should live together, and which information should be stored separately and referenced?

## 2. Example: A User Profile

A user profile contains a name, previous jobs, education, and contact information.

### Relational Representation

Fields that occur once per user, such as `first_name` and `last_name`, belong in a `users` table.

Repeating information can live in separate tables:

- `positions`: Jobs held by a user.
- `education`: Educational history.
- `contact_info`: Contact details.

Each related row contains a `user_id` referencing the user.

<img width="481" height="441" alt="Relational schema for a user profile" src="https://github.com/user-attachments/assets/95ac9be8-4fd2-4cdd-9faa-fa42e33a4205" />

Fetching the complete profile requires joining the tables or making several queries.

### Document Representation

A document can store the profile and its related information together:

```json
{
  "user_id": 251,
  "first_name": "Jane",
  "last_name": "Smith",
  "positions": [
    {
      "organization": "Example Company",
      "title": "Software Engineer"
    }
  ],
  "education": [
    {
      "school_name": "Example University"
    }
  ]
}
```

This structure maps naturally to an application object. Fetching the document retrieves the embedded profile information together.

Embedding is especially useful for one-to-few relationships, such as a resume with a small number of jobs.

It becomes less convenient for very large collections, such as thousands of comments on one post.

> Storing related data together can simplify reads, but that does not make documents universally faster. The benefit depends on what the application reads and updates.

## 3. Keys and Relationships

### Primary Key

A primary key uniquely identifies a row.

For example, `users.id` identifies a particular user.

### Foreign Key

A foreign key references a key in another table.

For example, `positions.user_id` references `users.id`.

### Relationship Types

| Relationship | Meaning | Example |
|---|---|---|
| One-to-many | One record relates to several records. | One user has many positions. |
| Many-to-one | Several records relate to one record. | Many users live in one region. |
| Many-to-many | Records on both sides can relate to several records on the other side. | Users work for multiple organizations, and organizations employ multiple users. |

One-to-many and many-to-one describe the same relationship from opposite directions.

### Representing Many-to-Many Relationships

Relational databases typically use a join table:

```text
users
  id
  first_name

organizations
  id
  name

positions
  user_id
  org_id
  title
```

Each position connects one user to one organization.

This allows queries in both directions:

- Which organizations has this user worked for?
- Which users have worked for this organization?

Indexes on `positions.user_id` and `positions.org_id` help support these lookups.

Document databases can also store references. For example, a position inside a profile could contain an `org_id` instead of an embedded organization.

> A document database does not require every relationship to be embedded.

## 4. Normalization, Denormalization, and Joins

### Normalization: Store Shared Information Once

Suppose many users live in the same region.

Instead of storing `"Washington, DC"` on every profile, store the name in a `regions` table and reference it through `region_id`.

```text
users
  id: 251
  region_id: 10

regions
  id: 10
  region_name: "Washington, DC"
```

Benefits include:

- Consistent names and spelling.
- Easier updates to shared information.
- Less duplicated information.
- Support for displaying localized names.

To display the region name, resolve the reference using a join:

```sql
SELECT users.id, regions.region_name
FROM users
JOIN regions ON users.region_id = regions.id
WHERE users.id = 251;
```

This finds user `251` and matches their `region_id` to the corresponding region.

### Denormalization: Copy Information to Simplify Reads

A denormalized profile might contain the region name directly:

```json
{
  "user_id": 251,
  "region_name": "Washington, DC"
}
```

The profile can display the name without another lookup. However, changing the shared name may require updating many profiles.

The same trade-off applies to organization names and logos:

- Store an organization once and reference its ID: Shared updates are easier.
- Copy its name and logo into profiles: Reads are more self-contained, but copies can become inconsistent.

### The Main Trade-Off

| Approach | Main benefit | Main cost |
|---|---|---|
| Normalization | Easier management of shared data and updates. | Reads may require joins or additional lookups. |
| Denormalization | Fewer lookups for the intended read pattern. | More duplication and update coordination. |

These are trade-offs, not universal performance guarantees.

> Normalization and document storage are separate choices. Documents can contain references, and relational tables can contain duplicated information.

### Duplicating Relationships

Suppose a user document lists organization IDs, and each organization document also lists user IDs.

The relationship is now stored twice. Updating one side without the other can make the data inconsistent.

An alternative is to store the relationship once and use indexes to query it in either direction.

## 5. Object-Relational Mapping

An object-relational mapper, or ORM, translates between application objects and relational tables.

Examples include ActiveRecord and Hibernate.

ORMs can help with:

- Reducing repetitive SQL and object-mapping code.
- Managing schema migrations.
- Abstracting database access.
- Caching query results in some frameworks.

However, an ORM does not remove the need to understand the database.

You still need to think about:

- Table structure and relationships.
- The queries being generated.
- Joins and indexes.
- Whether generated queries fit the application's access patterns.

> Use an ORM to reduce boilerplate, not as a replacement for understanding SQL.

## 6. Choosing Between Relational and Document Models

### Relational Models Fit Well When

- Data has many shared entities.
- Many-to-one and many-to-many relationships are important.
- You need to query relationships in several directions.
- Keeping shared information consistent is a priority.

### Document Models Fit Well When

- Related information forms a natural tree.
- The application often reads that information together.
- Embedded collections remain manageable.
- Records benefit from flexible structures.

Schema flexibility does not mean structure stops mattering. The application still needs to understand the fields it reads and writes.

### Think About Access Patterns

For a backend application, ask:

1. What does each endpoint need to read?
2. Which pieces of information are usually fetched together?
3. Which information is shared across many records?
4. What happens when that shared information changes?
5. Could an embedded collection grow very large?
6. Will relationships need to be queried in both directions?

These questions are more useful than treating either model as universally better.

## 7. Querying Relational and Document Data

Different query languages can express the same operation.

Suppose observation records contain a timestamp, animal family, and number of animals seen. We want the total number of sharks observed each month.

### PostgreSQL

```sql
SELECT
    date_trunc('month', observation_timestamp) AS observation_month,
    SUM(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks'
GROUP BY observation_month;
```

`date_trunc('month', ...)` groups timestamps into calendar months.

### MongoDB

```javascript
db.observations.aggregate([
    {
        $match: {
            family: "Sharks"
        }
    },
    {
        $group: {
            _id: {
                year: { $year: "$observationTimestamp" },
                month: { $month: "$observationTimestamp" }
            },
            totalAnimals: {
                $sum: "$numAnimals"
            }
        }
    }
]);
```

Both queries perform the same basic steps:

1. Filter for sharks.
2. Group observations by month.
3. Sum the number of animals.

The useful concept is the operation being performed, not memorizing both syntaxes.

---

# Analytics Schemas: The Essentials

Operational databases support application reads and writes. Data warehouses organize data for analysis.

ETL processes move and transform operational data into an analytical structure.

## 8. Star Schema

A star schema has a central fact table connected to dimension tables.

### Fact Table: What Happened?

A fact table stores events and their measurements.

For retail sales, a row might represent a particular product purchased in a sale.

```text
fact_sales
  product_id
  store_id
  customer_id
  date_id
  quantity
  sale_price
```

Be clear about what one row represents. For example, a purchased product and an entire shopping transaction are different units.

### Dimension Tables: The Context

Dimensions describe the event:

- `dim_product`: What was purchased?
- `dim_store`: Where was it purchased?
- `dim_customer`: Who purchased it?
- `dim_date`: When was it purchased?

Many fact rows can reference the same dimension row.

<img width="527" height="454" alt="Star schema with a central sales fact table and surrounding dimension tables" src="https://github.com/user-attachments/assets/df55c198-34fc-4b0a-83b1-cc6c52f78dae" />

## 9. Snowflake Schema and One Big Table

These are variations in how much information is separated or combined.

| Structure | Organization | Main trade-off |
|---|---|---|
| Star schema | Fact table references dimension tables. | Straightforward analytical structure. |
| Snowflake schema | Dimensions reference additional tables, such as separate brands and categories. | Less duplication, but more joins and complexity. |
| One big table (OBT) | Dimension information is copied into the fact table. | Fewer joins, but more duplicated data. |

For now, understand the structures and their trade-offs. Detailed warehouse design can wait until you work on an analytics pipeline.

---

# What to Remember

- Model data around how the application reads, updates, and connects it.
- Embedding keeps related information together; references connect separately stored information.
- Normalization reduces duplication; denormalization can simplify particular reads.
- Many-to-many relationships need an explicit representation, often a join table.
- Indexes help support the directions in which relationships are queried.
- ORMs reduce repetitive work, but SQL and database fundamentals still matter.
- In analytics, fact tables describe events and measurements; dimension tables provide context.
