# Summary: Encoding and Evolution

Encoding turns application data into bytes for storage or transmission. Format choices affect message size, interoperability, and how safely the application can change over time.

## 1. Compatibility Enables Independent Updates

Rolling upgrades, older clients, and historical data mean different versions must coexist.

- Backward compatibility: New code can read old data.
- Forward compatibility: Old code can read new data.

Compatibility depends on both the encoding format and application behavior. Successfully reading a record does not guarantee that rewriting it will preserve unfamiliar fields.

## 2. Encoding Formats Have Different Trade-Offs

| Category | Examples | Main consideration |
|---|---|---|
| Language-specific | Programming-language serialization formats | May tightly couple readers and writers to a language or runtime. |
| Text | JSON, XML, CSV | Readable and widely supported; compatibility still requires an agreed structure. |
| Binary without a required application schema | MessagePack | Compact representation, but applications still need shared expectations. |
| Schema-driven binary | Protobuf, Avro | Explicit structure and evolution rules; changes must follow those rules. |

> A schema helps define a contract. It does not make every change compatible automatically.

## 3. Four Modes of Dataflow

| Mode | How data moves |
|---|---|
| Databases | One process writes data that another reads later. |
| Service calls | Clients and servers exchange encoded requests and responses. |
| Workflow engines | Persisted progress coordinates tasks and recovery. |
| Asynchronous messaging | Producers send messages that consumers process independently. |

Across all four, ask:

- Can different versions interpret the data?
- What happens if processing fails?
- Can retries duplicate an action?
- Will existing data remain readable?

The central lesson: Design data contracts for change, not just for the versions running today.
