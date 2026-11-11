# Encoding and Evolution

Application changes often require data changes, such as adding a field or introducing a new record type.

The challenge is that code and data do not all update at once.

## 1. Different Versions Coexist

During a rolling upgrade, servers are updated gradually, so old and new versions run simultaneously.

Client applications may remain on older versions even longer because users control when they update.

Stored data can also outlive the code that originally wrote it.

This creates two compatibility requirements:

| Direction | Meaning |
|---|---|
| Backward compatibility | New code can read data written by old code. |
| Forward compatibility | Old code can read data written by new code. |

> Plan for mixed versions rather than assuming everything upgrades together.

## 2. Reading Successfully Is Not Enough

Older code may ignore an unfamiliar field and still read a record successfully.

The risk appears when it writes the record back.

Suppose a new version adds `nickname`:

```json
{
  "name": "Alex",
  "nickname": "Al"
}
```

An older version understands only `name`. If it replaces the entire record after changing the name, it might write:

```json
{
  "name": "Alexander"
}
```

The `nickname` has disappeared.

![Example of forward compatibility and potential data loss](https://github.com/user-attachments/assets/f13232fa-e888-4b22-a5ae-d49a0bec4bc3)

This loss is not inevitable. It depends on whether the application preserves unknown fields and whether the write updates selected fields or replaces the whole record.

## What to Remember

- New and old application versions may access the same data.
- Backward compatibility lets new readers handle old data.
- Forward compatibility lets old readers handle new data.
- Ignoring unknown fields can help reading, but does not guarantee safe rewriting.
- Check the full read-modify-write cycle when evolving a data format.
