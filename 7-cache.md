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
