## 📬 Kafka vs RabbitMQ – Comparison Cheat Sheet

| Attribute           | Kafka (Distributed Log)                                               | RabbitMQ (Message Broker / Queue)                    |
| ------------------- | --------------------------------------------------------------------- | ---------------------------------------------------- |
| Core Concept        | Append-only distributed log, persisted to disk                        | Smart message broker with queues                     |
| Message Ordering    | Ordered **within a partition**                                        | Ordered **within a queue**                           |
| Persistence Model   | Always persistent (commit log on disk)                                | In-memory by default, can persist to disk optionally |
| Scalability         | Scale by adding **partitions** and brokers                            | Scale by adding **queues** and consumers             |
| Consumer Model      | Pull-based; consumers fetch at their own pace                         | Push/pull; broker can push messages to consumers     |
| Consumer Groups     | Yes → exactly one consumer per partition in group                     | Competing consumers on queues                        |
| Delivery Guarantees | At-most/at-least/exactly once (with transactions)                     | At-most/at-least once                                |
| Message Retention   | Retention-based (time/size); can replay old messages                  | Queue-based; messages deleted once acknowledged      |
| Ideal For           | Event streaming, log aggregation, event sourcing, analytics           | Task distribution, RPC, short-lived jobs             |
| Latency             | Slightly higher end-to-end latency                                    | Very low (good for real-time tasks)                  |
| Protocol            | Custom TCP (binary), Kafka protocol                                   | AMQP 0-9-1, STOMP, MQTT etc.                         |
| Features            | High throughput, batch consumption, stream processing (Kafka Streams) | Routing, priorities, TTL, headers, dead-lettering    |

### 💡 Summary

- **Choose Kafka when** you need **high-throughput event streams**, replayable logs, or to power **event-driven architectures / analytics pipelines**.
- **Choose RabbitMQ when** you need a reliable **task/work queue**, **smart routing** (direct/topic/fanout), or **low-latency immediate delivery** with strong delivery semantics.
