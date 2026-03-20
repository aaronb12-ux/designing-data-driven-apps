## Understanding Load

First you need a clear understanding of the current load on the system. Only then can you discuss growth questions like "What happens if our load doubles"

Other statistical characteristics of the load affect the access patterns and hence the scalability requirements. For example, you may need to know the number of reads to writes in a database, the hit rate on a cache, or the number of data items per user.

Once you understand the load on your system, you can investigate what happens when the load increases:
* When you increase the load in a certain way and keep the system resources (CPUs, memory, network bandwidth, etc) unchanged, how is the performance of your system affected?
* When you increase the load in a certain way, how much do you need to increase the resources if you want to keep performance unchanged?

Usually the goal is to keep the performance of the system within the requirements of the SLA while also minimizing the cost of running the system.

If doubling the resources will enable you to handle twice the load while keeping performance the same, we say that you have **linear scalability**, and this is considered a good thing.
