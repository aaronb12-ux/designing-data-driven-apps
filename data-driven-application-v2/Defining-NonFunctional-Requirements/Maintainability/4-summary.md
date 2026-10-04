## Summary

We examined several examples of nonfunctional requirements: performance, reliability, scalability, and maintainability.

We discussed how to measure performance (e.g., using response time percentiles) and the load on a system (e.g., using throughput metrics), and how these metrics are used in SLAs.

Scalability is a closely related concept: it focuses on ensuring that performance stays the same when the load grows. We saw some general principles for scalability, such as breaking a task into smaller parts that can operate independently.

To achieve reliability, you can use fault-tolerance techniques, which allow a system to continue providing its services even if a component (e.g., a disk, a machine, or another service) is faulty. We saw examples of hardware faults that can occur and distinguished them from software faults, which can be harder to deal with because they are often strongly correlated. Another aspect of achieving reliability is to build resilience against humans making mistakes.

We finally examined several facets of maintainability, including supporting the work of operations teams, managing complexity, and making it easy to evolve an application's functionality over time.

There are no easy answers to how to achieve these goals, but one approach that can help is to build applications using well-understood building blocks that provide useful abstractions.
