## Data Models and Query Languages

Data models are perhaps the most important part of developing software, because of the profound effect they have not only on how the software is written, but also on how we think about the problem we are solving.

Most applications are built by layering one data model on top of another. For each layer, the key question is how it is represented in terms of the next-lower layer.

**Example of application layers from highest to lowest level:**

1. As an application developer, you look at the real world (people, organizations, goods, actions, money flows, sensors, etc.) and model it in terms of objects or data structures and APIs that manipulate those data structures.

2. When you want to store those data structures, you express them in terms of a general-purpose data model, such as JSON or XML documents, tables in a database, or vertices and edges in a graph.

3. The engineers who built your database software decided on a way of representing that document, relational, or graph data in terms of bytes in memory, on disk, or on a network. The representation may allow the data to be queried, searched, manipulated, and processed in various ways.

4. On yet lower levels, the hardware engineers have figured out how to represent bytes in terms of electrical currents, pulses of light, magnetic fields, and more.

**The basic idea is:** Each layer hides the complexity of the layers below it by providing a clean data model.

Several data models are widely used in practice, often for different purposes. Some types of data and some queries are easy to express in one model but awkward in another.

In this chapter we explore these trade-offs by comparing the relational model, the document model, graph-based data models, event sourcing, and dataframes.

### Terminology: Declarative Query Languages
