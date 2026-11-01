# Language-Specific Formats

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
- `integer` (a whole-number subset of `number`)
- `object`
- `array`
- `boolean`
- `null`

### Open Content Models

An **open content model** allows fields that are not explicitly defined in the schema to exist in the data.

These additional fields can contain values of any valid JSON data type, unless further restrictions are specified.
