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

**Advantages:**

- Stronger durability because the write exists on multiple replicas before the client receives success.

**Disadvantages:**

- Higher latency.
- If a required follower is unavailable, writes may be delayed or fail.

### Asynchronous Replication

With **asynchronous replication**, the leader does not wait for followers to confirm that they have completed the write before responding to the client.

**Advantages:**

- Lower write latency.
- The leader can continue accepting writes even if a follower is temporarily unavailable.

**Disadvantage:**

- Weaker durability. If a write is made to the leader and the leader fails before the write is replicated, that write may be lost.

### Quorum-Based Replication

A common approach is to require a **majority of replicas** to acknowledge a write synchronously while the remaining replicas replicate asynchronously.

This provides a balance between durability and performance.

**Key idea:** The more replicas that must acknowledge a write before success is returned, the stronger the durability guarantee, but generally at the cost of higher latency and lower availability when replicas are unavailable.

## Setting Up New Followers

Setting up a new follower typically occurs through the following process:

1. **Take a snapshot** of the leader's database at some point in time without locking the entire database.
2. **Copy the snapshot** to the new follower node.
3. The follower connects to the leader and requests all changes made since the snapshot was taken.
4. The snapshot must therefore be associated with an **exact position in the leader's replication log** so the follower knows where to resume.
5. Once the follower has processed all of the changes that occurred after the snapshot, we say that the follower has **caught up**.

### Databases Backed by Object Storage

**Object storage** is used to store large amounts of unstructured data at massive scale.

#### Advantages of Object Storage

- Useful for storing data that is not queried as frequently because object storage is inexpensive compared with typical cloud storage options.
- Object stores can provide **multi-zone, dual-region, or multi-region replication** with very high durability guarantees.

#### Tradeoffs

- Object stores generally have much higher read and write latencies than local disks.
- Objects are often **immutable**. Because an object is immutable, you cannot open an object, change a few bytes in the middle, and save it. Instead, you must upload a completely new version of the entire object to replace the old one.

## Handling Node Outages

The goal of replication is to keep the system as a whole running despite individual node failures and to keep the impact of a node outage as small as possible.

### Follower Failure: Catch-Up Recovery

Each follower keeps a log of the data changes it has received from the leader on its local disk.

If a follower crashes, or if the network connection between the follower and leader is disrupted, the follower can recover from its log.

The follower knows the last transaction or log position it successfully processed and can request the changes that occurred after that point from the leader.

Although this process is relatively simple, it can become a performance challenge.

If a follower is down for a long time, there may be a large number of writes that it needs to catch up on. This creates work for both:

- **The leader:** It must send the backlog of writes to the follower.
- **The follower:** It must process and apply the backlog of writes.

### Leader Failure: Failover

The goal of **failover** is to continue accepting writes when the leader fails.

The basic process is:

1. **Determine that the leader has failed.**
2. **Choose a new leader:** Typically, the follower with the most up-to-date data from the old leader is chosen.
3. **Reconfigure the system:** Clients must be directed to the new leader, and the other followers must begin consuming data changes from the new leader.

#### Problems with Failover

Things can go wrong during failover.

**1. Lost writes**

With asynchronous replication, writes accepted by the previous leader may not have reached any followers before the leader failed. Those writes can therefore be lost when a new leader is chosen.

**2. Split brain**

Two nodes can incorrectly believe that they are both the leader. This is known as **split brain**.

If both nodes accept writes, conflicting changes can occur and there may be no straightforward way to resolve them.

Protecting against this by limiting or shutting down the old leader after a new leader has been chosen is known as **fencing**.

**Most important consideration during failover:** Choose a follower that is as up-to-date as possible so that as few committed writes as possible are lost.

These problems—**node failures, unreliable networks, and trade-offs involving replica consistency, durability, availability, and latency**—are fundamental problems in distributed systems.

## Implementation of Replication Logs

    
  





