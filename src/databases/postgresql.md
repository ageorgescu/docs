## 🐘 PostgreSQL Deep-Dive (Indexes, Transactions, Locking)

### MVCC (Multi-Version Concurrency Control)

- Every row version has `xmin` / `xmax` system columns.
- **Readers never block writers** (and vice versa).
- Old versions stored until `VACUUM` cleans them.
- Allows **snapshot isolation**.

---

### Isolation Levels

| Level           | Behavior                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------ |
| READ COMMITTED  | Default. Each statement sees latest committed data.                                        |
| REPEATABLE READ | Same snapshot for entire transaction → avoids non-repeatable reads.                        |
| SERIALIZABLE    | Strictest. Emulates serial order; may abort with serialization failure (detect conflicts). |

---

### Index Types

| Type   | Use case                                   | Notes                          |
| ------ | ------------------------------------------ | ------------------------------ |
| B-Tree | Default. Range, equality searches          | Good all-purpose.              |
| Hash   | Equality only                              | Rarely used (used internally). |
| GIN    | Array, full-text search (`tsvector`)       | Inverted index.                |
| GiST   | Geospatial, ranges                         | PostGIS, fuzzy search.         |
| BRIN   | Huge data, naturally ordered (time series) | Lightweight.                   |

👉 Always run `EXPLAIN ANALYZE` to inspect query plans.

---

### Transactions

- Start with `BEGIN` … `COMMIT` / `ROLLBACK`
- Nested transactions supported with `SAVEPOINT`

```sql
BEGIN;
UPDATE ...;
SAVEPOINT s1;
UPDATE ...;
ROLLBACK TO s1;
COMMIT;
```

---

### Locking

| Lock             | Description        | Blocking?                     |
| ---------------- | ------------------ | ----------------------------- |
| Row Exclusive    | `UPDATE`, `DELETE` | Blocks concurrent row updates |
| Share            | `SELECT FOR SHARE` | Allows concurrent readers     |
| Access Share     | `SELECT`           | Blocks ALTER table            |
| Access Exclusive | `ALTER TABLE`      | Blocks everything             |

---

### JSON vs JSONB

| Attribute  | JSON        | JSONB                      |
| ---------- | ----------- | -------------------------- |
| Stored     | Text, as-is | Binary format              |
| Modifiable | No          | Yes (`                     |  | `, `-`, etc.) |
| Indexable  | No          | Yes (`GIN` index on JSONB) |

---

### Partitioning

- Range Partitioning (e.g. by date)
- List Partitioning (e.g. by category)
- Hash Partitioning (even distribution)

Improves large-table performance and maintenance (pruning, vacuum).

---

### Frequent Interview Questions

**Q: Explain MVCC and how PostgreSQL avoids read locks.**  
A: MVCC (Multi-Version Concurrency Control) keeps multiple versions of a row in the database. When a row is updated, a new version is created with a new transaction ID, but old transactions see the earlier snapshot. This allows readers to read consistent data without blocking writers, and writers can write without blocking readers. Old versions are eventually cleaned up by the `VACUUM` process.

---

**Q: What is the difference between SERIALIZABLE and REPEATABLE READ isolation?**  
A: `REPEATABLE READ` gives a consistent snapshot for the entire transaction; but phantom reads (new rows) are possible. `SERIALIZABLE` is stricter: it ensures that concurrently executed transactions behave as if they were executed one after another. It can abort transactions with a serialization failure error if it detects a conflict, whereas `REPEATABLE READ` would allow it to proceed.

---

**Q: Pros and cons of JSONB vs JSON columns?**  
A: `JSONB` stores JSON in a binary format, allows indexing (GIN) and efficient lookups, and strips duplicate keys when stored. It’s great for queries and performance. `JSON` stores raw text, keeps formatting, and requires re-parsing for every query. Use `JSONB` when you will query inside the JSON data and `JSON` when you only need to store arbitrary JSON in original form.

---

**Q: When would you choose a GIN index over a B-Tree index?**  
A: GIN (Generalized Inverted Index) is best for indexing collections/arrays, full-text search, or JSONB fields where each element inside the field should be searchable independently. B-Tree is better for general equality/range queries on scalar columns like integers or timestamps.

---

**Q: What causes deadlocks? How can you detect and resolve them?**  
A: Deadlocks occur when two transactions hold locks that the other needs, forming a cycle (e.g., Tx1 locks row A and waits for row B, while Tx2 locks B and waits for row A). PostgreSQL detects deadlocks by examining its lock graphs and terminates one transaction with an error to break the cycle. To avoid them, ensure a consistent locking order, keep transactions short, and avoid user interaction inside transactions.

---

