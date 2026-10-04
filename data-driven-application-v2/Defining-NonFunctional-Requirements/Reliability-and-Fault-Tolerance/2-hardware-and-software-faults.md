# Hardware Faults

When we think of causes of system failure, hardware faults quickly come to mind. Some common examples include:

- Magnetic hard drive failures
- Solid state drive (SSD) failures
- Power supply failures
- RAID controller failures
- Memory module failures
- CPU cores that occasionally compute the wrong result (roughly 1 in 1,000 machines)
- Data corruption in RAM

These events are rare enough that you often don't need to worry about them when working on a small system. In large-scale systems, however, hardware faults happen often enough that they become part of normal system operation.

## Tolerating Hardware Faults Through Redundancy

Our first response to unreliable hardware is usually to add redundancy to individual hardware components in order to reduce the failure rate of the system. Examples include:

- **RAID configuration** — data is spread across multiple disks in the same machine so that a failed disk does not cause data loss
- **Dual power supplies and hot-swappable CPUs** on servers
- **Batteries and diesel generators** in datacenters for backup power

Such redundancy can often keep a machine running uninterrupted for years.

Redundancy is most effective when component faults are independent — that is, when the occurrence of one fault does not increase the likelihood that another fault will occur.

## Tolerating the Loss of Entire Machines

The fault-tolerance techniques discussed here are designed to tolerate the loss of entire machines, racks, or availability zones. They generally work by allowing a machine in one datacenter to take over when a machine in another datacenter fails or becomes unreachable.

Systems that can tolerate the loss of entire machines also have operational advantages. A single-server system requires planned downtime if you need to reboot the machine, whereas a multi-node fault-tolerant system can be patched by restarting one node at a time without affecting the service for users. This is called a **rolling upgrade**.



