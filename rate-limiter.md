1️⃣ [**Problem Statement**]
Design a Rate Limiter that controls how many requests a client can make in a given time window.
We’ll design Token Bucket (primary) and briefly compare with Leaky Bucket.

2️⃣ [**High-Level Design (HLD): Goals**]
Prevent abuse/overload
Allow short bursts
Thread-safe
Extensible (multiple algorithms)
Works per user / API key / IP

[**Architecture (HLD)**]
        ┌───────────────┐
        │   Client      │
        └──────┬────────┘
               │
               ▼
        ┌──────────────────┐
        │  RateLimiter     │  ◄── Interface
        └──────┬───────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌──────────────┐   ┌──────────────┐
│ TokenBucket  │   │ LeakyBucket  │
└──────────────┘   └──────────────┘

[**HLD Decisions**]
Strategy Pattern → swap algorithms easily
Per-client bucket stored in a ConcurrentHashMap

3️⃣ [**UML / Class Diagram (LLD)**]

                 +---------------------+
                 |  RateLimiter        |<<interface>>
                 +---------------------+
                 | +allowRequest():boolean |
                 +----------▲----------+
                            |
        +-------------------+-------------------+
        |                                       |
+---------------------+              +---------------------+
| TokenBucketLimiter  |              | LeakyBucketLimiter  |
+---------------------+              +---------------------+
| - capacity:int      |              | - capacity:int      |
| - refillRate:double |              | - leakRate:double   |
| - tokens:double     |              | - waterLevel:double |
| - lastRefillTime:long|             | - lastLeakTime:long |
+---------------------+              +---------------------+
| +allowRequest()     |              | +allowRequest()     |
+---------------------+              +---------------------+

4️⃣ [**Algorithm Choice (Quick Interview Explanation)**]

1. [**Token bucket algorithm:**] The token bucket algorithm is widely used for rate limiting.The token bucket algorithm work as follows:
-> A token bucket is a container that has pre-defined capacity. Tokens are put in the bucket at preset rates periodically. Once the bucket is full, no more tokens are added. 
-> Each request consumes one token. When a request arrives, we check if there are enough tokens in the bucket. 
-> If there are enough tokens, we take one token out for each request, and the request goes through. If there are not enough tokens, the request is dropped.
-> The token bucket algorithm takes two parameters:
    Bucket size: the maximum number of tokens allowed in the bucket
    Refill rate: number of tokens put into the bucket every second

2. Leaking bucket algorithm: The leaking bucket algorithm is similar to the token bucket except that requests are processed at a fixed rate. It is usually implemented with a first-in-first-out (FIFO) queue. The algorithm works as follows:
-> When a request arrives, the system checks if the queue is full. If it is not full, the request is added to the queue.
-> Otherwise, the request is dropped.
-> Requests are pulled from the queue and processed at regular intervals.   
-> Leaking bucket algorithm takes the following two parameters:
    Bucket size: it is equal to the queue size. The queue holds the requests to be processed at a fixed rate.
    Outflow rate: it defines how many requests can be processed at a fixed rate, usually in seconds.   
Notes: A burst of traffic fills up the queue with old requests, and if they are not processed in time, recent requests will be rate limited.



    
