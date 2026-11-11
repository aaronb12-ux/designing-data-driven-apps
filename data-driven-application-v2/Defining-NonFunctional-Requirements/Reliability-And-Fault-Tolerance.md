# Reliability

Reliability means continuing to provide the intended service under stated conditions, even when some things go wrong. [54]

From a user's perspective, this includes:

- Correct behavior.
- Acceptable performance under the expected load.
- Safe handling of mistakes and unexpected inputs.
- Protection against unauthorized access and abuse.

The goal is not to prevent every possible problem. It is to prevent the faults we plan for from becoming user-visible failures.

## 1. Faults vs. Failures

| Term | Meaning | Example |
|---|---|---|
| Fault | A component stops working correctly. | A disk fails or a machine crashes. |
| Failure | The system no longer provides the required service. | Users cannot retrieve their stored data. |

The distinction depends on the system boundary.

A disk fault might cause an entire single-disk system to fail. In a system with suitable redundancy and recovery, the same disk fault might not interrupt service.

> A component can fail without the overall service failing.

Service-level objectives, or SLOs, define measurable reliability targets. They help evaluate service quality, but an individual failed request does not necessarily mean the service has breached its SLO over the measurement period.

## 2. Fault Tolerance and Single Points of Failure

A fault-tolerant system continues providing the required service despite certain faults.

A single point of failure, or SPOF, is a component whose loss can bring down the required service.

For example:

```text
Application → Only database server
```

If the application cannot operate without that database and there is no working alternative, the database is a single point of failure.

Adding another instance helps only if the system can actually use it when the first one becomes unavailable. Redundancy needs working detection, failover, and sufficient remaining capacity. [53]

### Example: Timeline Fan-Out

Suppose a worker crashes while adding a post to follower timelines.

A fault-tolerant design should allow another worker to resume the work:

- Pending work must not disappear with the crashed worker.
- Retrying must not permanently miss recipients.
- Repeated processing must not create duplicate timeline entries.

This connects reliability to durable queues, retries, and idempotent processing.

### Fault Tolerance Has Limits

Always ask:

> Which faults can this system tolerate, and how many at once?

Tolerating one machine crash does not automatically mean tolerating an entire datacenter outage.

## 3. Hardware Faults and Redundancy

Hardware faults include:

- Failed disks or SSDs.
- Power supply failures.
- Faulty memory.
- CPU errors.
- Data corruption.

As the number of machines grows, individual hardware faults become routine operational events rather than unusual exceptions. [52]

Redundancy provides alternatives when components stop working.

| Level | Example |
|---|---|
| Disk | A redundant RAID configuration. |
| Server | Multiple application instances. |
| Power | Redundant power supplies and backup power. |
| Location | Replicas across availability zones or datacenters. |

Redundancy works best when alternatives do not share the same likely causes of failure. [53]

### Correlated Faults

Two machines are not fully independent just because they are separate machines.

They might share:

- A power source.
- A network dependency.
- A software bug.
- A bad configuration or deployment.

A single problem can therefore affect both.

> Count independent failure domains, not just copies.

A failure domain is a group of components that can be affected by the same failure, such as machines in one rack or one availability zone. Spreading instances across failure domains reduces exposure to shared faults. [52]

## 4. Fault Injection: Test Recovery Deliberately

Fault injection means intentionally introducing controlled faults to test whether recovery works.

Examples include:

- Stopping a worker process.
- Taking an instance offline.
- Making a dependency unavailable.

The goal is to verify behavior, not simply create disruption.

Start with a limited scope, monitor the effects, and have a clear way to stop the experiment. Controlled recovery exercises also test whether engineers can diagnose and respond to failures. [55]

## 5. Rolling Upgrades

Fault tolerance also helps with planned maintenance.

A rolling upgrade updates instances gradually rather than taking the entire service offline.

```text
Update one instance
        ↓
Verify it is healthy
        ↓
Continue with the next instance
```

This requires the remaining instances to handle traffic and the system to tolerate the temporary mix of old and new versions.

Multiple instances make uninterrupted upgrades possible; they do not guarantee them.

## What to Remember

- A fault affects a component; a failure affects the required service.
- Fault tolerance prevents certain component faults from becoming service failures.
- A single point of failure can defeat an otherwise redundant design.
- Redundant components need working failover and enough remaining capacity.
- Shared dependencies can cause correlated faults.
- Recovery should be tested, not assumed.
- The same mechanisms that handle crashes can also make maintenance safer.
