## 🍃 MongoDB Deep-Dive (NoSQL, Architecture, Interview Points)

### Core Design

- **Document-oriented NoSQL database**
- Stores JSON-like documents internally as **BSON**
- **Schema-flexible** → collections don’t enforce rigid schema
- Great for **rapid iteration**, **unstructured or semi-structured data**

---

### Architecture

| Concept     | Description                                                 |
| ----------- | ----------------------------------------------------------- |
| Collection  | Similar to a table                                          |
| Document    | Record (JSON / BSON)                                        |
| Primary Key | `_id` (unique per document)                                 |
| Replica Set | 1 PRIMARY + N secondaries → automatic failover              |
| Sharding    | Horizontal partitioning across shards using a **shard key** |
| WiredTiger  | Default storage engine (document-level locking, journaling) |

---

### CAP Theorem in MongoDB

| Mode         | Behavior                         |
| ------------ | -------------------------------- |
| CP (default) | Consistent & partition-tolerant  |
| Trade-off    | Availability might be sacrificed |

---

### CRUD & Querying

- Documents can be nested (subdocuments, arrays)
- Query language resembles JSON
  - e.g. `{ age: { $gt: 25 } }`
- Indexes:
  - Single-field, compound, TTL, Text, Geospatial
  - Covered queries → all fields in index = no doc lookup

---

### Sharding

- Distribute data across many **shards** → scale horizontally
- Requires **config servers** & **mongos** query router
- Pick shard key wisely:
  - **Hashed** → even distribution (good for writes, bad for range queries)
  - **Ranged** → good for range queries, risk of hot shards

---

### Replication (Replica Sets)

```
Client → PRIMARY (reads/writes)
       ↙         ↘
   Secondary   Secondary
```

- Secondary nodes replicate oplog
- Failover → automatic leader election
- Reads can be routed to secondaries with `readPreference=secondary` → eventual consistency

---

### Aggregation Framework

- Equivalent to SQL’s `GROUP BY` & complex transformations:
  - `$match`, `$group`, `$sort`, `$lookup` (join with another collection)
- Runs stages like a pipeline.

Example pipeline:
```json
[ { $match: {} }, { $group: { _id: "$status", total:{$sum:1}}} ]
```

---

### Pros & Cons

| Pros                    | Cons                                                                            |
| ----------------------- | ------------------------------------------------------------------------------- |
| Schema-flexible         | Complex joins not as efficient                                                  |
| Easy horizontal scaling | Harder transactional guarantees                                                 |
| Powerful indexing       | ACID support only per-document (unless using multi-document txn on replica set) |

---

### Frequently Asked Interview Questions

**Q: When would you choose MongoDB over PostgreSQL?**  
A: When the data structure is dynamic or semi-structured (e.g. JSON documents that frequently change shape), when development speed and iteration are more important than rigid schema design, or when you expect high write throughput and need to horizontally shard easily. MongoDB is a good fit for content-management, log data, product catalogs, etc., where strict relations and ACID transactions aren’t critical.

---

**Q: Explain the difference between sharding and replication.**  
A: Replication is about **redundancy and high availability**: data is copied from a primary node to one or more secondaries. If the primary fails, a secondary can be promoted. Sharding is about **horizontal scaling**: splitting data across multiple servers (shards) based on a shard key so each shard only stores part of the dataset. They solve different problems — replication keeps copies, sharding spreads them.

---

**Q: What is a shard key and why is it important?**  
A: A shard key determines how documents are distributed across shards in a cluster. It’s critical because it impacts performance and scalability — it must be chosen so that requests are evenly distributed (to avoid “hot” shards). MongoDB uses the shard key to route queries; you can’t change it later without re-sharding your data.

---

**Q: How does replica-set failover work?**  
A: In a replica set, one node is elected as **primary**, while others are **secondaries** replicating data from the primary. If the primary goes down or becomes unreachable, the secondaries automatically hold an election. The node with the highest priority and most up-to-date oplog becomes the new primary. Clients automatically reconnect and resume operations — allowing high availability.

---

**Q: How does the aggregation pipeline differ from simple queries?**  
A: The aggregation pipeline processes documents through multiple transformation stages (like `$match`, `$group`, `$lookup`, `$sort`) to build more complex results similar to SQL GROUP BY, JOIN, HAVING. Simple queries (`find`) can only filter and return documents — aggregation pipelines can reshape, group, or enrich documents and are optimized internally using pipeline operators.

---

---
