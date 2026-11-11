# Event Sourcing and CQRS

A single data representation may not suit every operation. For example, recording a conference registration and displaying attendance statistics have different needs.

Two related patterns address this:

- Event sourcing: Store state changes as events that serve as the source of truth.
- CQRS: Separate the models used to handle writes and reads.

They are often combined, but you can use either independently. [17][18]

## 1. Event Sourcing: Store What Happened

In a typical application, you update the current state directly:

```text
Registration status: "pending" → "confirmed"
```

With event sourcing, you append an event describing what happened:

```text
RegistrationRequested
RegistrationConfirmed
```

The event history is the source of truth. Current state can be reconstructed by processing the relevant events in order. [17]

Events are treated as immutable during normal operation. Corrections are generally recorded as new events rather than edits to previous ones.

> An event log is not just an application log for debugging. In event sourcing, it is the authoritative record used to determine application state.

## 2. Commands vs. Events

A command requests an action. An event records a fact.

| Command | Possible resulting event |
|---|---|
| `RegisterAttendee` | `AttendeeRegistered` |
| `CancelRegistration` | `RegistrationCancelled` |

The application validates the command before recording a successful state change. A command can be rejected, so not every command produces an event.

A simplified flow is:

```text
Command
   ↓
Validate business rules
   ↓
Append event
   ↓
Update read models
```

For example, `RegisterAttendee` might be rejected if the conference is full.

## 3. CQRS: Separate Writes from Reads

CQRS stands for Command Query Responsibility Segregation.

- Commands change state.
- Queries retrieve information without changing it.

CQRS separates the read and write models so each can fit its purpose. It does not require separate databases or event sourcing. [18]

When combined with event sourcing:

```text
Command → Validation → Event store
                           ↓
                    Projection handlers
                           ↓
                       Read models
                           ↑
                         Queries
```

### Materialized Views and Projections

A materialized view stores a derived representation that is convenient to query.

A projection processes events to build or update that representation.

For a conference application, the same events could maintain:

- An attendee list.
- A count of registrations per session.
- A registration status page.

<img width="351" height="302" alt="Conference events used to build multiple materialized views" src="https://github.com/user-attachments/assets/e8f6a01b-b873-4258-8f7e-43263d1c8686" />

These views are derived from the event history rather than being independent sources of truth. They can be rebuilt by replaying the relevant events through the projection logic. [17]

## 4. Benefits

- History: Events preserve the sequence of recorded changes.
- Intent: An event such as `RegistrationCancelled` explains more than a changed status field alone.
- Read flexibility: Different views can serve different queries.
- Rebuilding: Replaying events can reconstruct read models or build new ones. [17]

An append-only log can support efficient writes, but event sourcing is not automatically simpler or faster than updating database rows.

## 5. Costs and Failure Cases

### Read Models Can Lag

If views update asynchronously, a successful write may not appear immediately in a query result.

This is eventual consistency: The read model catches up after processing the event. The application needs to account for that delay. [18]

### Events Can Be Delivered More Than Once

Consumers may receive duplicate events.

A handler should be idempotent: Processing the same event again must not apply its effect twice.

For example, receiving `AttendeeRegistered` twice should not increment attendance twice. [17]

### Ordering Matters

Events that affect the same entity must be processed in the required order.

For example:

```text
RegistrationCreated → RegistrationCancelled
```

Reversing these events could produce the wrong state.

A single global order across every unrelated event is not always necessary. The important requirement is preserving the ordering needed for correct state and projections. [17]

### Replays Must Not Repeat External Actions

Rebuilding a view should not accidentally resend confirmation emails or repeat payments.

Separate rebuilding derived state from triggering external side effects, and protect side-effect handlers against duplicates. [17]

### Event History Needs Long-Term Care

- Event formats may evolve while older events still need to be readable.
- Immutable history complicates deletion requirements when events contain personal data.
- Replaying a large history can take time.

These concerns make event sourcing more complex than simply appending records. [17]

## 6. What to Remember

- Event sourcing determines the source of truth: An event history rather than only the latest state.
- CQRS separates read and write models; it does not require event sourcing.
- Projections turn events into query-friendly read models.
- Asynchronous projections introduce lag, retries, duplicates, and ordering concerns.
- An ordinary database can store events, but reliable event sourcing requires more than an append-only table.

For now, focus on understanding the data flow and failure cases. You do not need to implement event sourcing in every backend application; it is a trade-off for systems that benefit enough from history, replay, and specialized read models to justify the complexity.
