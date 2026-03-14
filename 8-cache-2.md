# Request Coalescing (Single-Flight) in C++ and Production Mitigation Strategies

This document explains how to implement **request coalescing** (also called **single-flight**) in C++ and describes **production-grade mitigation strategies** used by large-scale systems.

The goal is to prevent **cache stampedes** where thousands of requests simultaneously hit the database after a cache miss.

---

# 1. What is Request Coalescing?

Request coalescing ensures that when **many requests arrive for the same key**, only **one request fetches the data from the database**, while the others **wait for the result**.

Example scenario:

```
1000 requests → key: product:123
```

Without coalescing:

```
1000 requests
      │
      ▼
1000 DB queries
```

With coalescing:

```
1000 requests
      │
      ▼
1 DB query
      │
      ▼
All requests receive the same result
```

---

# 2. High-Level Architecture

```
Client Requests
      │
      ▼
Application Server
      │
      ▼
Check Cache
      │
   ┌──┴─────┐
   ▼        ▼
Hit       Miss
 │          │
 ▼          ▼
Return   Check InFlight Map
            │
        ┌───┴─────┐
        ▼         ▼
     Exists     Not Exists
        │          │
        ▼          ▼
     Wait       Fetch DB
                  │
                  ▼
             Update Cache
                  │
                  ▼
            Notify Waiters
```

---

# 3. Core Idea

Maintain a map of **in-flight requests**.

```
inFlightMap[key] → promise/future
```

When the first request starts fetching data:

```
key → future
```

Subsequent requests **wait on that future** instead of hitting the database.

---

# 4. C++ Pseudocode Implementation

```cpp
#include <unordered_map>
#include <string>
#include <future>
#include <mutex>

using namespace std;

class CacheService {

private:

    unordered_map<string, string> cache;
    unordered_map<string, shared_future<string>> inFlight;

    mutex m;

public:

    string fetchFromDatabase(string key) {
        // simulate database call
        return "data_for_" + key;
    }

    string get(string key) {

        // Step 1: Check cache
        {
            lock_guard<mutex> lock(m);

            if (cache.find(key) != cache.end()) {
                return cache[key];
            }

            // If request already in progress
            if (inFlight.find(key) != inFlight.end()) {
                auto future = inFlight[key];
                return future.get();
            }
        }

        // Step 2: create promise
        promise<string> promiseObj;
        shared_future<string> future = promiseObj.get_future().share();

        {
            lock_guard<mutex> lock(m);
            inFlight[key] = future;
        }

        // Step 3: fetch from DB
        string value = fetchFromDatabase(key);

        {
            lock_guard<mutex> lock(m);

            cache[key] = value;
            promiseObj.set_value(value);

            inFlight.erase(key);
        }

        return value;
    }
};
```

---

# 5. Execution Example

Scenario:

```
1000 requests for product:123
```

Timeline:

```
Request 1 → cache miss → start DB query
Request 2 → sees inFlight → waits
Request 3 → waits
Request 4 → waits
...
Request 1000 → waits
```

After DB query finishes:

```
cache["product:123"] = data
```

All waiting requests return instantly.

---

# 6. Benefits

Request coalescing prevents **database overload during cache misses**.

Benefits include:

* Prevents thundering herd problem
* Reduces duplicate database queries
* Improves system stability
* Reduces latency spikes

---

# 7. Production Mitigation Strategies

Large-scale companies implement multiple techniques to prevent cache failures.

---

# Strategy 1: Multi-Level Caching

Systems often use **layered caching**.

```
Client
  │
  ▼
Application
  │
  ▼
Local Memory Cache
  │
  ▼
Distributed Cache
  │
  ▼
Database
```

Local cache absorbs most requests before they reach distributed cache.

This approach is widely used in high-scale backend services.

---

# Strategy 2: Request Coalescing

Backend services collapse identical requests.

```
10000 identical requests
        │
        ▼
1 DB query
```

This protects backend databases during traffic spikes.

---

# Strategy 3: Proactive Cache Warming

Popular data is **preloaded into cache before traffic arrives**.

Example:

```
Trending videos
Popular products
Top user feeds
```

Cache warming prevents cold-start latency.

---

# Strategy 4: Randomized TTL

Avoid simultaneous cache expiration.

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
key1 → 9 min
key2 → 11 min
key3 → 13 min
key4 → 10 min
```

This spreads cache refresh load over time.

---

# Strategy 5: Hot Key Replication

Highly requested keys are replicated across multiple cache nodes.

Example:

```
hotkey_1
hotkey_2
hotkey_3
```

Traffic is distributed among replicas.

---

# Strategy 6: Bloom Filters for Cache Penetration

Bloom filters are used to filter requests for **nonexistent keys**.

```
Request
  │
  ▼
Bloom Filter
  │
 ┌┴─────────┐
 ▼          ▼
Reject   Query Cache
```

This prevents unnecessary database queries.

---

# Strategy 7: Background Cache Refresh

Instead of waiting for TTL expiry, systems refresh data in the background.

```
Cache nearing expiration
        │
        ▼
Background refresh job
        │
        ▼
Cache updated
```

Users always receive fresh data without cache misses.

---

# Strategy 8: Rate Limiting and Circuit Breakers

When backend systems become overloaded:

```
Traffic Spike
     │
     ▼
Rate Limiter
     │
     ▼
Controlled DB traffic
```

If the database becomes unhealthy:

```
Circuit breaker activated
```

Requests return fallback responses.

---

# 8. Real Production Usage

Large-scale systems such as those used by companies like Netflix and Uber implement combinations of:

* request coalescing
* multi-layer caching
* cache warming
* randomized TTL
* Bloom filters
* hot key replication

These mechanisms allow their platforms to handle **millions of requests per second** while protecting backend databases.

---

# 9. Key Takeaways

* Cache stampedes can overload databases during traffic spikes.
* Request coalescing ensures only one database query occurs per cache miss.
* Production systems combine multiple mitigation strategies to maintain stability.
* Proper cache design is critical for building scalable distributed systems.
