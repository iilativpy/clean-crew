# Concurrency (Clean Code ch. 13 + Appendix A "Concurrency II")

Load this when the change touches threads, executors, async code, shared mutable state,
locks, connection pools, schedulers, or anything run in parallel.

## Why and when
- Concurrency decouples **what** gets done from **when** it gets done. It can improve
  throughput and structure - but only when there is waiting (I/O) to overlap.
- First find where time goes: **I/O-bound** work benefits from more threads;
  **CPU-bound** work does not beyond the number of cores.
- Myths to reject: "concurrency always improves performance", "design doesn't change",
  "the container (web/EJB/Spring) handles it for me".
- Truths: it adds overhead and code; correct concurrency is hard even for simple problems;
  its bugs are rarely repeatable; it usually requires a different design.

## Defense principles
- **SRP**: keep concurrency code separate from business code - it has its own lifecycle
  and failure modes. Concentrate it in a few classes.
- **Limit the scope of shared data**: encapsulate it; minimize the number of places that
  can update it (each one is a critical section you might forget).
- **Use copies of data**: share read-only copies, or let threads work on copies and merge
  results in one thread. Object creation is usually cheaper than locking.
- **Threads as independent as possible**: partition data so each thread works on its own
  subset with local variables (like a request handler that only uses its parameters).
- **Prefer immutability** (see also Clean Architecture: all race conditions, deadlocks and
  concurrent-update problems require mutable state).

## Know your library (Java examples; find the equivalents in your language)
- Use thread-safe collections (`ConcurrentHashMap`, `java.util.concurrent` queues) and their
  atomic composite operations.
- Use the **Executor** framework / thread pools instead of hand-made threads; use
  `Callable` + `Future` (or `CompletableFuture`) to run independent work and join results.
- Prefer **nonblocking** solutions (`AtomicInteger`, compare-and-swap) - usually cheaper
  than locks under moderate contention.
- Know what is **not** thread-safe (e.g. `SimpleDateFormat`, DB connections, plain
  `java.util` collections, servlet instance fields) and confine or wrap it.
- Useful primitives: `ReentrantLock`, `Semaphore`, `CountDownLatch`.

## Know your execution models
Terms: bound resources, mutual exclusion, starvation, deadlock, livelock.
- **Producer–Consumer**: producers and consumers coordinate through a bounded queue and
  signal "not empty" / "not full".
- **Readers–Writers**: balance reader throughput against writer starvation and stale data.
- **Dining Philosophers**: competing for multiple resources -> deadlock, livelock, poor throughput.
Most real problems are variations of these three - recognize which one you have and use the
known solution.

## Dependencies between synchronized methods
- Avoid calling more than one method on a shared object in a sequence that must be atomic.
- If you must: **client-based locking** (client locks around the whole sequence),
  **server-based locking** (server offers one method that does the whole sequence), or an
  **adapted server** (wrapper that adds the locking when you can't change the server).
- **Prefer server-based locking**: no repeated locking code in clients, clear ownership,
  and you can swap in a non-thread-safe single-thread version easily.
- "Tolerating the failure" (catch and clean up) is sloppy - a last resort.

## Keep synchronized sections small
Locks cost delays and contention. Guard every critical section, but keep each as small
as possible; don't widen them "to be safe".

## Shutdown
Graceful shutdown is hard (threads waiting forever on signals that never come, e.g. a
consumer blocked after its producer exits). Design and test shutdown **early**; expect it
to take longer than you think.

## Deadlock
Requires all four: **mutual exclusion, lock & wait, no preemption, circular wait**.
Break one:
- *Mutual exclusion*: resources that allow simultaneous use (atomics), enough resources for
  all threads, check all are free before seizing.
- *Lock & wait*: don't wait - if any resource is busy, release all and retry
  (risk: starvation, livelock).
- *Preemption*: allow a thread to ask an owner to release a resource.
- *Circular wait* (most common fix): **acquire resources in one global order**. Trade-off:
  resources may be held longer than needed.

## Testing threaded code
- Treat spurious, unrepeatable failures as **threading bugs**, never as "cosmic rays".
- **Get non-threaded code working first** - put logic in plain objects tested without threads.
- Make threaded code **pluggable** (1 thread / many threads, real or test doubles, fast/slow/
  variable doubles, configurable iterations) and **tunable** (thread counts adjustable,
  ideally at runtime).
- Run with **more threads than cores** to provoke task switching.
- Run on **all target platforms** early and often - schedulers differ.
- **Instrument** to force different interleavings ("jiggling": inject yields/sleeps, by hand
  or with tooling) so rare paths fail early.
- Write tests that can expose problems and run them frequently under varied configurations
  and load; never dismiss a failure because the next run passed.
