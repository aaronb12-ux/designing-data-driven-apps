# Formats for Encoding Data

## JSON, XML, and Binary Variants

Common standardized encoding formats that can be read by many programming languages include **JSON** and **XML**. **CSV** is also widely used, but it only supports tabular data without nesting.

### Limitations of JSON, XML, and CSV

- **XML:** Often criticized for being overly verbose and complicated.
- **JSON and XML:** Both support Unicode character strings (human-readable text), but neither natively supports binary strings. Binary data must typically be encoded into a text representation, such as Base64.
- **CSV:** Does not define a schema, so the application must determine the meaning and expected data type of each row and column.

There are other limitations, but these are some of the main ones.

Despite these drawbacks, JSON, XML, and CSV are sufficiently flexible and widely supported for many applications.

## JSON Schema

JSON Schema is a widely adopted standard for describing and validating the structure of JSON data exchanged between systems or stored in files.

### JSON Primitive Types

JSON supports the following data types:

- `string`
- `number`
- `integer` (a whole-number subset of `number` in JSON Schema)
- `object`
- `array`
- `boolean`
- `null`

### Open Content Models

An **open content model** allows fields that are not explicitly defined in the schema to exist in the data.

These additional fields can contain values of any valid JSON data type, unless further restrictions are specified.

![JSON Schema example](https://github.com/user-attachments/assets/38fc2df7-9f3d-4062-9a8b-46e7c9d249ab)

## Binary Encodings

JSON and XML are human-readable, but they can use considerably more space than binary encoding formats.

Binary encodings represent data using a compact sequence of bytes, reducing the amount of space required to store or transmit it.

Consider the following JSON object:

```json
{
    "userName": "Martin",
    "favoriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```

### MessagePack

**MessagePack** is a binary encoding format that converts JSON-like data structures into a compact binary representation.

Unlike JSON, which represents data using readable text, MessagePack encodes values into binary bytes.

![MessagePack encoding example](https://github.com/user-attachments/assets/aa185c09-b4cb-4948-9fb6-b9a1ade761e4)

## Protocol Buffers

**Protocol Buffers (Protobuf)** is a binary encoding library developed by Google. Unlike MessagePack, it requires a schema to define the structure and types of the data being encoded.

Here is how the previous JSON example would be represented using a Protocol Buffers schema:

```protobuf
syntax = "proto3";

message Person {
    string user_name = 1;
    int64 favorite_number = 2;
    repeated string interests = 3;
}
```

![Protocol Buffers encoding example](https://github.com/user-attachments/assets/b71d9fc7-9aa1-4e3d-bfd6-1364da718bf4)

### How Protocol Buffers Work

- **Field types:** Each field has a declared data type, such as `string` or `int64`. Protobuf strings are encoded using UTF-8.
- **Field tags:** Each field is assigned a unique numeric tag (`1`, `2`, `3`). These tags identify the fields in the encoded data without needing to store their full field names.
- **Compact encoding:** Protobuf combines the field tag and wire type into a single key, reducing the amount of space required. Larger tag numbers may require additional bytes.
- **Repeated fields:** Protobuf does not use a separate array type modifier in the schema. Instead, the `repeated` keyword indicates that a field can contain multiple values. Each occurrence is encoded using the same field tag.

## The Merits of Schemas

Many data systems also implement proprietary binary encodings for their data.

Most relational databases have a network protocol through which clients can send queries and receive responses. These protocols are generally specific to a particular database, and the database vendor provides a driver that decodes responses from the database's network protocol into in-memory data structures.

Although textual data formats such as JSON, XML, and CSV are widespread, binary encodings based on schemas are also a viable option. They offer several advantages:

- **Compactness:** Schema-based binary encodings can be considerably more compact than binary JSON variants because they can omit field names from the encoded data.
- **Documentation:** The schema serves as a valuable form of documentation. Since the schema is required for decoding, the data's structure is explicitly defined.
- **Compatibility checking:** Maintaining a database of schemas allows us to check whether schema changes are backward- and forward-compatible before deploying them.

### Schema Evolution

Schemas can evolve over time as applications introduce new fields, modify existing ones, or change their data structures.

Schema evolution allows applications to change their data formats while maintaining compatibility between different versions of the code and stored data.

In this way, schema-based formats can provide much of the flexibility associated with schemaless databases and schema-on-read approaches, while still maintaining an explicitly defined schema.


