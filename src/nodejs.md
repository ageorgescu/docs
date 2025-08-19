## 🔧 Node.js Deep-Dive (Internals, Performance & Best Practices)

### Event Loop Phases (Critical)

Node.js is single-threaded at the JavaScript level, using `libuv` under the hood. The event loop moves through these phases sequentially:

1. **Timers** – runs callbacks from `setTimeout()` / `setInterval()`
2. **Pending callbacks** – low-level I/O callbacks
3. **Idle / prepare** – internal
4. **Poll** – waits for incoming I/O; executes ready I/O callbacks
5. **Check** – runs `setImmediate()` callbacks
6. **Close callbacks** – e.g. `'close'` events from sockets

**Microtasks run between phases**:
- `process.nextTick()` → priority queue **before** other microtasks
- Promises (`.then/.catch`) → microtask queue

👉 Priority: `process.nextTick()` → microtasks → event loop phases

---

### libuv Thread-Pool vs Worker Threads

| Feature      | libuv Thread-pool              | Worker Threads (JS API)        |
| ------------ | ------------------------------ | ------------------------------ |
| Default size | 4 threads                      | 1 per worker instance          |
| Used for     | `fs`, crypto, DNS, compression | Heavy CPU-bound JS             |
| Scale        | `UV_THREADPOOL_SIZE=<n>`       | Manual control (spawn workers) |

Use Worker Threads for CPU-intensive code to avoid blocking the loop.

---

### Streams & Backpressure

Efficient data processing via:

- `Readable`, `Writable`, `Duplex`, `Transform`
- Built-in backpressure handling with `.pipe()`:

```js
fs.createReadStream('bigfile').pipe(res);
```

`highWaterMark` controls internal buffer size. Use `.pause()` & `.resume()` if implementing manually.

---

### Buffer / Binary

Raw byte handling in Node:

```js
const buf = Buffer.from('Hello', 'utf8');
```

- `Buffer.allocUnsafe()` → fast but not zero-filled ➝ may expose old memory.

---

### Process / Observability

| API                               | Purpose                 |
| --------------------------------- | ----------------------- |
| `process.memoryUsage()`           | Heap and RSS statistics |
| `process.cpuUsage()`              | CPU usage per process   |
| `process.on('uncaughtException')` | Catch fatal errors      |
| `perf_hooks`                      | High-resolution timers  |

---

### Error Handling Best Practices

- Wrap async/await in `try/catch`
- Handle:
  ```js
  process.on('unhandledRejection', ...)
  ```
- For critical crashes, **log and exit** instead of zombie state.

---

### Blocking the Event Loop (Anti-pattern)

Bad:
```js
while(true) { }
```

Instead:
- Use `worker_threads`
- Offload via queues (Rabbit, Kafka)
- Use `cluster` for multi-process scaling

---

### Modules: CommonJS vs ES Modules

| Feature            | CommonJS                       | ESM                          |
| ------------------ | ------------------------------ | ---------------------------- |
| Syntax             | `require()` / `module.exports` | `import` / `export`          |
| Default Node usage | `.js`                          | `"type": "module"` or `.mjs` |
| Loading            | Synchronous                    | Asynchronous                 |

Resolution order: `.js` → `.json` → `.node`

`"exports"` field in `package.json` restricts importable entrypoints.

---

### Frequently Asked Interview Questions

**Q: Explain the phases of the Node.js event loop.**  
A: The event loop moves through six phases in order: (1) **Timers** — runs callbacks from `setTimeout` / `setInterval`, (2) **Pending Callbacks** — executes deferred I/O callbacks, (3) **Idle/Prepare** — internal, (4) **Poll** — waits for incoming I/O and executes callbacks, (5) **Check** — executes `setImmediate()` callbacks, and (6) **Close Callbacks** — like `'close'` events. Between phases, Node executes all **microtasks**, including `process.nextTick()` (higher priority) and Promise callbacks.

---

**Q: What’s the difference between `process.nextTick()` and `setImmediate()`?**  
A: `process.nextTick()` places a callback at the **front of the microtask queue**, so it executes *before* the event loop continues to the next phase. `setImmediate()` queues the callback in the **check phase**, so it runs *after* pending I/O events. Overusing `process.nextTick()` can starve the loop.

---

**Q: How do you avoid blocking the event loop?**  
A: Move CPU-heavy or synchronous work (e.g. zipping, hashing, sorting large datasets) into worker threads (`worker_threads`), child processes, or distribute the task externally (e.g., message queue). Keep I/O async and avoid infinite loops in request handlers.

---

**Q: What is backpressure and how do streams handle it?**  
A: Backpressure happens when a data **producer is faster than the consumer**, causing buffers to grow. Streams handle this by pausing data flow when the buffer is full (`highWaterMark`) and resuming when drained. Using `.pipe()` automatically manages backpressure between read and write streams.

---

**Q: Why doesn’t `fs.readFile()` block the event loop?**  
A: Although it seems synchronous, `fs.readFile()` is implemented using the libuv thread-pool. The event loop schedules the operation and continues. The actual file I/O happens in another thread, and the callback is queued back into the event loop once ready.

---

**Q: How do you increase libuv’s thread pool size?**  
A: Before any async libuv-based methods are called, set the environment variable:  
`process.env.UV_THREADPOOL_SIZE = <n>` (default is 4, max is 128). This affects built-in modules like `fs`, `crypto`, `dns`.

---
