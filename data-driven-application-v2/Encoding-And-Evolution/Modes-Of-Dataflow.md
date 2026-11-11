# Modes of Dataflow

Data commonly moves between processes through:

| Mode | Basic idea |
|---|---|
| Databases | One process writes data that another process reads later. |
| Service calls | A client requests work from a server through an API. |
| Workflow engines | A coordinator manages a sequence of tasks and recovery. |
| Asynchronous messaging | Producers send messages that consumers process independently. |

Across all four modes, consider:

- How is the data encoded?
- Can different versions understand it?
- What happens when a process or network connection fails?
- Is retrying safe?

## 1. Dataflow Through Databases

Storing data is like sending a message to a future reader.

```text
Writer → Database → Reader
```

The reader might be another application instance—or a newer version of your own application months later.

### Compatibility Matters in Both Directions

During a rolling deployment, old and new application versions may access the same database.

- New code must understand existing data.
- Old code may encounter data written by new code.

Database changes therefore need to account for both historical data and temporarily mixed application versions.

> Updating the database schema does not automatically make every application version compatible with it.

### Schema Changes and Migrations

Some database changes can avoid rewriting every existing row immediately. Whether this is possible depends on the database, version, and specific change.

More substantial changes usually need an explicit migration, such as:

- Moving a field into a separate table.
- Replacing one value with a collection.
- Changing how existing values are interpreted.

The application may need to support both representations during the transition.

### Archives and Backups

Archived data may outlive the application version that created it.

Keep enough schema information to interpret it later. Formats such as Avro Object Container Files can store records together with their writer schema.

The important idea is not to memorize another format:

> Long-lived data needs a long-lived explanation of its structure.

## 2. Dataflow Through Services

A service exposes an API that clients call over a network.

```text
Client → Request → Server
Client ← Response ← Server
```

A client could be a browser, mobile app, or another backend service.

Unlike direct database access, an API exposes specific operations and applies business rules around them.

### HTTP APIs and RPC

| Style | Main idea |
|---|---|
| Resource-oriented HTTP API | Exchange representations of resources using HTTP operations. |
| RPC | Invoke an operation exposed by a remote service. |

REST and RPC are not simply “JSON versus binary.” An RPC API can use JSON, and an HTTP API can use other encodings.

### Remote Calls Are Not Local Calls

A network request introduces concerns that a local function call usually does not:

- Network or server failure.
- Variable response time.
- Serialization and transmission costs.
- Timeouts with uncertain outcomes.

The last point is particularly important:

```text
Client sends payment request
        ↓
Server processes payment
        ↓
Response is lost
        ↓
Client times out
```

The client does not know whether the payment happened.

> A timeout means the caller did not receive a result in time—not necessarily that the operation failed.

Retries must account for this uncertainty. Idempotent operations can be retried without repeating the intended effect.

## 3. API Contracts and Evolution

An API contract describes available operations, inputs, and outputs.

Interface description tools include:

- OpenAPI for HTTP APIs.
- Protocol Buffers service definitions for gRPC.
- AsyncAPI for asynchronous APIs.

Frameworks can help implement these contracts. For example, FastAPI can define endpoints and generate OpenAPI documentation from application code.

You do not need to memorize a full API specification to understand the purpose:

> A contract lets clients and servers agree on how to communicate without sharing the same implementation language.

### Evolving an API

Clients and servers may upgrade at different times.

Adding an optional request field or a response field is often compatible, but only if validation and application behavior permit it.

External clients may remain on old versions for a long time. Plan compatibility or explicit versioning rather than assuming everyone upgrades together.

## 4. Finding and Routing to Services

These concepts solve different parts of service communication:

| Concept | Question it answers |
|---|---|
| Service discovery | Where are the available service instances? |
| Load balancing | Which instance should receive this request? |
| Service mesh | How can common communication policies be managed across services? |

### Service Discovery and Load Balancing

Instead of hardcoding one machine's address, clients can use DNS, a discovery system, or a stable load-balancer endpoint.

```text
Client → Load balancer → Available service instance
```

Multiple instances can increase capacity and availability, provided traffic is routed to healthy instances.

### Service Meshes

A service mesh adds infrastructure for service-to-service communication, often using proxies.

It can help provide:

- Encrypted connections.
- Traffic routing and load balancing.
- Communication metrics.
- Configurable timeout and retry policies.

It also adds complexity. Recognize the concept; you do not need a service mesh merely because an application has multiple services.

## 5. Durable Execution and Workflows

A workflow coordinates multiple steps toward a business outcome.

```text
Check fraud → Charge payment → Confirm order
```

A workflow engine tracks progress and schedules work. Workers execute the individual tasks.

![Workflow diagram](https://github.com/user-attachments/assets/8099ef7d-be71-44b6-be6d-52cdacda1e7f)

### Durable Execution

Durable execution persists progress so execution can recover after crashes.

For example:

1. A fraud check completes and its result is recorded.
2. The payment activity encounters a failure.
3. The system recovers.
4. It reuses the recorded fraud-check result.
5. It retries the payment activity according to its retry policy.

In Temporal, workflow history supports replay, while activities perform operations such as API calls and database writes. [92][96]

### Deterministic Workflow Logic

Replay must produce decisions consistent with recorded history.

In Temporal:

- Keep workflow logic deterministic.
- Put external I/O in activities.
- Use framework-provided APIs for time and randomness when needed in workflow code. [92]

### Recovery Does Not Mean Exactly-Once Side Effects

An activity might complete an external action, then crash before its completion is recorded. The engine may retry it.

Payment and other side-effecting activities therefore need retry-safe behavior, such as an idempotency key. [95]

> Durable execution preserves workflow progress; it does not automatically prevent duplicate external actions.

## 6. Asynchronous Messaging

With messaging, producers send work or events through a broker rather than calling consumers directly.

```text
Producer → Broker → Consumer
```

The producer does not wait for the consumer to finish processing. It may still wait for the broker to confirm acceptance.

### Why Use a Broker?

- Buffering: Consumers can process work after a traffic burst.
- Availability separation: Producers and consumers need not be running simultaneously, subject to retention and durability settings.
- Redelivery: Failed processing can be retried.
- Fan-out: Multiple subscribers can receive the same event.
- Decoupling: Producers do not need to know each consumer's address.

### Queues vs. Publish/Subscribe

| Pattern | Typical use |
|---|---|
| Work queue | Distribute tasks among workers; one worker handles a given delivery. |
| Publish/subscribe | Deliver an event to multiple independent subscribers. |

Broker terminology varies. In log-based systems, a topic can have multiple consumer groups, with each group independently processing the stream.

Neither pattern automatically guarantees exactly-once processing.

### Acknowledgments and Retention

A consumer acknowledgment tells the broker that processing has completed, according to the application's acknowledgment policy.

If a consumer crashes before acknowledgment, the message may be delivered again. Consumers should therefore be prepared for duplicates.

Retention also varies:

- Some queues remove messages after acknowledgment.
- Some logs retain messages for a configured period regardless of consumption.

Persistence and delivery guarantees depend on configuration, not just the presence of a broker.

## What to Remember

- Databases connect writers and readers across time.
- Service calls provide direct request-response interaction but introduce uncertain failure outcomes.
- Workflow engines coordinate multi-step progress and recovery.
- Brokers separate message production from processing.
- Schema compatibility matters wherever different versions exchange data.
- Retries and duplicate processing are normal design concerns.
- Always ask: “If this action succeeds but its acknowledgment is lost, what happens next?”
