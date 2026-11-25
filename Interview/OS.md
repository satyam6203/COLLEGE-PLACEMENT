🖥️ OS Interview Questions (1–100)

> Format: **Answer** → **Example** → **Interview Follow-up** → **Short Interview Answer**

---

## 1. What is an Operating System?

**Answer:**
An OS is system software that manages the hardware and provides services to user applications — process management, memory, files, I/O and security. It acts as a resource manager and a virtual machine, so programs don't need to know the hardware details. It also provides a user interface and runs other programs.

**Example:**
When you open Chrome on Windows, the OS allocates memory for it, schedules its threads on the CPU, gives it file access, and isolates it from other apps.

**Interview Follow-up:**
- Is the OS part of the hardware or software? (Software, but it sits closest to hardware)
- Give two roles. (Resource manager + virtual machine/interface provider)

**Short Interview Answer:**
An Operating System is system software that manages hardware resources and provides services like process, memory and file management to applications.

---

## 2. What are the main functions of an Operating System?

**Answer:**
Main functions are process management, memory management, file system management, device and I/O management, plus security and user interface. On top of that it provides networking, system utilities and resource accounting. Basically everything a program needs to run safely and share hardware.

**Example:**
When a JVM starts, the OS gives it memory, opens sockets, schedules its threads, and denies it direct access to another process's memory.

**Interview Follow-up:**
- Which function matters most for performance? (Usually process scheduling and memory management)
- What is a system utility? (Tools like `df`, `ps`, `top` shipped with the OS)

**Short Interview Answer:**
An OS manages processes, memory, files, devices/I/O, security and networking so applications can use the hardware safely and efficiently.

---

## 3. What are the different types of Operating Systems?

**Answer:**
OSs are broadly classified by usage — batch, time-sharing, real-time, network, and multiprocessor/distributed — and by architecture — single-user vs multi-user, single-tasking vs multi-tasking. Examples: Windows/Linux (general purpose), Linux RT (real-time), Android (mobile). For an interview, mentioning the classification along with one example is enough.

**Example:**

| Type | Use case | Example |
|---|---|---|
| Batch | Offline bulk jobs | IBM mainframe Z/OS |
| Time-sharing | Many interactive users | Unix/Linux multi-user |
| Real-time | Strict deadlines | QNX, VxWorks, FreeRTOS |
| Network | Distributed resources | Windows Server |
| Multiprocessor | Multiple CPUs | Linux SMP, Unix |

**Interview Follow-up:**
- What is multiprogramming OS? (Runs many programs, switching CPU between them)
- Why do real-time OSs avoid virtual memory? (Unpredictable paging delays break deadlines)

**Short Interview Answer:**
OSs come as batch, time-sharing, real-time, network and multiprocessor systems, each optimized for a different workload.

---

## 4. What is the difference between Kernel and Operating System?

**Answer:**
The kernel is the core part of the OS that directly manages hardware — memory, CPU, devices. The OS is the whole package: kernel plus system utilities, libraries, the shell and the user interface. So every kernel is part of an OS, but an OS is much bigger than its kernel.

**Example:**
Linux kernel + coreutils (`ls`, `grep`) + bash + GNOME = the "Linux OS" you use, though "Linux" often refers to the kernel only.

**Interview Follow-up:**
- Is the shell part of the kernel? (No, it's a user-space program)
- What are the two kernel modes? (Kernel mode and user mode)

**Short Interview Answer:**
The kernel is the core that manages hardware directly, while the OS is the complete package — kernel plus utilities, libraries and the user interface.

---

## 5. What are system calls?

**Answer:**
A system call is the interface through which a user program asks the kernel to perform a privileged operation like opening a file or allocating memory. The call triggers a trap that switches the CPU from user mode to kernel mode. It's the controlled gateway that prevents programs from touching hardware directly.

**Example:**
Java's `FileInputStream.read()` ends up calling `read()` syscall, and `new` in native code calls `brk`/`mmap`.

**Interview Follow-up:**
- Are system calls slow? (Compared to a function call, yes — mode switch + validation overhead)
- Name a few. (`open`, `read`, `write`, `fork`, `exec`, `mmap`)

**Short Interview Answer:**
A system call is the interface a program uses to request a kernel-level service like file I/O or memory allocation, switching it into kernel mode.

---

## 6. What is the difference between user mode and kernel mode?

**Answer:**
Kernel mode gives unrestricted access to all hardware and memory — it's protected and only the OS runs there. User mode is restricted: applications run here and can only access their own memory plus whatever the kernel explicitly allows. The mode is enforced by hardware privilege levels so a buggy app can't crash the whole system.

**Example:**
A Java app writes to `System.out` from user mode; the actual write syscall runs in kernel mode.

**Interview Follow-up:**
- What switches the mode? (A trap, syscall, or interrupt)
- What happens on a segfault? (The kernel kills the process to protect memory)

**Short Interview Answer:**
User mode is restricted application execution, while kernel mode is privileged OS execution with full hardware and memory access.

---

## 7. What is a process?

**Answer:**
A process is a program in execution — it has its own address space, resources, and state tracked by the OS. It is the basic unit of resource allocation: each process gets its own memory, file descriptors and PCB. Two copies of the same program running at once are two separate processes.

**Example:**
Running Chrome twice gives two processes, each with its own tabs' memory and open files.

**Interview Follow-up:**
- Can two processes share memory? (Yes — shared memory / `fork` with `mmap`)
- What does the OS use to identify a process? (PID)

**Short Interview Answer:**
A process is an executing program with its own memory space and resources, and it is the OS's basic unit of resource allocation.

---

## 8. What is a thread?

**Answer:**
A thread is the smallest unit of execution within a process — it has its own program counter, registers and stack, but shares code, data and files with other threads of the same process. Threads are what get scheduled on the CPU. Sharing makes communication cheap but also introduces synchronization problems.

**Example:**
In a web server, Tomcat has a thread pool; each request is handled on a thread that shares the same JVM heap.

**Interview Follow-up:**
- Threads share or don't share what? (Share heap, static fields, files; don't share stack and local variables)
- Why is thread creation cheaper than process creation? (No separate address space or page table to set up)

**Short Interview Answer:**
A thread is the basic unit of CPU scheduling inside a process, sharing memory with other threads but having its own stack and registers.

---

## 9. What are the different states of a process?

**Answer:**
A process moves through New, Ready, Running, Waiting/Blocked, and Terminated. It goes Ready → Running when scheduled, Running → Waiting when it needs I/O, and Waiting → Ready when the I/O completes. Understanding these states is how you read a process state in `ps` or `top`.

**Example:**
A process doing `file.read()` goes Running → Waiting on disk, then back to Ready when the disk read completes.

**Interview Follow-up:**
- Can a Running process become Ready directly? (Yes — on preemption)
- What does `S` mean in `ps`? (Interruptible sleep, i.e. Waiting)

**Short Interview Answer:**
The process states are New, Ready, Running, Waiting and Terminated, and the transitions depend on scheduling, I/O and preemption.

---

## 10. What is a Process Control Block (PCB)?

**Answer:**
A PCB is the kernel's data structure that stores everything about a process — PID, state, program counter, registers, memory info and accounting. The OS keeps one PCB per process and switches between them to schedule the CPU. It's the process's identity in the kernel.

**Example:**
On a context switch, the kernel saves the current CPU registers into the PCB of process A and loads them from the PCB of process B.

**Interview Follow-up:**
- Where is it stored? (Kernel memory, one per process)
- Difference between PCB and TCB? (PCB for a process, TCB for a thread — several TCBs per process)

**Short Interview Answer:**
A PCB is the kernel's per-process record holding PID, state, registers and memory info, and it's swapped in and out during a context switch.

---

## 11. What is the difference between a process and a program?

**Answer:**
A program is a passive file on disk — just code. A process is that program loaded into memory and actively executing, with its own resources. One program can be executed many times, each producing a separate process.

**Example:**
`java` is a program; every time you run `java MyApp`, you create a new process for the JVM.

**Interview Follow-up:**
- Program vs executable file? (Program is any code; executable is a program in a runnable format)
- Is a program passive? (Yes — it doesn't execute by itself)

**Short Interview Answer:**
A program is a passive set of instructions stored on disk, while a process is that program loaded and running in memory with its own resources.

---

## 12. What is Inter-Process Communication (IPC)?

**Answer:**
IPC is the set of mechanisms processes use to exchange data and synchronize when they can't share memory directly. Since processes are isolated for safety, they need an OS-provided channel. The fastest option is shared memory, but it requires your own synchronization.

**Example:**
A web server process communicating with a worker process, or Kafka broker processes exchanging messages over sockets.

**Interview Follow-up:**
- Fastest IPC? (Shared memory — it avoids data copying)
- What's a pipe vs socket? (Pipe is one-way byte stream between related processes; socket supports bidirectional, networked communication)

**Short Interview Answer:**
IPC is how separate processes exchange data and synchronize — through pipes, message queues, shared memory, signals or sockets.

---

## 13. What are the different IPC mechanisms?

**Answer:**
Common IPC mechanisms are pipes, named pipes (FIFOs), message queues, shared memory, signals, semaphores and sockets. Some move data (pipes, queues, shared memory), some only notify (signals), and some only synchronize (semaphores). The right choice depends on whether you need data transfer, notification, or mutual exclusion.

**Example:**

| Mechanism | Purpose |
|---|---|
| Pipe / FIFO | One-way anonymous / named byte stream |
| Message queue | Structured messages with priority |
| Shared memory | Fastest — same memory region |
| Signal | Simple async notification |
| Semaphore | Synchronization only |
| Socket | Bidirectional, works across machines |

**Interview Follow-up:**
- Which is synchronous? (Message queues and sockets can be)
- Java equivalents? (`java.io.PipedInputStream`, `BlockingQueue`, `Semaphore`)

**Short Interview Answer:**
IPC mechanisms include pipes, message queues, shared memory, signals, semaphores and sockets — some transfer data, some notify, some synchronize.

---

## 14. What is a daemon process?

**Answer:**
A daemon is a background process that runs continuously without user interaction, started at boot and serving other processes. It has no controlling terminal and is reparented to init when its parent dies. On Linux it's called a service, managed by systemd.

**Example:**
`sshd`, `cron`, `mysqld`, `nginx`, and `dockerd` are all daemons.

**Interview Follow-up:**
- How to create one in Java? (Register with `init.d`/`systemd`, or `nohup ... &`, or use a service wrapper like `jlink`/`Spring Boot properties`)
- How does `kill` work for daemons? (SIGTERM lets them shut down gracefully)

**Short Interview Answer:**
A daemon is a long-running background process with no user interface, started at boot and used to serve other processes — like sshd or mysqld.

---

## 15. What is process scheduling?

**Answer:**
Process scheduling is the OS deciding which process runs on the CPU next and for how long. The scheduler picks from the ready queue using a policy that balances throughput, latency and fairness. It's the single biggest factor in perceived responsiveness.

**Example:**
When you open 20 browser tabs plus a video call, the scheduler decides which gets CPU time so that video stays smooth.

**Interview Follow-up:**
- Long-term vs short-term scheduler? (Admission/control vs CPU selection)
- Does I/O-bound vs CPU-bound scheduling differ? (Yes — time slicing matters more for I/O-bound work)

**Short Interview Answer:**
Process scheduling is the mechanism that selects which ready process gets the CPU next, based on a policy balancing fairness and responsiveness.

---

## 16. What are the different types of schedulers?

**Answer:**
The three classic schedulers are long-term (job/admission scheduler — which processes to admit from disk to memory), short-term (CPU scheduler — picks the next process to run), and medium-term (swapper — suspends and resumes processes to manage memory). In simple systems only the short-term scheduler exists.

**Example:**
In a time-sharing system, the long-term scheduler keeps a backup job pool on disk and admits one job when a process finishes, so the ready queue never empties.

**Interview Follow-up:**
- Which one affects user-perceived interactivity? (Short-term CPU scheduler)
- What does the medium-term scheduler do exactly? (Swaps a process out to free memory, later swaps it back in)

**Short Interview Answer:**
There are long-term (admission), short-term (CPU) and medium-term (swapping) schedulers, each controlling a different stage of a process's life.

---

## 17. What are scheduling criteria?

**Answer:**
Criteria include CPU utilization, throughput, turnaround time, waiting time, and response time. Also fairness and priority support. No single criterion is optimal — batch systems favour throughput, interactive systems favour response time.

**Example:**

| Criterion | Definition | Matters for |
|---|---|---|
| CPU utilization | % time CPU busy | Batch |
| Throughput | Processes completed per unit time | Batch |
| Turnaround time | Submission → completion | Batch |
| Waiting time | Time in ready queue | All |
| Response time | First response − arrival | Interactive |

**Interview Follow-up:**
- Response time vs turnaround time? (Response = first output; turnaround = full completion)
- Which is most important for a web server? (Response time / throughput together)

**Short Interview Answer:**
Scheduling criteria are CPU utilization, throughput, turnaround time, waiting time and response time, with different ones mattering for batch vs interactive systems.

---

## 18. What are the different CPU scheduling algorithms?

**Answer:**
The main ones are FCFS, SJF/SJF preemptive, Priority scheduling, Round Robin, and Multilevel Queue/Feedback Queue. Each optimises something different — FCFS is simple, SJF minimises average waiting time, Round Robin is fair, Priority respects importance. In practice modern OSs use MLFQ-like or CFS-type schedulers.

**Example:**

| Algorithm | Priority | Weakness |
|---|---|---|
| FCFS | Simplicity | Convoy effect |
| SJF | Minimum average wait | Needs burst-time estimate |
| Priority | Importance | Starvation |
| Round Robin | Fairness | Time-quantum tuning |
| MLFQ | Adaptive | Complexity |

**Interview Follow-up:**
- Which has starvation risk? (SJF and Priority)
- What does Linux CFS use? (A fair-share scheduler with virtual runtime per thread)

**Short Interview Answer:**
The main CPU scheduling algorithms are FCFS, SJF, Priority, Round Robin and multilevel queues — each optimising a different goal like speed, fairness or importance.

---

## 19. What is starvation? What is aging?

**Answer:**
Starvation is when a process keeps getting resources while another waits indefinitely because it's always lower priority. Aging fixes this by gradually increasing the priority of waiting processes as time passes. Every scheduling algorithm with priority or SJF needs aging to avoid indefinite waiting.

**Example:**
If new interactive processes always jump the queue, a long batch report may never get CPU — aging guarantees it eventually does.

**Interview Follow-up:**
- Name algorithms with starvation. (SJF, Priority, MLFQ without boosting)
- What is the fix in SJF? (Aging the wait time into the priority calculation)

**Short Interview Answer:**
Starvation is indefinite waiting for a low-priority process, and aging is the fix — gradually raising the priority of processes that have waited a long time.

---

## 20. What is a zombie process and an orphan process?

**Answer:**
A zombie is a process that has finished but whose exit status hasn't been read by its parent, so it lingers as a table entry consuming almost no memory. An orphan is a running process whose parent died — it's just adopted by init/systemd and keeps running normally. Zombies are harmless individually but a leak of thousands is a problem.

**Example:**
On Linux, a zombie still appears in `ps` with state `Z` until `wait()`/`waitpid()` is called or the parent exits.

**Interview Follow-up:**
- How to avoid zombies? (Always `wait()` on child processes; a reaper thread if you can't)
- What happens to orphans? (They get reparented to PID 1, which reaps them)

**Short Interview Answer:**
A zombie is a finished process whose exit status hasn't been collected, while an orphan is a live process whose parent died and is adopted by init.

---

## 21. What is the difference between a process and a thread?

**Answer:**
A process is the unit of resource allocation with its own address space, while a thread is a unit of execution that shares the process's memory. Threads are cheaper to create and can communicate directly, but they need synchronisation because they share data. Isolation is the process's strength; speed and shared memory are the thread's.

**Example:**
Running two Chrome instances uses processes (isolation), while each tab's JS runs on threads inside one renderer process.

**Interview Follow-up:**
- Context switch cost? (Threads are much cheaper — no page table/TLB change)
- What does `fork()` do in a multithreaded program? (Only the calling thread survives in the child — a classic gotcha)

**Short Interview Answer:**
Processes are isolated resource containers, threads are lightweight execution units that share one process's memory — threads are faster but need locking.

---

## 22. What are the advantages of multithreading?

**Answer:**
Multithreading lets a single process do several things concurrently, so CPU-bound work can use multiple cores and I/O-bound work can overlap waiting with computing. Threads are cheap to create, share data easily, and improve responsiveness. The cost is complexity — race conditions, deadlocks and hard debugging.

**Example:**
A web server handles 200 concurrent requests on a thread pool while the OS overlaps network waits with CPU work.

**Interview Follow-up:**
- Disadvantages? (Race conditions, deadlocks, non-deterministic bugs, harder testing)
- When is a process pool better? (When you need crash isolation or a different language runtime per task)

**Short Interview Answer:**
Multithreading uses multiple cores for CPU work, overlaps I/O waits, keeps the app responsive, and is cheaper than creating processes.

---

## 23. What is a critical section?

**Answer:**
A critical section is a block of code that accesses shared data and must not be executed by two threads at the same time. The pattern is entry section (request lock), critical section (shared work), exit section (release lock), and remainder section. Correctness depends on mutual exclusion — no two threads inside at once.

**Example:**
`balance = balance - 500` in two threads must be inside a critical section, or the read-modify-write can interleave.

**Interview Follow-up:**
- Requirements? (Mutual exclusion, progress, bounded waiting)
- Lock granularity? (Finer locks = more parallelism but higher overhead)

**Short Interview Answer:**
A critical section is code accessing shared data that must be protected by a lock so only one thread executes it at a time.

---

## 24. What is a race condition?

**Answer:**
A race condition happens when the result depends on the unpredictable order in which threads execute, so the output changes run to run. It occurs when shared data is accessed without proper synchronisation. It's the most common source of intermittent, hard-to-reproduce production bugs.

**Example:**
Two threads do `count++` — because it's a read-modify-write, both may read the same value and the final count is wrong.

**Interview Follow-up:**
- How to fix? (Locks, atomics, thread-safe collections, immutable objects)
- Is `volatile` enough? (No — it gives visibility, not atomicity for `count++`)

**Short Interview Answer:**
A race condition is a bug where unsynchronised shared data access gives different results depending on thread timing — fixed with locks, atomics or immutability.

---

## 25. What is a mutex?

**Answer:**
A mutex is a mutual exclusion lock — a flag with a queue — that lets exactly one thread hold it at a time. A thread locks before entering a critical section and unlocks after leaving; others block in FIFO order. In Java it's `ReentrantLock`; in POSIX it's `pthread_mutex`.

**Example:**
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    sharedMap.put(k, v);   // critical section
} finally {
    lock.unlock();
}
```

**Interview Follow-up:**
- `synchronized` vs `ReentrantLock`? (Simpler vs tryLock, timeout, fair ordering)
- What happens if you forget to unlock? (Deadlock — hence `finally`)

**Short Interview Answer:**
A mutex is a lock ensuring only one thread at a time enters a critical section; Java's equivalent is `ReentrantLock` or `synchronized`.

## 26. What is a semaphore?

**Answer:**
A semaphore is a counter with two atomic operations: `wait()` (decrement, or block if zero) and `signal()` (increment, possibly waking a waiter). It can control access to multiple resources, unlike a mutex which only allows one. A semaphore with a max value of 1 behaves like a mutex.

**Example:**
```java
Semaphore dbConnections = new Semaphore(10);   // max 10 concurrent DB calls
dbConnections.acquire();
try {
    runQuery();
} finally {
    dbConnections.release();
}
```

**Interview Follow-up:**
- `Semaphore` vs `CountDownLatch`? (Semaphore lets threads re-acquire; a latch is a one-time event counter)
- What if `release()` exceeds initial count? (Common Java bug — it increases permits beyond the intended limit)

**Short Interview Answer:**
A semaphore is a counter that controls how many threads may access a resource at once, using `acquire()` and `release()`.

---

## 27. What is the difference between binary semaphore and counting semaphore?

**Answer:**
A binary semaphore can only hold 0 or 1 — it's used purely for mutual exclusion and is released only by the thread that acquired it. A counting semaphore holds any value from 0 to N, so it can limit concurrency to N resources and any thread can release it. The main pitfall is that a binary semaphore allows recursive acquisition by the same thread, unlike a mutex.

**Example:**
Binary: `1` = lock/unlock for a print method. Counting: `5` = at most 5 simultaneous DB connections.

**Interview Follow-up:**
- Which is more error-prone? (Binary, because of recursion — a mutex fixes this)
- How many waiters can a counting semaphore have? (Unlimited)

**Short Interview Answer:**
A binary semaphore is 0/1 and used for mutual exclusion, while a counting semaphore goes from 0 to N and limits how many threads use a resource at once.

---

## 28. What is deadlock?

**Answer:**
A deadlock is when every thread is blocked, each waiting for a resource held by another thread, so nothing can progress. It requires at least two threads and two resources. Once deadlocked, the system recovers only through a timeout or a kill — there's no way out on its own.

**Example:**
T1 holds lock A and waits for B; T2 holds lock B and waits for A → both wait forever.

**Interview Follow-up:**
- Difference from starvation? (Deadlock = nobody progresses; starvation = some threads progress, one never does)
- How do you avoid it in Java? (Consistent lock ordering, `tryLock` with timeout)

**Short Interview Answer:**
A deadlock is a permanent state where each thread holds a lock another needs, so all of them are blocked forever.

---

## 29. What are the necessary conditions for deadlock?

**Answer:**
Four conditions must hold together: mutual exclusion (resource held exclusively), hold and wait (holding one while asking for another), no preemption (resource can't be forcibly taken), and circular wait (a cycle of waiting). If a system prevents even one of them, deadlock can't occur. These are the four Coffman conditions.

**Example:**
A global rule that all locks must be acquired in increasing ID order eliminates circular wait, so no deadlock is possible.

**Interview Follow-up:**
- Which condition do most real systems break? (Circular wait — via lock ordering; no preemption via timeouts)
- Is mutual exclusion avoidable? (Generally no — that's the point of locking)

**Short Interview Answer:**
Coffman's four conditions are mutual exclusion, hold-and-wait, no preemption and circular wait — breaking any one prevents deadlock.

---

## 30. What are the methods for handling deadlock?

**Answer:**
Three approaches exist: ignore it (the OS does nothing and hopes it doesn't happen — many systems do this), prevent it by denying one of the four conditions upfront, or detect and recover by building a wait-for graph, finding cycles, and aborting a victim. Recovery options include rollback, preemption and starvation of the victim.

**Example:**
Linux kernel has a wait-for graph and detects cycles, then kills the youngest process to break them.

**Interview Follow-up:**
- Why is ignoring common? (Deadlocks are rare in practice, and prevention adds overhead)
- Which is preferred in databases? (Detection + victim rollback)

**Short Interview Answer:**
Deadlock is handled by ignoring it, preventing it (breaking a Coffman condition), or detecting it with a wait-for graph and recovering by aborting a victim.

---

## 31. What is memory management in an OS?

**Answer:**
Memory management is how the OS allocates, tracks and reclaims physical memory for processes. It handles virtual addressing, allocation/deallocation, sharing and protection between processes. Its job is to make each process feel like it owns memory exclusively.

**Example:**
Each JVM process thinks it has its own address space starting at 0; the page tables map that to real physical frames.

**Interview Follow-up:**
- Data structures used? (Page tables, free lists, bitmaps, segment tables)
- Swapping vs paging? (Swapping moves whole processes; paging moves pages on demand)

**Short Interview Answer:**
Memory management is the OS's job of allocating, protecting and reclaiming physical memory so processes get safe, isolated virtual address spaces.

---

## 32. What is the difference between logical and physical address?

**Answer:**
A logical (virtual) address is what the program sees — it belongs to the process's own address space and is generated by the CPU. A physical address is the actual location in RAM where that byte lives. The MMU translates logical to physical using the page table on every memory access.

**Example:**
Your program addresses variable at `0x7fff1234`, but the MMU maps it to physical address `0x2a1b0`.

**Interview Follow-up:**
- When is translation done? (Every memory reference, cached in the TLB)
- What is a logical address space? (The full range of addresses a process can use)

**Short Interview Answer:**
Logical addresses are what a program sees in its own address space; physical addresses are the real RAM locations, and the MMU translates between them.

---

## 33. What is MMU (Memory Management Unit)?

**Answer:**
The MMU is a hardware component in the CPU that translates every logical address into a physical address using the page table held in main memory. It also enforces protection — it raises an exception if a process tries to access memory outside its allowed range. It's why memory isolation works without software effort.

**Example:**
On a segfault, the MMU detects the access violates a page table permission and raises an interrupt, which the OS handles by killing the process.

**Interview Follow-up:**
- What happens on a translation failure? (Page fault or protection fault interrupt)
- Is the MMU software or hardware? (Hardware, driven by page tables the OS maintains)

**Short Interview Answer:**
The MMU is hardware that translates virtual to physical addresses on every access and enforces memory protection.

---

## 34. What is paging?

**Answer:**
Paging splits logical memory into fixed-size pages and physical memory into equal-size frames, then maps any page to any frame. Because the blocks are fixed, there's no external fragmentation and you don't need contiguous memory. The trade-off is internal fragmentation and the page table overhead.

**Example:**
4KB pages — a program can use physical frames scattered anywhere while appearing contiguous.

**Interview Follow-up:**
- Page table entry size? (Usually 4 or 8 bytes, containing frame number + flags)
- Transparent to the app? (Yes — that's the point)

**Short Interview Answer:**
Paging divides memory into fixed-size pages and frames mapped by a page table, eliminating external fragmentation and allowing non-contiguous allocation.

---

## 35. What is segmentation?

**Answer:**
Segmentation divides memory into logical segments like code, data and stack, each with a size that matches its purpose. It uses a segment table with a base and limit per entry, and segments can grow dynamically. It provides a more natural view of programs but causes external fragmentation.

**Example:**
`CS: code segment`, `DS: data segment`, `SS: stack segment` — exactly how x86 real-mode addressing works.

**Interview Follow-up:**
- Fragmentation type? (External)
- Combined with paging? (x86 uses segmented paging — segmentation is largely vestigial)

**Short Interview Answer:**
Segmentation splits memory into logical variable-sized segments like code and data, which is programmer-friendly but causes external fragmentation.

---

## 36. What is virtual memory?

**Answer:**
Virtual memory lets each process believe it has more memory than the physical RAM, by mapping virtual addresses to physical frames and backing them with disk. It provides isolation, allows programs bigger than RAM, and enables efficient sharing. The OS and MMU work together to map pages on demand.

**Example:**
A machine with 8GB RAM can run a 20GB app; the parts never touched stay on disk.

**Interview Follow-up:**
- Overcommit? (Yes — OS promises more virtual memory than physical)
- What does it enable besides size? (Process isolation, shared libraries, copy-on-write on `fork`)

**Short Interview Answer:**
Virtual memory gives each process its own address space that can exceed physical RAM, with pages loaded on demand from disk.

---

## 37. What is demand paging?

**Answer:**
Demand paging loads a page into memory only when the process actually accesses it, not upfront. The page table entry is marked "not present", and the first access triggers a page fault which loads the page from disk. It saves a lot of memory and load time because unused pages are never brought in.

**Example:**
Starting a browser loads only the tabs and scripts you're actually using.

**Interview Follow-up:**
- What is a valid vs invalid page reference? (Valid = the page exists in the logical address space; invalid = the address is illegal)
- Downside? (Page faults are expensive — disk I/O latency)

**Short Interview Answer:**
Demand paging brings a page into memory only when it's first accessed, saving memory and startup time at the cost of page fault overhead.

---

## 38. What is a page fault?

**Answer:**
A page fault is a hardware exception raised when a process accesses a page that isn't currently in physical memory. If the page exists in the page file, the OS loads it, updates the page table and restarts the instruction — the process doesn't notice. If the address is genuinely invalid, the OS kills the process with a segfault.

**Example:**
Accessing memory for the first time is a "major" fault costing a disk read; accessing a page already in the page cache is a "minor" fault, which is cheap.

**Interview Follow-up:**
- Major vs minor? (Major = disk read; minor = already in memory)
- Why is restart needed? (The instruction must re-execute now that data is present)

**Short Interview Answer:**
A page fault is raised when a process touches a page not in RAM — the OS loads it from disk and restarts the instruction, or kills the process if the address is invalid.

---

## 39. What is TLB (Translation Lookaside Buffer)?

**Answer:**
The TLB is a small, fast cache inside the MMU that stores recent virtual-to-physical page translations. It lets most memory accesses skip the page table lookup, which would otherwise cost a memory access. Without it, every access would take two memory reads — one for the TLB/page table, one for the data.

**Example:**
A 1024-entry TLB covers the working set of most processes, so ~99% of accesses hit it and translate in a single cycle.

**Interview Follow-up:**
- TLB miss? (Walk the page table; cost is a memory access per level)
- What causes frequent misses? (Large working sets, poor locality, thrashing)

**Short Interview Answer:**
The TLB is a fast cache of recent address translations in the MMU, so most memory accesses skip the page table lookup.

---

## 40. What is thrashing?

**Answer:**
Thrashing is when a process spends more time paging than executing, because physical memory is over-committed and the working set doesn't fit. The CPU sits idle most of the time while the disk is busy swapping pages. It happens when too many processes are active for the available RAM.

**Example:**
16GB RAM with 20 Chrome tabs each with 2GB active sets → constant page faults, system becomes unresponsive despite a fast CPU.

**Interview Follow-up:**
- How to prevent? (Working set allocation, page-fault frequency control, more RAM, fewer concurrent processes)
- Symptom in OS? (High disk activity, near-zero CPU utilisation)

**Short Interview Answer:**
Thrashing is when processes spend more time paging in and out than executing because memory is over-committed — prevented with working-set control or more RAM.

## 41. What is a file?

**Answer:**
A file is a named collection of related information stored on secondary storage, managed by the OS. It's an abstract logical device — you don't care whether the bytes live on a disk, SSD or network share. Files have a name, extension, metadata and permissions.

**Example:**
`application.properties` in your Spring Boot project — the JVM only sees a stream of bytes with a name.

**Interview Follow-up:**
- File vs directory? (A directory is a special file that contains filenames)
- What metadata does a file carry? (Size, owner, permissions, timestamps, inode)

**Short Interview Answer:**
A file is a named block of data on secondary storage managed by the OS, treated as an abstract logical device independent of the physical medium.

---

## 42. What is a file system?

**Answer:**
A file system is the method and data structures the OS uses to store, name and organise files on a storage device. It defines how files are named, how blocks are allocated, how free space is tracked and how metadata is kept. Examples: NTFS, ext4, APFS, XFS.

**Example:**
ext4 uses an inode per file and an allocation bitmapped group structure; NTFS uses a Master File Table plus journaling.

**Interview Follow-up:**
- Journaling vs non-journaling? (Journaling logs metadata updates for faster crash recovery)
- What is a mount? (Attaching a file system to a directory path)

**Short Interview Answer:**
A file system is the OS's way of naming, storing and organising files on disk — like NTFS or ext4.

---

## 43. What are different file allocation methods?

**Answer:**
The three classic methods are contiguous allocation, linked allocation and indexed allocation. Contiguous is fastest but suffers external fragmentation; linked handles growth easily but has no random access; indexed supports random access and size growth but needs an index block per file. FAT uses linked allocation with a central table; i-nodes use indexed allocation.

**Example:**

| Method | Pros | Cons |
|---|---|---|
| Contiguous | Simple, fast sequential + random access | External fragmentation |
| Linked | No fragmentation, files grow easily | No random access, pointer overhead |
| Indexed | Random access, easy growth | Extra index block/block chain |

**Interview Follow-up:**
- How many blocks does a 100KB file need in indexed allocation? (ceil(100/4)=25 for 4KB blocks)
- Free-space management issue with contiguous? (Need compaction)

**Short Interview Answer:**
File allocation methods are contiguous (fast, fragments), linked (grows freely, no random access) and indexed (random access, needs an index block).

---

## 44. What is a directory? What are its types?

**Answer:**
A directory is a special file that maps names to files and other directories, forming a tree or graph structure from the root. Types vary by how they store that mapping: single-level, two-level, tree-structured, acyclic-graph, and general graph (allowing cycles).

**Example:**
Linux uses a tree-structured directory starting at `/`; NTFS and macOS use a general graph so the same folder can appear in multiple places.

**Interview Follow-up:**
- Hard link vs symbolic link? (Hard = another name for the same inode; symbolic = a pointer to a path that can dangle)
- What is an inode? (The metadata record holding size, owner, timestamps, block pointers)

**Short Interview Answer:**
A directory maps names to files and subdirectories — single-level, two-level, tree, acyclic-graph or general graph — and forms the namespace users browse.

---

## 45. What is mounting in a file system?

**Answer:**
Mounting is attaching a file system (or a device, network share or image) to a specific directory in the namespace, so its contents appear there. The OS records the device, mount point and options in a mount table. It lets a whole disk tree live inside what looks like a folder.

**Example:**
`mount /dev/sdb1 /mnt/data` makes `/mnt/data` show the contents of partition 2. `mount -t tmpfs none /dev/shm` creates a RAM-backed file system.

**Interview Follow-up:**
- What does /etc/fstab do? (Automatically mounts listed devices at boot)
- Can you mount a file? (Yes — loopback mount, common for ISO images and Docker volumes)

**Short Interview Answer:**
Mounting attaches a file system or device to a directory so its contents appear there, e.g. `mount /dev/sdb1 /mnt/data`.

---

## 46. What is disk scheduling?

**Answer:**
Disk scheduling is the OS deciding the order in which disk I/O requests are served. Because seeking to a different track is far slower than reading sequentially, reordering requests dramatically reduces average wait time. It's a classic example of balancing throughput and per-request latency.

**Example:**
If the head is at track 10 and requests are at 6, 15 and 40, FCFS takes 4+5+25 moves; SSTF takes 4+9+25 = fewer.

**Interview Follow-up:**
- Still relevant with SSDs? (Less so — SSDs have no seek time, but scheduler choice still matters for parallelism and wear)

**Short Interview Answer:**
Disk scheduling reorders I/O requests to minimise seek time and reduce average response time.

---

## 47. What are the disk scheduling algorithms?

**Answer:**
The main algorithms are FCFS, SSTF, SCAN (elevator), C-SCAN and LOOK/C-LOOK. SSTF minimises seek time but can starve far requests, and SCAN avoids that by sweeping back and forth. C-SCAN adds uniform wait times, which matters for disks with rotational latency.

**Example:**

| Algorithm | Strategy | Problem |
|---|---|---|
| FCFS | Arrival order | Unnecessarily long seeks |
| SSTF | Nearest request first | Starvation |
| SCAN | Sweep both directions | Reaches the end even when no requests remain |
| C-SCAN | Jump to one end, then serve | Better uniformity |

**Interview Follow-up:**
- Which is used in practice? (SCAN variants / C-SCAN)
- Difference between SCAN and LOOK? (LOOK reverses at the last request, not the disk end)

**Short Interview Answer:**
Disk scheduling algorithms are FCFS, SSTF, SCAN, C-SCAN and LOOK — SSTF is optimal but starves, so SCAN variants are used in practice.

---

## 48. What is RAID? Explain RAID levels.

**Answer:**
RAID (Redundant Array of Independent Disks) combines multiple disks into one logical volume for better performance, capacity or redundancy. Data is spread across disks using parity or mirroring. Key levels: RAID 0 (striping, no redundancy, fastest), RAID 1 (mirroring, full redundancy), RAID 5 (striping with single distributed parity, needs 3+ disks), and RAID 10 (mirrored stripes).

**Example:**

| Level | Technique | Min disks | Redundancy | Performance |
|---|---|---|---|---|
| 0 | Striping | 2 | None | Best |
| 1 | Mirroring | 2 | Full | Read fast, write slower |
| 5 | Single distributed parity | 3 | 1 disk | Good read, write penalty |
| 10 | Mirrored stripes | 4 | 1 disk per pair | Best overall |

**Interview Follow-up:**
- RAID 5 write penalty? (Every write requires a read-modify-write — often 4 I/Os)
- Why did RAID 6 exist? (Dual parity — survives 2 disk failures, which 5 can't)
- Hardware vs software RAID? (Software is more flexible; hardware has its own cache)

**Short Interview Answer:**
RAID combines multiple disks for redundancy or performance — RAID 0 for speed, RAID 1 for mirroring, RAID 5 for parity, RAID 10 for both.

---

## 49. What are free space management techniques?

**Answer:**
The main techniques are the free list (linked list of free blocks), bitmap (one bit per block), and extent table (list of free ranges). A bitmap is compact and fast because you can scan for consecutive free blocks quickly, which is why ext4 and NTFS use one. The linked free list becomes slow when fragmented.

**Example:**
On a 1TB disk with 4KB blocks there are ~268M blocks, so a 1-bit-per-block bitmap is only ~32KB — small enough to stay cached.

**Interview Follow-up:**
- Best fit vs first fit? (Best fit minimises leftover size but needs searching; first fit is faster)
- How is metadata about free space stored? (In the superblock, sometimes with a journal)

**Short Interview Answer:**
Free space is tracked with a free list, bitmap or extent table, and bitmaps are the common choice since they're compact and fast to search.

---

## 50. What is journaling in file systems?

**Answer:**
Journaling means the file system first writes the intended metadata change to a log (journal), applies it to the actual structure, then clears the journal. After a crash, the log tells the OS exactly what was in progress, so it rolls forward instead of scanning the whole disk. It mainly protects metadata consistency; ext3 had no journal, ext4 defaults to journalled metadata only for speed.

**Example:**
NTFS and ext4 replay the journal on mount after a power cut, so a long `fsck` becomes a fast, targeted recovery.

**Interview Follow-up:**
- Metadata vs full-data journaling? (Full journaling is safest but 2-3x slower)
- What about actual file contents? (Not journalled — a torn file write can still corrupt data)

**Short Interview Answer:**
Journaling logs metadata changes before applying them so a crash can be recovered quickly by replaying the log instead of a full disk check.

---

## 51. What is the role of the I/O subsystem in OS?

**Answer:**
The I/O subsystem manages all interaction between the CPU and devices — it provides a uniform interface, hides device specifics, and implements buffering, caching, spooling and scheduling. Through device drivers it translates generic requests into device commands. It decides whether the CPU waits (polling) or gets interrupted.

**Example:**
Your `FileReader.read()` goes through the VFS layer to a filesystem driver to a disk driver to the hardware, and back — the app never sees the hardware.

**Interview Follow-up:**
- What is VFS? (Virtual File System — lets Unix treat all filesystems uniformly)
- Layered vs modular I/O? (Layered = strict stack with lower layers only using the one above; modular = modules can call any other module)

**Short Interview Answer:**
The I/O subsystem provides a uniform interface between CPU and devices, handling drivers, buffering, caching, spooling and request scheduling.

---

## 52. What is a device driver?

**Answer:**
A device driver is kernel-mode software that controls a specific hardware device and translates generic OS I/O requests into device-specific commands. It provides an abstraction so the OS doesn't need to know each device's register layout and protocol. Without it, the device is unusable.

**Example:**
The GPU driver, printer driver and your Wi-Fi driver are all device drivers — usually the biggest source of OS instability when buggy.

**Interview Follow-up:**
- Character vs block device? (Char = byte stream, e.g. keyboard; block = fixed-size blocks, e.g. disk)
- User-mode vs kernel-mode driver? (Some modern GPUs/USB run in user mode for crash isolation)

**Short Interview Answer:**
A device driver is kernel software that controls specific hardware and translates OS I/O requests into device commands.

---

## 53. What is an interrupt?

**Answer:**
An interrupt is a hardware or software signal that temporarily suspends the CPU's current execution and transfers control to a handler. It lets the CPU respond to events immediately instead of constantly polling. After the handler finishes, execution resumes where it left off.

**Example:**
When you press a key, the keyboard controller raises an interrupt; the CPU saves its context, runs the driver's handler, and puts the character in the buffer.

**Interview Follow-up:**
- How many can happen at once? (Maskable ones can be masked; NMI cannot be blocked)
- Context switch vs interrupt? (Interrupt = jump to a handler; context switch = change which task runs)

**Short Interview Answer:**
An interrupt is a signal that pauses the CPU to run a handler immediately, then resumes — that's how devices notify the OS without polling.

---

## 54. What are the types of interrupts?

**Answer:**
Interrupts are hardware or software. Hardware interrupts come from external devices (I/O) or internal errors (exceptions like divide-by-zero, page fault). Software interrupts are triggered deliberately by instructions — traps, syscalls and int instructions. Hardware interrupts are asynchronous; software ones are synchronous.

**Example:**

| Type | Source | Example |
|---|---|---|
| External hardware | Devices | Keyboard, disk, timer |
| Internal (trap/exception) | CPU-detected error | Page fault, divide by zero |
| Software | Instruction | `int`, syscall |

**Interview Follow-up:**
- What's the difference between an interrupt and an exception? (Interrupt is external and async; exception is internal and sync)
- Is a page fault an interrupt? (A trap — a software-invoked exception)

**Short Interview Answer:**
Interrupts are hardware (external devices or internal exceptions) or software (syscalls and traps) — hardware ones are asynchronous, software ones synchronous.

---

## 55. What is interrupt-driven I/O?

**Answer:**
Interrupt-driven I/O is where the CPU issues a read/write command and moves on, letting the device raise an interrupt when it's done. This avoids wasting CPU cycles waiting for slow devices. It only works well when the transfer is large relative to the overhead of the interrupt.

**Example:**
Reading a 1MB file with interrupt-driven I/O triggers only a handful of interrupts instead of blocking per byte.

**Interview Follow-up:**
- When is it inefficient? (For small, frequent transfers — the interrupt overhead dominates)
- Which I/O mode is fastest for large transfers? (DMA)

**Short Interview Answer:**
Interrupt-driven I/O lets the CPU issue a command and continue working, getting notified when the device finishes, instead of busy-waiting.

---

## 56. What is polling?

**Answer:**
Polling means the CPU repeatedly checks a device status register in a loop to see if it's ready. It's simple and low-latency for very fast devices, but it wastes CPU cycles while waiting. It's exactly what interrupt-driven I/O avoids.

**Example:**
A spinlock is polling in software — the CPU spins checking a flag instead of sleeping. Good for very short waits, terrible for long ones.

**Interview Follow-up:**
- Polling vs busy wait? (Polling usually means checking a device; busy wait is spinning on memory/conditions)
- When is polling still used? (High-speed DMA devices, GPU fences, very short critical sections)

**Short Interview Answer:**
Polling is when the CPU repeatedly checks device readiness in a loop — simple but CPU-wasteful, which is why interrupts replaced it.

---

## 57. What is DMA (Direct Memory Access)?

**Answer:**
DMA lets a dedicated controller transfer data directly between a device and memory without the CPU copying every byte. The CPU just sets up the buffer address, length and direction once, and gets an interrupt when the transfer completes. This frees the CPU for other work and makes bulk I/O much faster.

**Example:**
Downloading a 100MB file with DMA: the CPU performs one setup and can serve other requests while the disk writes straight into the network buffer.

**Interview Follow-up:**
- Cycle stealing vs burst mode? (Burst blocks the bus and transfers a lot; cycle stealing interleaves to stay responsive)
- What is a bus master? (A device that can itself master the bus, like NVMe)

**Short Interview Answer:**
DMA is a controller that moves data directly between a device and memory, so the CPU only sets up the transfer and gets interrupted at the end.

---

## 58. What is buffering?

**Answer:**
Buffering is storing data temporarily in a memory region while it's transferred between producer and consumer, to smooth out speed mismatches. It lets a fast producer feed a slow device in appropriate chunks. It's especially useful with block-oriented devices that can't handle small or variable-size I/O.

**Example:**
Writing to a disk: instead of 100 tiny 10-byte writes, you buffer them and issue one 1KB write, which is far more efficient.

**Interview Follow-up:**
- Single vs double vs circular buffering? (Single = one buffer; double = producer/consumer overlap; circular = wrap-around)
- Buffering vs caching? (Buffering handles speed mismatch; caching avoids duplicate I/O)

**Short Interview Answer:**
Buffering holds data temporarily to smooth speed mismatches between a producer and a device, reducing the number of I/O operations.

---

## 59. What is spooling?

**Answer:**
Spooling is a technique where one device's output is first written to an intermediate disk file, and another process prints from it later. It turns an exclusive, slow device into a shareable one, letting multiple processes queue their requests. It also allows output to go to a printer while the user continues working.

**Example:**
Sending five documents to one printer — each goes to a spool file, and the printer daemon processes them one by one in the background.

**Interview Follow-up:**
- Buffering vs spooling? (Buffering is in memory for speed; spooling uses disk to allow sharing and deferred output)
- Where else is spooling used? (Print queues, fax, and some backup systems)

**Short Interview Answer:**
Spooling redirects a device's output to a disk file so multiple processes can share the device and continue working while it processes the queue.

---

## 60. What is caching?

**Answer:**
Caching keeps a copy of frequently accessed data in a faster location, so future requests avoid the slower device. It exploits locality — temporal (recently used) and spatial (nearby addresses) — and results in hits or misses. Caching trades memory for speed.

**Example:**
A CPU cache holds recently used instructions and data in L1/L2/L3; a page cache holds recently used disk blocks in RAM.

**Interview Follow-up:**
- CPU cache levels? (L1 fastest/smallest, L2, L3 larger/slower, then RAM)
- Buffer vs cache? (Caching serves repeat requests; buffering manages producer/consumer timing)
- Hit ratio formula? (hits / total requests — the key metric for cache effectiveness)

**Short Interview Answer:**
Caching stores frequently accessed data in a faster medium (CPU cache, page cache) so repeat requests skip the slow device, trading memory for speed.

## 61. What is CPU scheduling?

**Answer:**
CPU scheduling is the OS mechanism that decides which ready process or thread gets the CPU next, when it gets it, and for how long. A single CPU can only run one task at a time, so the scheduler time-shares it among everything in the ready queue using a policy that balances CPU utilisation, throughput, waiting time and responsiveness. Dispatch happens through the dispatcher, which hands the CPU over.

**Example:**
You type in a browser, watch a video and run a build — the CPU scheduler time-slices all three, keeping the video smooth even while the build saturates the cores.

**Interview Follow-up:**
- Where does it happen? (In the kernel, on every context switch, preemption, and process state change)
- Does it apply to threads too? (Yes — modern OSs schedule threads, since threads are the unit of CPU execution)
- What is the dispatcher? (The kernel code that performs the actual context switch and jumps to the new process)

**Short Interview Answer:**
CPU scheduling is the OS deciding which ready task runs on the CPU next and for how long, using a policy that balances efficiency and responsiveness.

---

## 62. What are the types of schedulers?

**Answer:**
Three types: the long-term (job/admission) scheduler chooses which processes move from disk into memory, the short-term (CPU) scheduler picks the next process from the ready queue to run, and the medium-term scheduler swaps processes out of memory and later back in to relieve memory pressure. Some systems also have scheduler classes or per-level policies (as in MLFQ). See also Q16.

**Example:**
On a desktop, the long-term scheduler admits new apps when memory frees up, the short-term scheduler runs the active threads, and the medium-term scheduler swaps out background tabs to keep RAM available.

**Interview Follow-up:**
- Does Windows/Linux have a long-term scheduler? (Not a classic one — admission is handled by the memory manager, and Windows has a "system" scheduler thread)
- Which is most important? (Short-term — it directly affects responsiveness)

**Short Interview Answer:**
There are long-term (admission), short-term (CPU) and medium-term (swapping) schedulers, each controlling a different stage of a process's life.

---

## 63. What is preemptive vs non-preemptive scheduling?

**Answer:**
In non-preemptive scheduling, once a process gets the CPU it runs until it terminates or blocks (voluntarily gives up the CPU). The OS cannot take the CPU back. In preemptive scheduling, the OS can interrupt a running process and switch to a higher-priority or simply next process — based on time slice, priority, or interrupt. Preemptive gives better responsiveness and fairness; non-preemptive has less context-switch overhead but can starve a process behind a long CPU-bound one.

**Example:**
Non-preemptive: a printer job starts and the OS won't switch away until it blocks. Preemptive: Linux interrupts a CPU-hogging process every few milliseconds so the mouse and keyboard stay responsive.

**Interview Follow-up:**
- Which do real OSs use? (Almost all general-purpose OSs are preemptive — Windows, Linux, macOS)
- What makes preemption possible? (A timer interrupt that forces a reschedule)
- Downside of preemption? (Context-switch overhead and possible race conditions if a process isn't designed for it)

**Short Interview Answer:**
Non-preemptive scheduling lets a process keep the CPU until it blocks or finishes, while preemptive scheduling lets the OS take the CPU back and switch tasks — the latter is far more responsive.

---

## 64. What are CPU scheduling criteria?

**Answer:**
The main criteria are CPU utilisation, throughput, turnaround time, waiting time and response time, plus fairness, priority support and context-switch cost. They conflict — maximising throughput can hurt response time — so the scheduler optimises for the workload: batch systems chase throughput, interactive and real-time systems chase response time and predictability. See also Q17.

**Example:**

| Criterion | Definition | Favoured by |
|---|---|---|
| CPU utilisation | Fraction of time CPU is busy | Batch |
| Throughput | Tasks finished per unit time | Batch |
| Turnaround time | Arrival → completion | Batch |
| Waiting time | Total time in ready queue | General |
| Response time | Arrival → first response | Interactive / RT |

**Interview Follow-up:**
- Which single metric is used for real-time OSs? (Deadline/response predictability, not throughput)
- Do these conflict? (Yes — SJF minimises average waiting time but can starve, hurting fairness)

**Short Interview Answer:**
Scheduling criteria are CPU utilisation, throughput, turnaround time, waiting time and response time — the important one depends on whether the workload is batch, interactive or real-time.

---

## 65. What is FCFS scheduling?

**Answer:**
First-Come-First-Served is the simplest algorithm: the process that arrives first is placed at the back of a FIFO queue and runs to completion. There is no preemption and no consideration of burst time, so it's easy to implement and cheap in overhead. Its weaknesses are the convoy effect — one long process delays everyone behind it — and poor average waiting time, plus unbounded wait for short jobs. It's non-preemptive.

**Example:**
Processes arrive with burst times 8, 4, 1: P1 runs 8 units, P4, P1 — and the 1-unit process waits the whole 8 units despite finishing fastest.

**Interview Follow-up:**
- Is FCFS fair? (Fair in arrival order, not in time — a long job blocks short ones)
- Where is it still used? (Disk scheduling queues, some print/batch queues, and as a tie-breaker)
- Convoy effect? (Many short processes stuck behind one long one, wasting CPU wait time)

**Short Interview Answer:**
FCFS is a non-preemptive FIFO scheduling algorithm — the earliest-arriving process runs to completion. It's simple and fair in order, but suffers from the convoy effect and high average waiting time.

---

## 66. What is SJF scheduling?

**Answer:**
Shortest Job First always runs the process with the shortest CPU burst. Non-preemptive SJF admits it in arrival order but schedules by burst; preemptive SJF (also called SRTF — Shortest Remaining Time First) can interrupt a running process if a new shorter job arrives. SJF provably minimises average waiting time, but burst time isn't known in advance, so it's estimated (often by exponential averaging from history). It can starve long processes, fixed with aging.

**Example:**
Bursts 6, 8, 7, 3: SJF runs the 3-unit job first, dropping average waiting time from 16 (FCFS) to about 8.3.

**Interview Follow-up:**
- SJF preemptive vs non-preemptive? (SRTF preempts, plain SJF doesn't — SRTF minimises average waiting but causes more context switches)
- How is burst time known? (Estimated, e.g. from the previous burst of the same process)
- How to avoid starvation? (Aging — increase priority as waiting time grows)

**Short Interview Answer:**
SJF runs the process with the shortest CPU burst to minimise average waiting time; preemptive SJF (SRTF) interrupts a longer running job. It needs burst estimation and needs aging to prevent starvation.

---

## 67. What is priority scheduling?

**Answer:**
In priority scheduling, each process gets a priority number and the highest-priority ready process runs first. Priorities may be static (fixed) or dynamic (adjusted by the OS, e.g. by aging). It's the basis of real-time and OS-internal scheduling — the kernel prioritises interrupts and system tasks over user work. Its main problem is starvation of low-priority processes; priority inversion is another problem when a high-priority task waits on a low-priority holder of a lock (fixed by priority inheritance).

**Example:**
An OS runs disk-write and input-handler system threads at high priority and a user's background backup at low priority, so the system never feels stuck.

**Interview Follow-up:**
- Static vs dynamic priority? (Static is simpler and predictable; dynamic adapts via aging or feedback)
- What is priority inversion? (A high-priority task blocked on a lock held by a lower-priority one — fixed with priority inheritance / mutexes)
- Real-time priority types? (Rate monotonic, EDF — EDF gives optimal utilisation but needs known deadlines)

**Short Interview Answer:**
Priority scheduling runs the highest-priority ready process first, with static or dynamic priorities. It's how real-time systems work, and starvation of low-priority tasks is handled with aging.

---

## 68. What is round robin scheduling?

**Answer:**
Round Robin is preemptive and fair: each ready process gets a fixed time quantum, and when the quantum expires the process is preempted and pushed to the back of the queue. It cycles through everyone, so waiting time is bounded and response is quick. It's ideal for time-sharing and interactive systems. The key design choice is quantum size — too small and context-switch overhead dominates, too large and it degenerates toward FCFS.

**Example:**
With a 4ms quantum and processes needing 10ms, 4ms, 6ms: P1(4) → P2(4) → P1(4) → P2 done → P3(4) → P1 done → P3(2), giving a much better average wait than FCFS.

**Interview Follow-up:**
- Ideal quantum? (Roughly 80% of the average CPU burst, or a few ms on modern systems)
- Does context-switch cost hurt? (Yes — with a 1ms quantum and 2µs switch, ~0.2% overhead is acceptable; much smaller quanta waste most of the time)
- Where is RR used? (Time-sharing systems, Windows per-quantum, and modern schedulers use similar slice-based ideas like Linux CFS's target latency)

**Short Interview Answer:**
Round Robin gives each ready process a fixed time quantum in rotation, giving bounded waiting and quick response — the classic choice for interactive time-sharing systems.

---

## 69. What is multilevel queue scheduling?

**Answer:**
Multilevel queue scheduling permanently partitions processes into separate queues, each with its own scheduling algorithm and priority. A common split is foreground/interactive processes (queue 1, RR with a short quantum) versus background/batch processes (queue 2, FCFS). Foreground queues always preempt lower queues, so interaction stays snappy while long batch work fills spare cycles. If a foreground process is CPU-bound, it can't be pushed down — that's the drawback MLFQ solves.

**Example:**
Interactive shell and editor commands get short RR quanta and always preempt, while a nightly report job runs FCFS in the background queue whenever the CPU frees up.

**Interview Follow-up:**
- Key limitation? (No way for a process to change queue — CPU-bound foreground work can starve background work)
- Difference from priority scheduling? (Each queue has its own policy, not just a number)
- How do you fix the limitation? (Add feedback — see MLFQ, Q70)

**Short Interview Answer:**
Multilevel queue scheduling permanently splits processes into separate queues, each with its own algorithm (e.g., RR for interactive, FCFS for batch), with higher queues preempting lower ones.

---

## 70. What is multilevel feedback queue (MLFQ) scheduling?

**Answer:**
MLFQ keeps multiple queues with different time quanta and priorities, but processes move between them based on how much CPU they've used. A process that keeps using its full quantum gets demoted to a lower, longer-quantum queue; one that blocks early (interactive) stays high. This lets the scheduler learn process behaviour without knowing it in advance. Periodically, priority boosting moves everything back up to prevent starvation. It's the closest real-world model to what modern schedulers do (Windows, BSD, and the pre-CFS Linux behaviour).

**Example:**
A shell prompt that returns immediately stays in the high-priority short-quantum queue and feels instant; a compile that burns CPU for 30s gradually drops to the low long-quantum queue and never delays the prompt.

**Interview Follow-up:**
- How does it prevent starvation? (Periodic priority boost / periodic reassignment to the top queue)
- Which OS uses it? (FreeBSD, NetBSD, OpenBSD, and Windows uses a similar multi-level approach)
- Why does priority boosting help interactive vs batch? (Recently-used pages get a cache boost; the OS refreshes a process's "credit" so it doesn't stay demoted)

**Short Interview Answer:**
MLFQ uses multiple queues with different quanta and moves processes up or down based on observed CPU usage, so the scheduler learns which processes are interactive and which are batch — with periodic boosting to avoid starvation.

---

## 71. What is system protection in OS?

**Answer:**
System protection is the set of mechanisms the OS uses to guard the system from faults, misuse and damage. It spans hardware-level protection (dual mode operation, MMU, page protection bits, privileged instructions), OS-level protection (system calls as the only controlled entry, resource quotas, process isolation), and recovery (backups, journaling, safe shutdown). The goal is that no single program, user or hardware glitch can compromise the whole system.

**Example:**
A buggy process tries to write to another process's memory → the MMU raises a protection fault → the kernel kills only that process; the rest of the system and its files stay safe.

**Interview Follow-up:**
- Protection vs security? (Protection is integrity/isolation; security also includes confidentiality and authentication)
- Key hardware features? (Dual mode, privilege levels, MMU, page protection bits)
- Examples of fault tolerance? (RAID, journaling, ECC memory, safe mode boot)

**Short Interview Answer:**
System protection is everything the OS does to prevent faults and misuse from harming the system — dual-mode operation, memory protection, process isolation, controlled system calls and recovery from failures.

---

## 72. What is authentication and authorization?

**Answer:**
Authentication is verifying *who* you are — proving identity with credentials (password, fingerprint, token, certificate). Authorization is deciding *what* you're allowed to do once identity is established — permissions, roles, ACLs. In short: authentication = identity, authorization = permission. They're always sequential — you can only authorize after authenticating.

**Example:**
You log in with a password and fingerprint (authentication); the OS then checks your role and grants read access to a folder but denies write (authorization via ACL/permissions).

**Interview Follow-up:**
- Difference in one line? (Authentication = are you who you say you are; authorization = what may you do)
- 2FA? (A second factor — something you know/have/are — dramatically stronger than a password alone)
- Examples of authorization models? (Role-based (RBAC), attribute-based (ABAC), access control lists (ACLs))

**Short Interview Answer:**
Authentication verifies identity (password, token, biometrics), while authorization decides what an authenticated user is permitted to do based on roles, policies or ACLs.

---

## 73. What is access control?

**Answer:**
Access control is the mechanism that enforces which users or processes can read, write, execute, or delete a resource. Access rights are attached to objects (files, directories, devices) and enforced by the OS through ACLs, permissions (rwx), mandatory access control (labels and policy, e.g. SELinux), and capability systems. The classic model is the access matrix (rows = subjects, columns = objects), implemented in practice as ACLs or capabilities.

**Example:**
Unix file permissions `rwxr-x---` let the owner read/write/execute, the group read/execute, and everyone else nothing — enforced on every file open by the kernel.

**Interview Follow-up:**
- ACL vs permission bits? (Permissions give three fixed classes; ACLs list arbitrary users/groups per file)
- Discretionary vs mandatory access control? (DAC — owner decides, flexible; MAC — system policy decides, used for military/high-security)
- Authentication vs access control? (Authentication proves who; access control decides what they can reach)

**Short Interview Answer:**
Access control is how the OS enforces permissions on resources — via rwx permissions, ACLs, or mandatory policies (SELinux) — so only authorised users/processes can read, write or execute them.

---

## 74. What is the difference between multitasking, multithreading, multiprocessing, and multiprogramming?

**Answer:**
Multiprogramming keeps several programs in memory and switches the CPU between them so the CPU is never idle. Multitasking is that same idea from the user's perspective — multiple tasks appear to run at once, typically by time-sharing. Multiprocessing runs multiple processes truly in parallel on multiple CPU cores. Multithreading runs multiple threads of one program concurrently (on multiple cores or interleaved on one), sharing one address space. Multiprogramming/multi-tasking is about switching between programs; multithreading is about concurrency inside one program; multiprocessing is about real parallelism on many cores.

**Example:**
Your OS is multiprogramming (Word, Chrome, Excel all loaded), multitasking (each feels responsive), multiprocessing (spreads across 8 cores), and Chrome internally multithreads (each tab's rendering on its own thread).

**Interview Follow-up:**
- Multi-tasking vs multi-programming? (Multiprogramming = switching between programs for CPU utilisation; multitasking = concurrent user-facing task experience built on it)
- Multithreading vs multiprocessing for CPU-bound work? (Both scale across cores; threads share data cheaply but need locking, processes are isolated but pay IPC cost)

**Short Interview Answer:**
Multiprogramming/multitasking switches the CPU among several loaded programs for utilisation; multithreading runs threads of one program concurrently sharing memory; multiprocessing runs separate processes in parallel on multiple cores.

---

## 75. What is the difference between a monolithic kernel and a microkernel?

**Answer:**
A monolithic kernel puts the whole OS — scheduling, memory, filesystems, drivers, network stack — in one big kernel-mode process communicating by direct function calls. It's fast (low overhead) but a single driver bug crashes everything. A microkernel keeps only the bare minimum in kernel mode (IPC, basic scheduling, memory) and moves filesystems, drivers and services to user-space daemons communicating via message passing. It's more modular, secure and easier to verify, but IPC overhead makes it slower. Real systems often split the difference (see hybrid kernel, Q76).

**Example:**
Linux is monolithic (drivers in kernel space) — a buggy NVIDIA driver can panic the whole machine. Windows NT and QNX are microkernel/hybrid, running most drivers as user-mode processes so a crash isolates the device.

**Interview Follow-up:**
- Pure example of microkernel? (QNX, seL4, MINIX 3)
- Which is faster and why? (Monolithic — no IPC, direct calls and shared address space)
- Why don't all OSs go microkernel? (Performance cost, plus drivers need fast, direct hardware access)

**Short Interview Answer:**
A monolithic kernel runs all OS services in kernel space (fast but one bug can crash everything), while a microkernel keeps only core functions in the kernel and runs services as user processes (modular and safer but slower due to IPC).

---

## 76. What is a hybrid kernel?

**Answer:**
A hybrid kernel is the compromise most modern OSs use: it's structured like a microkernel (modular, some services as user-mode servers) but keeps performance-critical parts — like the filesystem and some drivers — in kernel space for speed. It also usually supports loading kernel modules dynamically. Windows NT, macOS (XNU) and Android's Linux are commonly described as hybrid or monolithic-hybrid.

**Example:**
Windows runs the NT Executive and drivers mostly in kernel mode but isolates some drivers and subsystems in user mode (usermode drivers), getting microkernel-like reliability with monolithic-like speed.

**Interview Follow-up:**
- Is Linux hybrid? (It's monolithic, but with loadable modules and extensive use of user-space daemons — a "hybrid" in practice)
- What are kernel modules? (.ko loadable pieces of kernel code — feature modularity, but each is a crash risk)

**Short Interview Answer:**
A hybrid kernel mixes microkernel modularity with monolithic performance — some services run as user-mode servers, critical ones stay in kernel mode. Windows NT and XNU are examples.

---

## 77. What is a shell?

**Answer:**
A shell is a command interpreter — the program that reads what you type, parses it, resolves paths, launches the requested program, and manages things like pipes, redirection and background jobs. It's a user-space program (not part of the kernel) that exposes the OS's power through a text interface. Common shells: bash, zsh, fish, cmd.exe, PowerShell.

**Example:**
Typing `ls -l | grep ".java" > out.txt` — the shell (bash) parses the pipeline, runs `ls`, pipes to `grep`, redirects output to `out.txt`, and waits.

**Interview Follow-up:**
- System vs user shell? (Login shell for the user, e.g. bash; system shell used by scripts/services, e.g. /bin/sh)
- Is shell part of the OS? (It's a user program, but essential interface; kernels don't include it)
- How do Windows shells differ? (cmd.exe is a command interpreter; PowerShell is object-based with a .NET runtime)

**Short Interview Answer:**
A shell is a command interpreter that reads your input, runs programs, and wires together pipes, redirection and background jobs — like bash or PowerShell.

---

## 78. What is booting? What is the difference between cold boot and warm boot?

**Answer:**
Booting is the sequence that brings a machine from power-on to a ready state: the firmware (BIOS/UEFI) initialises hardware, loads the bootloader (GRUB, Windows Boot Manager), which loads the kernel, which mounts the root file system and starts `init`/`systemd`, which launches services and finally the login screen/GUI. A cold boot starts from power-off — the full sequence runs. A warm boot restarts without removing power, skipping some hardware initialisation and reloading the kernel more quickly.

**Example:**
A cold boot (power off then on) runs POST → UEFI → GRUB → kernel → systemd. A warm boot (Start → Restart) skips the full POST and reinitialises the OS faster.

**Interview Follow-up:**
- What does the bootloader do? (Locates and loads the kernel; GRUB also lets you pick OS/kernel)
- What is a live boot? (Loading the OS from USB/CD without installing to disk)
- What happens in single-user mode? (Only root is started for recovery — used to reset passwords)

**Short Interview Answer:**
Booting is the power-on sequence: firmware init → bootloader → kernel → init system → services → login. A cold boot starts from power-off (full init), while a warm boot restarts without losing power and skips some hardware re-initialisation.

---

## 79. What is a system call interface?

**Answer:**
The system call interface is the defined API through which user programs request privileged kernel services. A library routine (e.g. `read()` in libc) places arguments in registers, executes a trap instruction, and the kernel's handler runs the service and returns a result. The interface is the security boundary — it validates every argument, enforces the process's rights, and is the only legitimate way for user code to touch the kernel. Examples include POSIX, Win32, and Java's JNI layer on top of syscalls.

**Example:**
`open("file.txt", O_RDWR)` → libc syscall → trap to kernel → validate path & permissions → return file descriptor or -1/errno.

**Interview Follow-up:**
- How are arguments passed? (In registers per the calling convention; `strace` shows them)
- Are all syscalls traps? (Most are traps; a few are asynchronous (signals) or use special instructions)
- What is strace/ltrace? (Tools to observe syscalls and library calls a process makes)

**Short Interview Answer:**
The system call interface is the API and controlled entry point where user programs request kernel services (file I/O, memory, processes) — arguments are validated, the mode switches to kernel, and a result is returned.

---

## 80. Name some common operating systems and their kernels.

**Answer:**
Common OSs and their kernel type: Windows (Windows NT — hybrid, macro-kernel), Linux (monolithic, modular), macOS (XNU — hybrid, Mach microkernel + BSD), Android (Linux monolithic kernel with an Android runtime layer), iOS (XNU hybrid), Unix/BSD (varies, often hybrid, e.g. FreeBSD hybrid), Chrome OS (Linux). Most real OSs are monolithic or hybrid; pure microkernels are research/embedded (seL4, QNX).

**Example:**

| OS | Kernel | Type |
|---|---|---|
| Windows 10/11 | Windows NT | Hybrid |
| Linux (Ubuntu, RHEL) | Linux | Monolithic (modular) |
| macOS | XNU (Mach + BSD) | Hybrid |
| Android | Linux kernel | Monolithic + user-space framework |
| FreeBSD | FreeBSD kernel | Hybrid |

**Interview Follow-up:**
- Is Linux modular or monolithic? (Monolithic by design but modular in practice via loadable kernel modules)
- What is XNU made of? (Mach microkernel + BSD subsystems + IOKit drivers)
- Do all Unix kernels match? (No — Linux, BSD, Solaris and macOS have very different kernel designs)

**Short Interview Answer:**
Windows uses the hybrid NT kernel, Linux is a modular monolithic kernel, macOS uses the hybrid XNU (Mach + BSD), and Android runs on a Linux kernel. Pure microkernels (QNX, seL4) exist mainly in embedded/real-time use.

---

## 81. What is the difference between hard real-time and soft real-time systems?

**Answer:**
In a hard real-time system, missing a deadline is a system failure — the answer is wrong or a harm occurs. Deadlines are fixed, worst-case execution time must be bounded, and the scheduler uses fixed priorities (rate monotonic). In a soft real-time system, a missed deadline degrades quality but isn't catastrophic — late frame dropped, slightly late answer still useful. Soft RT can use dynamic priorities (EDF), preemption and statistical guarantees. Examples: hard — aircraft flight control, pacemaker, car brake ABS; soft — video streaming, live audio, gaming, VoIP.

**Example:**
An anti-lock brake system must react within a hard deadline or the car crashes (hard RT); a video call that drops a few frames is still usable (soft RT).

**Interview Follow-up:**
- Which is harder to build? (Hard RT — needs worst-case bounds, no dynamic allocation, no unbounded paging)
- Common scheduling? (Rate monotonic for hard; earliest-deadline-first for soft)
- Why avoid virtual memory in hard RT? (Unpredictable page-fault latency breaks the deadline guarantee)

**Short Interview Answer:**
Hard real-time demands deadlines always be met (failure if missed, e.g. flight control); soft real-time tolerates occasional misses that only degrade quality (e.g. video streaming).

---

## 82. What is the difference between preemptive and non-preemptive scheduling?

**Answer:**
(Same as Q63 — repeated for emphasis.) Non-preemptive: a running process keeps the CPU until it terminates or blocks. Preemptive: the OS can forcibly suspend it and run another process — via a timer tick, higher-priority arrival, or interrupt. Preemptive gives better response and fairness but costs more context switches; non-preemptive is cheaper but risks long uninterruptible runs. All major general-purpose OSs are preemptive.

**Example:**
Printing a long job non-preemptively would freeze your keyboard; preemptive time-slicing keeps the UI responsive.

**Interview Follow-up:**
- What triggers preemption most often? (The periodic timer interrupt ending the time slice)
- Any downside to preemption for CPU-bound math? (Yes — if a job needs to hold shared state, a mid-computation switch can cause livelock unless synchronised)

**Short Interview Answer:**
Non-preemptive lets a process hold the CPU until it finishes or blocks; preemptive lets the OS interrupt and switch tasks mid-run for better responsiveness, at the cost of extra context switches.

---

## 83. What happens during a page fault?

**Answer:**
The steps: the process accesses a page; the MMU finds the PTE's present bit is 0 and raises a page fault trap to the kernel. The kernel saves the faulting context and checks the address — if it's within the process's address space but not resident (valid reference), it locates the page on disk, reads it into a free frame (evicting a victim if needed via page replacement), updates the PTE and TLB, then resumes the process by re-executing the faulting instruction (transparent to the program). If the address is illegal (out of bounds, unmapped), the kernel sends SIGSEGV and kills the process.

**Example:**
Touching a lazily-loaded page: `malloc` succeeds (virtual), the first write to that memory triggers a fault, the OS reads the page from swap/file, and the write proceeds a microsecond-to-millisecond later.

**Interview Follow-up:**
- Minor vs major fault? (Minor = page already in RAM (page cache); major = disk read required — the expensive one)
- Why re-execute the instruction? (The instruction's operands weren't available; now they are)
- Does a page fault always mean a bug? (No — in demand paging, faults are the normal mechanism for loading pages)

**Short Interview Answer:**
On a page fault the kernel checks if the address is valid; if yes it loads the page from disk into a free frame (possibly evicting another), updates the page table, and restarts the instruction — if the address is invalid, it crashes the process with a segfault.

---

## 84. What is page replacement? Name page replacement algorithms.

**Answer:**
Page replacement is the OS's policy for choosing which memory page to evict when a free frame is needed but none is available (to make room for a faulted page). Good replacement should remove a page that isn't referenced soon (ideally one not in the working set) to minimise faults. Algorithms: Optimal (MIN, theoretical best), FIFO, LRU, LFU, and approximations like Clock/Second-Chance, and MRU. A page that was already written to disk must be written back (dirty bit) before eviction.

**Example:**
Clock (Second Chance) uses a reference bit per page: evict the first page with reference bit 0, set it to 1 and move on; pages used recently get a "second chance." It approximates LRU cheaply and is what real OSs use.

**Interview Follow-up:**
- Which is best? (Optimal — impossible in practice; LRU approximates it but needs hardware support or extra bookkeeping)
- Why is FIFO bad? (It can evict hot pages — "Belady's anomaly")
- Stack vs heap page replacement? (Stack is simpler but limited; heap is general and is what's used)

**Short Interview Answer:**
Page replacement picks which resident page to evict when a new page must be brought in. Algorithms include Optimal, FIFO, LRU, LFU and Clock/Second-Chance (LRU's practical approximation).

---

## 85. What is thrashing and how can it be prevented?

**Answer:**
Thrashing is the state where processes spend more time paging than executing — memory is over-committed and the working sets of active processes don't fit, so the system constantly pages in/out and the CPU sits idle. Symptoms: high disk I/O, near-zero CPU utilisation, everything feels slow despite a fast CPU. Prevention: (1) working-set allocation — ensure each process's working set fits; (2) page-fault frequency control — if a process's fault rate is too high, suspend it; (3) reduce multiprogramming level / more frames per process; (4) add more physical RAM. Detection is the easy part; the fix is throttling. See also Q40.

**Example:**
A 16GB laptop with 25 Chrome tabs and a 4GB video editor: page fault rate spikes, disk thrashes, and the whole UI stutters.

**Interview Follow-up:**
- How do you detect it? (Monitor page fault rate per process and overall CPU utilisation — high faults + low CPU is the signature)
- Fix without more RAM? (Lower the multiprogramming level / suspend some processes)
- Does it happen in virtual memory with overcommit? (Yes — overcommit is exactly what enables thrashing)

**Short Interview Answer:**
Thrashing is when processes spend more time paging than executing because memory is over-committed. It's prevented by working-set allocation, page-fault-frequency control (suspending processes), or simply more RAM / fewer concurrent processes.

---

## 86. What is the difference between paging and segmentation?

**Answer:**
Paging divides memory into fixed-size pages/frames and maps them via a page table — it eliminates external fragmentation (at the cost of some internal fragmentation) and is hardware-supported. Segmentation divides memory into variable-size logical segments (code, data, stack) described by a segment table with base+limit — it's programmer-friendly and allows segment growth, but suffers external fragmentation. Modern systems usually combine both: segments for logical organisation, paging underneath for physical placement. See also Q34, Q35.

**Example:**

| Aspect | Paging | Segmentation |
|---|---|---|
| Division | Fixed-size pages | Variable-size segments |
| Fragmentation | Internal | External |
| View | Physical-management oriented | Logical/user-oriented |
| Table | Page table (frame no.) | Segment table (base, limit) |

**Interview Follow-up:**
- Which does x86 use? (Segmented paging — segmentation for protection, paging for placement; mostly vestigial in 64-bit)
- Can you have both? (Yes — segments are the logical view, pages the physical mechanism)

**Short Interview Answer:**
Paging splits memory into fixed-size pages mapped to frames (no external fragmentation), while segmentation uses variable-size logical segments like code/data (programmer-friendly, but causes external fragmentation). Modern systems often use both.

---

## 87. What is the difference between internal and external fragmentation?

**Answer:**
External fragmentation is free memory scattered in many small, non-contiguous holes — total free space may be enough but no single block fits the request; it happens with variable-sized allocations (segmentation, contiguous file allocation) and is fixed by compaction. Internal fragmentation is wasted space inside an allocated block because the block is larger than requested — it happens with fixed-size allocation (paging: a 4KB frame for a 100-byte page) and can't be reused by others. Paging accepts internal fragmentation to eliminate external fragmentation.

**Example:**
Free memory = 10KB total but split as 4KB + 3KB + 3KB, and you request 5KB contiguous → external fragmentation (no block fits). Allocating a 4KB page for 100 bytes of data → internal fragmentation.

**Interview Follow-up:**
- Which does paging cause? (Internal)
- How is external fragmentation fixed? (Compaction, or use paging instead)
- Why is compaction expensive? (Copying all live data to a contiguous region takes time and I/O)

**Short Interview Answer:**
External fragmentation is free memory split into unusable scattered holes (fixed by compaction); internal fragmentation is unused space inside an allocated unit, which fixed-size paging intentionally accepts.

---

## 88. What is Belady's anomaly?

**Answer:**
Belady's anomaly is the surprising case where *adding more frames* (more physical memory) to a system *increases* the number of page faults — the opposite of intuition. It happens with some replacement algorithms (notably FIFO, and LRU is immune) when a page that would have been evicted early now survives and evicts a page that was needed sooner. It proves FIFO is not stack-like and can't be optimised by simply adding memory; it motivates LRU-like and stack algorithms.

**Example:**
With 3 frames FIFO gets 9 faults; with 4 frames FIFO gets 10 — more memory, more faults. LRU gets 10 and 8 respectively (faults drop as expected).

**Interview Follow-up:**
- Which algorithms suffer from it? (FIFO, and most non-stack algorithms; LRU and Optimal do not)
- Why LRU is immune? (It's a stack algorithm — the frame set is always a subset of the n+1-frame set)
- How to avoid? (Use LRU/Clock/Optimal instead of FIFO)

**Short Interview Answer:**
Belady's anomaly is when increasing the number of page frames paradoxically increases page faults (with FIFO), proving that memory alone can't fix a bad replacement algorithm — LRU avoids it.

---

## 89. What is a working set?

**Answer:**
A process's working set is the set of pages it has referenced in the last Δ (delta) references or time window — the pages it's actively using right now. It's a working definition of locality in time. The OS uses it to decide how many frames a process needs, for allocation, and to detect thrashing (if a process's fault rate implies it needs more frames than it has, suspend it).

**Example:**
A word processor: you touch the text buffer, the current paragraph, the font tables — those ~50 pages are the working set; the 5,000 pages of the other documents in the same file are not.

**Interview Follow-up:**
- How is it tracked? (A per-process sliding window of the last Δ references, e.g. a bit vector of page references)
- Fixed vs variable window? (Fixed Δ is simple; variable adapts to locality changes)
- How does it prevent thrashing? (Ensure each running process's working set fits in memory, else suspend one)

**Short Interview Answer:**
The working set is the collection of pages a process has used in its last Δ references, representing its current locality. The OS uses it to allocate frames and detect/prevent thrashing.

---

## 90. What is demand paging?

**Answer:**
Demand paging is a virtual-memory scheme where pages are loaded into physical memory only when the process actually references them, rather than loading everything at once. Unreferenced pages never occupy RAM. It's efficient because most programs use only a fraction of their code/data, and it makes programs start faster and allows overcommit. The cost is page faults and the need for virtual memory that exceeds physical memory. See also Q37.

**Example:**
Running a huge Java app: only the JARs and classes actually loaded into the JVM are paged in; unreferenced libraries stay on disk until needed.

**Interview Follow-up:**
- What is a "valid" vs "invalid" reference? (Valid = within the program's address space; invalid = illegal address → segfault)
- Pure demand paging vs pure swapping? (Demand paging doesn't require swapping — pages can come from the executable file itself)
- Problem it creates? (Page faults and external fragmentation of disk I/O; can cause thrashing)

**Short Interview Answer:**
Demand paging loads a page into memory only on first access, rather than loading the whole program upfront, saving memory and startup time but paying the cost of page faults.

---

## 91. What is the difference between logical and physical address?

**Answer:**
A logical (virtual) address is what the process's code sees — it belongs to the process's own isolated address space and is what the CPU generates from a pointer. A physical address is the real location in RAM. The MMU translates logical→physical on every memory access using the page table, and the mapping is per-process, which is what gives each process the illusion of its own memory starting at 0. See also Q32.

**Example:**
A process dereferences `0x400500` (logical); the page table maps page 1 to frame 7 at base `0x1C000`, so the physical address is `0x1C500`.

**Interview Follow-up:**
- Who performs the translation? (The MMU hardware, driven by the page table the OS maintains)
- Why does it matter? (It gives isolation — the OS can map different physical frames to each process's page 1)
- Which one does the programmer see in a segfault? (The logical address)

**Short Interview Answer:**
The logical/virtual address is what the program uses in its own address space, while the physical address is the actual RAM location; the MMU translates between them on every access using the page table.

---

## 92. What is address binding? What are its types?

**Answer:**
Address binding is the mapping of a program's logical addresses to physical addresses — deciding *when* and *how* a program's addresses are tied to real memory. Three classic types: (1) compile-time binding — the absolute address is fixed in the binary (only works if the load location is known in advance); (2) load-time binding — the absolute address is computed when the program is loaded, so the whole program must be loaded together; (3) execution-time (run-time) binding — addresses are translated per access by the MMU/page table, so the process can be moved and the memory shared. Modern OSes use run-time binding, which is what enables virtual memory.

**Example:**
A statically linked binary with a hardcoded load address is compile-time bound; it fails to load if placed elsewhere. A dynamically linked program using paging is run-time bound — the loader just maps it anywhere.

**Interview Follow-up:**
- Which does modern OS use? (Execution-time/run-time, via MMU + page tables)
- What does run-time binding enable? (Virtual memory, process isolation, swapping/moving a process, shared libraries)

**Short Interview Answer:**
Address binding maps a program's logical addresses to physical ones. Compile-time and load-time binding fix the address before/at loading, while run-time (execution-time) binding translates on every access via the MMU — that's what modern virtual memory uses.

---

## 93. What is swapping?

**Answer:**
Swapping is the process of moving a whole process (or part of it) out of main memory to secondary storage, and later back in, to free physical RAM or improve locality. It's used to support more processes than fit in memory and to bring cold data closer to disk. A swapper (medium-term scheduler) decides what to evict. Swapping the entire process is different from paging (which moves individual pages on demand); pure swapping can bring a whole process in, and with demand paging the concepts overlap.

**Example:**
With 8GB RAM and 20GB of open apps, the OS swaps out the least-recently-used background apps to disk so the active ones stay in RAM.

**Interview Follow-up:**
- Swap vs page? (Swap = whole process; page = individual page on demand)
- What is swap space / pagefile? (Reserved disk area used as backing store for swapped/virtual memory)
- Cost? (Swap I/O is slow, and swapping thrashes when programs need their pages back immediately)

**Short Interview Answer:**
Swapping is moving an entire process between main memory and secondary storage to free RAM or manage more processes than fit in memory, managed by the medium-term scheduler (swapper).

---

## 94. What is the difference between a thread and a process?

**Answer:**
A process is the unit of resource allocation — it has its own address space, open files, and PCB, and is isolated from other processes. A thread is a unit of execution within a process — it has its own program counter, registers and stack but shares the code, data, heap and file descriptors with sibling threads. Threads are far cheaper to create and switch, communicate directly, but because they share memory they need synchronisation and can suffer races and deadlocks. Processes need IPC but give crash and fault isolation. See also Q21.

**Example:**
Chrome uses processes for tabs (a crashing tab shouldn't kill the browser) and threads within each renderer for smooth rendering, event handling and network I/O.

**Interview Follow-up:**
- Context switch cost? (Threads are much cheaper — same address space, no page-table/TLB reload)
- Can threads share memory? (They do by default within a process; processes can also share via shared memory)
- `fork()` in a multithreaded program? (Only the calling thread survives in the child — a classic bug source)

**Short Interview Answer:**
A process is an isolated unit of resource allocation with its own memory, while a thread is a lightweight execution unit that shares its process's memory. Threads are faster and share data cheaply but need locking; processes are isolated but need IPC.

---

## 95. What are the types of threads?

**Answer:**
Threads can be classified by user/kernel level: user-level threads (managed by a library like GNU pthreads or JVM, cheap but one blocking syscall blocks the whole process) and kernel-level threads (managed by the OS, true parallelism across cores, more costly to switch). They can also be classified by purpose/relationship: single-level (one process, many threads), multilevel, and heterogeneous (different roles like the thread pool vs main). Model-wise, many-to-one, one-to-one (POSIX), and many-to-many are the mappings between user threads and kernel threads.

**Example:**
A thread pool in a web server: worker threads are user-level (many threads) mapped to kernel threads by the JVM's one-to-one model, so a blocking JDBC call parks just that thread, not the whole process.

**Interview Follow-up:**
- Many-to-one vs one-to-one? (Many-to-one is cheap but no parallelism and one syscall blocks all; one-to-one gives real parallelism at higher overhead — that's what Java and pthreads use)
- What are daemon threads? (Threads that die when the main exits — e.g. background GC threads)

**Short Interview Answer:**
Threads are user-level (cheap, but a blocking call blocks the process) or kernel-level (true parallelism, costlier). By mapping they can be many-to-one, one-to-one, or many-to-many; by purpose they're single-level, multilevel, or heterogeneous.

---

## 96. What is the difference between multitasking, multithreading, and multiprocessing?

**Answer:**
Multitasking is running several tasks so they appear concurrent, usually by time-sharing one CPU (or across cores). Multithreading runs several threads inside one process, sharing its address space — cheap and allows direct data sharing. Multiprocessing runs several separate processes, each with its own isolated address space — heavier but isolated and truly parallel. Multitasking is the user-visible effect; multithreading and multiprocessing are the two mechanisms (process-level vs thread-level concurrency). See also Q74.

**Example:**
Your OS is multitasking (all apps feel responsive simultaneously); Chrome multithreads each renderer, and spawns multiple processes for sandboxing — both at once.

**Interview Follow-up:**
- Why choose processes over threads? (Isolation — a crash or memory leak in one doesn't kill the others, and you can sandbox untrusted code)
- Do they need multiple cores? (No — even on one core, concurrency is achieved by interleaving; only true parallelism needs multiple cores)

**Short Interview Answer:**
Multitasking is the appearance of running many tasks at once; multithreading achieves it with threads sharing one process's memory, and multiprocessing with isolated separate processes. Threads are lighter but need locking; processes are heavier but isolated.

---

## 97. What is the purpose of TLB?

**Answer:**
The TLB (Translation Lookaside Buffer) is a small, fast cache inside the MMU that stores recently used virtual-page→physical-frame translations. Without it, every memory reference would need a page-table walk — an extra memory access (or several, for multi-level tables) — roughly doubling memory-access cost. The TLB makes the common case a single-cycle lookup, so it's essential to usable virtual memory performance. It works because of locality: a program's working set fits in a few hundred pages. See also Q39.

**Example:**
A 64-entry TLB holding the current process's hot pages means nearly all loads/stores translate in ~1 cycle instead of a 2–4 level page-table walk costing hundreds of cycles.

**Interview Follow-up:**
- TLB miss? (Hardware page-table walk, or a trap to software walk; extra latency per access)
- What causes thrashing of the TLB? (A working set larger than the TLB — "TLB thrashing", common in databases/VMs)
- ASID/PID in the TLB? (Tags the entries per process so the TLB isn't flushed on every context switch)

**Short Interview Answer:**
The TLB is a fast cache of recent virtual-to-physical address translations in the MMU, so most memory accesses avoid a slow page-table walk — it's what makes virtual memory performant.

---

## 98. What is a system call? Give examples.

**Answer:**
A system call is the mechanism by which a user-space program asks the kernel to perform a privileged operation — file I/O, memory management, process control, device access. The call goes through a library stub that loads arguments and executes a trap, switching the CPU from user mode to kernel mode; the kernel validates the arguments against the process's rights and returns a result. It is the only controlled doorway to hardware, which is what keeps the system safe. Examples below.

**Example:**

| Category | Examples |
|---|---|
| Process control | `fork`, `exec`, `exit`, `wait`, `kill` |
| File I/O | `open`, `read`, `write`, `close`, `stat` |
| Memory | `brk`, `mmap`, `mprotect` |
| Other | `socket`, `connect`, `ioctl`, `getpid` |

**Interview Follow-up:**
- How is a system call different from a normal function call? (Mode switch + privilege check + trap into the kernel — much slower)
- Are they always expensive? (Modern kernels use fast paths like vDSO/`syscall`/`sysenter` — e.g. `gettimeofday` may not even enter the kernel)
- Do languages compile to syscalls directly? (Usually a runtime/stdlib does — Java goes through JNI to native, then the syscall)

**Short Interview Answer:**
A system call is the controlled interface where a user program requests a privileged kernel service, switching from user to kernel mode. Examples include `fork`, `exec`, `open`, `read`, `write`, `mmap` and `socket`.

---

## 99. What is the difference between trap and interrupt?

**Answer:**
Both are events that transfer control to the kernel, but they differ in origin and timing. A trap (or exception) is raised *synchronously by the CPU while executing a specific instruction* — a deliberate trap instruction (syscall), or a fault caused by the current instruction (divide-by-zero, page fault, invalid opcode). An interrupt is raised *asynchronously by an external device* at an arbitrary time, unrelated to the instruction being executed (timer, keyboard, disk, I/O). Traps are predictable and tied to the instruction; interrupts are unpredictable and device-driven. Modern systems often blur the line and call traps "exceptions."

**Example:**
A `syscall` is a trap — the CPU deliberately switches to the kernel. A keyboard press is an interrupt — it arrives asynchronously, possibly mid-instruction, and the CPU finishes the current instruction before handling it.

**Interview Follow-up:**
- Is a page fault a trap or an interrupt? (A trap/exception — synchronous, caused by the accessing instruction)
- Can interrupts be masked? (Maskable ones yes; NMI — non-maskable — cannot)
- How does the CPU decide priority? (Interrupt priority levels and the vector table, e.g. x86 IDT)

**Short Interview Answer:**
A trap is a synchronous exception caused by the current instruction (syscall, page fault, divide-by-zero), while an interrupt is an asynchronous signal from an external device (timer, I/O) handled after the current instruction finishes.

---

## 100. What are the goals of an Operating System?

**Answer:**
An OS exists to: (1) **abstract hardware** — present a uniform, easy interface (files, sockets, processes) instead of raw registers; (2) **manage resources** — CPU, memory, storage, devices, so nothing is wasted or double-allocated; (3) **provide concurrency** — many tasks make progress efficiently and independently; (4) **ensure isolation & security** — one program/user can't harm another or the system; (5) **guarantee reliability & fairness** — protection, error recovery, and no starvation; (6) **convenience & portability** — a user interface and a stable environment programs can rely on; and (7) **accounting & monitoring** — usage tracking and performance. The two classic "big goals" are convenience and efficiency (Silberschatz), extended by effectiveness, flexibility and reliability.

**Example:**
You run a text editor: the OS turns keystrokes into characters via a driver, stores the file on disk, schedules the process fairly, isolates it from other programs, and hides all the hardware details so the editor only deals with bytes and keys.

**Interview Follow-up:**
- How do efficiency and convenience conflict? (A fast, direct API exposes hardware; a safe abstract API adds overhead — the OS balances them)
- What's the "resource manager vs virtual machine" view? (Resource manager = efficiency; virtual machine = abstraction and convenience)
- Which goal is hardest in real-time systems? (Meeting deadlines — predictability must be designed in, not hoped for)

**Short Interview Answer:**
An OS's goals are to abstract the hardware, manage resources efficiently, enable concurrency, isolate and protect processes, guarantee reliability and fairness, and give users a convenient, portable environment.

<!-- OS-MORE -->