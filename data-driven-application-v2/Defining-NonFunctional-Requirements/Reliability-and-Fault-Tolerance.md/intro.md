# Reliability

For software, something is reliable when:

- The application performs the function that the user expected
- The application can tolerate the user making mistakes or using the software in unexpected ways
- Its performance is good enough for the required use case, under the expected load and data volume
- The system prevents any unauthorized access and abuse

If all those things together mean "working correctly", then we can understand reliability as meaning roughly: **"continuing to work correctly, even when things go wrong."**

## Faults vs. Failures

**Fault:** A fault occurs when a particular part of a system stops working correctly — for example, if a single hard drive malfunctions, a single machine crashes, or an external service has an outage.

**Failure:** A failure occurs when the system as a whole stops providing the required service to the user — in other words, when it does not meet the SLO (Service Level Objectives).

### Examples

- If a hard drive stops working, we say that the hard drive has *faulted*; if the system consists of only that one hard drive, it has stopped providing the required service and has therefore also *failed*.
- However, if the system contains multiple hard drives, the failure of a single hard drive is only a fault from the point of view of the larger system.
