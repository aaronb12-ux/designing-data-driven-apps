# Response Time Distribution

Because response time varies from one request to the next, we need to think of it not as a single number, but as a distribution of values that we can measure.

In the figure below, each gray bar represents a request to a service, and the height shows how long that request took. Most requests are fast, but there are occasional outliers that take much longer.

<img width="363" height="162" alt="Response time distribution chart" src="https://github.com/user-attachments/assets/e34ea79d-321d-40b4-8229-2f93c3a94cd2" />

It's common to report the average response time of a service. The mean response time is useful for estimating throughput limits.

It's best to use percentiles to understand a "typical" response time. A good approach is to use the median by sorting the response times. The median is also known as the 50th percentile, or p50.

High response-time percentiles, also known as **tail latencies**, are important because they directly affect users' experience of the service.


