# Encoding and Evolution

Changes to an application's features often require corresponding changes to the data it stores. For example, a new feature might require adding a field, introducing a new record type, or presenting existing data in a different way.

When a data format or schema changes, the application code that reads and writes that data often needs to change as well.

## Rolling Upgrades and Compatibility

With server-side applications, we can perform a **rolling upgrade** by deploying a new version to a few nodes at a time, monitoring its behavior, and gradually upgrading the remaining nodes.

Client-side applications are more difficult because updates depend on users installing the new version. Some users may continue running older versions for an extended period.

As a result, old and new versions of application code—and old and new data formats—may coexist within the same system.

To maintain compatibility, we need to consider two directions:

- **Backward compatibility:** Newer code can read data written by older code.
- **Forward compatibility:** Older code can read data written by newer code.

## Why Forward Compatibility Is Tricky

Forward compatibility is particularly difficult because older code may not understand fields or data structures introduced by newer versions.

If older code encounters an unfamiliar field and simply ignores it, it may cause **data loss** when that data is later written back.

For example, imagine a newer version adds a `nickname` field to a user record. An older version might successfully read the record but ignore the unfamiliar field. If it then saves the record, it could overwrite the original data without preserving `nickname`.

This is why systems need to account for how different application versions handle unknown fields when reading and writing data.

![Example of forward compatibility and potential data loss](https://github.com/user-attachments/assets/f13232fa-e888-4b22-a5ae-d49a0bec4bc3)
