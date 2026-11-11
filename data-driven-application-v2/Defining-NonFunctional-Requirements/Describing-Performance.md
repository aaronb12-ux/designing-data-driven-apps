# Software Performance

## 1. Response Time and Throughput

Two core performance metrics answer different questions:

| Metric | Meaning | Example |
|---|---|---|
| Response time | How long a request takes from the client's perspective. | A timeline loads in 200 ms. |
| Throughput | How much work the system processes per unit of time. | The service handles 10,000 requests per second. |

Response time describes how quickly users receive results. Throughput describes the volume of work the system handles.

A system can have high throughput and still have slow individual requests.

## 2. Load, Capacity, and Queuing

As incoming traffic approaches the system's capacity, requests spend more time waiting for resources.

Those resources might include:

- CPU time.
- Database connections.
- Worker threads.
- Disk or network bandwidth.

<img width="333" height="215" alt="Response time increasing as throughput approaches system capacity" src="https://github.com/user-attachments/assets/2062941f-b9c0-416c-bc45-f44c9857c2c3" />

Response time can rise sharply near capacity because of queuing. Beyond capacity, more incoming requests may increase waiting, timeouts, or rejections rather than completed throughput. Keeping spare capacity helps absorb bursts. [40]

> Incoming request rate is demand. Completed throughput is how much work the system actually finishes.

A system is scalable if adding resources can substantially increase the workload it handles while maintaining acceptable performance.

## 3. Where Request Time Goes

### Response Time

The total elapsed time observed by the client, including processing, waiting, and network delays.

### Service Time

The time spent servicing the request at a particular component, excluding time waiting in that component's queue.

### Queuing Delay

Time spent waiting for an available resource.

Queues can exist on the client, server, database, and network—not just on the client.

### Latency

The meaning depends on context. Teams often use “request latency” to mean response time, while “network latency” refers to network delay. Do not assume latency means only time when no processing occurs. [37]

<img width="362" height="173" alt="Request timing showing processing and delays" src="https://github.com/user-attachments/assets/6bcae271-8202-49d4-9785-3b8e1fb07138" />

A useful simplified model is:

```text
Client-observed response time
≈ Network delays + Queuing delays + Processing time
```

For a multi-service request, some work may overlap, so total response time depends on the critical path rather than simply adding every component's duration.

## 4. Why Identical Requests Take Different Amounts of Time

Repeated requests can have different response times because of:

- Queuing behind other work.
- CPU scheduling and context switches.
- Garbage collection pauses.
- Disk reads or page faults.
- Network packet loss and retransmission.
- Slow downstream services.

One slow request can also hold up requests behind it when they share an ordered queue or resource. This is called head-of-line blocking.

Measure both client-observed response time and server-side timings. A server's internal measurements may miss delays outside the measured boundary. [41]

## 5. Response Time Is a Distribution

A single number cannot describe every request.

<img width="363" height="162" alt="Request durations showing mostly fast requests with occasional slow outliers" src="https://github.com/user-attachments/assets/e34ea79d-321d-40b4-8229-2f93c3a94cd2" />

The average is useful, but it can hide a small group of very slow requests. Percentiles show more of the distribution. [37]

| Metric | Interpretation |
|---|---|
| p50, or median | About 50% of requests finish within this time. |
| p95 | About 95% finish within this time; roughly 5% take longer. |
| p99 | About 99% finish within this time; roughly 1% take longer. |

For example:

```text
p50 = 100 ms
p95 = 400 ms
p99 = 2 seconds
```

Most requests are relatively fast, but the slowest portion experiences much longer waits.

High-percentile response times are called tail latencies. They matter because averages can look healthy while some requests remain painfully slow. [37]

> A p99 value is not the maximum. Some requests are still slower.

## 6. Tail Latency Amplification

An endpoint may call several backend services to produce one response.

If all results are required, even parallel execution must wait for the slowest required call.

```text
Timeline request
  ├─ Fetch posts:          40 ms
  ├─ Fetch user profiles:  60 ms
  └─ Fetch permissions:   800 ms
                           ↓
              Response waits for permissions
```

As the number of required backend calls grows, there are more opportunities for one call to be slow. This is tail latency amplification.

<img width="349" height="212" alt="Parallel backend calls where one slow call delays the overall response" src="https://github.com/user-attachments/assets/e3820fe4-a6b2-488f-9b22-d2f3d170696d" />

The important distinction:

- Sequential calls accumulate delays.
- Parallel calls reduce that accumulation, but the slowest required call still determines when the combined result is ready.

## 7. Overload and Retry Feedback Loops

Overload can become self-reinforcing:

```text
More load
   ↓
Longer queues
   ↓
Slower responses and timeouts
   ↓
Clients retry
   ↓
Even more load
```

Retries can keep a struggling system overloaded instead of helping it recover. Real outages have involved exactly this kind of retry amplification. [40]

In a metastable failure, the system can remain stuck in a degraded state even after the original trigger subsides. Recovery requires breaking the feedback loop—not necessarily rebooting.

Useful protections include:

- Limiting accepted work instead of allowing unlimited queues.
- Using bounded retries with backoff and jitter.
- Keeping spare capacity.
- Adding capacity before critical resources become saturated. [40]

## What to Remember

- Response time measures how long a request takes; throughput measures work per second.
- Near capacity, queuing can make response times rise sharply.
- Queues exist throughout the system, not only on the client.
- Track percentiles alongside averages.
- Parallel backend calls can still suffer from one slow dependency.
- Retries add work and can worsen overload.
- Always identify the measurement boundary: client, API server, database, or end-to-end.
