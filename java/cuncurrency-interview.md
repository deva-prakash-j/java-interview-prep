# 🧵 Java Concurrency – High-Level Interview Q\&A
---

## ✅ Threading and Basics

### 1. What is the difference between a thread and a process?

* **Process**: Independent execution unit with its own memory space.
* **Thread**: Lightweight unit within a process; threads share memory and resources.
* Java uses **multi-threading** to achieve concurrency within a single JVM process.

---

### 2. How does Java create and manage threads internally?

* Threads are represented by the `java.lang.Thread` class.
* Managed by the **OS kernel** but abstracted by the JVM.
* Thread management APIs (`Executors`, `ForkJoinPool`, etc.) decouple task submission from thread creation.

---

### 3. What is the lifecycle of a Java thread?

1. **New** → thread is created but not started.
2. **Runnable** → eligible to run but not necessarily running.
3. **Running** → scheduled and executing.
4. **Blocked/Waiting** → waiting on a lock or signal.
5. **Terminated** → execution finished or exception thrown.

---

### 4. What are the risks of using `Thread.sleep()` in multithreaded applications?

* Doesn’t release locks.
* Hard to control or coordinate.
* Prefer `wait/notify`, `CountDownLatch`, or scheduled executors for timing logic.

---

### 5. Why is the `start()` method preferred over calling `run()` directly?

* `start()` starts a **new thread**.
* `run()` executes the code in the **current thread**, not concurrently.

---

## 🔄 Synchronization and Locks

### 6. What are the different ways to achieve synchronization in Java?

* `synchronized` methods and blocks.
* Explicit locks (`ReentrantLock`, `ReadWriteLock`).
* Atomic variables (`AtomicInteger`, etc.).
* High-level constructs (`ExecutorService`, `Semaphore`, etc.).

---

### 7. Explain how the `synchronized` keyword works. What are its limitations?

* Enforces **mutual exclusion** by acquiring an intrinsic lock (monitor) on the object or class.
* Limitations:

  * Can't interrupt waiting threads.
  * No fairness or try-lock capabilities.
  * Only works with object-level or class-level locking.

---

### 8. What is a **deadlock**, and how can you detect and prevent it?

* A state where two or more threads are waiting indefinitely for each other’s resources.
* Detection: JVM thread dumps or tools like VisualVM.
* Prevention:

  * Lock ordering.
  * Use `tryLock()` with timeouts.
  * Avoid nested locks when possible.

---

### 9. What is a **livelock**, and how is it different from deadlock?

* Threads are **actively changing state** in response to others but make **no progress**.
* Unlike deadlock, threads are not blocked — just ineffective.

---

### 10. What is **starvation**, and what strategies prevent it?

* A thread is **denied CPU** or resources indefinitely due to thread priority or scheduling.
* Solutions:

  * Fair locks (`new ReentrantLock(true)`).
  * Avoid priority inversion.

---

### 11. How does Java handle **reentrant locks**?

* `ReentrantLock` allows a thread to acquire the **same lock multiple times** without blocking.
* Keeps a count of re-entries.
* More powerful than `synchronized`:

  * Supports timeout (`tryLock()`), interruption, and fairness.

---

### 12. What are the pros and cons of `synchronized` vs `ReentrantLock`?

| Feature         | `synchronized` | `ReentrantLock`      |
| --------------- | -------------- | -------------------- |
| Syntax          | Simpler        | Verbose              |
| Interruptible?  | No             | Yes                  |
| Timeout?        | No             | Yes                  |
| Fairness?       | No             | Yes                  |
| Reentrancy      | Yes            | Yes                  |
| Recommended for | Simpler cases  | Fine-grained control |

---

## 🧩 Concurrency Utilities

### 13. How does the `ExecutorService` framework work in Java?

* Separates **task submission** from **thread management**.
* Uses thread pools (`ThreadPoolExecutor`) to reuse threads and improve performance.
* Supports task cancellation, future results, and graceful shutdown.

---

### 14. Differences between `ExecutorService`, `ForkJoinPool`, and `CompletableFuture`?

| Feature           | ExecutorService | ForkJoinPool          | CompletableFuture     |
| ----------------- | --------------- | --------------------- | --------------------- |
| Use Case          | General-purpose | Recursive parallelism | Async pipelines       |
| Parallelism Model | Fixed/cached    | Work-stealing         | Callback chaining     |
| API               | submit/invoke   | invoke/submit         | thenApply/thenCompose |

---

### 15. How does `CountDownLatch`, `CyclicBarrier`, and `Semaphore` work?

| Utility          | Purpose                                                         |
| ---------------- | --------------------------------------------------------------- |
| `CountDownLatch` | Wait until a fixed number of operations complete. One-time use. |
| `CyclicBarrier`  | Threads wait until all reach a barrier. Reusable.               |
| `Semaphore`      | Limit concurrent access to a resource.                          |

---

### 16. What is `ThreadLocal`, and what are the risks in using it?

* Stores thread-specific variables.
* Risk: memory leaks in long-lived threads (e.g., thread pools).
* Always clean up with `.remove()` when done.

---

### 17. Difference between `BlockingQueue` and `ConcurrentLinkedQueue`?

* `BlockingQueue`: Thread-safe, supports blocking operations (used in producer-consumer).
* `ConcurrentLinkedQueue`: Non-blocking, lock-free queue for high concurrency, no blocking ops.

---

### 18. What is `CopyOnWriteArrayList`? When would you use it?

* Thread-safe list that copies the underlying array on **write**.
* Ideal for **read-heavy, write-light** scenarios.

---

### 19. Difference between `ConcurrentHashMap` and synchronized `HashMap`?

* `ConcurrentHashMap`: Fine-grained locking (segments or bucket locks); non-blocking reads.
* `Collections.synchronizedMap`: Coarse-grained lock on the entire map; slower under contention.

---

## 📦 Advanced Topics

### 20. How do atomic operations work in Java?

* Leverage **CAS (Compare-And-Swap)** via `sun.misc.Unsafe` or `VarHandle` in newer JDKs.
* Classes like `AtomicInteger`, `AtomicReference` provide lock-free thread-safe operations.

---

### 21. What is the role of the `Unsafe` class in Java concurrency?

* Provides low-level operations: memory access, CAS, fences.
* Used internally in `java.util.concurrent`.
* Not recommended for direct use—superseded by `VarHandle` in Java 9+.

---

### 22. How does Java ensure memory visibility between threads?

* Follows the **Java Memory Model (JMM)**.
* Synchronization (`synchronized`, `volatile`, `locks`) establishes **happens-before** relationships.

---

### 23. What are **false sharing** and **cache coherence**?

* **False sharing**: multiple threads modify variables on the **same cache line**, causing performance degradation.
* Avoid by padding variables or using `@Contended` (requires JVM flag `-XX:-RestrictContended`).

---

### 24. How does the `volatile` keyword work?

* Guarantees **visibility** of changes to variables across threads.
* Does **not** guarantee atomicity or mutual exclusion.

---

### 25. What's the difference between `volatile` and `synchronized`?

| Feature     | `volatile` | `synchronized`          |
| ----------- | ---------- | ----------------------- |
| Visibility  | Yes        | Yes                     |
| Atomicity   | No         | Yes                     |
| Blocking    | No         | Yes                     |
| Performance | Faster     | Slower under contention |

---

## 🕹️ Thread Pooling and Executors

### 26. Fixed thread pool vs cached thread pool?

* **Fixed**: Limited threads, blocks if all are busy.
* **Cached**: Creates new threads as needed, good for lightweight short-lived tasks.

---

### 27. How do you design a custom thread pool for IO-bound vs CPU-bound workloads?

* **IO-bound**: Use large pools (e.g., `2 * #cores`) due to blocking.
* **CPU-bound**: Use small pools (`#cores` or `#cores - 1`) to avoid context switching.

---

### 28. Tuning parameters in `ThreadPoolExecutor`?

* `corePoolSize`
* `maximumPoolSize`
* `keepAliveTime`
* `workQueue`
* `rejectedExecutionHandler`

Carefully tune based on CPU, memory, and request pattern.

---

### 29. How can thread pools cause memory leaks?

* Tasks with strong references to large objects remain in memory.
* Threads holding on to `ThreadLocal` variables.
* Not shutting down pools in applications leads to resource leaks.

---

### 30. How to monitor and gracefully shut down a thread pool?

```java
executor.shutdown();
if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

Use metrics: active threads, queue size, completed tasks, etc.

---

## ⚙️ Reactive and Asynchronous Patterns

### 31. How does Java handle async programming? Compare `CompletableFuture` and reactive frameworks.

* `CompletableFuture`: Built-in async programming with a future API (imperative).
* **Reactive** (Project Reactor, RxJava): Stream-oriented, supports backpressure and transformation.

---

### 32. Pros and cons of reactive programming?

| Pros             | Cons                         |
| ---------------- | ---------------------------- |
| Non-blocking     | Steep learning curve         |
| Backpressure     | Debugging harder             |
| Efficient for IO | Poor fit for CPU-heavy tasks |

---

### 33. What is backpressure in reactive streams?

* Mechanism to prevent overwhelming subscribers.
* Publishers must respect demand signals from subscribers (`request(n)`).

---

## 📐 Architecture and Design-Level

### 34. Thread-safe Singleton Design?

Use `enum` or double-checked locking with `volatile`.

```java
public enum Singleton {
    INSTANCE;
}
```

---

### 35. How to implement producer-consumer in Java?

* Use `BlockingQueue` with a fixed-capacity queue and worker threads.

---

### 36. Implement a rate limiter in Java?

* Token Bucket or Leaky Bucket algorithms.
* Use libraries like Bucket4j or Guava RateLimiter.

---

### 37. Ensure thread-safety in a high-throughput REST API?

* Avoid mutable shared state.
* Use stateless controllers, `@RequestScope`, concurrent collections.

---

### 38. How to detect thread contention?

* Analyze thread dumps.
* Monitor GC logs, CPU spikes.
* Use profilers: JFR, JMC, VisualVM.

---

### 39. Handle concurrency in distributed systems?

* Use distributed locks (`Redisson`, Zookeeper).
* Idempotent APIs, retry strategies.
* Eventual consistency mechanisms (e.g., Saga pattern).

---

## 📊 Monitoring and Debugging

### 40. Tools for analyzing thread behavior?

* **Java Flight Recorder (JFR)**
* **JDK Mission Control**
* **VisualVM**
* **JStack**, `kill -3` for thread dumps

---

### 41. How to analyze thread leaks?

* Look for unbounded threads in dump.
* Long-lived thread pools.
* Monitor for `OutOfMemoryError: unable to create new native thread`.

---

### 42. How can GC pauses affect concurrency?

* GC pauses **stop-the-world**; all threads halt.
* Tune GC (`G1GC`, `ZGC`) to reduce pause impact.

---

### 43. Daemon vs Non-daemon threads?

* **Daemon**: background tasks; JVM exits if only daemon threads remain.
* **Non-daemon**: keep JVM alive until they finish.

---

## 🧠 Behavioral and Scenario-Based

### 44. Describe a production incident due to concurrency?

*Tailor your real-world story to show diagnosis, resolution, and learnings.*

---

### 45. Concurrency in microservices accessing shared resources?

* Use locks sparingly.
* Prefer optimistic concurrency (e.g., ETags, version fields).
* Isolate writes, use distributed caches carefully.

---

### 46. Concurrency design patterns used?

* Producer-Consumer
* Read-Write Lock
* Thread Pool
* Future/Promise
* Circuit Breaker (resilience)

---

## 🧩 Language-Specific Enhancements

### 47. Concurrency improvements in Java 8–21?

* **Java 8**: `CompletableFuture`, lambda-based concurrency
* **Java 9**: `Flow` API (reactive streams), `VarHandle`
* **Java 19–21**: **Virtual Threads (Project Loom)**, **Structured Concurrency**

---

### 48. What is `StructuredTaskScope` in Java 19+?

* Part of **structured concurrency**.
* Manage and cancel related tasks hierarchically.
* Automatically joins child tasks when parent completes.

---

### 49. What are virtual threads?

* Lightweight threads managed by JVM, not OS.
* Introduced in **Project Loom** (Java 21).
* Scale to millions of threads with minimal memory.

---
