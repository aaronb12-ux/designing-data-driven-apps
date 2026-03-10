## Vocab:

* **The response time** is what the client sees; it includes all delays incurred anywhere in the system
* **The service time** is the duration for which the service is actively processing the client's request
* **Queuing delays** can occur at several points in the flow - for example, after a request is received, it might need to wait until a CPU is available before it can be processed, or a response packet might need to be buffered before it is sent over the network if other tasks on the same machine are sending a lot of data via the outbound network interface
* **Latency** is the catchall term for time during which a request is not being actively processed - that is, during which it is latent. In particular, a network latency or network delay refers to the time that a request and response spend traveling through the network

<img width="362" height="173" alt="Screenshot 2026-03-10 at 2 19 36 AM" src="https://github.com/user-attachments/assets/6bcae271-8202-49d4-9785-3b8e1fb07138" />

The response time can vary significantly from one request to the next, even if you keep making the same request over and over again. Some examples that can cause delays are: a context switch to a background process, the loss of a network packet and TCP retransmission, a garbage collection pause, a page fault forcing a read from disk, etc

Queueing delays often account for a large part of the variability in response times. As a server can process only a small amount of things in parallel, it takes only a small number of slow requests to hold up the processing of subsequent requests -> an effect known as head-of-line blocking. Queuing delays are not part of the service time, they happen on the client side, so it's important to take into account these response times on the client side as well as the server side.
