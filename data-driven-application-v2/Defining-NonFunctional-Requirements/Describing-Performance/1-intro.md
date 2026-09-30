Most discussions of software performance consider two main types of metric:

**Response time:** the elapsed time from the moment when a user makes a request until they receive the requested answer. The unit of measurement is seconds

**Throughput:** the number of requests per second, or the data volume per second, that the system is processing. For a given allocation of hardware resources, there is a maximum throughput that can be handled. The unit of measurement is "somethings per second"

Throughput and response time are often related. In the below graph, we see that when there is low throughput, the response time is lower. As the throughput grows the response time increases. This is because of queuing: when a request arrives on a highly loaded system, the CPU is likely already in the process of handling an earlier request, and therefore the incoming request needs to wait until the earlier request has been completed.

<img width="333" height="215" alt="Screenshot 2026-03-10 at 2 08 27 AM" src="https://github.com/user-attachments/assets/2062941f-b9c0-416c-bc45-f44c9857c2c3" />

## When An Overloaded System Won't Recover

If a system is close to overload, with throughput pushed close to the limit, it can sometimes enter a cycle where it becomes less efficient and hence even more overloaded. If a long queue of requests is waiting to be handled, response times may increase so much that clients time out and resend their requests. Even when the load is reduced again, such a system may remain in an overloaded state until it is rebooted or otherwise reset. This situation is called a metastable failure, and it can cause serious outages in production systems.

---

In terms of performance metrics, the response time is usually what the users care about the most, whereas the throughput determines the required computing resources (like how many servers you need) and hence the cost of serving a particular workload. 

If throughput is likely to increase beyond the current hardware's capability, the capacity needs to be expanded; a system is said to be scalable if its maximum throughput can be significantly increased by adding computing resources.
