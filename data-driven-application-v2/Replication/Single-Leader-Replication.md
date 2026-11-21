# Single-Leader Replication

Single-leader replication keeps copies of data on multiple database nodes:

- Leader: Accepts writes.
- Followers: Receive and apply changes from the leader.

The leader communicates changes through a replication log or change stream.

```text
Client writes → Leader
                  ├─ Replication → Follower
                  └─ Replication → Follower
```

Clients may read from the leader or from followers that support reads. Some standbys are reserved for recovery and do not serve queries. PostgreSQL supports both arrangements. [108]

<img width="346" height="158" alt="Single-leader replication with writes to the leader and changes propagated to followers" src="https://github.com/user-attachments/assets/9d40fa3a-d154-49aa-8c79-a806424500dd" />

> Replication can improve availability and distribute reads. Adding followers does not automatically increase write capacity, because writes still pass through the leader.

## 1. Synchronous vs. Asynchronous Replication

The key question is:

> What must happen before the leader tells the client that a write succeeded?

| Mode | When the leader responds | Main trade-off |
|---|---|---|
| Synchronous | After the required followers acknowledge the change. | Better protection against leader loss, but additional latency and dependence on those followers. |
| Asynchronous | Without waiting for follower acknowledgment. | Lower replication-related write latency, but acknowledged writes may be lost during failover. |

An acknowledgment can mean different things: The follower has received the change, persisted it, or applied it so queries can see it. The exact guarantee depends on configuration. [104][111]

### Synchronous Replication

The leader waits for the configured acknowledgment requirement.

This does not necessarily mean waiting for every follower. A system might require one designated follower or a specified number of followers. [109]

If the requirement cannot be met, writes may block or fail.

### Asynchronous Replication

The leader responds before followers necessarily have the change.

```text
Leader accepts write
        ↓
Client receives success
        ↓
Leader crashes before replication
        ↓
Promoted follower does not contain the write
```

The write may have been durable on the original leader but still be absent from the new leader.

### Majority Acknowledgments

Some systems require acknowledgments from a majority of voting replicas.

This can provide strong protection when combined with the system's election and commit rules. Majority acknowledgment alone is not a complete failover protocol.

> Evaluate acknowledgment requirements together with persistence guarantees and rules for choosing a new leader—not just the number of copies.

## 2. Replication Lag and Reads

A follower may temporarily trail the leader. This delay is replication lag.

For example:

1. A user updates their profile through the leader.
2. Their next request reads from a lagging follower.
3. They see the previous profile value.

Even synchronous replication does not imply every follower has applied the latest write. Acknowledgment may require persistence rather than query visibility, and some followers may replicate asynchronously. [104][111]

> Decide which reads can tolerate stale data before routing them to followers.

## 3. Setting Up a New Follower

A new follower needs both an initial copy and the changes made while that copy is being transferred.

Typical process:

1. Take a consistent snapshot using the database's supported backup mechanism.
2. Associate it with a position in the replication log.
3. Restore the snapshot on the new follower.
4. Replay changes after that position.
5. Continue receiving new changes.

```text
Snapshot at position X
        +
Log entries after X
        =
Up-to-date follower
```

The required log history must remain available until the follower has consumed it. Otherwise, a new snapshot may be necessary. [104][105]

## 4. Follower Failure: Catch-Up Recovery

After a crash or network interruption, a follower can resume from a safely recorded replication position.

Recovery requires:

- The follower's existing data to remain usable.
- The needed log history to remain available.
- Enough capacity to replay the backlog while new writes continue arriving.

A long outage can create substantial catch-up work. Retaining logs for followers also consumes storage on the source system. [104][113]

If the required history is gone, the follower may need to be rebuilt.

## 5. Leader Failure: Failover

Failover promotes a follower so the system can resume accepting writes.

Typical steps:

1. Detect that the leader appears unavailable.
2. Select an eligible follower under the system's failover rules.
3. Prevent the old leader from continuing to accept conflicting writes.
4. Promote the replacement.
5. Redirect clients and reconnect other followers.

### Failure Detection Is Uncertain

A timeout does not prove a machine has crashed. The leader may still be running but unreachable because of a network problem.

This uncertainty is one reason failover is difficult.

### Lost Writes

With asynchronous replication, the promoted follower may lack writes already acknowledged by the old leader.

Choosing an up-to-date follower reduces this risk, but cannot recover changes that never reached any surviving replica.

### Split Brain and Fencing

Split brain occurs when multiple nodes act as leader for the same data.

If both accept writes, their histories can diverge.

Fencing prevents an old or isolated leader from continuing to act as an authorized writer.

> Safe failover requires more than picking the freshest follower. It must preserve the system's commit rules and prevent conflicting leaders.

## 6. How Replication Logs Represent Changes

### Statement-Based Replication

Send the operations that produced the changes:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 42;
```

Followers execute the statements.

Potential problems include:

- Nondeterministic expressions producing different values.
- Different execution ordering changing results.
- Triggers or other side effects behaving differently.

The system needs rules or restrictions that make replay produce the intended result.

### Physical Replication / WAL Shipping

Send low-level storage changes through the write-ahead log, or WAL.

Followers replay those changes to maintain a compatible physical copy.

Advantages:

- Closely reproduces the leader's storage changes.
- Does not require re-executing the original application SQL.

Trade-off:

- Tight coupling to the storage engine and supported database versions.

PostgreSQL physical replication uses WAL and generally requires matching major versions, which limits its use for major-version rolling upgrades. [104]

### Logical / Row-Based Replication

Send logical changes to records rather than physical storage details.

Conceptually:

```text
Table: users
Operation: UPDATE
Key: id = 42
Changed value: name = "Alex"
```

Logical replication is less tied to the physical storage format. It can support selected-table replication and, where supported, migration between database versions. [103][106]

However, schema compatibility and supported operations still matter.

### Comparison

| Method | What is replicated | Main concern |
|---|---|---|
| Statement-based | Operations to execute. | Replaying them must produce equivalent results. |
| Physical / WAL | Low-level storage changes. | Storage-format and version compatibility. |
| Logical / row-based | Record-level changes. | Compatible schemas and interpretation of changes. |

## 7. Object Storage: A Related but Separate Topic

Some database architectures use object storage for durable database files, logs, or snapshots.

For these notes, keep it separate from the replication model:

- Replication describes how database copies receive changes.
- Object storage describes where some underlying data is stored.

You do not need object-storage internals to understand single-leader replication.

## 8. Problems With Replication Lag

Read scaling means creating multiple followers and distributing read traffic across them.

Asynchronous replication allows the leader to continue accepting writes without waiting for every follower. Requiring every follower to acknowledge writes would make write availability depend on all of them.

The trade-off is that followers may temporarily return outdated information.

If writes stop and replication continues successfully, followers eventually catch up with the leader. This behavior is called eventual consistency. The delay between a change on the leader and its appearance on a follower is replication lag.

### Reading Your Own Writes

A user may submit an update, reload the page, and not see it because the read was served by a lagging follower.

<img width="470" height="220" alt="A user reading from a lagging follower after submitting an update" src="https://github.com/user-attachments/assets/5aecb1a0-6cd1-4a0a-8a1f-a7e093c62e55" />

Read-after-write consistency guarantees that a user's subsequent reads reflect their own successful writes.

For example, after submitting a discussion comment, the user should see that comment when they reload the thread.

Possible approaches:

- Read recently modified information from the leader or a follower known to have applied the write.
- Use a simple routing rule, such as reading a user's own profile from the leader and other users' profiles from followers.
- Track the position of the user's latest write and serve subsequent reads only from a replica that has reached at least that position.

A replication position or logical timestamp can identify the required progress; it need not be a wall-clock time.

#### Cross-Device Read-After-Write Consistency

A user who updates information on one device should also see the update when reading from another.

This introduces additional coordination:

- The second device may not know about the first device's latest write.
- Tracking the required replication position only on one device is therefore insufficient.
- Shared, server-side metadata can track the user's latest required position.
- If devices connect through different regions, routing must account for whether the selected replica has received the update.

Routing all of a user's devices to the same region is one possible approach, not a universal requirement.

### Monotonic Reads

A user may first read from an up-to-date replica and then from a lagging replica, making information appear to disappear.

Monotonic reads guarantee that a user's successive reads do not move backward to an older state.

<img width="352" height="216" alt="A user seeing newer data followed by older data when reading from different replicas" src="https://github.com/user-attachments/assets/ede7ea5b-9e0f-4d90-910d-cfbaea9a0adf" />

One approach is to route each user's reads to the same replica, while distributing different users across replicas.

If that replica becomes unavailable, switching to another replica must preserve the user's required progress.

> Read-after-write consistency protects visibility of your own writes. Monotonic reads protect against losing visibility of data you have already seen.

### Consistent Prefix Reads

Consistent prefix reads preserve the ordering of a sequence of writes.

If one write depends on an earlier write, readers should not see the later write without the earlier one.

For example, a discussion reply should not appear before the message it answers.

<img width="341" height="257" alt="Related writes appearing out of order because of replication delays" src="https://github.com/user-attachments/assets/faed51ae-987f-4f88-b9c2-879145650c30" />

This becomes more difficult when related writes are spread across shards that replicate independently. Each shard may preserve its own order without automatically preserving the order across shards.

### Comparing the Guarantees

| Guarantee | What it prevents |
|---|---|
| Read-after-write consistency | Your successful update appears missing when you read afterward. |
| Monotonic reads | Data you have already seen disappears because a later read returns older state. |
| Consistent prefix reads | Related writes become visible in the wrong order. |

## What to Remember

- One leader accepts writes; followers apply replicated changes.
- Follower reads can be stale.
- Synchronous replication waits for a configured acknowledgment requirement—not necessarily every replica.
- Asynchronous replication can lose acknowledged writes during failover.
- New and recovering followers need a snapshot or existing state plus sufficient log history.
- Failover must prevent conflicting leaders, not merely promote another node.
- Physical replication copies storage-level changes; logical replication communicates record-level changes.
- Always ask: “What does success guarantee, and which failures does that guarantee cover?”
