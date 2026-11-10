# Single-Leader Replication

There is a **leader replica** that clients send write requests to.

The other replicas are called **followers**. The leader sends all data changes to its followers as part of a **replication log** or **change stream**, and the followers update their databases accordingly.

A client can query either the leader or a follower:

- **Writes:** Can only be performed through the leader.
- **Reads:** Can be performed from any replica.

Single-leader replication is built into many databases, including:

- PostgreSQL
- MySQL
- MongoDB
- DynamoDB

<img width="346" height="158" alt="Single-leader replication diagram" src="https://github.com/user-attachments/assets/9d40fa3a-d154-49aa-8c79-a806424500dd" />

## Synchronous vs. Asynchronous Replication

### Synchronous Replication

With **synchronous replication**, the leader waits for a follower to confirm that the write has been successfully replicated before considering the operation complete.

In a strict synchronous setup, the leader must wait for all designated synchronous replicas to log or apply the change and send back an acknowledgment before it can tell the client, `"Success."`

**Advantage:**

- Stronger durability because the write exists on multiple replicas before the client receives success.

**Disadvantage:**

- Higher latency.
- If a required follower is unavailable, writes may be delayed or fail.

### Asynchronous Replication

With **asynchronous replication**, the leader does not wait for followers to confirm that they have completed the write before responding to the client.

**Advantage:**

- Lower write latency.
- The leader can continue accepting writes even if a follower is temporarily unavailable.

**Disadvantage:**

- Weaker durability. If a write is made to the leader and the leader fails before the write is replicated, that write may be lost.

### Quorum-Based Replication

A common approach is to use a **quorum**, where a majority of replicas must acknowledge a write synchronously while the remaining minority can replicate asynchronously.

This provides a balance between durability and performance.

**Key idea:** The more replicas that must acknowledge a write before success is returned, the stronger the durability guarantee, but generally at the cost of higher latency and lower availability when replicas are unavailable.



