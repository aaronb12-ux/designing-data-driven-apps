## The Document Data Model for One-to-Many Relationships

Not all data lends itself to a relational representation. Here is an example of the limitation of the relational model.

We are working with the below resume/LinkedIn profile and represent it as a relational schema:

<img width="481" height="441" alt="Screenshot 2026-03-23 at 6 53 58 PM" src="https://github.com/user-attachments/assets/95ac9be8-4fd2-4cdd-9faa-fa42e33a4205" />

The profile as a whole can be identified by a unique identifier, user_id. And fields like first_name and last_name appear exactly once per user, so they can be modeled as columns on the users table.
