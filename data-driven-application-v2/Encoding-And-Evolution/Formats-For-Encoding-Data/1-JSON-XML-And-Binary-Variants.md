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


<img width="327" height="199" alt="Screenshot 2026-11-02 at 2 17 14 PM" src="https://github.com/user-attachments/assets/38fc2df7-9f3d-4062-9a8b-46e7c9d249ab" />


Binary Encodings (section)

Its clear that both JSON and XML used a lot of space compared to binary encodings

Below is an example of data that would be binary encoded:
{
    "userName": "Martin",
    "favoriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}

Here is an example using MessagePack

<img width="373" height="344" alt="Screenshot 2026-11-02 at 2 28 07 PM" src="https://github.com/user-attachments/assets/aa185c09-b4cb-4948-9fb6-b9a1ade761e4" />



