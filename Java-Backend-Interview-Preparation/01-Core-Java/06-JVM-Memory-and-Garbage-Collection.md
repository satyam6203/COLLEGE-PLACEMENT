# Chapter 6: JVM, Memory and Garbage Collection

**Chapter goal:** Explain how the JVM loads classes, manages memory, executes bytecode, and collects garbage — and diagnose real production issues such as `OutOfMemoryError`, `StackOverflowError`, leaks, and bad GC pause times.

**Topics covered:** JVM architecture · class loading · class loader delegation · runtime data areas · heap and stack · metaspace · object layout · GC roots · mark-sweep-compact · generational GC · Serial/Parallel/CMS/G1/ZGC · G1 internals · GC tuning · OOM errors · memory leaks · reference types · monitoring and troubleshooting.

**Cross-references:** Java 8 functional internals are in [Chapter 5](05-Java-8-Features-and-Streams.md). Collection internals are in [Chapter 7](../02-Collections/07-Collections-Framework.md). Locks and threads are in [Chapter 8](../03-Concurrency/08-Multithreading-and-Concurrency.md).

---

## Part A — JVM architecture and class loading

### Question 1: Explain the JVM architecture and the lifecycle of a Java program

**1. Interview-ready answer**

The JVM is an abstract virtual machine specified by the Java Virtual Machine Specification; HotSpot is the implementation most of us run. A `.java` file is compiled to `.class` bytecode, the class loader subsystem loads, links, and initializes it, the execution engine runs bytecode through the interpreter and the JIT compiler, and garbage collection reclaims unreachable heap objects. Native calls go through JNI to operating-system libraries.

**2. Detailed explanation**

```text
.java source
   |
javac
   v
.class bytecode
   |
Class Loader Subsystem: loading -> linking -> initialization
   |
Runtime Data Areas: heap, metaspace, stacks, PC registers, native stacks
   |
Execution Engine: interpreter, JIT (C1/C2), GC
   |
Native Method Interface -> native libraries -> OS
```

The main components are:

- **Class loader subsystem:** loads classes, verifies bytecode, allocates static storage, resolves symbolic references, and runs static initializers.
- **Runtime data areas:** memory regions shared by threads (heap, metaspace) and per-thread regions (stack, PC register, native method stack).
- **Execution engine:** interprets bytecode and uses the Just-In-Time compiler to compile hot methods to native machine code.
- **Garbage collector:** finds unreachable objects and reclaims heap space.
- **JNI/native interface:** allows Java code to call C/C++ code and use OS-level features.

**3. Internal working**

Running `java -jar app.jar` starts a new JVM process. The bootstrap loader loads core classes such as `java.lang.Object`, the application loader loads classes from the classpath, and the verifier checks bytecode safety before linking. The interpreter starts execution immediately; the JIT profiles call counts and loop back-edges, then compiles hot bytecode. Compiled code may be deoptimized if an assumption breaks, such as a class being loaded later.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class OrderServiceJvmDemo {

    private static final List<String> CACHE = new ArrayList<>();

    public static void main(String[] args) {
        for (int i = 0; i < 100_000; i++) {
            CACHE.add("ORDER-" + i);
        }
        System.out.println(CACHE.size());
    }
}
```

Run it with explicit memory settings:

```bash
java -Xms64m -Xmx64m -XX:+UseG1GC OrderServiceJvmDemo
```

Here `CACHE` lives in the heap, its class metadata lives in metaspace, `main`'s frame lives on the main thread's stack, and string constants are interned in the string table.

**5. Real-world use case**

When a Spring Boot service in Kubernetes gets a 1 GB memory limit, you need to know whether the limit covers Java heap, metaspace, thread stacks, direct buffers, code cache, and JVM overhead — not just `-Xmx`.

**6. Interview follow-up questions**

- **Is the JVM the same as HotSpot?** No. JVM is a specification; HotSpot, OpenJ9, and GraalVM are implementations.
- **Where do local variables live?** In stack frames; the objects they reference live on the heap.
- **Does the JIT replace all interpretation?** No. Cold code stays interpreted; hot code is compiled.
- **Can the JVM run non-Java languages?** Yes — Kotlin, Scala, Groovy, and Clojure compile to bytecode.
- **What is the code cache?** Native memory where JIT-compiled machine code is stored.

**7. Common mistakes**

- Saying the JVM only executes Java source directly.
- Treating heap and stack as the same memory.
- Assuming `-Xmx` is the total process memory.
- Forgetting that GC runs inside the JVM process.

**8. Interview difficulty**

**Beginner**

---

### Question 2: What happens during class loading, linking, and initialization?

**1. Interview-ready answer**

Class loading has three main phases: loading finds the binary class and creates a `Class` object; linking verifies the bytecode, prepares static fields with default values, and resolves symbolic references; initialization runs static field assignments and static blocks in source order. Initialization is lazy and thread-safe — the JVM uses a class initialization lock so `<clinit>` runs once.

**2. Detailed explanation**

| Phase | What happens |
|---|---|
| Loading | Find the `.class` bytes, create the runtime `Class` object, store it in the method area/metaspace |
| Verification | Check bytecode structure, stack maps, access rules, and type safety |
| Preparation | Allocate static fields and set default values (`0`, `null`, `false`) |
| Resolution | Convert symbolic references to direct references where needed |
| Initialization | Execute static initializers and explicit static assignments |
| Using | Instantiate objects, call methods, read fields |
| Unloading | Reclaim class metadata when its loader and classes become unreachable |

Preparation and initialization are different: `static int x = 10;` sets `x = 0` during preparation and `x = 10` during initialization.

**3. Internal working**

`<clinit>` is generated by the compiler by collecting static field assignments and static blocks in textual order. The JVM guarantees that a class's superclass is initialized first. Initialization triggers include `new`, calling a static method, assigning a static field, reflective access, or initializing a subclass. Merely declaring a `Class<?>` reference does not trigger it.

**4. Practical Java/Spring Boot example**

```java
class TaxConfig {

    static final int RATE = 18;

    static {
        System.out.println("TaxConfig initialized");
    }

    static int calculate(int amount) {
        return amount + (amount * RATE / 100);
    }
}

public class ClassInitDemo {

    public static void main(String[] args) {
        System.out.println("before");
        System.out.println(TaxConfig.calculate(1000));
    }
}
```

Output:

```text
before
TaxConfig initialized
1180
```

**5. Real-world use case**

A Spring Boot `@Configuration` class with a static initializer loads a native library; knowing when initialization happens avoids surprises during startup and tests.

**6. Interview follow-up questions**

- **When is `RATE` assigned?** At initialization, not preparation, because it is a compile-time constant only if declared with a constant expression; here it is still stored in the constant pool for `int`.
- **Can initialization fail?** Yes — an exception in `<clinit>` wraps in `ExceptionInInitializerError`.
- **Can classes be unloaded?** Yes, but only when their defining loader is collectible; application classes usually are not.
- **What is resolution vs initialization?** Resolution converts references; initialization runs code.
- **Are interfaces initialized?** Only when a static field they define is actually used.

**7. Common mistakes**

- Assuming static fields get final values during preparation.
- Thinking class loading always happens at startup for every class.
- Putting heavy I/O in static blocks.
- Forgetting superclass initialization order.

**8. Interview difficulty**

**Intermediate**

---

### Question 3: Explain the class loader hierarchy and parent-first delegation

**1. Interview-ready answer**

Java uses a hierarchy: bootstrap (loads `java.*` from modules), platform (JDK extension modules), application/system (classes on the classpath), and custom loaders. Delegation is parent-first: a loader asks its parent before trying itself, which prevents user code from replacing core classes like `java.lang.String`.

**2. Detailed explanation**

```text
Bootstrap ClassLoader   -> java.base, java.lang.Object (getClassLoader() returns null)
        ^
Platform ClassLoader    -> JDK platform modules
        ^
Application ClassLoader -> classpath / target/classes / dependency jars
        ^
Custom ClassLoader      -> plugins, application servers, isolated tenants
```

The algorithm in `ClassLoader.loadClass` is:

1. Check whether the class is already loaded (`findLoadedClass`).
2. Delegate to the parent.
3. If the parent fails, call the loader's own `findClass`.
4. Define the class and optionally resolve it.

**3. Internal working**

Each loader has its own namespace. Two classes with the same fully qualified name loaded by different loaders are different runtime types. This is why plugin systems can ship different versions of a library. `Class.forName(name)` uses the caller's loader and initializes the class by default; `ClassLoader.loadClass(name)` does not initialize until first active use.

**4. Practical Java/Spring Boot example**

```java
public class ClassLoaderDemo {

    public static void main(String[] args) throws Exception {
        ClassLoader applicationLoader = ClassLoaderDemo.class.getClassLoader();
        ClassLoader contextLoader = Thread.currentThread().getContextClassLoader();

        System.out.println("JDK String loader: " + String.class.getClassLoader());
        System.out.println("Application loader: " + applicationLoader);
        System.out.println("Context loader: " + contextLoader);
        System.out.println("Same loader: " + (applicationLoader == contextLoader));
    }
}
```

Typical output:

```text
JDK String loader: null
Application loader: jdk.internal.loader.ClassLoaders$AppClassLoader@1f34f4c2
Context loader: jdk.internal.loader.ClassLoaders$AppClassLoader@1f34f4c2
Same loader: true
```

**5. Real-world use case**

Spring Boot uses the context class loader to discover JDBC drivers, JAXB providers, and logging implementations from the application classpath.

**6. Interview follow-up questions**

- **Why is `String.class.getClassLoader()` null?** It is loaded by the bootstrap loader, which is not a Java object.
- **How do you load a resource?** `ClassLoader.getResourceAsStream("application.yml")`.
- **What is a child-first loader?** It tries itself before the parent; used by Tomcat and OSGi.
- **Can you load two versions of the same library?** Yes, with separate loaders.
- **What is a loader constraint violation?** Two loaders define incompatible types that meet in one method signature.

**7. Common mistakes**

- Assuming one class name always means one runtime class.
- Using `Class.forName` when `loadClass` is intended, or vice versa.
- Replacing JDK classes on the classpath.
- Ignoring the thread context class loader in frameworks.

**8. Interview difficulty**

**Intermediate**

---

### Question 4: Why do frameworks use the thread context class loader?

**1. Interview-ready answer**

The thread context class loader, or TCCL, lets framework code discover application classes without depending on the framework's own loader. Parent-first delegation works from child to parent, but a framework loaded higher in the hierarchy cannot see classes loaded by the application. JDBC, JNDI, JAXB, Spring, and servlet containers use the TCCL to bridge that gap.

**2. Detailed explanation**

In a servlet container, the container's loader sits above each web application's loader. The framework needs to instantiate a class supplied by the application, such as a JDBC driver or a JAXB provider. It reads `Thread.currentThread().getContextClassLoader()`, which the container sets to the web app loader, and uses it for `Class.forName` or `ServiceLoader`.

**3. Internal working**

`ServiceLoader.load(Class, loader)` scans `META-INF/services` resources visible to the supplied loader and instantiates providers. If the wrong loader is used, the provider is invisible or a `ClassCastException` appears because the same interface was loaded by two loaders.

**4. Practical Java/Spring Boot example**

```java
import java.sql.Connection;
import java.sql.DriverManager;

public class JdbcLoaderDemo {

    public static void main(String[] args) throws Exception {
        ClassLoader contextLoader = Thread.currentThread().getContextClassLoader();
        Class.forName("org.postgresql.Driver", true, contextLoader);

        try (Connection connection = DriverManager.getConnection(
                "jdbc:postgresql://localhost:5432/orders", "app", "secret")) {
            System.out.println(connection.isValid(2));
        }
    }
}
```

**5. Real-world use case**

A Spring Boot application with PostgreSQL discovers `org.postgresql.Driver` from the fat jar through the TCCL even though the JDBC API itself belongs to the JDK.

**6. Interview follow-up questions**

- **What if the TCCL is null?** Fall back to the class's own loader.
- **Does modular Java change this?** Modules add another layer of encapsulation; `ServiceLoader` remains the standard mechanism.
- **Why not always use TCCL?** It can be wrong in virtual threads or executor threads if not propagated.
- **How do you set it?** `thread.setContextClassLoader(loader)`.
- **How does Spring use it?** Bean resolution, resource loading, proxies, and provider discovery.

**7. Common mistakes**

- Using `ClassLoader.getSystemClassLoader()` inside libraries.
- Forgetting to restore a temporarily changed TCCL.
- Assuming executors inherit the submitting thread's TCCL.
- Loading provider interfaces with one loader and implementations with another.

**8. Interview difficulty**

**Advanced**

---

### Question 5: What are the JVM runtime data areas?

**1. Interview-ready answer**

The JVM has shared areas — heap and metaspace — and per-thread areas — Java stack, program counter register, and native method stack. The heap holds objects and arrays; metaspace holds class metadata in native memory; each stack holds frames with locals, operand stacks, and return addresses; the PC register tracks the current bytecode instruction; the native stack supports JNI calls. Direct buffers live outside the heap in native memory.

**2. Detailed explanation**

| Area | Scope | Stores | Typical errors |
|---|---|---|---|
| Heap | Shared | Objects, arrays | `java.lang.OutOfMemoryError: Java heap space` |
| Metaspace | Shared | Class metadata, constant pool, method metadata | `OutOfMemoryError: Metaspace` |
| Java stack | Per thread | Frames, local variables, operand stack | `StackOverflowError` |
| PC register | Per thread | Current instruction address | None directly |
| Native method stack | Per thread | Native/JNI frames | `StackOverflowError`, native crashes |
| Direct memory | Shared/native | `ByteBuffer.allocateDirect`, Netty buffers | `OutOfMemoryError: Direct buffer memory` |

**3. Internal working**

A thread invoking a method pushes a frame. The frame contains the local variable table and operand stack. Primitive locals and references can live in the frame, but the objects they point to live on the heap. When the method returns, the frame pops. Heap memory is managed by GC; stack memory is reclaimed automatically with the frame.

**4. Practical Java/Spring Boot example**

```java
record Order(long id, String customerId) {
}

public class RuntimeAreasDemo {

    static Order createOrder(long id) {
        String customerId = "CUST-" + id;
        return new Order(id, customerId);
    }

    public static void main(String[] args) {
        Order order = createOrder(10L);
        System.out.println(order.customerId());
    }
}
```

Inside `createOrder`, `id` and `customerId` are in the stack frame; the `Order` object and the `String` it references are on the heap; the definitions of `Order` and `RuntimeAreasDemo` are in metaspace.

**5. Real-world use case**

A Spring WebFlux service with many concurrent requests may be heap-limited, but a thread-per-request MVC service can fail first because of thread stacks or direct buffers.

**6. Interview follow-up questions**

- **Are strings on the stack or heap?** String objects are on the heap; string literals are interned and referenced from the string table.
- **Does the stack grow?** Yes, HotSpot stacks grow dynamically up to `-Xss`.
- **Where are static fields?** The field metadata is in metaspace; referenced objects are on the heap.
- **What is TLAB?** A thread-local allocation buffer inside Eden that avoids locking for small allocations.
- **Where does JIT code live?** In the code cache, native memory.

**7. Common mistakes**

- Saying all primitives are always on the stack — object fields are inside heap objects.
- Ignoring direct memory in container sizing.
- Confusing metaspace with heap.
- Assuming stack frames are garbage collected.

**8. Interview difficulty**

**Beginner**

---

---

## Part B — Heap, stack, and metaspace

### Question 6: How is the heap structured and how does object allocation work?

**1. Interview-ready answer**

In HotSpot the heap is split into the young generation and old generation. The young generation has Eden and two survivor spaces. New objects are allocated in Eden, usually inside a thread-local allocation buffer. A minor GC copies live objects from Eden and one survivor space into the other survivor space, incrementing their age; objects that survive long enough are promoted to the old generation. G1 still has young and old regions logically, but implements them as dynamic regions instead of one contiguous young block.

**2. Detailed explanation**

```text
Heap
├── Young generation
│   ├── Eden
│   ├── Survivor 0
│   └── Survivor 1
└── Old generation
```

Allocation flow:

1. The JIT may allocate scalar-replaced objects outside the heap entirely.
2. Small allocations use a TLAB to avoid global locks.
3. Large allocations may go directly to old generation or become humongous objects under G1.
4. When Eden fills, a minor GC runs.
5. Live objects copy to the empty survivor space; dead objects are reclaimed by ignoring them.
6. After repeated survival, usually after 15 cycles or when a survivor space is full, objects promote to old.
7. Old generation collection is less frequent but usually more expensive.

**3. Internal working**

Copying collection makes allocation cheap: the JVM keeps a pointer to the next free address in Eden rather than searching free lists. The collector updates references to copied objects. Promotion is a copy from young to old, not a move in place. `-XX:MaxTenuringThreshold` controls the age limit; HotSpot may promote earlier based on survivor occupancy.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class HeapAllocationDemo {

    public static void main(String[] args) {
        List<byte[]> retained = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
            retained.add(new byte[1024 * 1024]);
        }

        System.out.println(retained.size());
    }
}
```

Run with GC logging:

```bash
java -Xms64m -Xmx64m -XX:+UseG1GC -Xlog:gc HeapAllocationDemo
```

The `byte[]` objects are retained, so they eventually promote and exhaust the old generation. Short-lived request DTOs normally die in Eden.

**5. Real-world use case**

A REST API creates millions of short-lived DTOs per minute. Tuning young generation size and avoiding early promotion keeps collections cheap and pause times low.

**6. Interview follow-up questions**

- **Why two survivor spaces?** Copying live objects into the empty space avoids fragmentation.
- **What is a humongous object?** With G1, an object larger than half a region; allocated directly in humongous regions.
- **What is a promotion failure?** Old generation cannot receive promoted objects.
- **Does stack allocation exist in Java?** Escaping objects go to heap; JIT scalar replacement can eliminate non-escaping allocations.
- **What is TLAB exhaustion?** The thread requests a new TLAB from Eden under a short lock.

**7. Common mistakes**

- Believing minor GC collects the whole heap.
- Setting a huge young generation for long-lived data.
- Confusing heap size with RSS in containers.
- Assuming all allocation is lock-free.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: What is inside a Java stack frame?

**1. Interview-ready answer**

Each method invocation pushes a frame onto the thread's Java stack. The frame contains the local variable table, the operand stack, a reference to the runtime constant pool, and return/exception-handling information. Locals hold primitives and references; the operand stack is where bytecode instructions push and pop values during expression evaluation.

**2. Detailed explanation**

A frame is created when a method is invoked and destroyed when it returns or throws. Its size is known from the method's bytecode, so the JVM can allocate it efficiently. The operand stack is not the same as the CPU stack used by native code; it is the JVM's evaluation stack.

Example method:

```java
public class FrameDemo {

    static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        System.out.println(add(10, 20));
    }
}
```

Conceptual bytecode for `add`:

```text
iload_1
iload_2
iadd
ireturn
```

`iload_1` pushes `a`, `iload_2` pushes `b`, `iadd` pops both and pushes the sum, and `ireturn` pops the result to the caller's operand stack.

**3. Internal working**

Slot 0 in an instance method holds `this`; static methods start at slot 0 with the first parameter. `long` and `double` occupy two slots. Deep call chains push many frames, which is why unbounded recursion eventually throws `StackOverflowError`. Native methods use the native method stack instead of Java frames.

**4. Practical Java/Spring Boot example**

```java
public class RecursivePricing {

    static long factorial(int n) {
        if (n == 1) {
            return 1L;
        }
        return n * factorial(n - 1);
    }

    public static void main(String[] args) {
        System.out.println(factorial(5));
    }
}
```

Each recursive call adds a frame; changing `5` to a very large number demonstrates stack growth failure.

**5. Real-world use case**

A recursive tree-flattening routine in a category service works for shallow trees but fails on deep trees; rewriting it with an explicit stack moves state to the heap.

**6. Interview follow-up questions**

- **Are objects stored in frames?** No, references are; objects are on the heap.
- **Why `long` takes two slots?** JVM local variable slots are 32 bits.
- **Can frames be shared between threads?** No; stacks are per thread.
- **What happens on exception?** Stack unwinding pops frames until a handler is found.
- **Does `-Xss` affect heap?** No, it sets thread stack size.

**7. Common mistakes**

- Calling the operand stack “the heap.”
- Assuming recursion is free because method calls are cheap.
- Increasing `-Xss` without considering thread count.
- Forgetting that each Spring request thread has its own stack.

**8. Interview difficulty**

**Intermediate**

---

### Question 8: What is metaspace and how is it different from PermGen?

**1. Interview-ready answer**

PermGen was a fixed-size heap-adjacent area used before Java 8 for class metadata and interned strings. Java 8 removed it and moved class metadata to metaspace, which grows in native memory by default. Interned strings had already moved to the heap in Java 7. Metaspace can be capped with `-XX:MaxMetaspaceSize`; if classes are generated or deployed repeatedly without unloading, it can throw `OutOfMemoryError: Metaspace`.

**2. Detailed explanation**

| Feature | PermGen | Metaspace |
|---|---|---|
| Java version | Java 7 and earlier | Java 8 onward |
| Memory type | Heap-like, fixed size | Native memory |
| Default max | Fixed by `-XX:MaxPermSize` | Practically limited by system memory |
| Interned strings | PermGen in Java 6, heap from Java 7 | Heap |
| Tuning flag | `-XX:MaxPermSize` | `-XX:MaxMetaspaceSize`, `-XX:MetaspaceSize` |
| Class unloading | Possible but JVM-managed | Possible with collectible loaders |

Metaspace stores class metadata such as constant pools, method metadata, field metadata, and class hierarchies. It does not store ordinary object instances.

**3. Internal working**

Each class loader gets metaspace chunks from native memory. When a loader is unreachable, the JVM can unload its classes and release chunks. Reflection, proxies, lambdas, bytecode libraries, and hot deployment create many classes and loaders, which is why frameworks must avoid loader leaks.

**4. Practical Java/Spring Boot example**

```java
public class MetaspaceLoaderDemo {

    static Class<?> loadProvider(ClassLoader loader, String className) throws Exception {
        return Class.forName(className, false, loader);
    }

    public static void main(String[] args) throws Exception {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        Class<?> provider = loadProvider(loader, "com.example.payment.StripeGateway");
        System.out.println(provider.getName());
    }
}
```

Useful production flags:

```bash
java -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m -Xlog:class+unload=info -jar app.jar
```

**5. Real-world use case**

A developer running Spring DevTools or deploying WAR files repeatedly sees metaspace grow because old application class loaders remain referenced by static fields or threads.

**6. Interview follow-up questions**

- **Where are string literals in Java 17?** In the heap string table.
- **Can metaspace be garbage collected?** Metadata is reclaimed when classes unload; objects are never stored there.
- **Why set MaxMetaspaceSize in containers?** To fail predictably instead of consuming all native memory.
- **What causes class loader leaks?** Static references, threads, JDBC drivers, and scheduled tasks holding application classes.
- **Does lambda creation always create new classes?** Lambdas use hidden classes and invokedynamic; repeated generated lambdas can still add metadata.

**7. Common mistakes**

- Using `-XX:MaxPermSize` on Java 17 — the flag was removed.
- Treating metaspace as part of `-Xmx`.
- Ignoring native memory when debugging container OOM kills.
- Assuming class unloading is guaranteed immediately.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: What are the PC register and native method stack used for?

**1. Interview-ready answer**

The program counter register holds the address of the bytecode instruction currently being executed by a thread. Each thread has its own PC. The native method stack stores frames for native methods called through JNI. When a thread executes a native method, the Java PC is undefined because execution is no longer bytecode-driven.

**2. Detailed explanation**

The PC is small but essential: after each instruction, it advances to the next one, and it is restored when a thread is switched back. Branching, exception handling, and `goto` all work by changing it.

The native method stack is used by calls into C/C++ code — for example `System.nanoTime()`, file system operations, compression libraries, or Netty's native transport. HotSpot usually creates native stacks as OS threads rather than explicit JVM structures, but the conceptual role is the same.

**3. Internal working**

When Java calls a native method, the JVM saves the Java frame, switches to native execution, and uses the native stack for that call. On return, the Java frame resumes. If native code calls back into Java through JNI, a new Java frame is pushed on the same Java stack.

**4. Practical Java/Spring Boot example**

```java
public class NativeCallDemo {

    public static void main(String[] args) {
        long start = System.nanoTime();

        String payload = "order-payload".repeat(10_000);
        int hash = payload.hashCode();

        long elapsed = System.nanoTime() - start;
        System.out.println(hash + " in " + elapsed + " ns");
    }
}
```

`System.nanoTime()` enters native code using the native stack; the Java bytecode that calls it has its PC tracked separately.

**5. Real-world use case**

A JVM crash inside a compression library produces a native stack in the hs_err file; knowing it is separate from the Java stack avoids looking in the wrong place.

**6. Interview follow-up questions**

- **Does every thread have a PC?** Yes.
- **Is the PC the same as a CPU register?** Conceptually similar, but it tracks bytecode for the thread.
- **Can native code throw Java exceptions?** It can pending exceptions through JNI.
- **What is JNI?** Java Native Interface, the boundary between Java and native code.
- **Can the native stack overflow?** Yes, often with a native crash rather than a Java `StackOverflowError`.

**7. Common mistakes**

- Saying the PC is shared.
- Assuming native methods run on the Java stack.
- Ignoring `-XX:MaxJavaStackTraceDepth` when reading truncated traces.
- Blaming Java code for a crash that originated in a native library.

**8. Interview difficulty**

**Intermediate**

---

### Question 10: How is a Java object laid out in memory?

**1. Interview-ready answer**

A Java object on HotSpot has a header, instance fields, and padding. The header contains a mark word and a klass pointer, and arrays also store a length. Fields are usually ordered by size for alignment. With compressed oops enabled, references are 4 bytes instead of 8, and the whole object is padded to an 8-byte boundary.

**2. Detailed explanation**

Typical 64-bit HotSpot layout:

```text
Object
├── Mark word            8 bytes: hash, GC age, lock state
├── Klass pointer        4 bytes with compressed class pointers
├── Instance fields      primitive and reference fields
├── Array length         4 bytes, arrays only
└── Padding              aligns total size to 8 bytes
```

For this class:

```java
class OrderSnapshot {

    long orderId;
    boolean paid;
    String customerId;
}
```

A rough estimate is 12 bytes header + 8 bytes `long` + 1 byte `boolean` + 4 bytes compressed reference = 25 bytes, padded to 32 bytes. The JVM may reorder fields to reduce padding.

**3. Internal working**

The mark word stores identity hash code, GC age, biased/epoch and lock information, and forwarding pointers during compaction. Compressed oops use a base address plus a 32-bit offset and are enabled by default for heaps below roughly 32 GB. Above that threshold, references may grow to 8 bytes, causing a noticeable heap footprint increase.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class ObjectFootprintDemo {

    static final class OrderSnapshot {
        final long orderId;
        final boolean paid;
        final String customerId;

        OrderSnapshot(long orderId, boolean paid, String customerId) {
            this.orderId = orderId;
            this.paid = paid;
            this.customerId = customerId;
        }
    }

    public static void main(String[] args) {
        List<OrderSnapshot> snapshots = new ArrayList<>();

        for (int i = 0; i < 1_000_000; i++) {
            snapshots.add(new OrderSnapshot(i, i % 2 == 0, "CUST-" + i));
        }

        System.out.println(snapshots.size());
    }
}
```

A million tiny wrapper objects plus a `String` each can consume far more memory than a million `long` values in an array.

**5. Real-world use case**

A reporting service holds millions of DTOs. Replacing boxed `Long` and per-row `String` objects with primitive arrays or compact DTOs reduces heap pressure and GC pauses.

**6. Interview follow-up questions**

- **How do you measure this precisely?** Use JOL (`org.openjdk.jol`) or a heap dump analyzer.
- **Why does heap above 32 GB hurt?** Compressed oops are disabled beyond the range.
- **What is in the mark word during locking?** Monitor or lightweight lock state.
- **Do records save memory?** No special layout benefit; they are final classes with fields.
- **What is false sharing?** Adjacent fields on the same cache line written by different threads.

**7. Common mistakes**

- Assuming a `boolean` field costs only 1 byte in total.
- Ignoring object headers for tiny objects.
- Using boxed primitives in large collections.
- Expecting exact size from source order alone.

**8. Interview difficulty**

**Advanced**

---

---

## Part C — Object creation and garbage collection fundamentals

### Question 11: What happens when you execute `new Order(...)`?

**1. Interview-ready answer**

The JVM first ensures the class is loaded and initialized, then allocates memory — usually from Eden via a TLAB — zeroes the memory, sets the object header, runs the constructor with `invokespecial`, and finally stores the reference in the local variable. The JIT may eliminate the allocation entirely if the object does not escape the method.

**2. Detailed explanation**

Steps:

1. **Class resolution:** Check that `Order` is loaded, linked, and initialized.
2. **Allocation:** Reserve contiguous space in Eden, normally from the thread's TLAB.
3. **Zeroing:** Set fields to `0`, `false`, or `null` before constructor code runs.
4. **Header setup:** Write the mark word and klass pointer.
5. **Constructor execution:** Run field initializers and constructor body.
6. **Reference assignment:** Store the heap address in the local variable table.

**3. Internal working**

`new`, `dup`, `invokespecial <init>`, and `astore` are the typical bytecode sequence. `dup` keeps a reference on the operand stack while the constructor runs. Escape analysis may replace a non-escaping object with scalar values, so no heap allocation occurs. Lock elision can also remove synchronization on thread-local objects.

**4. Practical Java/Spring Boot example**

```java
record Order(long id, String customerId) {
}

public class NewOperatorDemo {

    public static Order create(long id) {
        return new Order(id, "CUST-" + id);
    }

    public static void main(String[] args) {
        Order order = create(10L);
        System.out.println(order.id());
    }
}
```

If `Order` is used only inside `create`, the JIT may scalar-replace it; if it is returned and stored in a list, it must be heap-allocated.

**5. Real-world use case**

A controller maps JSON to a DTO per request. Millions of these allocations are normal, but retaining them in a static cache turns short-lived objects into long-lived heap pressure.

**6. Interview follow-up questions**

- **Is memory zeroed before the constructor?** Yes, so fields have default values.
- **Can allocation fail?** Yes, with `OutOfMemoryError: Java heap space`.
- **Does `new` always call a constructor?** Yes, some constructor must run unless deserialization or Unsafe is used.
- **What is escape analysis?** JIT analysis that determines whether an object is visible outside a thread/method.
- **Can allocation happen outside TLAB?** Yes, for large objects or after TLAB exhaustion.

**7. Common mistakes**

- Thinking constructors allocate memory — they initialize already-allocated memory.
- Assuming escape analysis always eliminates allocations.
- Forgetting that returned objects usually escape.
- Confusing object references with pointers visible to Java code.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: What are GC roots and how does reachability work?

**1. Interview-ready answer**

Garbage collection starts from GC roots — references held by live threads, static fields, JNI handles, monitors, and JVM internals — and traverses object graphs. Any object reachable from a root is live; unreachable objects are garbage. The collector does not use reference counting, so cyclic references are reclaimed if no root reaches them.

**2. Detailed explanation**

Common GC roots:

- Local variables and operand stack values in Java thread frames.
- Static fields loaded by live class loaders.
- Active thread objects and classes.
- JNI local/global references.
- Objects used as synchronization monitors.
- JVM system class loader and important internal data structures.
- Exception objects being propagated.

Reachability levels matter in older APIs: strongly reachable objects are never collected; softly reachable objects may be collected under memory pressure; weakly reachable objects are collected at the next GC; phantom reachable objects have been finalized and await cleanup.

**3. Internal working**

Collectors trace from roots using mark bits in the object header or side bitmaps. Starting points change constantly as stack frames appear and disappear, so GC must stop application threads at safepoints or use concurrent techniques with synchronization barriers.

**4. Practical Java/Spring Boot example**

```java
public class GcRootDemo {

    private static Object staticRef;

    public static void main(String[] args) {
        Object localRef = new Object();
        staticRef = new Object();

        localRef = null;

        System.gc();
        System.out.println(staticRef != null);
    }
}
```

The object assigned to `staticRef` is reachable from a static field and survives. The original `localRef` object becomes unreachable after the assignment to `null`. `System.gc()` is only a hint and should not be used in application code.

**5. Real-world use case**

A static `Map<String, Object>` in a Spring service accidentally caches every request payload. Those payloads are GC roots via the static field, so they can never be reclaimed.

**6. Interview follow-up questions**

- **Does reference counting handle cycles?** No; HotSpot uses tracing.
- **Are class loaders roots?** Live loaders and their static fields act as roots.
- **Can a local variable be a root after method returns?** No, its frame is gone.
- **How do finalizers affect reachability?** Finalizable objects need extra reachability treatment; `Cleaner` is preferred.
- **Why are safepoints needed?** To get a consistent snapshot of roots and object graph.

**7. Common mistakes**

- Believing `null` immediately frees an object.
- Keeping large objects in static caches.
- Calling `System.gc()` to “fix” memory issues.
- Forgetting that thread-local variables are roots while the thread lives.

**8. Interview difficulty**

**Intermediate**

---

### Question 13: Compare mark-sweep, mark-compact, and copying collectors

**1. Interview-ready answer**

Mark-sweep marks live objects then sweeps dead ones, leaving fragmentation. Mark-compact marks live objects then slides them together, producing contiguous free space but requiring reference updates and a pause. Copying collectors mark live objects and copy them into a fresh region, which gives fast bump-pointer allocation and no fragmentation but needs spare space. HotSpot combines these strategies: young collectors copy survivors, while old-generation collectors may compact or sweep.

**2. Detailed explanation**

| Collector style | Steps | Strength | Weakness |
|---|---|---|---|
| Mark-sweep | Mark live, sweep dead | Simple, can be concurrent | Fragmentation |
| Mark-compact | Mark live, slide live objects together | No fragmentation, dense heap | Longer pauses, reference updates |
| Copying | Copy live objects to empty space | Fast allocation, no fragmentation | Needs spare region, moves objects |

The **generational hypothesis** says most objects die young. HotSpot exploits it by collecting the young generation frequently and the old generation rarely. A minor GC is usually cheap because it copies only the few survivors.

**3. Internal working**

Marking sets bits in object headers or side bitmaps. Moving objects means every reference must be updated; collectors use forwarding pointers or forwarding tables. Read/write barriers intercept reference updates so concurrent collectors keep the graph consistent while application threads run.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class GenerationalDemo {

    static List<Object> retained = new ArrayList<>();

    static void request() {
        Object requestDto = new byte[64 * 1024];
        retained.add(requestDto);
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100_000; i++) {
            request();
        }
    }
}
```

Without `retained.add`, each DTO dies in Eden. With it, the DTOs survive, promote to old, and eventually force a major collection.

**5. Real-world use case**

A cache with no size limit violates the generational hypothesis by turning short-lived request data into long-lived old-generation objects.

**6. Interview follow-up questions**

- **Why is young GC faster?** It scans a small area and copies few survivors.
- **What is fragmentation?** Free memory split into small non-contiguous blocks.
- **What is a write barrier?** Code executed on reference writes to maintain collector metadata.
- **What is a safepoint?** A point where all threads are paused so the JVM can inspect/move objects safely.
- **Why do moving collectors need reference updates?** Object addresses change.

**7. Common mistakes**

- Assuming full GC and minor GC are the same.
- Treating all dead objects as immediately reclaimed.
- Ignoring the cost of moving large heaps.
- Believing fragmentation only happens in old generation.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: Compare Serial, Parallel, G1, and ZGC collectors

**1. Interview-ready answer**

Serial is single-threaded and suited to tiny heaps; Parallel uses multiple GC threads for high throughput but longer pauses; G1 is the default from Java 9 and targets pause times with region-based incremental collection; ZGC aims for sub-millisecond to low-millisecond pauses using concurrent compaction and colored pointers/load barriers. Choose based on heap size, latency SLA, and CPU budget.

**2. Detailed explanation**

| Collector | Flags | Threads | Goal | Typical use |
|---|---|---|---|---|
| Serial | `-XX:+UseSerialGC` | One | Low overhead | Small apps, CLI tools |
| Parallel | `-XX:+UseParallelGC` | Many | Throughput | Batch jobs, large non-latency workloads |
| G1 | `-XX:+UseG1GC` | Many | Balanced pause time | Default server choice |
| ZGC | `-XX:+UseZGC` | Many | Very low latency | Large heaps, strict SLAs |
| Epsilon | `-XX:+UseEpsilonGC` | None | No-op GC | Performance tests only |

CMS was deprecated in Java 9 and removed in Java 14. Shenandoah exists in some builds and also targets low pauses.

**3. Internal working**

Serial and Parallel stop-the-world for the whole collection. G1 builds collection sets from regions and can collect partly concurrently. ZGC performs marking, relocation, and remapping largely concurrently; tiny stop-the-world phases handle root processing. More concurrent work means more CPU overhead and barriers on application read/write paths.

**4. Practical Java/Spring Boot example**

```bash
java -Xms2g -Xmx2g -XX:+UseParallelGC -jar orders-service.jar

java -Xms2g -Xmx2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -jar orders-service.jar

java -Xms8g -Xmx8g -XX:+UseZGC -XX:+ZGenerational -jar orders-service.jar
```

For Java 17, ZGC generational mode is available with `-XX:+ZGenerational` and became the default in later releases; verify support for your exact JDK build.

**5. Real-world use case**

A payment API with a 200 ms p99 SLA should prefer G1 or ZGC over Parallel GC, even if total throughput is slightly lower.

**6. Interview follow-up questions**

- **Which collector is default in Java 17?** G1.
- **Does ZGC have zero pauses?** No; it has very short stops for roots and thread synchronization.
- **Why not always Parallel GC?** Pause times grow with heap size.
- **What does MaxGCPauseMillis guarantee?** A soft target, not a hard guarantee.
- **Can you switch collectors at runtime?** No; it is chosen at JVM startup.

**7. Common mistakes**

- Copying GC flags from old Java 8 articles.
- Using CMS flags on Java 17.
- Setting pause goals without measuring.
- Assuming low pauses mean better throughput.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: How does Garbage-First (G1) GC work?

**1. Interview-ready answer**

G1 divides the heap into equal-sized regions and tracks the amount of garbage in each. It prioritizes regions with the most reclaimable space — hence “garbage first” — and collects a bounded set of regions to meet `-XX:MaxGCPauseMillis`. It uses remembered sets for cross-region references, SATB snapshots for concurrent marking, and mixed collections to reclaim both young and old regions.

**2. Detailed explanation**

G1 concepts:

- **Regions:** Heap split into ~1–32 MB regions, assigned roles dynamically: Eden, survivor, old, or humongous.
- **Remembered sets:** Per-region metadata listing incoming references from other regions.
- **SATB:** Snapshot-at-the-beginning marking preserves the object graph as it existed when marking started.
- **Collection set:** Regions selected for a pause based on reclaimable bytes and pause prediction.
- **Young-only GC:** Evacuates young regions.
- **Concurrent mark:** Finds live old-region objects while the application runs.
- **Mixed GC:** Evacuates young regions plus selected old regions with high garbage ratios.
- **Humongous objects:** Objects larger than half a region occupy contiguous humongous regions.

**3. Internal working**

Application writes are intercepted by write barriers that update remembered sets. During evacuation, live objects are copied out of collection-set regions so whole regions can be reclaimed at once. G1 maintains pause predictions from previous collections and adjusts the collection set size. If it cannot keep up, it may degrade to a full compacting collection.

**4. Practical Java/Spring Boot example**

Recommended baseline for a latency-sensitive service:

```bash
java -Xms2g -Xmx2g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:InitiatingHeapOccupancyPercent=45 \
  -Xlog:gc*,safepoint:file=gc.log:time,uptime,level,tags \
  -jar orders-service.jar
```

Analyze `gc.log` before changing flags. If old regions never enter mixed collections, investigate allocation rate and retention rather than lowering the pause target.

**5. Real-world use case**

A product catalog API has a 4 GB heap and strict tail latency. G1 collects the dirtiest regions first, keeping most pauses near the target without a full-heap stop.

**6. Interview follow-up questions**

- **Why remembered sets?** They avoid scanning the whole heap for references into a region.
- **What is SATB?** A marking snapshot that keeps concurrently changed graphs consistent.
- **What happens if the pause target is unrealistic?** G1 misses it and may accumulate garbage.
- **Why are humongous objects costly?** They occupy whole regions and are less flexible to evacuate.
- **When does G1 full GC happen?** When it cannot reclaim fast enough or regions are exhausted.

**7. Common mistakes**

- Setting `MaxGCPauseMillis=10` without evidence.
- Increasing heap to hide a leak.
- Ignoring humongous allocation logs.
- Tuning `InitiatingHeapOccupancyPercent` before inspecting logs.
- Confusing G1 regions with fixed young/old contiguous spaces.

**8. Interview difficulty**

**Advanced**

---

---

## Part D — OOM, leaks, and production troubleshooting

### Question 16: Which JVM flags would you use to tune and diagnose GC in production?

**1. Interview-ready answer**

I start with measurements, not flags: enable GC logs, capture heap dumps on OOM, enable native memory tracking, and record container limits. Then I set heap size consistently with `-Xms` and `-Xmx`, choose a collector, and tune only one goal at a time. For Java 17 services I usually start with G1, a realistic pause target, and structured GC logs.

**2. Detailed explanation**

| Flag | Purpose |
|---|---|
| `-Xms` / `-Xmx` | Initial and maximum heap |
| `-XX:MaxRAMPercentage=70` | Heap as a percentage of container memory |
| `-XX:+UseG1GC` | Select G1 |
| `-XX:MaxGCPauseMillis=200` | Soft pause-time target |
| `-Xlog:gc*,safepoint` | Unified GC logging |
| `-XX:+HeapDumpOnOutOfMemoryError` | Write a heap dump on OOM |
| `-XX:HeapDumpPath=/dumps` | Control dump location |
| `-XX:NativeMemoryTracking=summary` | Track native memory categories |
| `-XX:ErrorFile=/logs/hs_err_pid%p.log` | Capture JVM crash details |
| `-XX:ActiveProcessorCount=N` | Override CPU detection in constrained containers |

**3. Internal working**

GC logs report pause phases, collection causes, heap occupancy before and after collection, and region statistics. Heap dumps are snapshots of all objects and references, but taking one causes a stop-the-world pause. Native memory tracking adds small overhead and separates JVM categories such as class, thread, code, GC, and internal memory.

**4. Practical Java/Spring Boot example**

```bash
java -Xms1g -Xmx1g \
  -XX:MaxRAMPercentage=70 \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/app/dumps \
  -XX:NativeMemoryTracking=summary \
  -Xlog:gc*,safepoint:file=/var/log/app/gc.log:time,uptime,level,tags \
  -jar /app/orders-service.jar
```

In Kubernetes, set the container memory limit above `-Xmx` to leave room for metaspace, thread stacks, direct buffers, code cache, and JVM overhead.

**5. Real-world use case**

A service's RSS grows to the pod limit even though heap usage is flat. NMT reveals 300 MB of thread stacks because the application creates too many platform threads.

**6. Interview follow-up questions**

- **Should Xms equal Xmx in production?** Often yes for stable heap sizing, but autoscaling and memory limits must be considered.
- **Does MaxRAMPercentage replace Xmx?** It is an alternative; explicit Xmx is clearer when limits are known.
- **Why not always enable detailed GC logs?** Disk cost and overhead, but `-Xlog:gc*` is usually acceptable.
- **Can heap dumps contain secrets?** Yes; protect and delete them securely.
- **What is a safepoint log?** It shows time to stop threads and can explain hidden pauses.

**7. Common mistakes**

- Raising heap without finding the retention cause.
- Setting Xmx equal to the container limit.
- Tuning many flags at once.
- Forgetting to rotate GC logs.
- Enabling NMT permanently without measuring overhead.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: What are the common types of `OutOfMemoryError`?

**1. Interview-ready answer**

`OutOfMemoryError` has several messages: Java heap space means object allocation failed; GC overhead limit exceeded means the JVM spent too long reclaiming too little; Metaspace means class metadata exhausted its cap; Direct buffer memory means off-heap buffers are exhausted; unable to create native thread often means too many threads or stack memory exhaustion. Each points to a different resource, so the fix depends on the message.

**2. Detailed explanation**

| Message | Meaning | Typical fix |
|---|---|---|
| `Java heap space` | Heap cannot satisfy allocation | Find leak, reduce retention, or size heap |
| `GC overhead limit exceeded` | GC runs constantly with little recovery | Fix leak or request volume |
| `Metaspace` | Class metadata cap reached | Raise cap or fix class loader leak |
| `Compressed class space` | Compressed klass metadata exhausted | Raise class space or reduce classes |
| `Direct buffer memory` | Off-heap buffer limit reached | Close buffers, tune `-XX:MaxDirectMemorySize` |
| `unable to create new native thread` | OS cannot create thread/stack | Reduce threads, lower `-Xss`, raise OS limits |
| `map failed` | mmap failed, often too many mapped files | Reduce mapped files or raise VM limits |
| `Requested array size exceeds VM limit` | Array too large | Chunk data or fix logic |

**3. Internal working**

The JVM throws OOM when an allocation request fails after collection has been attempted where applicable. `-XX:+HeapDumpOnOutOfMemoryError` captures heap state for heap-related failures. Native failures may produce an `hs_err_pid.log` instead of or alongside the Java exception.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class HeapOomDemo {

    public static void main(String[] args) {
        List<byte[]> retained = new ArrayList<>();

        while (true) {
            retained.add(new byte[1024 * 1024]);
        }
    }
}
```

Run with:

```bash
java -Xmx32m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./dump HeapOomDemo
```

The list keeps every array reachable, so GC cannot reclaim anything.

**5. Real-world use case**

A report endpoint loads all rows into a `List`. Under load, heap OOM appears; streaming results with pagination fixes the root cause better than increasing `-Xmx`.

**6. Interview follow-up questions**

- **Can OOM be caught?** It is an `Error`; catching it is unreliable and generally not recommended.
- **What does GC overhead limit mean?** By default, more than 98% of time in GC with less than 2% heap recovered over several GCs.
- **Why direct buffer OOM with small heap?** Direct memory is outside `-Xmx`.
- **Does metaspace count against Xmx?** No.
- **Can many threads cause OOM with a small heap?** Yes, via native stack memory.

**7. Common mistakes**

- Treating every OOM as a heap problem.
- Catching OOM and continuing.
- Raising heap without reading the error message.
- Ignoring off-heap memory in Netty applications.

**8. Interview difficulty**

**Intermediate**

---

### Question 18: What is the difference between `StackOverflowError` and heap `OutOfMemoryError`?

**1. Interview-ready answer**

`StackOverflowError` happens when a thread's Java stack is exhausted, usually from unbounded recursion or excessively deep call chains. Heap `OutOfMemoryError` happens when objects cannot be allocated because the heap is full or fragmented. The first is per-thread stack memory; the second is shared heap memory.

**2. Detailed explanation**

| Aspect | StackOverflowError | OutOfMemoryError: Java heap space |
|---|---|---|
| Region | Thread stack | Shared heap |
| Common cause | Infinite recursion | Retaining too many objects |
| Fix | Bound recursion, use iteration, increase `-Xss` carefully | Remove leak, stream data, size heap |
| Scope | One thread can fail | Whole JVM or allocation path |
| Diagnostic | Stack trace with repeated frames | Heap dump and GC logs |

**3. Internal working**

Every method call pushes a frame. If the next frame exceeds the remaining stack, the JVM throws `StackOverflowError` in that thread. Other threads can continue running. A heap OOM affects allocation globally and may leave the JVM unstable.

**4. Practical Java/Spring Boot example**

```java
public class DeepRecursionDemo {

    static int count(int depth) {
        return 1 + count(depth + 1);
    }

    public static void main(String[] args) {
        System.out.println(count(0));
    }
}
```

Run with a smaller stack to reproduce quickly:

```bash
java -Xss256k DeepRecursionDemo
```

The fix is an iterative traversal or an explicit heap-based stack, not repeatedly increasing `-Xss` in a thread-per-request service.

**5. Real-world use case**

A JSON serializer recurses through a deeply nested tree. One request thread throws `StackOverflowError`, while the rest of the service remains alive; adding a maximum depth prevents the failure.

**6. Interview follow-up questions**

- **Does tail-call optimization exist in Java?** HotSpot does not perform general tail-call elimination.
- **Can you catch StackOverflowError?** Technically yes, but the stack may be too exhausted to handle it safely.
- **Does a bigger heap help recursion?** No; stack depth is controlled by `-Xss`.
- **Can native code cause stack overflow?** Yes, in the native method stack.
- **What about `unable to create new native thread`?** That is usually OOM from too many threads, not Java stack overflow.

**7. Common mistakes**

- Raising `-Xss` without considering thread count.
- Using recursion for unbounded user input.
- Confusing a long stack trace with heap exhaustion.
- Catching the error and retrying the same recursive path.

**8. Interview difficulty**

**Beginner**

---

### Question 19: What causes memory leaks in Java and how do you find them?

**1. Interview-ready answer**

A Java memory leak is accidental retention: objects remain strongly reachable from GC roots even though the application no longer needs them. Common causes are unbounded static collections, uncleared `ThreadLocal`s, caches without eviction, unremoved listeners, class loader leaks, and unclosed resources. I find them by watching live-set growth in GC logs/heap metrics, then taking a heap dump and inspecting retained paths in Eclipse MAT or VisualVM.

**2. Detailed explanation**

Common leak patterns:

- Static `Map` used as a cache with no TTL or size bound.
- `ThreadLocal` not removed before returning a pooled thread.
- Listeners or callbacks registered but never unregistered.
- Completable futures holding large request objects.
- Bytecode generation creating classes that keep loaders alive.
- `ClassLoader` references from static fields or threads after redeploy.
- Unclosed streams and direct buffers.

**3. Internal working**

A leaked object is still reachable, so tracing collectors correctly keep it alive. Heap dump analyzers compute the **retained heap** — memory that would be freed if the object were removed — and show the shortest path to a GC root. That path usually identifies the exact field holding the reference.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashMap;
import java.util.Map;

public final class TenantContext {

    private static final ThreadLocal<Map<String, String>> CONTEXT = new ThreadLocal<>();

    public static void set(String tenantId, String userId) {
        Map<String, String> values = new HashMap<>();
        values.put("tenantId", tenantId);
        values.put("userId", userId);
        CONTEXT.set(values);
    }

    public static String get(String key) {
        Map<String, String> values = CONTEXT.get();
        return values == null ? null : values.get(key);
    }

    public static void clear() {
        CONTEXT.remove();
    }
}
```

In a Spring application, call `TenantContext.clear()` in a `finally` block or a servlet filter; otherwise Tomcat's pooled thread keeps the last tenant's values alive.

Useful commands:

```bash
jcmd <pid> GC.class_histogram
jcmd <pid> GC.heap_dump /tmp/orders.hprof
jstat -gcutil <pid> 1000
jcmd <pid> VM.native_memory summary
```

**5. Real-world use case**

After each deploy, metaspace usage rises and never drops. A heap dump shows a `ThreadLocal` in a container thread referencing an application class, preventing the web app class loader from unloading.

**6. Interview follow-up questions**

- **Why doesn't GC collect leaked objects?** They are reachable from roots.
- **Which tool shows retained size?** Eclipse MAT dominator tree.
- **Can a native memory leak appear in heap dump?** No; use NMT or native profilers.
- **How do you detect a slow leak?** Trend live-set size after full GCs over hours.
- **Is SoftReference a fix for a leak?** No; it only changes collection behavior.

**7. Common mistakes**

- Looking only at heap usage before GC instead of after GC.
- Assuming OOM means the heap is simply too small.
- Taking a dump without capturing the allocation context.
- Forgetting that thread pools reuse threads and their ThreadLocals.

**8. Interview difficulty**

**Advanced**

---

### Question 20: Explain strong, soft, weak, and phantom references with backend examples

**1. Interview-ready answer**

A strong reference prevents GC entirely. A soft reference is cleared only under memory pressure, useful for memory-sensitive caches. A weak reference is cleared at the next GC, useful for canonical mappings such as `WeakHashMap`. A phantom reference offers post-collection notification and is mostly replaced by `Cleaner` for resource cleanup.

**2. Detailed explanation**

| Reference type | Collection behavior | Typical use |
|---|---|---|
| Strong | Never collected while reachable | Normal object references |
| Soft | Collected when memory is low | Image/report caches |
| Weak | Collected on next GC | WeakHashMap, metadata associates |
| Phantom | Enqueued after object is collected | Cleanup notifications |

Reachability order is: strong → soft → weak → phantom. `finalize()` is deprecated for removal; `Cleaner` and phantom references are the modern mechanisms for tying cleanup to object lifetime, though deterministic `AutoCloseable` is preferred.

**3. Internal working**

The reference package uses special reference objects processed by the collector. When reachability changes, the JVM enqueues registered references to a `ReferenceQueue`. `Cleaner` runs a separate thread that polls the queue and invokes cleanup actions. The cleanup action must not hold a strong reference to the owned object, or it will never become unreachable.

**4. Practical Java/Spring Boot example**

```java
import java.lang.ref.Cleaner;
import java.nio.ByteBuffer;
import java.util.Map;
import java.util.WeakHashMap;

public class ReferenceTypesDemo {

    record Product(String sku, String description) {
    }

    static final class DirectBufferHolder implements AutoCloseable {

        private static final Cleaner CLEANER = Cleaner.create();

        private final State state;
        private final Cleaner.Cleanable cleanable;

        DirectBufferHolder(int size) {
            this.state = new State(ByteBuffer.allocateDirect(size));
            this.cleanable = CLEANER.register(this, state);
        }

        public void read(byte[] destination) {
            state.buffer.get(destination);
        }

        @Override
        public void close() {
            cleanable.clean();
        }

        static final class State implements Runnable {

            private ByteBuffer buffer;

            State(ByteBuffer buffer) {
                this.buffer = buffer;
            }

            @Override
            public void run() {
                buffer = null;
            }
        }
    }

    public static void main(String[] args) {
        Map<Product, String> labels = new WeakHashMap<>();
        Product product = new Product("SKU-1", "Laptop");
        labels.put(product, "Premium laptop");

        try (DirectBufferHolder holder = new DirectBufferHolder(1024)) {
            byte[] destination = new byte[8];
            holder.read(destination);
            System.out.println(labels.get(product));
        }
    }
}
```

The `WeakHashMap` entry disappears after `product` becomes unreachable. The holder uses `Cleaner` as a safety net; production code should still close it deterministically.

**5. Real-world use case**

A cache associates metadata with `ClassLoader` or `Connection` objects. Using `WeakHashMap` avoids keeping the keys alive solely because of the cache.

**6. Interview follow-up questions**

- **Why not use WeakHashMap for a business cache?** Entries vanish unpredictably after GC.
- **What is a memory-sensitive cache?** It releases entries when memory pressure rises.
- **Why must Cleaner state avoid the owner reference?** Otherwise the owner stays strongly reachable.
- **Is cleanup immediate?** No; it depends on GC and the cleaner thread.
- **Why prefer try-with-resources?** Deterministic release beats GC-dependent cleanup.

**7. Common mistakes**

- Using weak references to fix a retention bug.
- Keeping strong references to keys in a WeakHashMap value.
- Relying on `finalize()`.
- Assuming `SoftReference` guarantees no OOM.

**8. Interview difficulty**

**Advanced**

---

## Chapter 6 Verification

| Syllabus topic | Question(s) |
|---|---|
| JVM architecture and execution flow | Q1 |
| Class loading, linking, initialization | Q2 |
| Class loader hierarchy and delegation | Q3, Q4 |
| Runtime data areas | Q5, Q9 |
| Heap structure and allocation | Q6, Q11 |
| Stack frames and recursion | Q7, Q18 |
| Metaspace vs PermGen | Q8 |
| Object layout and compressed oops | Q10 |
| GC roots and reachability | Q12, Q20 |
| Mark-sweep, compact, copying, generational GC | Q13 |
| Collector comparison and G1 internals | Q14, Q15 |
| GC tuning and JVM flags | Q16 |
| OutOfMemoryError and StackOverflowError | Q17, Q18 |
| Memory leaks and troubleshooting | Q19 |
| Reference types and Cleaner | Q20 |

**Total interview questions: 20** — all with the required 8-part format and practical Java/JVM examples.

**Next:** [Chapter 7 — Collections Framework](../02-Collections/07-Collections-Framework.md). Say `NEXT CHAPTER` to continue.
