# Summary

We discussed several ways of encoding data structures into byte sequences for transmission over a network or storage on disk.

The details of these encoding formats affect not only **efficiency**, but more importantly, the **architecture of applications and their ability to evolve over time**.

## Rolling Upgrades and Compatibility

Many services need to support **rolling upgrades**, where a new version of a service is gradually deployed to a few nodes at a time rather than updating all nodes simultaneously.

Benefits include:

- Less risky deployments.
- Reduced downtime.
- Greater **evolvability**: the ease of making changes to an application over time.

To support rolling upgrades, data flowing through a system must be encoded in a way that provides:

- **Backward compatibility:** Newer code can read data written by older code.
- **Forward compatibility:** Older code can read data written by newer code.

## Encoding Formats and Their Compatibility Properties

We discussed several encoding formats:

- **Programming-language-specific formats:** Often restricted to applications written in a particular programming language.
- **Textual formats (JSON, XML, CSV):** Widely used, but their compatibility depends on how they are structured and how applications handle changes.
- **Binary schema-driven formats (Protocol Buffers and Avro):** Provide compact, efficient encoding with clearly defined rules for forward and backward compatibility.

## Modes of Dataflow

We discussed three common ways encoded data flows between processes:

- **Databases:** The process writing to the database encodes the data, and the process reading from the database decodes it.
- **RPC and REST APIs:** The client encodes a request, the server decodes it, and the server then encodes a response that the client decodes.
- **Event-driven architectures (message brokers):** Processes communicate by sending messages that are encoded by the sender and decoded by the recipient.
  
