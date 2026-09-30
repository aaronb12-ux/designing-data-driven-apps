## Tail Latency Amplification

High percentiles are especially important in backend services that are called multiple times as part of serving a single end-user request. Even if you make the calls in parallel, the request still needs to wait for the slowest of the parallel calls to complete — meaning it takes just one slow call to make the entire end-user request slow.

Even if only a small percentage of backend calls are slow, the chance of getting a slow call increases if an end-user request requires multiple backend calls, so a higher proportion of such end-user requests end up being slow. This is known as **tail latency amplification**.

<img width="349" height="212" alt="Tail latency amplification diagram" src="https://github.com/user-attachments/assets/e3820fe4-a6b2-488f-9b22-d2f3d170696d" />

