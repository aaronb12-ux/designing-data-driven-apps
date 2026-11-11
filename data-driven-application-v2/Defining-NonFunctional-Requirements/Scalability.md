# Scalability

Scalability is a system's ability to handle increased load, often by adding resources, while maintaining acceptable performance.

Calling a system “scalable” is incomplete without describing what grows:

- Requests per second.
- Stored data.
- Concurrent users or connections.
- Request size.
- Work performed per request.

The useful question is:

> If this particular workload grows, what becomes the bottleneck, and how can we increase capacity?

## 1. Understand the Workload First

Describe the load before choosing a scaling strategy.

| Characteristic | Why it matters |
|---|---|
| Request rate and peak traffic | Determines processing capacity and spare capacity needed. |
| Read-to-write ratio | Reads and writes may need different scaling strategies. |
| Data volume and growth | Affects storage, indexes, and working memory. |
| Request and response size | Affects network and processing costs. |
| Cache hit rate | Determines how much traffic reaches downstream storage. |
| Distribution of activity | A few popular users or records can create hotspots. |

Equal data throughput does not mean equal workloads.

For example, many small requests and a few large transfers may move the same bytes per second but place very different demands on the system.

### Two Questions to Test

1. With resources fixed, what happens to response times, errors, and throughput as load increases?
2. How many additional resources are needed to maintain acceptable performance?

The goal is to meet performance and reliability targets at a reasonable cost.

## 2. Vertical vs. Horizontal Scaling

| Strategy | Meaning | Main trade-off |
|---|---|---|
| Vertical scaling, or scaling up | Give a machine more CPU, RAM, storage, or bandwidth. | Often simpler, but limited by available machine sizes. |
| Horizontal scaling, or scaling out | Add machines and distribute work across them. | Adds capacity, but requires coordination and workload distribution. |

Both strategies can be combined. [65]

### Vertical Scaling

A larger machine can help when the workload needs more resources on one node—for example, more memory for a large working set.

However:

- A larger machine does not necessarily process proportionally more work.
- Software bottlenecks can remain.
- Increasing machine size alone does not provide redundancy.

### Horizontal Scaling

Multiple machines handle portions of the workload.

For example:

```text
                 ┌─ Application instance
Client → Balancer├─ Application instance
                 └─ Application instance
```

Application instances are easier to distribute across when requests do not depend on state stored only in one particular instance. [64]

Adding application instances will not fix an overloaded database. Scale the component that is actually limiting capacity. [62]

## 3. Fix the Bottleneck, Not Everything

Possible bottlenecks include:

- CPU-intensive processing.
- Insufficient memory.
- Slow or excessive database queries.
- Disk or network limits.
- Lock contention.
- A slow downstream dependency.

More hardware is not always the first answer. Inefficient queries, poor data distribution, or unsuitable configurations can waste the capacity already available. [61]

A practical process is:

```text
Measure load and performance
            ↓
Identify the limiting resource or operation
            ↓
Improve it or add appropriate capacity
            ↓
Measure again
```

After fixing one bottleneck, another may become the new limit.

## 4. Architecture Terms to Recognize

These describe how processing and storage resources are shared. They are not exact synonyms for vertical and horizontal scaling.

| Architecture | Basic idea | Main concern |
|---|---|---|
| Shared-memory | Processing units access common memory, usually within one machine. | Contention and single-machine limits. |
| Shared-disk | Machines have separate CPUs and memory but access shared storage. | Storage contention and coordination. |
| Shared-nothing | Nodes have their own processing, memory, and storage resources and communicate over a network. | Partitioning data and coordinating work. |

For now, understand the resource-sharing distinction rather than memorizing implementation details.

> Horizontal scaling does not automatically imply shared-nothing. For example, multiple application servers may still depend on one shared database.

## 5. Independent Work Is Easier to Scale

Work is easier to distribute when components can operate with limited coordination.

Examples include:

- Workers processing separate background jobs.
- Application instances handling independent requests.
- Shards storing different subsets of data.

Separating components can also let you scale them differently. A busy worker tier may need more capacity while the API tier does not. [64]

However, splitting a system adds communication and operational complexity.

> Microservices are not a prerequisite for scalability. Split components when independent scaling or other requirements justify the cost.

## 6. Linear Scalability

A system has approximately linear scalability if doubling resources lets it handle roughly twice the load while maintaining comparable performance.

This is a useful goal, not a guarantee.

Shared bottlenecks, coordination overhead, and uneven work distribution can produce diminishing returns. Adding nodes may increase capacity by less than the proportional resource increase. [62]

Always evaluate scaling using the relevant workload rather than node count alone.

## 7. Keep the Design Proportional to the Need

Start with the simplest architecture that meets current requirements and reasonably expected growth.

- A single-machine database may be sufficient.
- A larger instance may solve the immediate constraint.
- Distributed storage introduces additional failure modes and operational work.
- Autoscaling can adjust capacity to demand, but it does not remove application bottlenecks. [65][70]

Do not wait for an outage to think about capacity, but do not build for hypothetical scale without evidence.

Large growth may require architectural changes; there is no rule that every tenfold increase requires a redesign.

## What to Remember

- Scalability depends on what grows and which performance targets must hold.
- Measure workload characteristics before choosing a solution.
- Scale up for more resources per machine; scale out to distribute work.
- Adding machines helps only if the bottleneck can use them.
- Independent work is easier to distribute than tightly coordinated work.
- More resources do not guarantee proportional capacity.
- Prefer the simplest design that meets the workload and leaves reasonable room for growth.
