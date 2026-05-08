## Many-to-One and Many-to-Many Relationships

A `region_id` field of our case study is an example of a **many-to-one relationship** (many people live in the same region, but we assume that each person lives in only one region at any one time).

If we introduce entities for organizations and schools and reference them by ID from the resume, then we also have **many-to-many relationships** (one person may have worked for several organizations, and an organization has several past or present employees).

In a relational model, such a relationship is usually represented as an **associative table**, or **join table**. Each position associates one user ID with one organization ID.

<img width="572" height="287" alt="Screenshot 2026-05-08 at 2 40 03 PM" src="https://github.com/user-attachments/assets/1f9dea7b-136e-4a00-b567-2a902ca20a67" />

Many-to-one and many-to-many relationships do not easily fit within one self-contained JSON document; they lend themselves more to a normalized representation.

---

### Querying Many-to-Many Relationships

Many-to-many relationships often need to be queried in "both directions" - for example, finding all the organizations that a particular person has worked for, and finding all the people who have worked at a particular organization. One way of enabling such queries is to store ID references on both sides, such that a resume includes the ID of each organization where the person has worked, and the organization document includes the IDs of the resumes that mention that organization.

This representation is **denormalized**, since the relationship is stored in two places, which could become inconsistent with each other.

<img width="352" height="274" alt="Screenshot 2026-05-08 at 2 43 34 PM" src="https://github.com/user-attachments/assets/f7290c5e-ab0d-4f70-b3d8-1f35674b88e9" />

A **normalized representation** stores the relationship in only one place and relies on secondary indexes to allow the relationships to be efficiently queried in both directions.

In the relational schema shown in Figure 3-3, we would tell the database to create indexes on both the `user_id` and `org_id` columns of the positions table.

In the document model in Figure 3-2, the database needs to index the `org_id` field of objects inside the positions array. Many document databases and relational databases with JSON support are able to create such indexes on values inside a document.
