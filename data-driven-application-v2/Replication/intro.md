# Replication

**Replication** means keeping a copy of the same data on multiple machines that are connected via a network.

## Why Replicate Data?

Replication can be used to:

- **Keep data geographically close to users**, reducing latency.
- **Increase availability and durability**, allowing the system to continue operating if some machines fail.
- **Scale read queries** by distributing reads across multiple machines.

Replication becomes complicated and interesting when handling **changes to replicated data**.

There are three main algorithms for replicating changes:

1. **Single-leader**
2. **Multi-leader**
3. **Leaderless**

## Replication and Backups

Replication and backups serve different purposes:

- **Replication:** Continuously copies writes from one node to other nodes so that multiple machines have up-to-date copies of the data.
- **Backups:** Store older snapshots or versions of data so that you can recover or **go back in time** after accidental deletion, corruption, or other problems.

**Key distinction:** Replication primarily helps with **availability and keeping multiple copies synchronized**, while backups primarily help with **data recovery and historical restoration**.
