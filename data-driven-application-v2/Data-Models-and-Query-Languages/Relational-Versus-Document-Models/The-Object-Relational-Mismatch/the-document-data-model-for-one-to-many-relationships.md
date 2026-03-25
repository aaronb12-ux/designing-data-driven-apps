## The Document Data Model for One-to-Many Relationships

Not all data lends itself to a relational representation. Here is an example of the limitation of the relational model.

We are working with the below resume/LinkedIn profile and represent it as a relational schema:

<img width="481" height="441" alt="Screenshot 2026-03-23 at 6 53 58 PM" src="https://github.com/user-attachments/assets/95ac9be8-4fd2-4cdd-9faa-fa42e33a4205" />

The profile as a whole can be identified by a unique identifier, user_id. And fields like first_name and last_name appear exactly once per user, so they can be modeled as columns on the users table.

Most people have had more than one job in their career, and people may have varying numbers of periods of education and any number of pieces of contact information. One way of representing such one-to-many relationships is to put positions, education, and contact information in separate tables, each with a foreign-key reference to the users table.

Another way of representing the same information, which is perhaps more natural and maps more closely to an object structure in application code, is as a JSON document shown below:

<img width="462" height="383" alt="Screenshot 2026-03-24 at 8 26 11 PM" src="https://github.com/user-attachments/assets/54918ad5-fd20-4706-98fa-e9a77d97c32b" />

The JSON representation has better locality than the multi-table schema shown above. If you want to fetch a profile in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multiway join between the users table and its subordinate tables. In the JSON representation, all the relevant information is in one place, making the query both faster and simpler.

The one-to-many relationships from the user profile to the user's positions, educational history, and contact information imply a tree structure in the data, and the JSON representation makes this tree structure explicit.

<img width="335" height="279" alt="Screenshot 2026-03-24 at 8 29 40 PM" src="https://github.com/user-attachments/assets/76ef157f-3f05-469d-bffe-b8916bb75ae4" />

Note that a one-to-many relationship is sometimes called a one-to-few, since a resume typically has a small number of positions. If you have a genuinely large number of related items -- say, comments on a celebrity's social media post, of which there could be many thousands -- embedding them all in the same document may be too unwieldy, so the relational approach is preferable.
