# Social Network System Design

## 1. Requirements and Assumptions

Design a Twitter-style platform where users can:

- Publish posts.
- Follow other users.
- View recent posts from followed accounts in a home timeline.

For this example, the timeline is chronological rather than algorithmically ranked.

### Assumed Scale

These are design assumptions, not claims about Twitter's current traffic.

| Metric | Assumption |
|---|---|
| Posts per day | 500 million |
| Average posts per second | Approximately 5,800 |
| Peak posts per second | 150,000 |
| Average accounts followed per user | 200 |
| Average followers per user | 200 |

The central challenge is serving frequently refreshed timelines without repeatedly rebuilding them from scratch.

## 2. Basic Storage Model

```text
users
  id
  name

posts
  id
  sender_id
  timestamp
  content

follows
  follower_id
  followee_id
```

Each `follows` row means that one user follows another.

<img width="374" height="178" alt="Storage model for users, posts, and follow relationships" src="https://github.com/user-attachments/assets/21e91a54-e851-4bd7-81d8-1dc2fe2da6c5" />

Useful access patterns include:

- Finding accounts a user follows.
- Finding an author's followers.
- Finding an author's recent posts.

These patterns guide which indexes the storage system needs.

## 3. Approach One: Build the Timeline on Read

When a user opens their home timeline:

1. Find the accounts they follow.
2. Fetch recent posts from those accounts.
3. Merge the results by timestamp.
4. Return the newest posts.

An illustrative SQL query is:

```sql
SELECT posts.*, users.name
FROM posts
JOIN follows
  ON posts.sender_id = follows.followee_id
JOIN users
  ON posts.sender_id = users.id
WHERE follows.follower_id = :current_user_id
ORDER BY posts.timestamp DESC
LIMIT 1000;
```

`:current_user_id` represents a query parameter.

This approach is called fan-out on read: The timeline request gathers posts from multiple followed accounts. [33]

### Why It Becomes Expensive

Suppose 10 million concurrent users refresh every five seconds:

```text
10 million users ÷ 5 seconds
= 2 million timeline requests per second
```

With 200 followed accounts per user, a simplified model gives:

```text
2 million requests × 200 accounts
= 400 million author-post lookups per second
```

This is a rough workload estimate—not a guarantee that the database performs 400 million separate queries or disk reads. Indexes, caching, batching, and query execution affect the actual work.

The key issue is repeated computation: Many requests gather and merge overlapping sets of posts.

## 4. Approach Two: Precompute Timelines

Instead of rebuilding a timeline for every request, maintain a recent-post list for each user.

When someone publishes a post, background workers add its ID to their followers' timelines.

This is called fan-out on write: One post causes updates to many follower timelines. [33]

### Write Path

```text
Author publishes post
        ↓
Persist post
        ↓
Enqueue fan-out work
        ↓
Workers find followers
        ↓
Add post ID to follower timeline caches
```

### Read Path

```text
User requests timeline
        ↓
Read cached post IDs
        ↓
Fetch post content and author details
        ↓
Return timeline
```

A timeline cache can store post IDs rather than full copies of every post. The read service then retrieves the corresponding content. [30]

<img alt="Architecture for precomputing follower timelines" src="https://github.com/user-attachments/assets/02c81f73-df08-49f4-8684-733e3387457d" />

### Estimated Write Volume

If each post reaches an average of 200 followers:

```text
5,800 posts/second × 200 followers
≈ 1.16 million timeline insertions/second
```

This estimate assumes the average follower count per post is 200. The average follower count across users may differ if highly followed users post more frequently.

The approach trades additional write work for simpler reads.

> Do not compare 1.16 million cache insertions and 400 million lookups as if they were identical operations. The useful comparison is where the work happens and how much repeated computation is avoided.

## 5. Background Processing and Spikes

A queue lets workers process timeline updates asynchronously.

During a traffic spike:

- New posts can be stored before all follower timelines are updated.
- Workers process the queued fan-out work.
- Followers may see posts after a short delay.

This is eventual consistency: The timeline can temporarily lag behind the stored posts.

A queue absorbs temporary bursts; it does not create unlimited capacity. If incoming work consistently exceeds processing capacity, the backlog keeps growing.

Workers also need retry-safe processing so that retrying a fan-out task does not create duplicate timeline entries.

## 6. Precomputation and Live Delivery Are Separate

Two different decisions are involved:

1. How is the timeline built?
2. How does the client learn that it changed?

Precomputation reduces the server-side work needed to serve a timeline.

Polling or a live connection determines how the client receives updates.

For example:

```text
Post → Timeline update → Live notification to an online client
```

You can still poll a precomputed timeline. Likewise, pushing notifications does not by itself eliminate the work of building the timeline.

## 7. The High-Follower Account Problem

An average of 200 followers hides uneven distribution.

An account with millions of followers can trigger millions of timeline updates from a single post.

A hybrid design handles this by:

- Precomputing timeline entries for ordinary accounts.
- Fetching posts from very high-follower accounts when a follower reads their timeline.
- Merging both sets of posts before returning the result. [35]

There is no universal follower threshold; it depends on the workload and system capacity.

## 8. Connection to Materialized Views

A precomputed home timeline is a materialized view: A stored result derived from posts and follow relationships.

- Source data: Posts and follow relationships.
- Derived data: Each user's home timeline.
- Benefit: Less work during reads.
- Cost: Extra writes and responsibility for keeping the view up to date.

This connects to CQRS: The representation optimized for reading the timeline differs from the representation used to store posts.

It does not automatically mean the system uses event sourcing.

## What to Remember

| Approach | Work happens mainly when | Main trade-off |
|---|---|---|
| Fan-out on read | A user requests their timeline. | Simpler writes, more read-time gathering and merging. |
| Fan-out on write | An author publishes a post. | Faster timeline reads, more write amplification. |
| Hybrid | Both publication and reading. | Handles high-follower accounts, but adds complexity. |

The central lesson is to move repeated work away from the busiest request path when the workload justifies it.

Follow the data flow:

```text
Post storage → Background work → Timeline cache → Timeline response
```

Then ask what happens when a step is delayed, retried, or unavailable.
