# Materializing and Updating Timelines

## The Approach

Instead of polling, we can do two things:

1. Actively push new posts to users that are currently online.
2. Precompute the results of the query so that a request for the home timeline can be served from a cache.

For each user, we store a data structure containing their home timeline — the recent posts made by people they follow. Every time a user makes a post, we look up their followers and insert that post into each follower's home timeline, like delivering a message to a mailbox. When a user logs in, we serve them this precomputed timeline directly.

The user's client simply subscribes to the stream of posts being added to their home timeline.

The downside is that we now have to do more work whenever a user makes a post, because the home timeline is derived data that needs to be kept up to date.

![Timeline architecture diagram](https://github.com/user-attachments/assets/02c81f73-df08-49f4-8684-733e3387457d)

## Write Volume

| Metric                              | Value          |
|-------------------------------------|----------------|
| Average posts per second            | 5,800          |
| Average followers reached per post  | 200            |
| Home timeline writes per second     | ~1,160,000     |

At roughly 1 million writes per second, this is far better than the previous approach of 400 million lookups per second.

## Handling Spikes

If the post rate spikes due to a special event, home timeline writes do not need to happen immediately. We can enqueue them and accept that it will temporarily take a bit longer for posts to appear in followers' timelines.

## Materialized Views

This process of precomputing and keeping query results up to date is called **materialization**. The timeline cache is an example of a **materialized view**. A materialized view speeds up reads, but in return requires more writes.
