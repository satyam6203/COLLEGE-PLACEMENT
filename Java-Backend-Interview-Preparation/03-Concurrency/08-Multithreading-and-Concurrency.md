# Chapter 8: Multithreading and Concurrency

**Chapter goal:** Answer JVM concurrency, synchronization, memory-model, executor, and `CompletableFuture` questions — and avoid the race conditions, deadlocks, and thread-pool mistakes seen in production Spring Boot services.

**Topics covered:** Process vs thread · thread lifecycle · thread creation · `start` vs `run` · `sleep`, `wait`, `park`, `yield` · `notify`/`notifyAll` · `synchronized` · `volatile` · Java Memory Model and happens-before · race conditions · `ReentrantLock` · `ReadWriteLock` · `StampedLock` · atomics and CAS · deadlock, livelock, starvation · `ThreadLocal` · executors and thread pools · `Future` and `CompletableFuture` · `ForkJoinPool` · concurrent coding exercises.

**Cross-references:** Concurrent collections are in [Chapter 7](../02-Collections/07-Collections-Framework.md). JVM stacks, safepoints, and thread memory are in [Chapter 6](../01-Core-Java/06-JVM-Memory-and-Garbage-Collection.md). Transaction concurrency is in [Chapter 12](../04-Spring/12-Transactions-and-Scheduling.md).

---

## Part A — Threads and lifecycle

### Question 1: What is the difference between a process and a thread?

**1. Interview-ready answer**

A process is an OS-level unit of resources: it has its own memory space, file descriptors, and at least one thread. A thread is a unit of execution inside a process; threads share heap, metaspace, and file handles but have their own stack, program counter, and thread-local state. Java platform threads map to OS threads; Java 21+ virtual threads are JVM-managed and scheduled onto a small set of carrier threads.

**2. Detailed explanation**

| Aspect | Process | Thread |
|---|---|---|
| Memory | Separate address space | Shared heap, separate stack |
| Creation cost | High | Lower |
| Communication | IPC, sockets, shared files | Shared objects with synchronization |
| Failure isolation | Strong | One thread can corrupt shared state |
| Scheduling | OS scheduler | OS scheduler for platform threads |

A Java application normally runs in one process with many threads: request threads, GC threads, JIT compiler threads, and JVM internal threads.

**3. Internal working**

A platform thread's Java stack stores frames, while shared heap objects require visibility control via `synchronized`, `volatile`, locks, or atomics. Virtual threads still use the same Java memory model; they only change scheduling and resource cost. Blocking a virtual thread usually unmounts it from its carrier, unlike blocking a platform thread.

**4. Practical Java/Spring Boot example**

```java
public class ThreadVsProcessDemo {

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> System.out.println(Thread.currentThread().getName() + " working");

        Thread worker = new Thread(task, "worker-1");
        worker.start();
        worker.join();

        System.out.println("main done");
    }
}
```

Typical output:

```text
worker-1 working
main done
```

**5. Real-world use case**

A Spring Boot service handles each HTTP request on a platform thread (or a virtual thread in newer setups), while one shared connection pool and cache live in the process heap.

**6. Interview follow-up questions**

- **Do threads share stack memory?** No.
- **Are virtual threads always faster?** No; they help with blocking I/O concurrency, not CPU-bound work.
- **Can threads of one process communicate without IPC?** Yes, through shared memory with proper synchronization.
- **Does each Java thread map to an OS thread?** Platform threads do; virtual threads do not.
- **Why are too many platform threads dangerous?** Native stack memory and scheduler overhead.

**7. Common mistakes**

- Assuming threads have separate heaps.
- Creating unbounded threads instead of using an executor.
- Treating virtual threads as a replacement for synchronization.
- Counting only application threads and ignoring JVM threads.

**8. Interview difficulty**

**Beginner**

---

### Question 2: What are the thread lifecycle states in Java?

**1. Interview-ready answer**

`Thread.State` has six states: `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, and `TERMINATED`. Java does not expose a separate `RUNNING` state — a running thread is reported as `RUNNABLE`. `BLOCKED` means waiting for a monitor; `WAITING`/`TIMED_WAITING` mean waiting because of `wait`, `join`, `park`, or timed sleeps.

**2. Detailed explanation**

```text
NEW
 |
start()
 v
RUNNABLE <----> BLOCKED    waiting for intrinsic lock
 |
 | wait() / join() / park()
 v
WAITING / TIMED_WAITING
 |
 notify / timeout / interrupt
 v
RUNNABLE
 |
run() completes
 v
TERMINATED
```

| State | Meaning |
|---|---|
| `NEW` | Thread object created, `start` not called |
| `RUNNABLE` | Ready to run or currently running |
| `BLOCKED` | Waiting to enter a synchronized block/method |
| `WAITING` | Waiting indefinitely for another thread's action |
| `TIMED_WAITING` | Waiting with a timeout |
| `TERMINATED` | `run` finished or threw |

**3. Internal working**

Thread state is a JVM-level abstraction, not a complete OS scheduler state. A `RUNNABLE` Java thread may be parked by the OS but still appears `RUNNABLE` in a thread dump if it is not blocked on a Java monitor or wait condition.

**4. Practical Java/Spring Boot example**

```java
public class ThreadStateDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            try {
                Thread.sleep(200);
            } catch (InterruptedException exception) {
                Thread.currentThread().interrupt();
            }
        }, "state-worker");

        System.out.println(worker.getState());
        worker.start();
        Thread.sleep(50);
        System.out.println(worker.getState());
        worker.join();
        System.out.println(worker.getState());
    }
}
```

Typical output:

```text
NEW
TIMED_WAITING
TERMINATED
```

**5. Real-world use case**

A stuck Spring endpoint thread appears `WAITING` on a database pool or `BLOCKED` on a lock; a thread dump maps that state to the exact wait reason.

**6. Interview follow-up questions**

- **Is there a RUNNING state?** No.
- **What state waits on `synchronized`?** `BLOCKED`.
- **What state does `Thread.sleep` produce?** `TIMED_WAITING`.
- **Can a thread go from BLOCKED to WAITING directly?** Not directly through normal transitions.
- **Does `getState` show CPU usage?** No; use profilers or OS metrics.

**7. Common mistakes**

- Assuming `RUNNABLE` always means consuming CPU.
- Calling `start` twice — throws `IllegalThreadStateException`.
- Ignoring interrupted status.
- Confusing `BLOCKED` with `WAITING`.

**8. Interview difficulty**

**Beginner**

---

### Question 3: What is the difference between `Thread`, `Runnable`, `Callable`, and `start` vs `run`?

**1. Interview-ready answer**

`Thread` is the execution unit; `Runnable` is a task with `run()` and no result; `Callable<T>` is a task with `call()` that can return a value and throw checked exceptions. Calling `start()` creates a new thread and schedules `run`; calling `run()` directly executes on the current thread. In application code, submit `Runnable`/`Callable` to an `ExecutorService` instead of creating threads manually.

**2. Detailed explanation**

| Type | Result | Checked exception | Execution |
|---|---|---|---|
| `Thread` | No | No | `start` launches a thread |
| `Runnable` | No | No | Run by a thread or executor |
| `Callable<T>` | Yes | Yes | Submitted to executor, returns `Future<T>` |
| `ExecutorService` | Futures | Yes | Manages workers and lifecycle |

**3. Internal working**

`start()` asks the JVM to create a native thread and eventually invokes `run()` on it. Calling `run()` directly is a normal method call, so no parallelism occurs. A `Callable` submitted to an executor is wrapped in a `FutureTask`, which stores the result or exception.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ThreadCreationDemo {

    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(() -> System.out.println("Runnable on " + Thread.currentThread().getName()));
        thread.start();

        Callable<String> callable = () -> "prices-loaded";
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Future<String> future = executor.submit(callable);
        System.out.println(future.get());

        executor.shutdown();
    }
}
```

Typical output:

```text
Runnable on Thread-0
prices-loaded
```

**5. Real-world use case**

A Spring `@Async` method is effectively a `Callable`/`Runnable` submitted to a configured `TaskExecutor`, not a manually created `Thread`.

**6. Interview follow-up questions**

- **Why not extend Thread?** It mixes task logic with execution machinery; Runnable/Callable separates them.
- **Can you call start twice?** No.
- **Can Runnable throw checked exceptions?** No; wrap or use Callable.
- **What does Future.get do?** Blocks until completion and rethrows failures wrapped in `ExecutionException`.
- **Why use an executor?** Resource control, reuse, queueing, lifecycle management.

**7. Common mistakes**

- Calling `run()` and expecting concurrency.
- Creating a new thread per request.
- Forgetting to shut down executors.
- Swallowing exceptions inside tasks.

**8. Interview difficulty**

**Beginner**

---

### Question 4: Compare `sleep`, `wait`, `join`, `park`, and `yield`

**1. Interview-ready answer**

`sleep` pauses the current thread without releasing locks. `wait` releases the monitor and must be called while holding it; `join` waits for another thread to finish; `park` blocks via `LockSupport` and is the primitive behind many locks; `yield` is only a scheduler hint. `sleep`, `wait`, and `join` can be timed, and all blocking calls should handle interruption properly.

**2. Detailed explanation**

| Method | Releases lock? | Requires monitor? | Purpose |
|---|---|---|---|
| `Thread.sleep(ms)` | No | No | Pause execution |
| `Object.wait()` | Yes | Yes | Wait for condition/notification |
| `Thread.join()` | No | No | Wait for another thread |
| `LockSupport.park()` | No | No | Block a thread primitive |
| `Thread.yield()` | No | No | Hint to scheduler |

`wait` is a method on `Object` because every object has an intrinsic monitor. `sleep` is a method on `Thread` because it acts on the current thread.

**3. Internal working**

`wait` moves the thread to the object's wait set and releases the monitor. After notification, it must reacquire the monitor before returning. `park` tracks a permit; a thread can be unparked by another thread or interrupted.

**4. Practical Java/Spring Boot example**

```java
public class WaitVsSleepDemo {

    private final Object lock = new Object();

    void waitForCondition() throws InterruptedException {
        synchronized (lock) {
            lock.wait(1000);
        }
    }

    void pause() throws InterruptedException {
        Thread.sleep(1000);
    }

    public static void main(String[] args) throws InterruptedException {
        WaitVsSleepDemo demo = new WaitVsSleepDemo();
        demo.pause();
        demo.waitForCondition();
        System.out.println("done");
    }
}
```

`pause` holds no lock; `waitForCondition` releases `lock` while waiting.

**5. Real-world use case**

A retry worker uses timed `sleep` between attempts, while a connection-pool waiter uses condition waiting that releases the pool lock.

**6. Interview follow-up questions**

- **Does sleep release synchronized lock?** No.
- **Why call wait in a loop?** To handle spurious wakeups and changed conditions.
- **What happens on interrupt during wait?** InterruptedException is thrown and the interrupt flag is cleared.
- **Is yield reliable?** No; it is a hint only.
- **Which is preferred in modern locks?** park/Condition rather than wait/notify.

**7. Common mistakes**

- Calling `wait` outside synchronized — `IllegalMonitorStateException`.
- Holding locks while sleeping.
- Ignoring `InterruptedException`.
- Using `yield` to solve contention.

**8. Interview difficulty**

**Intermediate**

---

### Question 5: Why should you use `notifyAll` instead of `notify`?

**1. Interview-ready answer**

`notify` wakes one arbitrary waiter; if it wakes a thread whose condition is still false, the right thread may remain asleep. `notifyAll` wakes all waiters, and each rechecks its condition in a loop. Spurious wakeups are allowed, so `wait` must always be inside a `while` loop rather than an `if`.

**2. Detailed explanation**

A wait set can contain threads waiting for different conditions on the same monitor — for example producers waiting for space and consumers waiting for items. `notify` may wake the wrong category. `notifyAll` is safer and usually cheap because awakened threads immediately reacquire the lock and go back to waiting if their condition is false.

**3. Internal working**

After `notifyAll`, all waiters move to the entry set and compete for the monitor. Only one holds the monitor at a time. Each thread re-evaluates its predicate before proceeding.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class WaitNotifyDemo {

    private final Queue<Integer> queue = new ArrayDeque<>();
    private final int capacity = 2;

    public void put(int value) throws InterruptedException {
        synchronized (this) {
            while (queue.size() == capacity) {
                wait();
            }
            queue.add(value);
            notifyAll();
        }
    }

    public int take() throws InterruptedException {
        synchronized (this) {
            while (queue.isEmpty()) {
                wait();
            }
            int value = queue.remove();
            notifyAll();
            return value;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        WaitNotifyDemo demo = new WaitNotifyDemo();
        demo.put(1);

        Thread consumer = new Thread(() -> {
            try {
                System.out.println(demo.take());
            } catch (InterruptedException exception) {
                Thread.currentThread().interrupt();
            }
        });

        consumer.start();
        consumer.join();
    }
}
```

Output:

```text
1
```

**5. Real-world use case**

A custom blocking queue notifies both producers and consumers because either group may become able to proceed after a state change.

**6. Interview follow-up questions**

- **Are spurious wakeups real?** Yes; the JVM may wake a thread without notify.
- **Why while, not if?** The condition may change before the thread reacquires the lock.
- **Is notifyAll always better?** Usually safer; optimize only with proof.
- **Can wait release nested locks?** No, only the monitor it was called on.
- **What replaces wait/notify?** `ReentrantLock` with `Condition` or blocking queues.

**7. Common mistakes**

- Using `if` around wait.
- Calling notify outside synchronized.
- Assuming notify wakes a specific thread.
- Holding a lock while doing slow I/O after notification.

**8. Interview difficulty**

**Intermediate**

---

---

## Part B — Synchronization and the memory model

### Question 6: How does `synchronized` work internally?

**1. Interview-ready answer**

`synchronized` uses an object's intrinsic monitor. A `synchronized` method locks `this` (or the `Class` object for static methods); a synchronized block locks the object inside the parentheses. Only one thread can hold a monitor at a time, entry and exit create happens-before edges, and the lock is reentrant so the same thread can acquire it repeatedly.

**2. Detailed explanation**

Usage forms:

```java
public synchronized void increment() {
    count++;
}

public void add(int value) {
    synchronized (this) {
        count += value;
    }
}

public static synchronized void globalUpdate() {
}
```

Monitor rules:

- A thread must own the monitor before entering the block.
- Other threads block at entry (`BLOCKED` state).
- Exiting releases the monitor even on exception.
- Monitors are reentrant: nested synchronized on the same object succeeds.
- A `null` lock object throws NPE.

**3. Internal working**

The object header's mark word stores lock state. HotSpot uses lightweight locking for uncontended cases and inflates to an OS-level monitor under contention, where waiting threads may park. Biased locking was disabled and later removed from modern HotSpot, so do not rely on it. Monitor acquisition and release also flush/invalidate visibility through the JMM.

**4. Practical Java/Spring Boot example**

```java
public final class SynchronizedCounter {

    private int count;

    public synchronized void increment() {
        count++;
    }

    public int snapshot() {
        synchronized (this) {
            return count;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedCounter counter = new SynchronizedCounter();

        Thread worker = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        worker.start();
        worker.join();

        System.out.println(counter.snapshot());
    }
}
```

Output:

```text
1000
```

**5. Real-world use case**

A singleton in-memory metrics collector protects its counters with synchronized methods until contention becomes high enough to justify `LongAdder`.

**6. Interview follow-up questions**

- **Which object does a synchronized instance method lock?** `this`.
- **What does static synchronized lock?** The `Class` object.
- **Is it reentrant?** Yes.
- **Does it prevent visibility problems?** Yes, via happens-before.
- **Can you use a string literal as a lock?** Technically yes, but it is dangerous due to interning.

**7. Common mistakes**

- Locking on mutable fields.
- Synchronizing only the read but not the write.
- Holding a lock during remote calls.
- Using `synchronized` on high-contention hot paths without measuring.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: What does `volatile` guarantee and what does it not guarantee?

**1. Interview-ready answer**

`volatile` guarantees visibility and ordering: a write to a volatile field happens-before every subsequent read of that field, and volatile accesses act as memory barriers. It does not make compound operations atomic — `count++` on a volatile int is still read-modify-write and can lose updates. Use `volatile` for status flags and one-time publication, or atomics/locks for compound state.

**2. Detailed explanation**

Guarantees:

- Visibility: writes are seen by other threads.
- Ordering: normal writes before a volatile write cannot be reordered after it.
- 64-bit `long`/`double` volatile accesses are atomic.

Not guaranteed:

- Atomicity of `++`, `+=`, check-then-act.
- Mutual exclusion.
- Atomicity across multiple variables.
- Safe publication of complex mutable state by itself unless paired with a volatile ready flag.

**3. Internal working**

The JIT inserts acquire/release barriers around volatile accesses. On x86, the cost is often small for reads but larger for writes; on weaker architectures, explicit barriers are needed. The JMM defines behavior, not specific CPU instructions.

**4. Practical Java/Spring Boot example**

```java
public class OrderWorker implements Runnable {

    private volatile boolean running = true;

    public void stop() {
        running = false;
    }

    @Override
    public void run() {
        while (running) {
            processBatch();
        }
    }

    private void processBatch() {
        try {
            Thread.sleep(10);
        } catch (InterruptedException exception) {
            Thread.currentThread().interrupt();
            running = false;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        OrderWorker worker = new OrderWorker();
        Thread thread = new Thread(worker, "order-worker");

        thread.start();
        Thread.sleep(50);
        worker.stop();
        thread.join();

        System.out.println("stopped");
    }
}
```

Output:

```text
stopped
```

**5. Real-world use case**

A graceful-shutdown flag in a background poller is a good `volatile` use case: one writer, many readers, no compound update.

**6. Interview follow-up questions**

- **Is volatile int++ thread-safe?** No.
- **Does volatile replace synchronization?** No.
- **Does volatile make an object deeply immutable?** No.
- **Can volatile stop an infinite loop?** Yes, if the loop reads the field each iteration.
- **Why prefer AtomicBoolean for stop flags?** It also supports compare-and-set and clear semantics.

**7. Common mistakes**

- Assuming visibility means atomicity.
- Using volatile for counters under load.
- Expecting volatile arrays to make elements volatile.
- Using it to protect multiple related fields.

**8. Interview difficulty**

**Intermediate**

---

### Question 8: Explain the Java Memory Model and happens-before

**1. Interview-ready answer**

The Java Memory Model defines when one thread's writes are visible to another. Happens-before is the key relation: if action A happens-before action B, A's effects are visible to B. Rules include program order, monitor unlock/lock, volatile write/read, `Thread.start`, `Thread.join`, and transitivity. Without a happens-before edge, the JVM may reorder operations or cache values in registers.

**2. Detailed explanation**

Common happens-before rules:

| Rule | Meaning |
|---|---|
| Program order | Actions in one thread are ordered |
| Monitor | Unlock happens-before subsequent lock on same monitor |
| Volatile | Volatile write happens-before subsequent volatile read |
| Thread start | `start()` happens-before actions in the started thread |
| Thread join | Actions in a thread happen-before `join()` returns |
| Transitivity | If A hb B and B hb C, then A hb C |

**3. Internal working**

Compilers, JITs, and CPUs reorder instructions as long as single-thread semantics are preserved. Memory barriers and synchronization prevent reorderings that would violate JMM guarantees. This is why unsynchronized code can observe stale or surprising values without being “broken” at the hardware level.

**4. Practical Java/Spring Boot example**

```java
public class SafePublicationDemo {

    static final class ConfigHolder {

        int timeout;
        volatile boolean ready;

        void publish(int timeout) {
            this.timeout = timeout;
            this.ready = true;
        }

        int read() {
            if (ready) {
                return timeout;
            }
            return -1;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ConfigHolder holder = new ConfigHolder();

        Thread writer = new Thread(() -> holder.publish(3000));
        Thread reader = new Thread(() -> System.out.println(holder.read()));

        writer.start();
        writer.join();
        reader.start();
        reader.join();
    }
}
```

Output:

```text
3000
```

The volatile write to `ready` makes `timeout = 3000` visible to the reader.

**5. Real-world use case**

A config refresh thread builds a new immutable config object and publishes it through one volatile field so request threads never see a half-built object.

**6. Interview follow-up questions**

- **Does synchronized create happens-before?** Yes, unlock → subsequent lock.
- **Does Thread.join create happens-before?** Yes.
- **What is safe publication?** Making an object visible in a fully constructed state.
- **Can final fields help?** Yes, final field semantics support safe publication.
- **Does volatile guarantee atomic multi-field updates?** No.

**7. Common mistakes**

- Assuming `System.out.println` synchronizes program state.
- Sharing mutable DTOs between threads without publication.
- Using `Thread.sleep` as synchronization.
- Relying on x86 behavior instead of JMM guarantees.

**8. Interview difficulty**

**Advanced**

---

### Question 9: What is a race condition and how do you fix it?

**1. Interview-ready answer**

A race condition occurs when correctness depends on the timing of threads accessing shared mutable state. The classic `count++` is three operations — read, add, write — so two threads can read the same value and both write the same result. Fix it with atomic variables, locks, confinement, or immutable data, depending on the operation.

**2. Detailed explanation**

Unsafe patterns:

- `count++` from multiple threads.
- `if (!map.containsKey(k)) map.put(k, v)` check-then-act.
- Lazy singleton without safe publication.
- Multiple threads mutating the same `ArrayList`.

Fixes:

| Fix | Best for |
|---|---|
| `AtomicInteger` / `AtomicReference` | Single-variable atomic updates |
| `synchronized` / `ReentrantLock` | Multi-step critical sections |
| `ConcurrentHashMap.computeIfAbsent` | Atomic map updates |
| Thread confinement | No sharing at all |
| Immutable objects | No mutation |

**3. Internal working**

`count++` compiles roughly to `getfield`, `iadd`, `putfield`. A context switch between these steps loses updates. Atomics use CAS loops; locks serialize entry; confinement avoids shared state.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class RaceConditionDemo {

    static final class Counter {

        int unsafe;
        final AtomicInteger safe = new AtomicInteger();

        void incrementUnsafe() {
            unsafe++;
        }

        void incrementSafe() {
            safe.incrementAndGet();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        Thread first = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) {
                counter.incrementUnsafe();
                counter.incrementSafe();
            }
        });

        Thread second = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) {
                counter.incrementUnsafe();
                counter.incrementSafe();
            }
        });

        first.start();
        second.start();
        first.join();
        second.join();

        System.out.println("unsafe=" + counter.unsafe);
        System.out.println("safe=" + counter.safe.get());
    }
}
```

Typical output:

```text
unsafe=137842
safe=200000
```

The unsafe value varies between runs.

**5. Real-world use case**

Two concurrent requests both see “inventory available” and decrement stock, overselling the product. Use an atomic DB update or lock/version strategy, not just an in-memory check.

**6. Interview follow-up questions**

- **Is volatile enough for count++?** No.
- **Does ConcurrentHashMap make compound logic safe?** Only its atomic methods like compute/merge.
- **How do you test race conditions?** Stress tests, jcstress, thread sanitizers, repeated runs.
- **What is linearizability?** Each operation appears to take effect atomically at one point.
- **Why is thread confinement safest?** No shared mutable state means no race.

**7. Common mistakes**

- Fixing races with sleeps.
- Logging as synchronization.
- Synchronizing only the writer.
- Assuming single CPU means no races.

**8. Interview difficulty**

**Beginner**

---

### Question 10: When would you use `ReentrantLock` instead of `synchronized`?

**1. Interview-ready answer**

Use `synchronized` by default for simple mutual exclusion. Use `ReentrantLock` when you need try-lock timeouts, interruptible lock acquisition, fairness policy, multiple `Condition` objects, or explicit lock inspection. Always unlock in a `finally` block, because a lock is not released automatically on exception.

**2. Detailed explanation**

| Feature | `synchronized` | `ReentrantLock` |
|---|---|---|
| Acquisition timeout | No | `tryLock(timeout)` |
| Interruptible acquisition | No | `lockInterruptibly()` |
| Fairness | No | Optional constructor flag |
| Multiple conditions | One monitor queue | Multiple `Condition`s |
| Automatic release | Yes | No, use finally |
| JVM-level optimization | Strong | Also optimized |

**3. Internal working**

`ReentrantLock` uses AbstractQueuedSynchronizer (AQS) with a CLH-style queue and CAS state changes. Fair locks queue strictly; nonfair locks allow barging, which usually improves throughput. Conditions use `await`/`signal` and require the lock to be held.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class PaymentSlotGuard {

    private final ReentrantLock lock = new ReentrantLock(true);

    public boolean tryProcess(String paymentId) {
        boolean acquired = false;

        try {
            acquired = lock.tryLock(1, TimeUnit.SECONDS);
            if (!acquired) {
                return false;
            }
            System.out.println("processing " + paymentId);
            return true;
        } catch (InterruptedException exception) {
            Thread.currentThread().interrupt();
            return false;
        } finally {
            if (acquired) {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        PaymentSlotGuard guard = new PaymentSlotGuard();
        System.out.println(guard.tryProcess("PAY-1"));
    }
}
```

Output:

```text
processing PAY-1
true
```

**5. Real-world use case**

A payment processor uses `tryLock` with a timeout so a request fails fast instead of waiting indefinitely behind a stuck critical section.

**6. Interview follow-up questions**

- **Is ReentrantLock reentrant?** Yes.
- **Does fairness guarantee no starvation?** It reduces it but does not guarantee scheduling.
- **Can you pass a lock between methods?** Yes, but unlock must happen in the owner thread.
- **What happens if you forget unlock?** The lock is never released.
- **Is synchronized slower?** Not necessarily; modern JVM optimizes it well.

**7. Common mistakes**

- Missing finally.
- Using fair locks without measuring — they are slower.
- Using ReentrantLock when synchronized is enough.
- Calling condition methods without holding the lock.

**8. Interview difficulty**

**Intermediate**

---

---

## Part C — Advanced locking and thread coordination

### Question 11: Compare `ReentrantReadWriteLock` and `StampedLock`

**1. Interview-ready answer**

A `ReentrantReadWriteLock` allows many concurrent readers or one exclusive writer, which helps read-heavy caches. A `StampedLock` adds optimistic reads: read without taking a lock, then validate a stamp to detect a concurrent write. StampedLock is not reentrant, does not support conditions, and requires careful validation, but it can outperform read-write locks under heavy read loads.

**2. Detailed explanation**

| Feature | ReentrantReadWriteLock | StampedLock |
|---|---|---|
| Readers | Many | Many |
| Writers | One | One |
| Optimistic read | No | Yes |
| Reentrant | Yes | No |
| Conditions | Yes | No |
| Upgrade read → write | No | via `tryConvertToWriteLock` |
| Interruptible reads | No | Yes |

**3. Internal working**

`ReentrantReadWriteLock` encodes reader count and writer state in one integer. Writers block new readers and wait for existing readers to finish. `StampedLock` returns a stamp representing lock state; an optimistic reader copies state, reads fields, then calls `validate(stamp)`. If validation fails, retry with a read lock.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class RouteCache {

    private final Map<String, String> routes = new HashMap<>();
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public String get(String serviceName) {
        lock.readLock().lock();
        try {
            return routes.get(serviceName);
        } finally {
            lock.readLock().unlock();
        }
    }

    public void put(String serviceName, String url) {
        lock.writeLock().lock();
        try {
            routes.put(serviceName, url);
        } finally {
            lock.writeLock().unlock();
        }
    }

    public static void main(String[] args) {
        RouteCache cache = new RouteCache();
        cache.put("orders", "http://orders-service");
        System.out.println(cache.get("orders"));
    }
}
```

Output:

```text
http://orders-service
```

**5. Real-world use case**

A service-discovery cache is read on every request but refreshed occasionally; a read-write lock allows parallel lookups while refresh holds the write lock.

**6. Interview follow-up questions**

- **Can you upgrade read lock to write lock?** Not with ReentrantReadWriteLock; release read first.
- **Can you downgrade write to read?** Yes, by acquiring read while still holding write, then releasing write.
- **Is StampedLock reentrant?** No.
- **Why validate optimistic reads?** A writer may have changed state during the read.
- **When is read-write lock worse?** When writes are frequent or critical sections are tiny.

**7. Common mistakes**

- Holding read lock while calling write logic.
- Forgetting validation with StampedLock.
- Using StampedLock where conditions are needed.
- Assuming many readers are always faster.

**8. Interview difficulty**

**Advanced**

---

### Question 12: How do atomic variables and CAS work?

**1. Interview-ready answer**

Atomic classes such as `AtomicInteger` and `AtomicReference` use compare-and-set, a CPU-level operation that updates a value only if it still matches the expected value. CAS loops retry on contention, avoiding locks for simple state changes. `LongAdder` improves on `AtomicLong` for high-contention counters by spreading updates across cells and summing at read time.

**2. Detailed explanation**

CAS steps:

1. Read current value.
2. Compute new value.
3. CAS expected → new.
4. If another thread changed it, retry.

| Class | Use |
|---|---|
| `AtomicInteger` | Counters, IDs, state machine ints |
| `AtomicBoolean` | Flags with compare-and-set |
| `AtomicReference<T>` | Atomic object reference updates |
| `AtomicStampedReference` | ABA-sensitive updates |
| `LongAdder` | High-throughput accumulating counters |
| `LongAccumulator` | Custom high-contention accumulation |

**3. Internal working**

CAS maps to atomic CPU instructions under the JMM. It is lock-free at the API level but can still suffer retries. ABA occurs when a value changes A → B → A; CAS sees A and succeeds even though state changed. Stamped references attach a version to detect it.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.LongAdder;

public class AtomicDemo {

    static final AtomicInteger retries = new AtomicInteger();
    static final AtomicReference<String> leader = new AtomicReference<>("none");
    static final LongAdder processedOrders = new LongAdder();

    static int incrementWithCas() {
        while (true) {
            int current = retries.get();
            if (retries.compareAndSet(current, current + 1)) {
                return current + 1;
            }
        }
    }

    public static void main(String[] args) {
        System.out.println(incrementWithCas());
        leader.compareAndSet("none", "node-1");
        processedOrders.increment();
        processedOrders.increment();

        System.out.println(leader.get());
        System.out.println(processedOrders.sum());
    }
}
```

Output:

```text
1
node-1
2
```

**5. Real-world use case**

A metrics layer counts processed events with `LongAdder` because thousands of threads increment it, and exact intermediate values are not needed.

**6. Interview follow-up questions**

- **Is CAS always faster than locks?** No; retries waste CPU under high contention.
- **What is ABA?** A value returns to its old value between read and CAS.
- **Why LongAdder instead of AtomicLong?** It reduces contention by using multiple cells.
- **Is LongAdder exact under concurrency?** The sum is not a linearizable snapshot during updates.
- **Can atomics replace DB transactions?** No.

**7. Common mistakes**

- Using AtomicLong for extreme contention.
- Assuming CAS means no contention cost.
- Using atomics for multi-field invariants.
- Ignoring ABA in lock-free stacks.

**8. Interview difficulty**

**Intermediate**

---

### Question 13: What are deadlock, livelock, and starvation?

**1. Interview-ready answer**

Deadlock is a cycle of threads each holding a resource the other needs. Livelock is when threads keep changing state but make no progress. Starvation is when a thread never gets access to a resource because others always win. Prevent deadlock by acquiring locks in a global order, using timeouts, and minimizing nested locks.

**2. Detailed explanation**

Deadlock needs four conditions:

1. Mutual exclusion.
2. Hold and wait.
3. No preemption.
4. Circular wait.

Breaking any one prevents deadlock. In Java, the most practical fix is consistent lock ordering and `tryLock` timeouts. A thread dump shows `Found one Java-level deadlock` with lock ownership and waiting chains.

**3. Internal working**

Two threads acquiring `lockA → lockB` and `lockB → lockA` can each hold one lock and block on the other. JVM does not automatically detect or resolve this. `jcmd <pid> Thread.print` or `jstack <pid>` exposes the cycle.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class DeadlockDemo {

    private final ReentrantLock inventoryLock = new ReentrantLock();
    private final ReentrantLock paymentLock = new ReentrantLock();

    public boolean reserveSafe(boolean paymentFirst) {
        ReentrantLock first = paymentFirst ? paymentLock : inventoryLock;
        ReentrantLock second = paymentFirst ? inventoryLock : paymentLock;

        boolean firstHeld = false;
        boolean secondHeld = false;

        try {
            firstHeld = first.tryLock(1, TimeUnit.SECONDS);
            if (!firstHeld) {
                return false;
            }
            secondHeld = second.tryLock(1, TimeUnit.SECONDS);
            if (!secondHeld) {
                return false;
            }
            System.out.println("reserved");
            return true;
        } catch (InterruptedException exception) {
            Thread.currentThread().interrupt();
            return false;
        } finally {
            if (secondHeld) {
                second.unlock();
            }
            if (firstHeld) {
                first.unlock();
            }
        }
    }

    public static void main(String[] args) {
        DeadlockDemo demo = new DeadlockDemo();

        Thread one = new Thread(() -> demo.reserveSafe(false));
        Thread two = new Thread(() -> demo.reserveSafe(false));

        one.start();
        two.start();
    }
}
```

Both threads acquire locks in the same order, so no cycle forms.

**5. Real-world use case**

An order service locks inventory then payment. If another flow locks payment then inventory, deadlocks appear under concurrent checkout.

**6. Interview follow-up questions**

- **Can synchronized deadlock?** Yes.
- **How do you detect deadlock in production?** Thread dumps and JVM deadlock detection.
- **Does fair locking prevent deadlock?** No, it reduces starvation.
- **What is livelock example?** Two threads repeatedly politely yielding to each other.
- **Why timeouts help?** They break hold-and-wait cycles.

**7. Common mistakes**

- Nested locks in inconsistent order.
- Calling external code while holding a lock.
- Retrying endlessly without backoff.
- Assuming deadlocks only happen with many threads.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: How should `ThreadLocal` be used in backend services?

**1. Interview-ready answer**

`ThreadLocal` gives each thread its own copy of a value, useful for request context such as tenant ID, trace ID, or user principal. In pooled-thread servers it must always be cleared in a `finally` block or filter, otherwise the next request on the same thread sees stale data and objects remain GC-rooted.

**2. Detailed explanation**

Each `Thread` has a `ThreadLocalMap` keyed by weak references to `ThreadLocal` objects. Values are strongly referenced by the map. With a fixed thread pool, static ThreadLocals can retain large objects indefinitely if not removed.

Use cases:

- Request-scoped context without passing parameters everywhere.
- `SimpleDateFormat` per thread (though java.time is preferred).
- `ThreadLocalRandom` for per-thread random generators.

**3. Internal working**

`initialValue()` supplies a default on first `get`. `set` stores a value in the current thread's map. `remove` deletes the entry. `InheritableThreadLocal` copies values to child threads, but executor threads are created long before tasks arrive, so it is unreliable for request context. Structured task scopes/context propagation libraries are used for modern async contexts.

**4. Practical Java/Spring Boot example**

```java
public final class RequestContext {

    private static final ThreadLocal<String> TENANT_ID = new ThreadLocal<>();

    private RequestContext() {
    }

    public static void setTenantId(String tenantId) {
        TENANT_ID.set(tenantId);
    }

    public static String getTenantId() {
        return TENANT_ID.get();
    }

    public static void clear() {
        TENANT_ID.remove();
    }
}
```

A servlet filter should call `clear()` in a finally block after `chain.doFilter`.

**5. Real-world use case**

A multi-tenant Spring service stores tenant ID in a ThreadLocal so repositories can add tenant predicates automatically.

**6. Interview follow-up questions**

- **Does ThreadLocal solve concurrency?** It avoids sharing, not synchronizes.
- **Why remove in finally?** Pooled threads outlive requests.
- **Are keys weak?** Yes, but values are strong; leaks still happen.
- **Does it work across async threads?** Not by default.
- **Is InheritableThreadLocal enough for executors?** No.

**7. Common mistakes**

- Forgetting `remove`.
- Storing large request objects.
- Expecting child tasks to inherit context automatically.
- Using ThreadLocal as a global cache.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: How does the Executor framework work?

**1. Interview-ready answer**

`Executor` separates task submission from execution. `ExecutorService` adds lifecycle control and `Future` results. A `ThreadPoolExecutor` is configured by core size, max size, keep-alive time, work queue, thread factory, and rejection policy. Prefer an explicitly configured pool over `Executors` factory methods, which can create unbounded queues.

**2. Detailed explanation**

Execution flow:

1. Submit a `Runnable`/`Callable`.
2. If fewer than core threads are running, create a worker.
3. Otherwise queue the task.
4. If the queue is full and threads < maximum, create more workers.
5. If still full, apply the rejection policy.

Rejection policies: `AbortPolicy` (default), `CallerRunsPolicy`, `DiscardPolicy`, `DiscardOldestPolicy`.

**3. Internal working**

Workers are `Thread` objects wrapped in `Worker` nodes in a `HashSet`. The queue is a `BlockingQueue`. `shutdown` stops accepting new tasks and drains queued tasks; `shutdownNow` attempts interruption and returns pending tasks.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ExecutorDemo {

    public static void main(String[] args) {
        ExecutorService executor = new ThreadPoolExecutor(
                4,
                8,
                60,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(100),
                new ThreadPoolExecutor.AbortPolicy());

        executor.submit(() -> System.out.println("processing order"));
        executor.shutdown();
    }
}
```

Output:

```text
processing order
```

Spring Boot equivalent configuration uses `ThreadPoolTaskExecutor` with core/max pool size and queue capacity.

**5. Real-world use case**

A Spring `@Async` email sender uses a bounded executor so a slow mail server cannot cause unbounded queuing and memory growth.

**6. Interview follow-up questions**

- **What is the difference between core and max pool size?** Core is the steady state; max is the burst ceiling.
- **What happens when queue is unbounded?** Max pool size may never be reached.
- **Which rejection policy backpressures the caller?** CallerRunsPolicy.
- **How do you shut down gracefully?** `shutdown`, await termination, then `shutdownNow` if needed.
- **Why not Executors.newCachedThreadPool?** Unbounded thread creation.

**7. Common mistakes**

- Using unbounded `LinkedBlockingQueue`.
- Never shutting down executors.
- Assuming @Async uses the default executor safely.
- Setting pool size without measuring I/O wait.

**8. Interview difficulty**

**Intermediate**

---

---

## Part D — Executors, async composition, and coding

### Question 16: How do you size a thread pool in a Spring Boot service?

**1. Interview-ready answer**

For CPU-bound work, start near the number of CPU cores. For I/O-bound work, use `threads = cores × (1 + waitTime / computeTime)` and validate with load tests. In Spring Boot, configure a bounded `ThreadPoolTaskExecutor` explicitly, set queue capacity, thread-name prefix, and graceful shutdown, and avoid unbounded defaults.

**2. Detailed explanation**

Examples:

- CPU-bound: 8 cores → around 8 threads.
- I/O-bound: 8 cores, 90% waiting, 10% computing → 8 × (1 + 9) = 80 threads.
- Blocking external calls: isolate pools per dependency so one slow API cannot starve all async work.
- Virtual threads (Java 21+) allow many blocking tasks, but shared resources like DB pools still need limits.

**3. Internal working**

Too few threads underutilize CPU; too many increase context switching, native stack memory, and contention on shared pools. A bounded queue applies backpressure. A bounded maximum pool prevents thread explosion. Metrics should include queue depth, active threads, rejection count, and task duration.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean("orderExecutor")
    public Executor orderExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(16);
        executor.setMaxPoolSize(32);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("order-async-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        executor.initialize();
        return executor;
    }
}
```

Use with `@Async("orderExecutor")` on a method in another Spring bean.

**5. Real-world use case**

An order confirmation async task calls an email API with 500 ms latency. The pool is sized from measured wait/compute ratio and isolated from payment webhook processing.

**6. Interview follow-up questions**

- **What happens with an unbounded queue?** Max pool size is often never reached.
- **Which rejection policy applies backpressure?** CallerRunsPolicy.
- **Why separate pools?** Bulkhead isolation.
- **Does @Async work on same-class calls?** No; Spring proxies do not intercept self-invocation.
- **What metrics matter?** Active count, queue size, completed task time, rejections.

**7. Common mistakes**

- Copying pool sizes from another service.
- Using default SimpleAsyncTaskExecutor in production.
- Sharing one pool for CPU and blocking tasks.
- Ignoring DB connection pool limits.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: Compare `Future` and `CompletableFuture`

**1. Interview-ready answer**

`Future` represents a result that may not be ready yet, but it is limited: `get` blocks, cancellation is weak, and there is no composition. `CompletableFuture` supports non-blocking callbacks, chaining, combining, exception recovery, and explicit executor selection. Prefer `CompletableFuture` for asynchronous workflows in modern Java.

**2. Detailed explanation**

| Feature | `Future` | `CompletableFuture` |
|---|---|---|
| Blocking get | Yes | Yes, plus non-blocking callbacks |
| Chaining | No | `thenApply`, `thenCompose`, `thenAccept` |
| Combining | No | `thenCombine`, `allOf`, `anyOf` |
| Exception recovery | No | `exceptionally`, `handle` |
| Manual completion | No | `complete`, `completeExceptionally` |
| Async executor choice | No | Yes |

**3. Internal working**

`CompletableFuture` stores a result or exception and a stack of dependent completions. When the future completes, it triggers dependents synchronously or on the supplied executor. Methods ending in `Async` schedule work on an executor; others may run in the completing or calling thread.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class FutureComparisonDemo {

    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Future<String> oldStyle = executor.submit(() -> "order-loaded");
        System.out.println(oldStyle.get());

        CompletableFuture<String> modern = CompletableFuture
                .supplyAsync(() -> "price-loaded", executor)
                .thenApply(value -> value.toUpperCase());

        modern.exceptionally(error -> "fallback")
                .thenAccept(System.out::println)
                .get();

        executor.shutdown();
    }
}
```

Output:

```text
order-loaded
PRICE-LOADED
```

**5. Real-world use case**

A product detail page loads price, inventory, and reviews concurrently. `CompletableFuture.allOf` waits for all three before building the response DTO.

**6. Interview follow-up questions**

- **Does CompletableFuture always run asynchronously?** No; non-Async stages may run in the completing thread.
- **What is the default async executor?** ForkJoinPool.commonPool unless supplied.
- **How do you avoid blocking in callbacks?** Use Async variants or avoid get/join in stages.
- **What exception does join throw?** `CompletionException`.
- **How do you cancel?** `cancel`, but underlying tasks may keep running.

**7. Common mistakes**

- Calling `get()` inside async callbacks.
- Using commonPool for blocking I/O.
- Ignoring exceptions until `get`.
- Confusing `thenApply` and `thenCompose`.

**8. Interview difficulty**

**Intermediate**

---

### Question 18: How do you compose `CompletableFuture` stages correctly?

**1. Interview-ready answer**

Use `thenApply` for sync transformation of a value, `thenCompose` when the next step returns another `CompletableFuture`, and `thenCombine` to merge two independent futures. Use `allOf` to wait for many futures, `anyOf` for the first completion, and `exceptionally`/`handle` to recover. Always consider which executor runs each stage.

**2. Detailed explanation**

| Method | Use |
|---|---|
| `thenApply` | T → R |
| `thenAccept` | T → void |
| `thenCompose` | T → CompletableFuture<R> |
| `thenCombine` | (T, U) → R |
| `allOf` | Wait for all |
| `anyOf` | First completed |
| `exceptionally` | Recover from failure |
| `handle` | Inspect success or failure |

**3. Internal working**

`thenCompose` avoids nested `CompletableFuture<CompletableFuture<T>>` by flattening. `allOf` completes when all inputs complete but does not return their values — you read them from the original futures.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureCompositionDemo {

    static CompletableFuture<BigDecimal> loadPrice(String sku) {
        return CompletableFuture.supplyAsync(() -> BigDecimal.valueOf(100));
    }

    static CompletableFuture<BigDecimal> loadTax(String sku) {
        return CompletableFuture.supplyAsync(() -> BigDecimal.valueOf(18));
    }

    static CompletableFuture<String> loadInventory(String sku) {
        return CompletableFuture.supplyAsync(() -> "IN_STOCK");
    }

    public static void main(String[] args) {
        CompletableFuture<BigDecimal> price = loadPrice("SKU-1");
        CompletableFuture<BigDecimal> tax = loadTax("SKU-1");
        CompletableFuture<String> inventory = loadInventory("SKU-1");

        CompletableFuture<BigDecimal> total = price.thenCombine(tax, BigDecimal::add);

        CompletableFuture<Void> all = CompletableFuture.allOf(total, inventory);

        all.thenApply(ignored -> new StringBuilder()
                        .append("total=").append(total.join())
                        .append(", status=").append(inventory.join())
                        .toString())
                .exceptionally(error -> "fallback-response")
                .thenAccept(System.out::println)
                .join();
    }
}
```

Output:

```text
total=118, status=IN_STOCK
```

**5. Real-world use case**

An aggregation endpoint fetches price, stock, and shipping estimates in parallel, then combines results into one response.

**6. Interview follow-up questions**

- **Why thenCompose instead of thenApply for async calls?** It flattens nested futures.
- **What does allOf return?** `CompletableFuture<Void>`.
- **How do you timeout a stage?** `orTimeout` or `completeOnTimeout`.
- **Which thread runs thenApply?** The thread completing the future, unless Async variant is used.
- **How do you recover only on failure?** `exceptionally`.

**7. Common mistakes**

- Using allOf and expecting combined values automatically.
- Using commonPool for JDBC calls.
- Calling join inside thenApply.
- Losing exceptions because the derived future is never observed.

**8. Interview difficulty**

**Intermediate**

---

### Question 19: What is `ForkJoinPool` and how do parallel streams use it?

**1. Interview-ready answer**

`ForkJoinPool` is an executor designed for divide-and-conquer tasks. It uses work stealing: idle workers take subtasks from other workers' deques. Parallel streams use the common ForkJoinPool by default. They are good for CPU-bound, purely functional operations over large in-memory data, but poor for blocking I/O or shared-resource calls.

**2. Detailed explanation**

Key concepts:

- `RecursiveTask<T>` returns a result.
- `RecursiveAction` performs work without a result.
- `fork()` schedules a subtask.
- `join()` waits for and returns its result.
- Work stealing improves CPU utilization.
- Common pool parallelism defaults to available processors minus one.

**3. Internal working**

A worker splits a task until a threshold is reached, performs small tasks directly, and steals from the tail of other workers' deques. Parallel streams split a source using spliterators and execute leaf operations on the pool.

**4. Practical Java/Spring Boot example**

```java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

public class ForkJoinDemo {

    static final class SumTask extends RecursiveTask<Long> {

        private final int from;
        private final int to;

        SumTask(int from, int to) {
            this.from = from;
            this.to = to;
        }

        @Override
        protected Long compute() {
            if (to - from <= 100) {
                long sum = 0;
                for (int i = from; i < to; i++) {
                    sum += i;
                }
                return sum;
            }

            int middle = (from + to) / 2;
            SumTask left = new SumTask(from, middle);
            SumTask right = new SumTask(middle, to);
            left.fork();
            return right.compute() + left.join();
        }
    }

    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool(4);
        long custom = pool.invoke(new SumTask(0, 1000));
        int streamSum = IntStream.range(0, 1000).parallel().sum();

        System.out.println(custom);
        System.out.println(streamSum);
    }
}
```

Output:

```text
499500
499500
```

**5. Real-world use case**

A reporting service computes aggregates over a large in-memory list in parallel, but never uses parallel streams for calls to a shared database or payment API.

**6. Interview follow-up questions**

- **Can you change common pool parallelism?** Yes with `java.util.concurrent.ForkJoinPool.common.parallelism`.
- **Are parallel streams always faster?** No; overhead can dominate.
- **Why not block in commonPool?** It can starve unrelated application tasks.
- **How do you run a parallel stream in a custom pool?** Submit the stream task to a custom ForkJoinPool.
- **Are spliterators always splittable well?** ArrayList yes, LinkedList poorly.

**7. Common mistakes**

- Using parallel streams for network calls.
- Mutating shared state inside parallel operations.
- Assuming ordering is always preserved.
- Running nested parallel streams on the common pool.

**8. Interview difficulty**

**Advanced**

---

### Question 20: Coding challenge — implement a bounded blocking queue

**1. Interview-ready answer**

I use a circular array with `head`, `tail`, and `count`, guarded by a `ReentrantLock`. Two `Condition`s represent “not full” and “not empty”. Producers wait while full, consumers wait while empty, and both use `while` loops to handle spurious wakeups. `put` signals not-empty; `take` signals not-full.

**2. Detailed explanation**

**Problem statement:** Implement a thread-safe fixed-capacity queue where `put` blocks when full and `take` blocks when empty.

**Approach:**
1. Store elements in a circular array.
2. Protect all state with one `ReentrantLock`.
3. Wait on `notFull` in `put` and `notEmpty` in `take`.
4. Use `lockInterruptibly` so blocked callers can be interrupted.
5. Signal the opposite condition after state changes.

**3. Internal working**

The circular array avoids shifting elements. `tail = (tail + 1) % capacity` wraps around. Conditions release the lock while waiting and reacquire it before returning, matching the intrinsic wait/notify model with more control.

**4. Practical Java solution**

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public final class BoundedQueue<T> {

    private final Object[] items;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notEmpty;
    private final Condition notFull;

    private int head;
    private int tail;
    private int count;

    public BoundedQueue(int capacity) {
        if (capacity <= 0) {
            throw new IllegalArgumentException("capacity must be positive");
        }
        this.items = new Object[capacity];
        this.notEmpty = lock.newCondition();
        this.notFull = lock.newCondition();
    }

    public void put(T item) throws InterruptedException {
        if (item == null) {
            throw new IllegalArgumentException("item must not be null");
        }

        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                notFull.await();
            }
            items[tail] = item;
            tail = (tail + 1) % items.length;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                notEmpty.await();
            }
            T item = (T) items[head];
            items[head] = null;
            head = (head + 1) % items.length;
            count--;
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        BoundedQueue<String> queue = new BoundedQueue<>(2);

        queue.put("ORD-1");
        queue.put("ORD-2");

        Thread producer = new Thread(() -> {
            try {
                queue.put("ORD-3");
            } catch (InterruptedException exception) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        TimeUnit.MILLISECONDS.sleep(50);

        System.out.println(queue.take());
        producer.join();
        System.out.println(queue.take());
        System.out.println(queue.take());
        System.out.println(queue.size());
    }
}
```

Output:

```text
ORD-1
ORD-3
ORD-2
0
```

**5. Dry run**

| Step | Action | State |
|---|---|---|
| 1 | put ORD-1 | `[ORD-1]`, tail wraps next |
| 2 | put ORD-2 | `[ORD-1, ORD-2]`, full |
| 3 | producer put ORD-3 | waits on notFull |
| 4 | take | returns ORD-1, signals notFull |
| 5 | producer wakes | inserts ORD-3 |
| 6 | two takes | returns ORD-3, ORD-2 |

**6. Complexity**

- `put`: O(1) plus blocking wait.
- `take`: O(1) plus blocking wait.
- Space: O(capacity).

**7. Edge cases**

- Capacity ≤ 0 → `IllegalArgumentException`.
- Null item → `IllegalArgumentException`.
- Interrupt while waiting → `InterruptedException`.
- Many producers/consumers → condition loops preserve correctness.
- Spurious wakeups → `while` loops handle them.

**8. Real-world use case**

This is the pattern behind `ArrayBlockingQueue`, used for bounded handoff between webhook receivers and background workers.

**9. Interview follow-up questions**

- **Why not use ArrayBlockingQueue?** In production you should; this demonstrates internals.
- **Why while around await?** Spurious wakeups and changed conditions.
- **Why signal instead of signalAll?** One insert can wake one consumer; one remove can wake one producer.
- **What if using fair lock?** FIFO-ish waiter ordering at lower throughput.
- **How do you add timeout?** Use `await(timeout, unit)` and return boolean.

**10. Common mistakes**

- Using `if` instead of `while`.
- Forgetting to unlock in finally.
- Nulling the wrong slot.
- Using two independent locks for head and tail.

**11. Interview difficulty**

**Advanced**

---

## Chapter 8 Verification

| Syllabus topic | Question(s) |
|---|---|
| Process vs thread | Q1 |
| Thread states and lifecycle | Q2 |
| Thread creation, Runnable/Callable, start vs run | Q3 |
| sleep, wait, join, park, yield | Q4 |
| notify and notifyAll | Q5 |
| synchronized and monitors | Q6 |
| volatile | Q7 |
| Java Memory Model and happens-before | Q8 |
| Race conditions and atomicity | Q9 |
| ReentrantLock | Q10, Q20 |
| ReadWriteLock and StampedLock | Q11 |
| Atomics and CAS | Q12 |
| Deadlock, livelock, starvation | Q13 |
| ThreadLocal | Q14 |
| Executor framework and thread pools | Q15, Q16 |
| Future and CompletableFuture | Q17, Q18 |
| ForkJoinPool and parallel streams | Q19 |
| Concurrent coding exercise | Q20 |

**Total interview questions: 20** — all with the required 8-part format; coding questions include problem statement, approach, Java solution, dry run, complexity, and edge cases.

**Next:** [Chapter 9 — Spring Framework Fundamentals](../04-Spring/09-Spring-Framework.md). Say `NEXT CHAPTER` to continue.
