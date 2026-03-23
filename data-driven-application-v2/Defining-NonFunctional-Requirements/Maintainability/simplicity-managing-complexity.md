## Simplicity: Managing Complexity

Reducing complexity greatly improves the maintainability of software, and thus simplicity should be a key goal for the systems we build.

One system may hide a complex implementation behind a simple interface, whereas another may have a simple implementation that exposes more internal detail to its users -- which one is simpler?

One attempt at reasoning about complexity breaks it into two categories: essential and accidental. The idea is that essential complexity is inherent in the problem domain of the application, while accidental complexity arises only because of limitations of our tooling.

One of the best tools we have for managing complexity is abstraction. A good abstraction can hide a great deal of implementation detail behind a clean, simple-to-understand facade.

For example, high-level programming languages are abstractions that hide machine code, CPU registers, and system calls. SQL is an abstraction that hides complex on-disk and in-memory data structures, concurrent requests from other clients, and inconsistencies after crashes.

Abstractions for application code that aim to reduce its complexity can be created using methodologies such as design patterns and domain-driven design.
