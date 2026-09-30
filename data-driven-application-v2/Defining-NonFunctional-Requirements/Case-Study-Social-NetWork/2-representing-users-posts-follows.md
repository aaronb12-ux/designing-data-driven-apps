# Home Timeline — Data Architecture

## Storage

<img width="374" height="178" alt="Screenshot 2026-03-06 at 2 16 52 AM" src="https://github.com/user-attachments/assets/21e91a54-e851-4bd7-81d8-1dc2fe2da6c5" />

## Main Read Operation

The primary read operation is the home page, which displays recent posts from people the user is following.

### Query

```sql
SELECT posts.*, users.* FROM posts
  JOIN follows ON posts.sender_id = follows.followee_id
  JOIN users   ON posts.sender_id = users.id
  WHERE follows.follower_id = current_user
  ORDER BY posts.timestamp DESC
  LIMIT 1000
```

### How it works

1. Use the `follows` table to find everyone `current_user` is following.
2. Look up recent posts made by those users.
3. Sort by timestamp and return the most recent 1000 posts.

## Scaling Problem

If we want up-to-date posts, the users client could poll this query every 5 seconds while the user is online.

At 10 million active users, that means roughly 2 million queries per second.

The query itself is also expensive. If a user follows 200 people, the database must fetch recent posts from all 200 accounts and merge the results. At 2 million polls per second, that comes out to 400 million lookups per second.
