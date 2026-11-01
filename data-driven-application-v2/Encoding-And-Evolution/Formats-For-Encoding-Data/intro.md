# Formats for Encoding Data

Programs typically work with data in two different representations:

- **In-memory representation:** Data is stored in objects, structs, lists, and other data structures. These structures are optimized for efficient access and manipulation by the CPU, often using pointers and references.
- **Byte sequence representation:** When data needs to be written to a file or transmitted over a network, it must be encoded into a self-contained sequence of bytes.

Therefore, systems need a way to translate between these two representations.

## Encoding and Decoding

- **Encoding** (also called serialization or marshaling) is the process of converting an in-memory representation into a sequence of bytes.
- **Decoding** (also called parsing, deserialization, or unmarshaling) is the reverse process: converting a sequence of bytes back into an in-memory representation.

**Note:** The term *serialization* is also used in the context of database transactions, but it has a completely different meaning there.

## Encoding Formats

Most systems need to convert between in-memory objects and flat byte sequences. There are many libraries and encoding formats available, each with different trade-offs in readability, performance, compatibility, and data size.
