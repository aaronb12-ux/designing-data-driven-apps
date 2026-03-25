## Normalization, Denormalization, and Joins

In the below JSON document:

<img width="462" height="383" alt="Screenshot 2026-03-24 at 8 26 11 PM" src="https://github.com/user-attachments/assets/ca3db42b-c239-45f0-a03c-cdfa6a670064" />

You notice how region_id is given as an ID, not as the plain-text string like: Washington, DC, United States. Why is this?

There are advantages to having standardized lists of geographic regions and letting users choose from a drop-down list or autocomplete. These include the following:

- Consistent style and spelling across profiles
- Avoiding ambiguity if several places have the same name
- Ease of updating - the name is stored in only one place, so it is easy to update across the board if it ever needs to be changed
- Localization support - when the site is translated into other languages, the standardized lists can be localized, so the region can be displayed in the viewer's language
- Better search functionality

Whether you store an ID or a text string is a question of **normalization**. When you use an ID, your data is more normalized: the information that is meaningful to humans (such as the text "Washington, DC") is stored in only one place, and everything that refers to it uses an ID.

When you store the text directly, you are duplicating the human-meaningful information in every record that uses it; this representation is **denormalized**.

The advantage of using an ID is that because it has no meaning to humans, it never needs to change: the ID can remain the same even if the information it identifies changes. Anything that is meaningful to humans may need to change sometime in the future - and if that information is duplicated, all the redundant copies will need to be updated. This requires more code, write operations, and disk space.

The downside of normalized representation is that every time you want to display a record containing an ID, you have to do an additional lookup to resolve the ID into something human-readable. In a relational data model, this is done using a join. For example:
```sql
SELECT users.*, regions.region_name
FROM users
JOIN regions ON users.region_id = regions.id
WHERE users.id = 251;
```

Query from users table of the user with id = 251. Now find the record in the regions table with id == users.region_id.

Document databases can store both normalized and denormalized data, but they are often associated with denormalization - partly because the JSON data model makes it easy to store additional denormalized fields, and partly because the weak support for joins in many document databases makes normalization inconvenient.
