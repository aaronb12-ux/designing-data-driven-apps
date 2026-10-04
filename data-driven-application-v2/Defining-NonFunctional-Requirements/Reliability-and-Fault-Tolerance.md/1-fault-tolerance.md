# Fault Tolerance

We call a system **fault-tolerant** if it continues providing the required service to users in spite of certain faults occurring.

If a system cannot tolerate a certain part becoming faulty, we call that part a **single point of failure (SPOF)**, because a fault in that part escalates to cause the failure of the whole system.

In the social network example, a fault that might happen is that during the fan-out process, a machine involved in updating the materialized timelines crashes or becomes unavailable. To make this process fault-tolerant, we would need to ensure that another machine can take over the task without missing any posts that should have been delivered, and without duplicating any posts.

Fault tolerance is always limited to a certain number of certain types of faults. For example, a system might be able to tolerate a maximum of two hard drives failing at the same time, or a maximum of one out of three nodes crashing.

## Fault Injection

In some fault-tolerant systems, it can make sense to increase the rate of faults by triggering them deliberately — for example, by randomly killing individual processes without warning. This is called **fault injection**.

By deliberately inducing faults, you ensure that the fault tolerance machinery is continually exercised and tested, which increases your confidence that faults will be handled correctly when they occur naturally.
