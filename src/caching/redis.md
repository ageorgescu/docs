## 🔴 Redis Deep-Dive (Data Structures, Persistence, Use-cases)

### Core Concepts

- In-memory, single-threaded, extremely fast.
- Great for **caching**, **pub/sub**, **rate-limiting**, and **distributed locks**.
- Runs commands atomically.

---

### Built-in Data Structures

| Type              | Description / Use case                             |
| ----------------- | -------------------------------------------------- |
| String            | Basic key/value (`SET`, `GET`)                     |
| Hash              | Key → map of field/value pairs (`HSET`, `HGET`)    |
| List              | Linked list (`LPUSH`, `RPUSH`, `LPOP`)             |
| Set               | Unordered unique values (`SADD`, `SMEMBERS`)       |
| Sorted Set (ZSET) | Each element has a score → leaderboards (`ZADD`)   |
| Bitmaps           | Store bits → user on/off flags                     |
| HyperLogLog       | Probabilistic count (approx. distinct count)       |
| Streams           | Time-series with IDs, consumer groups (Kafka-like) |

---

### Eviction Policies (when memory is full)

- `noeviction` – default ⇒ error on write
- `allkeys-lru` – evict least-recently-used from any key
- `allkeys-lfu` – least-frequently-used (approximate)
- `volatile-ttl` – evict keys with TTL only

---

### Persistence Options

| Mode   | Description               | Pros         | Cons                        |
| ------ | ------------------------- | ------------ | --------------------------- |
| RDB    | Snapshot on interval      | Fast restore | Data loss between snapshots |
| AOF    | Append-only log of writes | Durable      | Larger disk usage           |
| Hybrid | RDB + AOF rewriting       | balance      | complexity                  |

- Configure with `save`, `appendonly yes`

---

### Distributed Locking (Redlock)

```
SET <key> <value> NX PX <ms>
```

- NX = only if key does not exist
- PX = expiration in ms
- Value should be a UUID to verify owner on release.

---

### Rate Limiting (token bucket)

```
INCR user:counter
EXPIRE user:counter 60
```

Allows X requests per 60 seconds.

---

### Use-cases

- Cache DB query results (`SETEX`)
- Queue (LPUSH / RPOP)
- Leaderboards (`ZADD`, `ZREVRANGE`)
- Pub/Sub (`PUBLISH`, `SUBSCRIBE`)
- Session store for web apps

---

### Frequently Asked Interview Questions

- How does Redis implement locks? What is Redlock?
- When would you use Sorted Sets over Lists?
- Compare `LRU` vs `LFU` eviction strategy.
- Explain the difference between RDB and AOF persistence.
- Can Redis guarantee strong consistency? (No — single thread, but don’t use as authoritative store)

---
