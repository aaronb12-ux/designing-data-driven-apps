# Formats for Encoding Data

Encoding, also called serialization, converts application data into bytes that can be stored or transmitted. Decoding, or deserialization, converts those bytes back into usable data.

```text
Application data → Encode → Bytes → Store or transmit
                                      ↓
Application data ← Decode ←───────────┘
```

The format determines how those bytes represent fields, values, and structure.

## 1. JSON, XML, and CSV

These text formats are widely supported across programming languages.

| Format | Structure | Main consideration |
|---|---|---|
| JSON | Objects, arrays, and values. | Easy to inspect, but no native raw binary type. |
| XML | Nested elements and attributes. | Flexible, but often verbose. |
| CSV | Rows and columns. | Convenient for tabular data, but lacks built-in nesting and explicit data types. |

JSON and XML commonly represent embedded binary data using a text encoding such as Base64.

CSV requires an agreement about what columns mean and how values should be interpreted—for example, whether `00123` is an identifier or a number.

> A readable format does not remove the need for an agreed data structure.

## 2. JSON Schema: Describe and Validate JSON

JSON Schema defines rules for JSON data, including field types, required fields, and whether additional fields are allowed.

Example data:

```json
{
  "userName": "Martin",
  "favoriteNumber": 1337,
  "interests": ["daydreaming", "hacking"]
}
```

A corresponding schema:

```json
{
  "type": "object",
  "properties": {
    "userName": { "type": "string" },
    "favoriteNumber": { "type": "integer" },
    "interests": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["userName"],
  "additionalProperties": true
}
```

Important distinctions:

- `properties` describes fields; it does not automatically make them required.
- `required` lists fields that must be present.
- `additionalProperties: true` allows fields not listed in the schema. This is also the default.
- `additionalProperties: false` rejects those extra fields. [86]

Allowing extra fields is called an open content model.

JSON has strings, numbers, objects, arrays, booleans, and `null`. JSON Schema additionally distinguishes whole-number values using `integer`; JSON itself does not have separate integer syntax.

![JSON Schema example](https://github.com/user-attachments/assets/38fc2df7-9f3d-4062-9a8b-46e7c9d249ab)

## 3. Binary Encodings

Binary formats encode structured data without representing everything as readable text. They can reduce message size, but the savings depend on the data and format. MessagePack and Protocol Buffers are two examples. [87][75]

The trade-off is that you generally need decoding tools to inspect the data.

### MessagePack

MessagePack represents JSON-like structures using compact binary encodings for maps, arrays, strings, numbers, and other values. It does not require a predefined application schema. [87]

When an object is encoded as a map with string keys, its field names are still included. Binary encoding alone does not eliminate them.

![MessagePack encoding example](https://github.com/user-attachments/assets/aa185c09-b4cb-4948-9fb6-b9a1ade761e4)

### Protocol Buffers

Protocol Buffers, or Protobuf, uses a schema to define message fields and types. Tools can generate code for serializing and deserializing those messages. [75]

```protobuf
syntax = "proto3";

message Person {
  string user_name = 1;
  int64 favorite_number = 2;
  repeated string interests = 3;
}
```

The key parts are:

- `string` and `int64`: Field types.
- `1`, `2`, and `3`: Unique field numbers within this message.
- `repeated`: A field containing zero or more values.

The binary representation identifies fields by number rather than storing their full names. The decoder uses the message definition to interpret them. [72]

![Protocol Buffers encoding example](https://github.com/user-attachments/assets/b71d9fc7-9aa1-4e3d-bfd6-1364da718bf4)

You do not need to memorize the byte layout. Understand why stable field numbers matter.

## 4. Why Schemas Matter

A schema makes the data contract explicit:

- Which fields exist?
- What types do they have?
- Which are required?
- How should different versions interpret them?

JSON Schema validates JSON structure. A Protobuf schema also guides encoding, decoding, and generated code. These are related benefits, but not identical roles. [86][75]

Schemas can support compatibility checks, but having a schema does not automatically make every change safe.

## 5. Schema Evolution

Applications change while older clients, services, and stored messages may still exist.

Two compatibility directions matter:

| Term | Meaning |
|---|---|
| Backward compatibility | New code can read data written by old code. |
| Forward compatibility | Old code can read data written by new code. |

For example, adding an optional field may allow:

- New readers to handle older messages where the field is absent.
- Old readers to ignore the unfamiliar field.

Whether this works depends on the format, validation rules, and application behavior.

### Protobuf Rules to Remember

For Protobuf's binary format:

- Adding fields is generally wire-compatible.
- Do not change existing field numbers.
- Do not reuse deleted field numbers; reserve them.
- Do not assume changing a field's type is safe.
- Application code still needs sensible behavior when fields are absent. [73][75]

Compatibility means more than successfully decoding bytes. The application must still interpret the message correctly.

## What to Remember

- Encoding turns application data into a format suitable for storage or transmission.
- JSON, XML, and CSV favor readability and broad interoperability.
- MessagePack provides binary encoding without requiring a predefined application schema.
- Protobuf uses schemas and numeric field identifiers.
- Schemas document contracts and help validate data.
- Safe evolution matters whenever different versions exchange or retain data.
- Focus on format trade-offs and compatibility rules before learning byte-level details.

