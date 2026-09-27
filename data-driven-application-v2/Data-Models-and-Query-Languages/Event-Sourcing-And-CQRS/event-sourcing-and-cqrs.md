# Event Sourcing and CQRS

- In all the data models looked at, the data is queried in the same form as it is written.
- In complex applications it can sometimes be difficult to find a single data representation that is able to satisfy all the ways that the data needs to be queried and presented.
- In some situations it can be beneficial to write data in one form and then derive from it representations that are optimized for different types of reads (an example of this is ETL).

The simplest, fastest, and most expressive way of writing data is an event log: every time you want to write some data, you encode it as a self-contained string, including a timestamp, and then append it to a sequence of events. The events in this log are immutable (they can only be added to, never changed or removed).

The example below shows a conference management system — we keep a log of immutable events as the source of truth and derive materialized views from it.

<img width="351" height="302" alt="Screenshot 2026-09-26 at 6 38 45 PM" src="https://github.com/user-attachments/assets/e8f6a01b-b873-4258-8f7e-43263d1c8686" />

Whenever an event is appended to the log, several materialized views are also updated to reflect the effect of that event.

- The idea of using events as the source of truth and expressing every state change as an event is known as **event sourcing**.
- The principle of maintaining separate read-optimized representations and deriving them from the write-optimized representation is called **command query responsibility segregation (CQRS)**.

Requests are called commands, and they need to be validated before being added to the official log of events.

A similarity between event sourcing and a star schema fact table is that both are collections of events that happened in the past.

## Advantages and Disadvantages of Event Sourcing and CQRS

**Advantages:**
- Events better communicate the intent of why something happened.
- The materialized views are derived from the event log in a reproducible way.
- You can have multiple materialized views that are optimized for the particular queries that your application requires.
- Event logs can typically handle higher write throughput than databases because of their sequential access patterns.

**Disadvantages:**
- Need to be careful if external side effects are involved.
- The requirement that events are immutable creates problems if events contain personal data from users, as users may exercise their right to request deletion of their data.

Event sourcing can be implemented on top of any database. The only important requirement is that the event storage system must guarantee that all materialized views process the events in exactly the same order as they appear in the log.
