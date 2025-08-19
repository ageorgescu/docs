## 🐇 RabbitMQ Deep-Dive (AMQP, Exchanges, Patterns, Interview Tips)

### Message Broker Basics

RabbitMQ is a **message broker** based on the AMQP protocol. Producers publish messages; RabbitMQ routes them to queues; consumers pull/ack messages.

- **Decouples** producers and consumers.
- Ideal for **task queues, RPC, asynchronous workflows**.

---

### Exchange Types & Routing

| Exchange Type | Routing Behavior                                  | Use Cases                   |
| ------------- | ------------------------------------------------- | --------------------------- |
| direct        | Routes by exact routing key                       | Simple task queues          |
| topic         | Routing key supports wildcards (`*`, `#`)         | Logs, grouped subscriptions |
| fanout        | Broadcasts message to all bound queues            | Pub/Sub, notifications      |
| headers       | Matches on message headers instead of routing key | Custom attribute routing    |

Diagram:

```
 Producer → Exchange → Queue(s) → Consumer(s)
```

A *binding* connects exchange → queue with a routing key/pattern.

---

### Guaranteed Delivery

- Message ack = `basic.ack`
- If not acked → message requeued
  - Can lead to **redelivery / duplicates** → implement **idempotency**
- **Durable Queue + Persistent Message** → survives broker restart
- **Dead Letter Exchange (DLX)** → handle message failures / TTL expiry

---

### QoS & Prefetch Count

- Controls consumer throughput: `basic.qos(prefetch=<n>)`
- Limits how many unacknowledged messages a consumer can process → **prevents overwhelming slow consumers**.

---

### Clustering & HA

| Mechanism               | Description                                          |
| ----------------------- | ---------------------------------------------------- |
| Classic Queue Mirroring | Entire queue replicated to other nodes (HA)          |
| Quorum Queues           | Raft-based replication → recommended (RabbitMQ 3.8+) |

---

### Important Interview Concepts

- **At-most-once vs at-least-once** → RabbitMQ is at-least-once by default
- **Consumer local ordering only** → global ordering not guaranteed if multiple queues
- **RPC over RabbitMQ** → use `reply_to` + `correlation_id`
- **When to use RabbitMQ vs Kafka?** → RabbitMQ for low-latency tasking & routing; Kafka for streaming & replay

---

### Frequently Asked Interview Questions

**Q: How does RabbitMQ guarantee delivery?**  
A: RabbitMQ guarantees at-least-once delivery by requiring consumers to send an acknowledgement (`basic.ack`). Until a message is acked, it stays “unacked.” If the consumer disconnects or crashes, RabbitMQ requeues the message for another consumer. To survive broker crashes, the queue must be declared as *durable* and the message sent as *persistent*.

---

**Q: What is the difference between fanout and topic exchange?**  
A: A **fanout exchange** routes the message to *all* bound queues, ignoring the routing key (broadcast). A **topic exchange** uses a routing key with wildcards — `*` for one word and `#` for multiple — to deliver messages to matching queues. Fanout is used for pub-sub where routing rules are simple; topic exchanges are used for more granular routing.

---

**Q: If a consumer crashes before acking a message, what happens?**  
A: The message remains unacknowledged. When the broker detects the consumer’s connection has closed, the unacked message is returned to the queue so another consumer can process it. This provides fault tolerance and ensures messages are not lost.

---

**Q: How can you scale consumers to handle 10k messages per second?**  
A: Use multiple consumer instances in parallel, each with its own connection/channel → classic “competing consumers” pattern. Tune `prefetch` (QoS) so each consumer only preloads a reasonable number of messages and prevent one slow consumer from blocking. You can also use multiple queues bound to the same exchange to parallelize even further.

---

**Q: What is a Dead Letter Exchange (DLX) and how is it used?**  
A: A DLX is an exchange where messages are routed if they expire (due to TTL), are rejected (`basic.reject`), or reach their delivery attempt limit. This allows failed messages to be decoupled from the normal queue for later inspection, alerting, or retry logic — important for building resilient systems.


---
