# When To Use Which Model

The document data models provide schema flexibility, better performance due to locality, and — for some applications — a structure that is closer to the object model used by the application.

The relational model counters by providing better support for joins and many-to-one and many-to-many relationships.

## Schema Flexibility in the Document Model

Most document databases, and the JSON support in relational databases, do not enforce any schema on the data in documents.

When all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure.

## Data Locality For Reads and Writes

A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB's BSON).

If your application often needs to access the entire document, this storage locality has a performance advantage. If data is split across multiple tables, then multiple index lookups are required to retrieve it all, which may require more disk seeks and take more time.

## Query Languages for Documents

Relational databases often use SQL for operations, and document databases are more varied.

An example is to imagine you are a marine biologist, and you add an observation record to your database every time you see animals in the ocean. Now you want to generate a report saying how many sharks you sighted per month. In PostgreSQL, you might express this query like:

```sql
SELECT date_trunc('month', observation_timestamp) AS observation_month,
       sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks'
GROUP BY observation_month;
```

`date_trunc('month', observation_timestamp)` -> determines the calendar month containing the timestamp.

This query first filters the observations to show only species in the Sharks family, then groups the observations by the calendar month in which they occurred, and finally adds up the number of animals seen in all observations in that month.

This same query in MongoDB's aggregation pipeline is:

```javascript
db.observations.aggregate([
    { $match: { family: "Sharks" } },
    { $group: {
        _id: {
            year:  { $year:  "$observationTimestamp" },
            month: { $month: "$observationTimestamp" }
        },
        totalAnimals: { $sum: "$numAnimals" }
    } }
]);
```



