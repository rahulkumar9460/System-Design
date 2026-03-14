# Caching
---

# 1. What is Caching

Caching is the practice of storing the result of expensive operations in **fast-access storage (usually memory)** so that future requests can be served quickly without recomputation or database queries.

Instead of repeatedly fetching or computing the same data, the system returns a previously stored result.

### Core idea

```
Cache = Trade freshness for speed
```

Caching primarily improves:

* **Latency** (faster responses)
* **Throughput** (handle more requests)
* **Scalability** (reduce backend load)

---

# 2. Why Caching is Needed

Databases are expensive resources in terms of:

* CPU
* disk I/O
* network overhead
* query execution time

Without caching:

```
Client
  │
  ▼
Application Server
  │
  ▼
Database
```

Every request hits the database.

Problems:

* high latency
* database overload
* poor scalability
* expensive infrastructure

---

With caching:

```
Client
  │
  ▼
Application Server
  │
  ▼
Cache (fast memory)
  │
  ├── HIT → return response
  │
  └── MISS
        │
        ▼
     Database
        │
        ▼
      Cache store
        │
        ▼
      Return response
```

Benefits:

* faster responses
* lower database load
* improved scalability

---

# 3. Real World Example

Example API:

```
GET /user/123
```

### Without cache

```
Request
  │
  ▼
Application
  │
  ▼
Database Query
  │
  ▼
Return Response
```

Every request hits the database.

---

### With cache

```
Request
  │
  ▼
Check Cache
  │
  ├── HIT → return cached response
  │
  └── MISS
        │
        ▼
      Query Database
        │
        ▼
      Store result in Cache
        │
        ▼
      Return response
```

Subsequent requests become extremely fast.

---

# 4. Cache Layers in Modern Systems

Caching exists at **multiple layers** in real systems.

Example DNS resolution flow:

```
Browser Cache
     │
     ▼
Operating System Cache
     │
     ▼
Router Cache
     │
     ▼
DNS Resolver Cache
     │
     ▼
Authoritative DNS Server
```

Each layer reduces traffic to the next layer.

---

Example web request path:

```
Browser Cache
      │
      ▼
CDN Cache
      │
      ▼
Load Balancer
      │
      ▼
Application Cache
      │
      ▼
Database
```

Each cache layer improves performance.

---

# 5. Types of Caches

## Browser Cache

Stores HTTP responses locally inside the browser.

Important HTTP headers:

```
Cache-Control
ETag
Expires
Last-Modified
```

---

## Application Cache

Stored in application memory or external cache.

Common examples:

* user profiles
* session data
* product information
* configuration data

---

## Distributed Cache

A centralized cache shared by many application servers.

Common systems include:

* Redis
* Memcached

Architecture:

```
App Server 1
       │
       ├────► Cache Cluster
       │
App Server 2
       │
       ├────► Cache Cluster
       │
App Server 3
```

---

# 6. Cache Read Strategy (Cache Aside)

The most common caching pattern.

Also called **Lazy Loading**.

```
Client
  │
  ▼
Application
  │
  ▼
Check Cache
  │
  ├── HIT → return
  │
  └── MISS
        │
        ▼
      Query Database
        │
        ▼
      Store in Cache
        │
        ▼
      Return response
```

Advantages:

* simple
* widely used
* flexible

---

# 7. Cache Write Strategies

Different approaches exist for writing data when caches are involved.

---

## 7.1 Write Through

Writes go to both cache and database synchronously.

```
Client
  │
  ▼
Application
  │
  ▼
Write Cache
  │
  ▼
Write Database
```

Pros:

* strong consistency
* cache always updated

Cons:

* slower writes

---

## 7.2 Write Back (Write Behind)

Write first to cache.

Database updated asynchronously.

```
Client
  │
  ▼
Application
  │
  ▼
Write Cache
  │
  ▼
Async Worker
  │
  ▼
Database
```

Pros:

* extremely fast writes

Cons:

* risk of data loss if cache crashes

---

## 7.3 Write Around

Writes bypass cache.

```
Write:
Client → Database

Read:
Client → Cache → Database
```

Pros:

* avoids caching rarely used data

Cons:

* initial read latency

---

# 8. Cache Eviction

Cache memory is limited.

When capacity is full, some data must be removed.

---

## LRU — Least Recently Used

Removes the least recently accessed item.

Data structure:

```
HashMap + Doubly Linked List
```

Structure:

```
Most Recent
   │
   ▼
[A] ⇄ [B] ⇄ [C] ⇄ [D]
                     ▲
                     │
               Least Recent
```

Operations:

```
Read → move node to head
Write → insert at head
Evict → remove tail
```

Time complexity:

```
O(1)
```

---

# 9. Distributed Cache

Large-scale systems require multiple cache nodes.

Architecture:

```
                ┌─────────────┐
                │ LoadBalancer│
                └──────┬──────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     App 1           App 2           App 3
        │              │              │
        └──────────► Cache Cluster ◄──┘
                         │
                         ▼
                     Database
```

---

# 10. Cache Sharding

Keys must be distributed across cache nodes.

Simple hashing approach:

```
node = hash(key) % N
```

Problem:

If nodes change:

```
Add node → almost all keys move
Remove node → massive redistribution
```

---

# 11. Consistent Hashing

Solution to redistribution problem.

Nodes are placed on a **hash ring**.

```
          (0)
           │
       Node A
        /   \
       /     \
   Node D   Node B
       \     /
        \   /
        Node C
```

Keys are assigned clockwise.

Example:

```
Key1 → Node B
Key2 → Node C
Key3 → Node D
```

When a node fails:

```
Keys move only to next node
```

Minimal redistribution.

---

# 12. Virtual Nodes

To balance load evenly.

Each physical node creates multiple virtual nodes.

Example:

```
Node A → A1 A2 A3
Node B → B1 B2 B3
Node C → C1 C2 C3
```

Benefits:

* better load balancing
* supports nodes with different capacity

---

# 13. Cache Stampede (Thundering Herd)

Occurs when cache expires and many requests hit the database simultaneously.

Example:

```
10,000 requests
       │
       ▼
Cache MISS
       │
       ▼
Database overload
```

This can crash the backend.

---

# Cache Failure Patterns in Distributed Systems

Caching improves system performance, reduces database load, and lowers latency.
However, poorly designed caching systems can introduce serious failure patterns that may overload backend systems.

Three major cache-related failure scenarios frequently discussed in system design are:

* Cache Penetration
* Cache Breakdown (Cache Stampede / Hot Key Expiry)
* Cache Avalanche

Understanding these problems and their mitigation strategies is essential for building scalable systems.

---

# 1. Cache Penetration

## Definition

Cache penetration occurs when requests are made for **data that does not exist in the database**.

Because the data does not exist:

* Cache lookup fails
* Database query is executed
* Database returns `null`
* Result is not cached

Future requests repeat the same process.

This leads to unnecessary database load.

---

## Request Flow

```
Client Request (user_id = 999999)
        │
        ▼
Check Cache
        │
        ▼
Cache Miss
        │
        ▼
Query Database
        │
        ▼
Data Not Found
        │
        ▼
Return Null
```

Since the result is not cached, every request follows the same path.

---

## Example Scenario

An attacker sends requests for random IDs.

```
user:999111
user:999112
user:999113
user:999114
```

Flow:

```
Requests
   │
   ▼
Cache Miss
   │
   ▼
Database Query
   │
   ▼
Database Overload
```

This can be exploited as a **denial-of-service attack** against the database.

---

## Solutions

### Cache Null Results

When the database returns `null`, cache a placeholder value.

Example:

```
Key: user:999999
Value: NULL
TTL: 5 minutes
```

Future requests will hit the cache instead of the database.

---

### Bloom Filter

A Bloom Filter is a probabilistic data structure that helps determine whether a key **may exist**.

Flow:

```
Incoming Request
        │
        ▼
Bloom Filter Check
        │
   ┌────┴─────┐
   ▼          ▼
Not Present   Possibly Present
   │          │
   ▼          ▼
Reject      Query Cache
```

If the Bloom Filter says the key definitely does not exist, the request is rejected early.

---

### Input Validation

Validate request parameters before querying cache or database.

Examples:

* Reject invalid user IDs
* Reject malformed queries

---

# 2. Cache Breakdown (Cache Stampede / Hot Key Expiry)

## Definition

Cache breakdown occurs when a **highly popular key (hot key)** expires and many requests attempt to rebuild the cache simultaneously.

Since the key is requested very frequently, all incoming requests miss the cache and hit the database.

---

## Request Flow

```
Hot Key: product:123
TTL Expired
        │
        ▼
Thousands of Requests
        │
        ▼
Cache Miss
        │
        ▼
All Requests Query Database
        │
        ▼
Database Overload
```

---

## Example Scenario

Consider a trending product page.

```
Key: product:123
Traffic: 100,000 requests per second
```

When the cache expires:

```
Cache Entry Expired
        │
        ▼
100,000 Requests
        │
        ▼
100,000 Database Queries
```

This can easily overwhelm the database.

---

## Solutions

### Request Coalescing (Single Flight)

Allow only **one request** to fetch data from the database.

Other requests wait until the first request populates the cache.

```
Cache Miss
     │
     ▼
First Request → Query Database
Other Requests → Wait
     │
     ▼
Cache Updated
     │
     ▼
All Requests Served from Cache
```

---

### Distributed Lock / Mutex

Use a lock to ensure only one process rebuilds the cache.

```
Cache Miss
     │
     ▼
Acquire Lock
     │
     ▼
Fetch Data from Database
     │
     ▼
Update Cache
     │
     ▼
Release Lock
```

Other requests wait until the cache is updated.

---

### Never Expire Hot Keys

For extremely popular keys, disable TTL.

```
product:123 → no expiration
```

The application updates the cache manually when the underlying data changes.

---

### Background Cache Refresh

Refresh cache entries before they expire.

```
Cache Entry Near Expiry
        │
        ▼
Background Refresh
        │
        ▼
Cache Updated
```

This prevents sudden expiration spikes.

---

# 3. Cache Avalanche

## Definition

Cache avalanche occurs when **many cache entries expire at the same time**, causing a massive spike in database queries.

---

## Example Scenario

Cache entries have identical expiration times.

```
product1 TTL = 10 minutes
product2 TTL = 10 minutes
product3 TTL = 10 minutes
product4 TTL = 10 minutes
```

When they expire simultaneously:

```
Mass Expiration
       │
       ▼
Thousands of Cache Misses
       │
       ▼
Database Queries Spike
       │
       ▼
Database Crash
```

---

## Real System Scenario

If cache is populated during system startup:

```
Cache populated at startup
All keys have same TTL
```

Later:

```
All cache entries expire together
```

This leads to sudden database overload.

---

## Solutions

### Randomized TTL

Add randomness to expiration time.

Instead of:

```
TTL = 10 minutes
```

Use:

```
TTL = 10 minutes ± random(0–5 minutes)
```

Example:

```
product1 TTL = 9 minutes
product2 TTL = 11 minutes
product3 TTL = 13 minutes
product4 TTL = 10 minutes
```

This spreads expiration events over time.

---

### Multi-Level Cache

Use layered caching.

Architecture:

```
Client
  │
  ▼
Application Server
  │
  ▼
Local Cache (in-memory)
  │
  ▼
Distributed Cache
  │
  ▼
Database
```

Even if the distributed cache fails, the local cache absorbs traffic.

---

### Rate Limiting

Limit requests during cache failures.

```
Cache Failure
      │
      ▼
Rate Limiter
      │
      ▼
Controlled Database Traffic
```

---

### Circuit Breaker

Temporarily stop database calls if the backend is overloaded.

```
Database Overloaded
        │
        ▼
Circuit Breaker Activated
        │
        ▼
Serve Fallback Response
```

---

# Summary

| Problem           | Cause                           | Result                | Common Solutions                 |
| ----------------- | ------------------------------- | --------------------- | -------------------------------- |
| Cache Penetration | Requests for nonexistent data   | Continuous DB queries | Cache null values, Bloom filters |
| Cache Breakdown   | Hot key expires                 | Sudden DB spike       | Request coalescing, locks        |
| Cache Avalanche   | Many keys expire simultaneously | Massive DB overload   | Random TTL, multi-level cache    |

---

# Key Takeaways

* Caching significantly improves performance but introduces new failure scenarios.
* Systems must guard against penetration, stampede, and avalanche patterns.
* Defensive techniques such as Bloom filters, request coalescing, randomized TTLs, and multi-level caching are essential in large-scale systems.

---

# 14. Request Coalescing

Solution to stampede.

Only one request fetches from database.

Others wait.

Example:

```
Request 1
   │
   ▼
Cache MISS
   │
   ▼
Start DB query
   │
   ▼
Store Promise in inflightMap
```

Other requests:

```
Request 2..1000
   │
   ▼
Check inflightMap
   │
   ▼
Wait for same promise
```

Result:

```
1000 requests → 1 DB query
```

---

# 15. Hot Keys

A **hot key** is a key that receives extremely high traffic.

Examples:

```
/homepage
/trending
/popular-products
```

Problem:

```
Millions of requests
      │
      ▼
Single cache node overloaded
```

Solutions:

* replicate the key
* rate limiting
* load balancing
* cache pre-warming

---

# 16. Stale-While-Revalidate

Serve stale data while refreshing cache in background.

Two TTL values are used.

Example:

```
Fresh TTL = 10 minutes
Stale TTL = 30 minutes
```

Timeline:

```
|------Fresh------|------Stale------|Expired|
```

Behavior:

```
Fresh → serve cache
Stale → serve cache + refresh async
Expired → DB fetch
```

Flow:

```
Request
  │
  ▼
Check Cache
  │
  ├── Fresh → return
  │
  ├── Stale → return stale + refresh
  │
  └── Expired → DB query
```

Advantages:

* avoids latency spikes
* prevents stampede

---

# 17. Probabilistic Early Expiration

Another strategy to avoid cache stampede.

Instead of refreshing exactly at TTL expiration, refresh **randomly before TTL**.

Normal TTL behavior:

```
|------------TTL------------|
                          Refresh spike
```

Probabilistic refresh behavior:

```
|------------TTL------------|
       Refresh
            Refresh
                 Refresh
```

Logic:

```
if random() < probability(age_of_cache):
    refresh_cache()
```

Benefits:

* spreads refresh load over time
* prevents mass expiration events

---

# 18. Full Production Cache Architecture

Large-scale systems combine multiple caching strategies.

Example architecture:

```
Client
   │
   ▼
CDN Cache
   │
   ▼
Application Cache
   │
   ▼
Distributed Cache
   │
   ▼
Database
```

Protection mechanisms:

```
Cache Aside
Request Coalescing
Stale While Revalidate
Probabilistic Early Expiration
Consistent Hashing
Hot Key Mitigation
LRU Eviction
```

---

# 19. Key Tradeoffs in Caching

Advantages:

```
Low latency
High scalability
Reduced database load
```

Trade-offs:

```
Data staleness
Consistency challenges
Operational complexity
```

---

# 20. Summary

Caching is a fundamental technique used to scale modern distributed systems.

Key ideas include:

* cache read/write patterns
* eviction strategies
* distributed cache sharding
* consistent hashing
* handling cache stampede
* handling hot keys
* advanced refresh techniques

A robust caching architecture protects the database while delivering fast responses to users.

---

# End of Caching Guide
