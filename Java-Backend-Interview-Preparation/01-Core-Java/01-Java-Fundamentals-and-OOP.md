# Chapter 1: Java Fundamentals and Object-Oriented Programming

**Chapter goal:** Answer confidently when an interviewer asks how Java works from source code to a running backend service, and prove with code that you understand classes, objects, constructors, the four OOP pillars, polymorphism, interfaces, access control, composition, and SOLID design.

**Topics covered:** JVM/JDK/JRE · compilation and execution lifecycle · bytecode · platform independence · classes and objects · constructors · OOP pillars · encapsulation · inheritance · polymorphism · abstraction · overloading vs overriding · interfaces vs abstract classes · static and instance members · `this` and `super` · access modifiers · composition vs inheritance · association/aggregation/composition · SOLID · output-based questions.

**Cross-references:** Class loading, heap/stack and JIT are explored deeply in [Chapter 6](../01-Core-Java/06-JVM-Memory-and-Garbage-Collection.md). Strings and immutability are in [Chapter 2](02-Strings-and-Object-Handling.md). Spring beans and proxies that rely on these OOP ideas are in [Chapter 9](../04-Spring/09-Spring-Framework.md).

---

### Question 1: What is the difference between JVM, JRE, and JDK?

**1. Interview-ready answer**

The JVM is the virtual machine that actually executes Java bytecode and provides runtime services like memory management, garbage collection, and security checks. The JRE is the runtime environment: the JVM plus the standard library classes needed to run a Java program. The JDK is the full development kit: the JRE plus development tools such as `javac`, `jar`, `javadoc`, and `jstack`. In short, **JDK = JRE + tools**, and **JRE = JVM + libraries**. Since Java 9, Oracle no longer ships a separate standalone JRE; you use a JDK or build a custom runtime image with `jlink`. The JVM itself is platform-specific — you install a Windows JVM or a Linux JVM — but the bytecode it runs is platform-independent.

**2. Detailed explanation**

- **JVM (Java Virtual Machine):** A specification and a running process. It loads `.class` files, verifies them, and executes instructions. It also manages the heap, stacks, garbage collector, threads, and native method calls.
- **JRE (Java Runtime Environment):** Everything needed to *run* an already compiled Java application: the JVM plus `java.base` and other runtime modules.
- **JDK (Java Development Kit):** Everything needed to *develop and run*: the JRE plus the compiler `javac`, packaging tools, debuggers, and monitoring utilities (`jcmd`, `jmap`, `jfr`).

Why the distinction exists: it lets you ship only the runtime to production instead of the compiler, reducing attack surface and image size. With modules (Java 9+), `jlink` can create a tiny custom runtime containing only the modules your application uses.

Advantages of the layered model: write once, run anywhere; strong runtime verification; consistent memory management. Disadvantages: startup cost and memory overhead compared with native binaries; behavior differences between JVM vendors and versions.

**3. Internal working**

When you run `java -jar order-service.jar`, the launcher creates a JVM instance. The JVM: (1) loads the bootstrap/system classes, (2) finds and loads the application's main class through the application class loader, (3) verifies and initializes classes, (4) invokes `main` on the main thread, (5) manages objects on the heap and executes bytecode via the interpreter plus the JIT compiler, and (6) keeps running until non-daemon threads finish or `System.exit()` is called. The detailed JVM architecture is covered in Chapter 6.

**4. Practical Java/Spring Boot example**

```java
public class RuntimeInfo {

    public static void main(String[] args) {
        Runtime runtime = Runtime.getRuntime();

        System.out.println("Java version: " + Runtime.version());
        System.out.println("JVM name: " + System.getProperty("java.vm.name"));
        System.out.println("JVM vendor: " + System.getProperty("java.vm.vendor"));
        System.out.println("Available processors: " + runtime.availableProcessors());
        System.out.println("Max heap (MB): " + (runtime.maxMemory() / (1024 * 1024)));
    }
}
```

Important lines:
- `Runtime.version()` returns structured version information (Java 9+), useful for verifying that production actually runs Java 17 or 21.
- `Runtime.getRuntime()` gives access to JVM runtime information such as processors and memory limits.
- `System.getProperty("java.vm.name")` distinguishes HotSpot, OpenJ9, and other JVM implementations.

Typical commands:

```bash
javac RuntimeInfo.java
java RuntimeInfo
java --version
```

For a Spring Boot container, the Dockerfile decision is the practical version of this question:

```dockerfile
FROM eclipse-temurin:17-jre AS runtime
COPY target/order-service.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

Using the `jre` image keeps the container smaller than the `jdk` image.

**5. Real-world use case**

A payment service is built on Java 17 and deployed as a container. The build stage uses a full JDK image to compile with Maven; the final stage copies only the fat JAR into a JRE-based image. When production shows `UnsupportedClassVersionError`, the engineer checks `java --version` in the running container and discovers the runtime image is Java 11 while the JAR was compiled with `--release 17`.

**6. Interview follow-up questions**

- **Is the JVM platform-independent?** No. Each OS has its own JVM implementation. Bytecode and `.class` files are platform-independent.
- **Can Java run without a JVM?** No. Bytecode needs a JVM; native images (GraalVM) are a different deployment model where compilation happens ahead of time.
- **What is `javac`?** The Java compiler in the JDK that converts `.java` source into `.class` bytecode.
- **Why do Docker images use JRE instead of JDK?** Smaller size and smaller attack surface because compilers and debugging tools are not needed at runtime.
- **What does `java -version` show?** The installed JVM/JDK version and vendor, used to confirm runtime compatibility.

**7. Common mistakes**

- Saying "JVM includes the compiler" — compilation is a JDK tool, not a JVM runtime responsibility.
- Saying "Java is platform-independent because the JVM is platform-independent" — the opposite is true.
- Assuming Java 8-style separate JRE installers still exist in modern JDKs.
- Confusing `java` (launcher) with `javac` (compiler).

**8. Interview difficulty**

**Beginner**

---

### Question 2: Explain the complete Java compilation and execution lifecycle.

**1. Interview-ready answer**

A `.java` source file is compiled by `javac` into a `.class` file containing bytecode and a constant pool. At runtime, the JVM class loader loads that class, links it by verifying bytecode, preparing static storage, and resolving symbolic references, then initializes static fields and static blocks. The execution engine interprets bytecode and the JIT compiler compiles frequently executed methods into native machine code for speed. During execution, objects live on the heap, method calls use stack frames, and the garbage collector reclaims unreachable objects. Finally, the program ends when all non-daemon threads complete. For a Spring Boot application, this same lifecycle is wrapped by the SpringApplication bootstrap process.

**2. Detailed explanation**

The phases are:

1. **Compilation (`javac`):** Lexical analysis, parsing, type checking, and generation of `.class` files. No machine code is produced. Each class gets its own `.class` file, including nested classes.
2. **Loading:** The class loader finds the binary class bytes and creates a `Class<?>` object in the method area/metaspace.
3. **Linking:**
   - *Verification:* ensures bytecode is safe and does not violate JVM rules.
   - *Preparation:* allocates memory for static fields and sets default values (not explicit values).
   - *Resolution:* converts symbolic references in the constant pool into direct references.
4. **Initialization:** Executes static field initializers and static blocks in source order, starting with the superclass.
5. **Execution:** The interpreter starts; the JIT compiles hot methods; the GC manages heap objects.
6. **Unloading:** Classes can be unloaded when their class loader becomes unreachable (important in application servers and plugin systems).

Why this design: a verified intermediate representation gives portability plus safety; the JIT gives near-native performance after warm-up.

Advantages: portability, runtime safety, aggressive runtime optimizations. Disadvantages: startup and warm-up time, extra memory for bytecode and metadata.

**3. Internal working**

A `.class` file contains a magic number `CAFEBABE`, version numbers, a constant pool, access flags, fields, methods, attributes (including `Code` with the actual bytecode for each method), and optional debugging information. You can inspect it with `javap -c -p MyClass`.

The JIT uses profiling counters. When a method crosses a threshold, C2 (or Graal) compiles it to native code using profiles such as branch probabilities and observed types. This is why a Java service often gets faster after a few minutes of traffic — a key point for latency interviews.

**4. Practical Java/Spring Boot example**

```java
public class BookingLauncher {

    private static final String SERVICE_NAME = "booking-service";

    static {
        System.out.println("Initializing " + SERVICE_NAME);
    }

    public static void main(String[] args) {
        BookingLauncher launcher = new BookingLauncher();
        int total = launcher.addInventory(40, 2);
        System.out.println("Available seats: " + total);
    }

    private int addInventory(int existingSeats, int newSeats) {
        int sum = existingSeats + newSeats;
        return sum;
    }
}
```

Important lines:
- The static block runs during class initialization, before `main` is invoked.
- `addInventory` is a separate method so you can see a distinct bytecode block when running `javap`.

Inspect it:

```bash
javac BookingLauncher.java
javap -c -p BookingLauncher
```

For Spring Boot, the entry point looks like this; the class loading and initialization phases are identical, then Spring takes over:

```java
@SpringBootApplication
public class BookingServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(BookingServiceApplication.class, args);
    }
}
```

**5. Real-world use case**

An airline booking service reports slow responses only during the first minute after every deployment. The engineer explains that this is JIT warm-up plus lazy initialization of Hibernate and HTTP client pools, and fixes it with a readiness probe that delays traffic until warm-up completes, plus JVM flags for tiered compilation and CDS (Class Data Sharing).

**6. Interview follow-up questions**

- **What is the constant pool?** A table in the `.class` file holding literals, class names, method names, and symbolic references used by bytecode.
- **When are static fields assigned their declared values?** During initialization, after preparation assigned default values.
- **What is the difference between class initialization and object creation?** Initialization runs static code once per class; object creation runs instance field initializers and a constructor each time.
- **What does `javap` do?** Disassembles a `.class` file so you can read bytecode and verify compiler behavior.
- **Why can a Spring Boot app be slow to start?** Component scanning, bean creation, auto-configuration, and JIT warm-up.

**7. Common mistakes**

- Saying `javac` produces machine code.
- Thinking static fields get their declared values during loading — preparation only sets defaults.
- Forgetting that initialization order is superclass first.
- Believing JIT compilation happens before the program starts (it happens at runtime; GraalVM native image is the exception).

**8. Interview difficulty**

**Intermediate**

---

### Question 3: What is bytecode, and how does Java achieve platform independence?

**1. Interview-ready answer**

Bytecode is a compact, stack-based instruction set stored in `.class` files. It is not machine code for any real CPU; it is the instruction set of the JVM. `javac` always produces the same bytecode regardless of the operating system. Each platform has its own JVM that translates that bytecode into native instructions, using an interpreter for quick startup and a JIT compiler for hot code. That separation — one portable intermediate representation, many platform-specific JVMs — is how Java achieves "write once, run anywhere." Bytecode is also verified at load time, which makes Java more secure than running native binaries directly.

**2. Detailed explanation**

Bytecode instructions operate on an operand stack rather than CPU registers. Examples: `iload_1` pushes an int local variable, `iadd` pops two ints and pushes their sum, `ireturn` returns an int. Types are encoded in opcodes (`i` for int, `a` for reference, `d` for double).

Platform independence comes from three contracts:
- The class file format is fixed and documented.
- The bytecode instruction set is fixed.
- The JVM specification defines exact runtime behavior.

Any vendor that implements the spec can run the same JAR. The trade-off is that the JVM must perform verification, class loading, and JIT compilation, so startup is slower than a native executable, and behavior can differ slightly between JVM implementations or versions.

**3. Internal working**

The verifier performs data-flow analysis: it checks that stack depths and types are consistent at every branch target, that accesses respect visibility, and that no invalid casts occur. Verification is what prevents a malicious class file from crashing the JVM. After verification, the JIT can make optimistic assumptions because types are already known to be safe.

**4. Practical Java/Spring Boot example**

```java
public class FareCalculator {

    public int calculateFare(int baseFare, int tax) {
        int total = baseFare + tax;
        return total;
    }
}
```

Compile and disassemble:

```bash
javac FareCalculator.java
javap -c FareCalculator
```

Representative bytecode for `calculateFare`:

```text
iload_1
iload_2
iadd
istore_3
iload_3
ireturn
```

Explanation: `iload_1` loads `baseFare`, `iload_2` loads `tax`, `iadd` adds them, `istore_3` stores the result in local variable 3 (`total`), `iload_3` pushes it, and `ireturn` returns it. Local variable 0 is `this` for an instance method.

Important note: modern javac may optimize away the temporary variable. If `javap` shows a shorter sequence, that is the compiler producing equivalent bytecode — the reasoning about the stack machine is still correct.

**5. Real-world use case**

A fintech deploys the same `payment-service.jar` to an Azure Container Apps Linux node and a developer's Windows laptop. No recompilation is needed. The team pins the base image to `eclipse-temurin:17-jre` to guarantee identical bytecode verification and JIT behavior across environments.

**6. Interview follow-up questions**

- **Is bytecode the same as machine code?** No. Bytecode targets the JVM; machine code targets a physical CPU.
- **What is a stack-based VM?** Instructions work on an operand stack instead of hardware registers, simplifying portability.
- **What verifies bytecode?** The JVM verifier during the linking phase.
- **Does JIT compile everything?** No. It compiles methods that become hot; cold code stays interpreted.
- **How is GraalVM native image different?** It compiles ahead of time to a native binary, trading portability and dynamic features for instant startup.

**7. Common mistakes**

- Claiming bytecode runs directly on the CPU.
- Claiming all JVMs execute bytecode identically in performance terms — JIT strategies differ.
- Forgetting that verification exists and is a major part of JVM security.

**8. Interview difficulty**

**Beginner**

---

### Question 4: What are classes and objects? What happens when you create an object with `new`?

**1. Interview-ready answer**

A class is a blueprint or template that defines state through fields and behavior through methods. An object is a runtime instance of that class with its own copy of instance fields, plus an identity and a reference to its class. When you write `new FlightBooking(...)`, the JVM allocates memory on the heap, initializes instance fields to default values, runs instance initializers and the constructor to set the explicit state, and returns a reference that the variable holds on the stack. Multiple objects of the same class share method bytecode but not instance state. In backend code, entities like `Order` are classes and each row becomes an object managed by Hibernate.

**2. Detailed explanation**

A class can contain fields, methods, constructors, initializer blocks, nested types, and static members. An object has three characteristics: **state** (field values), **behavior** (methods it can invoke), and **identity** (distinct memory location and `equals`/`hashCode` semantics).

The reference variable is not the object. `FlightBooking booking = new FlightBooking("BK1");` creates a reference `booking` on the stack (or as a field inside another object) pointing to the object on the heap. Assigning `booking2 = booking` copies the reference, not the object — both variables now point to the same object. This distinction explains many bug reports about unexpected mutation.

Advantages of classes and objects: modularity, encapsulation, reuse, and a natural mapping from domain concepts to code. Disadvantage: if a class holds too much state or behavior, it becomes a "god object" that is hard to test.

**3. Internal working**

Allocation may use bump-the-pointer in the young generation of the heap; with several threads, TLABs (thread-local allocation buffers) reduce contention. Object layout includes a header (mark word and class pointer), instance fields, and padding. Field initializers compile into the constructor. The reference returned by `new` may be optimized away by escape analysis if the object never leaves the method — the JVM can even allocate such an object on the stack or eliminate it entirely.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

public class FlightBooking {

    private final String bookingReference;
    private String passengerName;
    private BigDecimal fare;

    public FlightBooking(String bookingReference, String passengerName, BigDecimal fare) {
        this.bookingReference = bookingReference;
        this.passengerName = passengerName;
        this.fare = fare;
    }

    public void applyDiscount(BigDecimal discount) {
        this.fare = this.fare.subtract(discount);
    }

    public String summary() {
        return bookingReference + " | " + passengerName + " | " + fare;
    }

    public static void main(String[] args) {
        FlightBooking first = new FlightBooking("BK1001", "Asha", BigDecimal.valueOf(8500));
        FlightBooking second = new FlightBooking("BK1002", "Ravi", BigDecimal.valueOf(9200));

        first.applyDiscount(BigDecimal.valueOf(500));
        second.applyDiscount(BigDecimal.valueOf(700));

        System.out.println(first.summary());
        System.out.println(second.summary());
    }
}
```

Output:

```text
BK1001 | Asha | 8000
BK1002 | Ravi | 8500
```

Important lines:
- `private final String bookingReference` makes identity immutable after construction.
- Each `new` call produces independent `fare` state; `first` and `second` do not affect each other.
- Methods live once in the class metadata; only fields are duplicated per object.

**5. Real-world use case**

In a Spring Boot order service, `Order` is a JPA entity class. Each order row loaded from PostgreSQL becomes a managed object in Hibernate's persistence context. Two requests loading the same row in different transactions get two different Java objects holding the same database identity — which is why `equals`/`hashCode` must be designed carefully (see Chapter 2).

**6. Interview follow-up questions**

- **Is an object stored on the stack?** No. The object is on the heap; the reference variable may be on the stack.
- **Can two references point to the same object?** Yes. Mutating through one is visible through the other.
- **How many copies of a method exist?** One set of bytecode per class, shared by all instances.
- **What is the default value of a reference field?** `null`.
- **Can a class have no objects?** Yes — utility classes such as `java.lang.Math` are never instantiated.

**7. Common mistakes**

- Saying a reference variable "is" the object.
- Assuming `new` always allocates on the heap — escape analysis can eliminate allocation.
- Forgetting that instance field initializers run before the constructor body.
- Confusing static fields (one per class) with instance fields (one per object).

**8. Interview difficulty**

**Beginner**

---

### Question 5: What is a constructor in Java? What are the rules and types?

**1. Interview-ready answer**

A constructor is a special block of code used to initialize a newly created object. It has the same name as the class and no return type. If you do not declare one, the compiler inserts a no-argument default constructor that calls `super()` and does nothing else. Constructors can be overloaded, chained with `this(...)` or `super(...)`, and made private to restrict instantiation — for example in a singleton or utility class. They cannot be inherited, overridden, `static`, `final`, `abstract`, or `synchronized`. Every constructor call implicitly starts with `super()` unless you explicitly call another constructor as the first statement.

**2. Detailed explanation**

Types of constructors:
- **Default constructor:** Compiler-generated no-arg constructor, present only if you declare no constructor at all.
- **No-arg constructor:** Explicitly written constructor with no parameters; needed by frameworks such as Hibernate and Jackson for reflection-based instantiation.
- **Parameterized constructor:** Accepts initial state; supports invariants at creation time.
- **Copy constructor:** Takes an object of the same class and copies its state. Java does not auto-generate one.
- **Private constructor:** Used in singletons, factories, and utility classes.

Rules:
- Name must match the class exactly, including case.
- No return type — a method named like the class with a `void` return type is just a method.
- You cannot use `return value;` inside a constructor.
- `this(...)` and `super(...)` must be the first statement and cannot both appear in one constructor.
- The first statement of any constructor is either `this(...)`, `super(...)`, or an implicit `super()`.
- Constructors are not inherited: `class Child extends Parent` does not get Parent's constructors.

Why it exists: to guarantee that an object starts life in a valid state. Advantages: invariants, dependency enforcement, immutability. Disadvantages: too many overloads create confusion; heavy work in constructors makes objects hard to test.

**3. Internal working**

At the bytecode level, a constructor compiles to a `<init>` method. It first invokes the superclass `<init>` (or another `<init>` via `this(...)`), then executes instance field initializers in source order, then the constructor body. Verification requires that `<init>` be called on a freshly allocated object before the reference can be used — this is why you cannot call instance methods before `super()` in Java source.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.Objects;

public class PaymentRequest {

    private final String orderId;
    private final BigDecimal amount;
    private final String currency;

    public PaymentRequest(String orderId, BigDecimal amount) {
        this(orderId, amount, "INR");
    }

    public PaymentRequest(String orderId, BigDecimal amount, String currency) {
        this.orderId = Objects.requireNonNull(orderId, "orderId must not be null");
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("amount must be positive");
        }
        this.amount = amount;
        this.currency = Objects.requireNonNull(currency, "currency must not be null");
    }

    public PaymentRequest(PaymentRequest original) {
        this(original.orderId, original.amount, original.currency);
    }

    public String display() {
        return orderId + " : " + amount + " " + currency;
    }

    public static void main(String[] args) {
        PaymentRequest defaultCurrency = new PaymentRequest("ORD-1", BigDecimal.valueOf(2500));
        PaymentRequest usdRequest = new PaymentRequest("ORD-2", BigDecimal.valueOf(3000), "USD");
        PaymentRequest copy = new PaymentRequest(defaultCurrency);

        System.out.println(defaultCurrency.display());
        System.out.println(usdRequest.display());
        System.out.println(copy.display());
    }
}
```

Output:

```text
ORD-1 : 2500 INR
ORD-2 : 3000 USD
ORD-1 : 2500 INR
```

Important lines:
- `this(orderId, amount, "INR")` chains to the main constructor — it must be the first statement.
- `Objects.requireNonNull` and the amount check enforce invariants at construction time.
- The copy constructor delegates to the same validating constructor, so no invalid object can be created through copying.

**5. Real-world use case**

A payment API receives JSON. Jackson deserializes it into a DTO using the no-arg constructor plus setters (or field access), then a service maps it into an immutable domain object like `PaymentRequest` through a validating constructor. This split lets the wire layer stay lenient while the domain layer stays strict. Hibernate similarly requires a no-arg constructor for entities but your application code should use the all-args constructor.

**6. Interview follow-up questions**

- **Can a constructor be `static`?** No. Constructors belong to instances; `<clinit>` handles static initialization.
- **Can a constructor be overloaded?** Yes, as long as parameter lists differ.
- **What is a default constructor?** The compiler-generated no-arg constructor; it disappears as soon as you declare any constructor.
- **Can a constructor call another constructor?** Yes, with `this(...)` as the first statement.
- **Why does Hibernate need a no-arg constructor?** It uses reflection to instantiate entities when loading rows from the database.

**7. Common mistakes**

- Adding a return type and accidentally creating a method instead of a constructor.
- Calling `this(...)` and `super(...)` in the same constructor.
- Doing heavy I/O or starting threads inside constructors — object creation should be cheap and side-effect-light; use an `init` method or factory instead.
- Assuming a copy constructor does deep copy automatically — nested mutable fields must be copied explicitly (Chapter 4).

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 6: What are the four pillars of object-oriented programming?

**1. Interview-ready answer**

The four pillars are encapsulation, inheritance, polymorphism, and abstraction. Encapsulation hides internal state and exposes controlled access through methods. Inheritance lets a subclass reuse and extend the behavior of a superclass, modeling an "is-a" relationship. Polymorphism lets the same call behave differently depending on the actual object type — compile-time through overloading and runtime through overriding. Abstraction exposes only the essential contract and hides implementation details, using abstract classes and interfaces. In a backend service these appear together: an `OrderService` depends on a `PaymentGateway` interface (abstraction), implementations inherit common behavior (inheritance), each gateway overrides `charge()` (polymorphism), and all hold private state (encapsulation).

**2. Detailed explanation**

- **Encapsulation** protects invariants. Making fields private and exposing `deposit()` instead of a raw `setBalance()` prevents negative balances.
- **Inheritance** promotes reuse and establishes type substitutability. Use it only for genuine "is-a" relationships; otherwise prefer composition.
- **Polymorphism** removes conditional logic. Instead of `if (type == CREDIT_CARD) ... else if (type == UPI) ...`, you call `gateway.charge(request)` and let dynamic dispatch choose the implementation.
- **Abstraction** reduces coupling. Callers know *what* a component does, not *how* it does it.

Advantages: maintainability, testability, extensibility. Disadvantages if overused: deep inheritance hierarchies, excessive indirection, and performance overhead from virtual dispatch (usually negligible compared with I/O).

**3. Internal working**

Runtime polymorphism works through a virtual method table (vtable). Each class has a table mapping method signatures to implementation addresses. An object's header points to its class metadata. When `gateway.charge()` is invoked, the JVM reads the object's actual class and dispatches through the vtable. JIT compilers optimize this with inline caches and even devirtualization when only one implementation is loaded.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

interface PaymentGateway {

    String charge(BigDecimal amount);

    default String providerName() {
        return "generic-gateway";
    }
}

abstract class AbstractPaymentGateway implements PaymentGateway {

    protected final String merchantId;

    protected AbstractPaymentGateway(String merchantId) {
        this.merchantId = merchantId;
    }

    protected void validateAmount(BigDecimal amount) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Invalid amount");
        }
    }
}

final class CardGateway extends AbstractPaymentGateway {

    CardGateway(String merchantId) {
        super(merchantId);
    }

    @Override
    public String charge(BigDecimal amount) {
        validateAmount(amount);
        return "CARD txn for " + amount + " by " + merchantId;
    }

    @Override
    public String providerName() {
        return "card-gateway";
    }
}

final class UpiGateway extends AbstractPaymentGateway {

    UpiGateway(String merchantId) {
        super(merchantId);
    }

    @Override
    public String charge(BigDecimal amount) {
        validateAmount(amount);
        return "UPI txn for " + amount + " by " + merchantId;
    }
}

public class OopPillarsDemo {

    public static void main(String[] args) {
        PaymentGateway gateway = new CardGateway("MERCHANT-9");
        System.out.println(gateway.charge(BigDecimal.valueOf(1200)));
        System.out.println(gateway.providerName());

        gateway = new UpiGateway("MERCHANT-9");
        System.out.println(gateway.charge(BigDecimal.valueOf(800)));
        System.out.println(gateway.providerName());
    }
}
```

Output:

```text
CARD txn for 1200 by MERCHANT-9
card-gateway
UPI txn for 800 by MERCHANT-9
generic-gateway
```

Important lines:
- `PaymentGateway gateway = new CardGateway(...)` — abstraction plus polymorphism: the variable type is the interface, the behavior comes from the runtime object.
- `validateAmount` is inherited shared behavior.
- Private/final fields and validation methods demonstrate encapsulation.

**5. Real-world use case**

An e-commerce checkout decides the gateway at runtime based on the payment method. Spring injects a `Map<String, PaymentGateway>` keyed by bean name, and the checkout service picks one. Adding a new wallet gateway requires only a new class, not changes to the checkout logic — the Open/Closed Principle in action.

**6. Interview follow-up questions**

- **Which pillar is most important in Spring?** Abstraction and polymorphism — Spring's entire container is built on interfaces and proxies.
- **Does inheritance break encapsulation?** It can, because subclasses may depend on superclass internals. Prefer protected hooks and composition.
- **Is polymorphism only runtime?** No. Overloading is compile-time polymorphism.
- **Can abstraction exist without inheritance?** Yes — an interface is abstraction without class inheritance.
- **What is the fragile base class problem?** A change to a superclass silently breaks subclasses that depended on its internal behavior.

**7. Common mistakes**

- Listing definitions without a code example — interviewers usually ask for one.
- Using inheritance for code reuse when the relationship is not "is-a".
- Claiming encapsulation means only making fields private; it also means controlling all state transitions.
- Confusing abstraction with simply hiding complexity behind a class name.

**8. Interview difficulty**

**Beginner**

---

### Question 7: What is encapsulation? How do you implement it correctly in Java?

**1. Interview-ready answer**

Encapsulation is bundling data and the methods that operate on that data into one unit, while restricting direct access to the internal state. In Java you make fields `private` and expose controlled methods — getters, setters with validation, and behavior methods such as `withdraw()`. The goal is not merely "private fields"; it is to make invalid states impossible. For example, an `Account` should not expose `setBalance(-500)`; it should expose `debit(amount)` that checks funds. Encapsulation also reduces coupling: you can change the internal representation later without breaking callers. In Spring, DTOs, JPA entities, and domain objects all rely on this principle.

**2. Detailed explanation**

Levels of encapsulation:
1. **Field encapsulation:** private fields with public accessors.
2. **Behavior encapsulation:** callers invoke intention-revealing methods rather than mutating fields directly.
3. **Representation encapsulation:** callers do not know whether a balance is stored as `BigDecimal`, cents as `long`, or a computed value.

Advantages: invariants are enforced centrally, debugging is easier because all state changes pass through known methods, and the class can evolve internally. Disadvantages: boilerplate getters/setters; excessive getter chains can produce anemic domain models where all logic lives in services.

**3. Internal working**

Private access is enforced by the JVM verifier and access flags in the class file. Reflection (used by Spring and Hibernate) can bypass it with `setAccessible(true)`, and Java modules can restrict deep reflection. This is why `private` is an application-level and JVM-level boundary, not a security boundary against a determined attacker with reflection.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.Objects;

public class BankAccount {

    private final String accountNumber;
    private BigDecimal balance;

    public BankAccount(String accountNumber, BigDecimal openingBalance) {
        this.accountNumber = Objects.requireNonNull(accountNumber);
        if (openingBalance == null || openingBalance.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Opening balance cannot be negative");
        }
        this.balance = openingBalance;
    }

    public String getAccountNumber() {
        return accountNumber;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public void debit(BigDecimal amount) {
        validateAmount(amount);
        if (balance.compareTo(amount) < 0) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance = balance.subtract(amount);
    }

    public void credit(BigDecimal amount) {
        validateAmount(amount);
        balance = balance.add(amount);
    }

    private void validateAmount(BigDecimal amount) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount("ACC-77", BigDecimal.valueOf(10000));
        account.debit(BigDecimal.valueOf(2500));
        account.credit(BigDecimal.valueOf(1000));
        System.out.println(account.getAccountNumber() + " balance = " + account.getBalance());
    }
}
```

Output:

```text
ACC-77 balance = 8500
```

Important lines:
- `private BigDecimal balance` prevents callers from setting an arbitrary value.
- `debit` and `credit` are intention-revealing operations that validate first.
- `balance = balance.subtract(amount)` uses `BigDecimal`'s immutability — the result is a new object assigned back to the field.

**5. Real-world use case**

A banking service exposes `POST /accounts/{id}/debit`. The controller maps the request to a command object; the `BankAccount` aggregate enforces the "no overdraft" rule inside `debit()`. Even if two different controllers or a Kafka consumer call the aggregate, the invariant is enforced in one place. The database adds a second line of defense with a check constraint.

**6. Interview follow-up questions**

- **Is a POJO with only getters/setters well encapsulated?** Not really — it is a data carrier. Encapsulation is stronger when behavior and invariants live with the data.
- **Can reflection break encapsulation?** Yes. Spring uses it for injection; modules and security managers restrict it.
- **Should all fields be private?** Almost always for domain objects; constants may be `public static final`.
- **Why is `BigDecimal` preferred for money?** It avoids floating-point rounding errors (Chapter 2 covers it).
- **What is an anemic domain model?** Data classes with no behavior and services containing all business logic.

**7. Common mistakes**

- Generating setters for every field and calling that encapsulation.
- Exposing mutable collections directly through a getter — return a defensive copy or unmodifiable view.
- Putting validation only in the controller; another caller can bypass it.
- Assuming `private` prevents debugging tools from reading the field.

**8. Interview difficulty**

**Beginner**

---

### Question 8: What is inheritance in Java? Explain with a payment hierarchy.

**1. Interview-ready answer**

Inheritance is a mechanism where a class, called the subclass, derives from another class, called the superclass, using the `extends` keyword. The subclass gets access to non-private members of the superclass, can add its own fields and methods, and can override methods to change behavior. Java supports single class inheritance to avoid the ambiguity of multiple inherited implementations, but a class can implement multiple interfaces. Every class implicitly extends `Object`. Inheritance models an "is-a" relationship — `CreditCardPayment is a Payment` — and constructor calls chain upward with `super()`.

**2. Detailed explanation**

Key rules:
- One direct superclass per class; interfaces can be multiple.
- `final` classes cannot be extended; `final` methods cannot be overridden.
- Constructors are not inherited.
- Private members are inherited in the sense that they exist in the object, but are not directly accessible from the subclass.
- `protected` members are accessible in subclasses and in the same package.
- A subclass constructor must call `super(...)` (explicitly or implicitly) as its first statement.
- Overriding methods can widen access but not narrow it, and can narrow checked exceptions but not widen them.

Why it exists: code reuse and substitutability. Advantages: shared behavior in one place, polymorphic collections. Disadvantages: tight coupling between parent and child, fragile base class problem, and the temptation to use inheritance for reuse where composition is better.

**3. Internal working**

A subclass object contains a full superclass portion plus its own fields — there is no separate parent object. The object header points to the subclass's class metadata, which references its superclass metadata. Method dispatch starts at the actual class and walks up until an implementation is found. Field access, in contrast, is resolved at compile time based on the reference type (see Question 19).

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

abstract class Payment {

    private final String paymentId;

    protected Payment(String paymentId) {
        this.paymentId = paymentId;
    }

    public abstract String pay(BigDecimal amount);

    public String receipt() {
        return "Receipt " + paymentId + " -> " + pay(BigDecimal.valueOf(0));
    }

    protected String paymentId() {
        return paymentId;
    }
}

class CardPayment extends Payment {

    private final String lastFourDigits;

    CardPayment(String paymentId, String lastFourDigits) {
        super(paymentId);
        this.lastFourDigits = lastFourDigits;
    }

    @Override
    public String pay(BigDecimal amount) {
        return "Charged " + amount + " to card ending " + lastFourDigits;
    }
}

class UpiPayment extends Payment {

    private final String upiId;

    UpiPayment(String paymentId, String upiId) {
        super(paymentId);
        this.upiId = upiId;
    }

    @Override
    public String pay(BigDecimal amount) {
        return "Charged " + amount + " via " + upiId;
    }
}

public class InheritanceDemo {

    public static void main(String[] args) {
        Payment payment = new CardPayment("PAY-1", "4242");
        System.out.println(payment.pay(BigDecimal.valueOf(1500)));

        payment = new UpiPayment("PAY-2", "user@bank");
        System.out.println(payment.pay(BigDecimal.valueOf(700)));
    }
}
```

Output:

```text
Charged 1500 to card ending 4242
Charged 700 via user@bank
```

Important lines:
- `super(paymentId)` initializes the superclass portion first.
- `Payment payment = ...` holds a superclass reference to a subclass object — safe because of the "is-a" relationship.
- `pay` is overridden, so dynamic dispatch runs the subclass version.

Note: the `receipt()` method above intentionally calls `pay(BigDecimal.ZERO)` only to illustrate inheritance; in real code you would pass the actual amount. Calling an overridable method from a superclass constructor is a serious mistake and is covered in Question 19.

**5. Real-world use case**

A notification module has `Notification` with shared fields (`recipient`, `createdAt`) and common validation, with subclasses `EmailNotification`, `SmsNotification`, and `PushNotification` implementing `deliver()`. Spring stores them in a `List<Notification>` and iterates. Adding WhatsApp later means one new subclass.

**6. Interview follow-up questions**

- **Why does Java not support multiple class inheritance?** To avoid the diamond problem of conflicting method implementations and ambiguous state.
- **What is the diamond problem?** With two parents defining the same method, the JVM cannot decide which implementation to inherit. Java 8 default methods handle this by forcing the implementer to override on conflict.
- **Are private members inherited?** They exist in the object but are not accessible from subclass code.
- **Can you override a `final` method?** No; compilation fails.
- **What is an "is-a" test?** If you can truthfully say "Subclass is a Superclass", inheritance is appropriate.

**7. Common mistakes**

- Extending a class only to reuse one utility method — use composition.
- Calling overridable methods from constructors.
- Assuming a superclass reference can call subclass-specific methods — you must downcast.
- Forgetting that all classes implicitly extend `Object`, so `toString`, `equals`, and `hashCode` are always available.

**8. Interview difficulty**

**Beginner**

---

### Question 9: What is polymorphism? Differentiate compile-time and runtime polymorphism.

**1. Interview-ready answer**

Polymorphism means "many forms" — the same name or call can take different shapes. In Java, compile-time polymorphism is method overloading: the compiler picks the method based on the argument list at compile time. Runtime polymorphism is method overriding: the JVM picks the implementation based on the actual object type at runtime. For example, `gateway.charge(amount)` behaves differently for `CardGateway` and `UpiGateway`, even though the reference type is the same interface. Runtime polymorphism is what makes frameworks like Spring so flexible — you inject an interface and the container supplies an implementation.

**2. Detailed explanation**

| Aspect | Compile-time (static binding) | Runtime (dynamic binding) |
|---|---|---|
| Mechanism | Overloading | Overriding |
| Decision made by | Compiler | JVM |
| Based on | Method signature and reference type | Actual object type |
| Also called | Early binding | Late binding / virtual dispatch |
| Examples | `pay(int)`, `pay(BigDecimal)` | `gateway.charge()` on different implementations |

Requirements for overriding: same method name and parameter list; instance methods only; return type must be covariant (same or subtype); access cannot be reduced; checked exceptions cannot be broadened; `static`, `final`, and `private` methods cannot be overridden.

Why both exist: overloading improves readability and API ergonomics; overriding enables substitution and extensibility.

**3. Internal working**

Overloads are encoded as distinct methods with different descriptors in the class file; the compiler emits an `invokevirtual` or `invokestatic` call to the exact descriptor. Overrides use `invokevirtual` (or `invokeinterface`) with a symbolic reference resolved to a vtable slot. The JIT profiles the receiver type at each call site; if only one type is seen, it can inline the method and later de-optimize if a new type appears.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

interface DiscountPolicy {

    BigDecimal discount(BigDecimal amount);
}

class NoDiscount implements DiscountPolicy {

    @Override
    public BigDecimal discount(BigDecimal amount) {
        return amount;
    }
}

class MemberDiscount implements DiscountPolicy {

    @Override
    public BigDecimal discount(BigDecimal amount) {
        return amount.multiply(BigDecimal.valueOf(0.9));
    }
}

public class PolymorphismDemo {

    public static BigDecimal checkout(DiscountPolicy policy, BigDecimal amount) {
        return policy.discount(amount);
    }

    public static String format(int amount) {
        return "INR " + amount;
    }

    public static String format(BigDecimal amount) {
        return "INR " + amount.toPlainString();
    }

    public static void main(String[] args) {
        System.out.println(checkout(new NoDiscount(), BigDecimal.valueOf(1000)));
        System.out.println(checkout(new MemberDiscount(), BigDecimal.valueOf(1000)));
        System.out.println(format(500));
        System.out.println(format(BigDecimal.valueOf(750.50)));
    }
}
```

Output:

```text
1000
900.0
INR 500
INR 750.50
```

Important lines:
- `checkout(DiscountPolicy, ...)` never knows the concrete policy — runtime polymorphism.
- The two `format` methods show compile-time selection purely by parameter type.
- `BigDecimal.valueOf(0.9)` constructs from a string-safe representation, avoiding double imprecision.

**5. Real-world use case**

A pricing engine receives a `List<PricingRule>` from Spring. Each rule implements `apply(Cart)`. Rules for coupon, loyalty, and seasonal discounts are invoked in order through the interface. Business users add new rules without touching the engine — a classic polymorphic design that supports the Open/Closed Principle.

**6. Interview follow-up questions**

- **Can static methods be overridden?** No, they can be hidden. Dispatch is based on the reference type.
- **Can constructors be polymorphic?** Overloaded yes, overridden no.
- **What is dynamic dispatch?** Selecting the method implementation at runtime from the receiver's actual class.
- **What is method hiding?** A static method in a subclass with the same signature as the parent's static method.
- **Does polymorphism hurt performance?** Modern JIT devirtualization and inline caches make the overhead negligible; I/O dominates in backend services.

**7. Common mistakes**

- Assuming the compiler chooses an override based on the reference variable's type.
- Trying to override `private` or `static` methods.
- Changing the return type to an unrelated type and expecting it to compile.
- Confusing overloading with overriding when the parameter list differs.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 10: What is abstraction? How do abstract classes differ from interfaces in practice?

**1. Interview-ready answer**

Abstraction means showing only the essential contract and hiding implementation details. In Java, you express it with abstract classes and interfaces. An abstract class can hold state, concrete methods, constructors, and access modifiers, and a subclass extends it with `extends`. An interface defines a contract; since Java 8 it can have `default` and `static` methods, and since Java 9 private helper methods, but it cannot hold instance state. A class can implement many interfaces but extend only one class. Use an abstract class when related classes share state or a common base implementation; use an interface when you want to define a capability that unrelated classes can implement.

**2. Detailed explanation**

Abstract class:
- Declared with `abstract`; cannot be instantiated.
- May have abstract and concrete methods, fields, constructors, and initializer blocks.
- Supports single inheritance.
- Can have `public`, `protected`, and package-private members.

Interface:
- Declared with `interface`; cannot be instantiated.
- Methods are implicitly `public abstract` unless `default`, `static`, or `private`.
- Fields are implicitly `public static final`.
- A class can implement multiple interfaces.
- Java 8: `default` and `static` methods; Java 9: `private` methods for shared logic inside default methods.

Why both exist: abstract classes model a shared base identity; interfaces model capabilities and enable multiple inheritance of type. Advantages: loose coupling, easier testing with mocks, and pluggable implementations. Disadvantage: adding a new abstract method to a widely implemented interface breaks all implementers unless it is a `default` method.

**3. Internal working**

Calling an abstract method compiles to `invokevirtual`/`invokeinterface` against the abstract declaration; resolution finds the concrete implementation in the runtime class. A `default` method is inherited only if the class does not provide its own implementation. If two interfaces provide conflicting defaults, the class must override the method explicitly — otherwise compilation fails.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

interface Auditable {
    String auditId();

    default String auditLabel() {
        return "AUDIT:" + auditId();
    }
}

abstract class AbstractGateway implements Auditable {

    private final String gatewayCode;

    protected AbstractGateway(String gatewayCode) {
        this.gatewayCode = gatewayCode;
    }

    public abstract BigDecimal charge(BigDecimal amount);

    protected String describe() {
        return gatewayCode + " / " + auditLabel();
    }
}

class StripeGateway extends AbstractGateway {

    StripeGateway() {
        super("STRIPE");
    }

    @Override
    public String auditId() {
        return "STR-7781";
    }

    @Override
    public BigDecimal charge(BigDecimal amount) {
        System.out.println(describe());
        return amount.multiply(BigDecimal.valueOf(1.02));
    }
}

public class AbstractionDemo {

    public static void main(String[] args) {
        AbstractGateway gateway = new StripeGateway();
        System.out.println(gateway.charge(BigDecimal.valueOf(1000)));
    }
}
```

Output:

```text
STRIPE / AUDIT:STR-7781
1020.00
```

Important lines:
- `Auditable` defines a capability with a default method.
- `AbstractGateway` holds shared state (`gatewayCode`) and a concrete helper — impossible in an interface.
- `charge` remains abstract, forcing each gateway to provide its own payment logic.

**5. Real-world use case**

Spring Data repositories are interfaces: `OrderRepository extends JpaRepository<Order, Long>`. You declare the contract and Spring generates the implementation as a proxy at startup. Services depend on the interface, so tests can inject a mock repository without a database.

**6. Interview follow-up questions**

- **Can an interface have a constructor?** No. An abstract class can.
- **Can an abstract class be `final`?** No — the modifiers contradict each other.
- **Can a class implement two interfaces with the same default method?** Yes, but it must override the method to resolve the conflict.
- **Why are interface fields `public static final`?** Interfaces define contracts, not per-object state.
- **When would you convert an abstract class to an interface?** When the shared state disappears and only a behavior contract remains.

**7. Common mistakes**

- Saying interfaces cannot have method bodies — Java 8 default methods disprove that.
- Putting mutable state in interface constants.
- Choosing an abstract class just to share one method — an interface with a default method is often enough.
- Expecting a class to extend two abstract classes.

**8. Interview difficulty**

**Intermediate**

---

### Question 11: Method overloading vs method overriding — which one is which?

**1. Interview-ready answer**

Overloading happens in the same class: two or more methods share a name but have different parameter lists. The compiler chooses the right one from the call arguments, so it is compile-time polymorphism. Overriding happens between a superclass and a subclass: the subclass provides a new implementation of an inherited instance method with the same signature. The JVM chooses the implementation from the actual object type at runtime. The rules differ sharply — overloading only requires a different parameter list, while overriding requires the same parameters, a compatible return type, no narrower access, and no broader checked exceptions.

**2. Detailed explanation**

| Rule | Overloading | Overriding |
|---|---|---|
| Where | Same class (or inherited methods) | Subclass overrides superclass/interface |
| Parameters | Must differ | Must be identical |
| Return type | Can be anything | Same or covariant subtype |
| Access modifier | Any | Cannot reduce visibility |
| Checked exceptions | Any | Cannot add broader checked exceptions |
| Static methods | Can be overloaded | Cannot be overridden (hidden) |
| Binding | Compile time | Runtime |
| `final`/`private` methods | Can be overloaded | Cannot be overridden |

Why both: overloading gives readable APIs (`valueOf(int)`, `valueOf(String)`), overriding gives substitutability. Overloading ambiguity occurs when the compiler cannot pick a unique method — for example `pay(null)` with both `pay(String)` and `pay(Integer)` overloads is a compile error because neither is more specific.

**3. Internal working**

Overloaded methods have distinct JVM descriptors such as `(I)V` for `int` and `(Ljava/lang/Integer;)V` for `Integer`. The compiler emits a call to the exact descriptor. Overrides share the same descriptor; dispatch uses `invokevirtual`/`invokeinterface` and the receiver's vtable slot.

**4. Practical Java/Spring Boot example — output-based**

```java
public class OverloadResolutionDemo {

    public void pay(int amount) {
        System.out.println("int");
    }

    public void pay(Integer amount) {
        System.out.println("Integer");
    }

    public void pay(long amount) {
        System.out.println("long");
    }

    public void pay(Object amount) {
        System.out.println("Object");
    }

    public static void main(String[] args) {
        OverloadResolutionDemo demo = new OverloadResolutionDemo();
        demo.pay(100);
        demo.pay(100L);
        demo.pay(null);
    }
}
```

Output:

```text
int
long
Integer
```

Reasoning:
- `100` is an `int` literal → exact match with `pay(int)`; widening to `long` or boxing to `Integer` is considered only if no exact match exists.
- `100L` is a `long` literal → exact match with `pay(long)`.
- `null` is not applicable to primitive parameters, so only `pay(Integer)` and `pay(Object)` remain. `Integer` is a subtype of `Object`, so it is more specific and wins.

Overriding example:

```java
class BaseService {

    public Object load() {
        return "base";
    }
}

class OrderService extends BaseService {

    @Override
    public String load() {
        return "order";
    }
}
```

`String` is covariant with `Object`, so this override is legal.

**5. Real-world use case**

Spring MVC uses both. `ResponseEntity.ok()` is overloaded for different bodies, while `@RestController` handler methods override interface-style contracts in generated proxies. In JPA, `save()` is overloaded through `CrudRepository` generics, and entity `toString()` overrides `Object.toString()`.

**6. Interview follow-up questions**

- **What happens with `pay(null)` if both `pay(String)` and `pay(Integer)` exist?** Compile-time ambiguity error.
- **Can I override a method with a wider return type?** No — the return type must be the same or a subtype.
- **Can an overriding method throw a broader checked exception?** No. Unchecked exceptions are unrestricted.
- **Can I reduce visibility from `public` to `private` while overriding?** No.
- **Is `@Override` mandatory?** No, but use it — the compiler then verifies that you really overrode something.

**7. Common mistakes**

- Assuming the parameter type of the reference variable decides an override.
- Expecting a `static` method in a subclass to override the parent's static method.
- Forgetting that autoboxing and widening have different priority in overload resolution.
- Omitting `@Override` and accidentally adding a new method because of a typo in the parameter type.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: Why is the `main` method `static`? Can it be overloaded or overridden?

**1. Interview-ready answer**

The JVM invokes `main` before any object of the application class exists, so it must be a static method; otherwise the JVM would need to construct an object first and there would be no defined way to choose which object. Its signature must be exactly `public static void main(String[] args)` — `public` so the JVM can call it from outside the class, `static` so no instance is required, `void` because the JVM does not consume a return value, and `String[]` to receive command-line arguments. You can overload `main` with other parameter lists and call those overloads yourself, but only the exact signature is a valid entry point. You cannot override `main` because static methods are hidden, not overridden.

**2. Detailed explanation**

Why not a constructor-based entry point? A constructor can be overloaded, and the JVM cannot know which one to call. A fixed static signature is unambiguous.

Command-line arguments are useful in backend jobs: `java -jar report-service.jar --spring.profiles.active=prod` passes arguments to Spring Boot, which are merged with properties and environment variables.

Can `main` be `final`, `synchronized`, or `native`? Technically yes for `final` and `synchronized`, though pointless; the JVM only checks the required modifiers (`public static void` and the parameter type).

**3. Internal working**

The launcher looks up a method with the exact descriptor `([Ljava/lang/String;)V` and the required access flags. If it is missing, you get `Main method not found in class ...`. If the class has a static initializer, it runs before `main` is called.

**4. Practical Java/Spring Boot example**

```java
public class JobLauncher {

    public static void main(String[] args) {
        System.out.println("Production main, arguments: " + args.length);
        main(42);
        main("manual-override");
    }

    public static void main(int jobId) {
        System.out.println("Overloaded main with job id: " + jobId);
    }

    public static void main(String profile) {
        System.out.println("Overloaded main with profile: " + profile);
    }
}
```

Output:

```text
Production main, arguments: 0
Overloaded main with job id: 42
Overloaded main with profile: manual-override
```

Important lines:
- The `String[]` version is the real entry point.
- The other two are ordinary static methods that happen to be named `main`; the JVM ignores them.
- Only the `String[]` signature is recognized by the launcher.

**5. Real-world use case**

A batch job is launched with `java -jar reconciliation.jar --spring.profiles.active=prod --batch.size=500`. Spring Boot reads those arguments into the `Environment`, letting the same JAR run in dev, QA, and prod without code changes.

**6. Interview follow-up questions**

- **Can `main` be private?** No — the JVM cannot access it; launch fails.
- **Can `main` return `int`?** No — it must be `void`.
- **Can a class have more than one `main`?** Yes, through overloading, but only one with the exact entry-point signature.
- **Can a subclass "override" `main`?** No. Hiding is not overriding; the launcher uses the class you name.
- **What runs before `main`?** Class loading, linking, initialization of static fields and static blocks.

**7. Common mistakes**

- Writing `static void main(String args)` without `public` or with `String` instead of `String[]`.
- Believing overloading `main` creates multiple entry points.
- Forgetting that Spring Boot's `main` delegates to `SpringApplication.run(...)`, which then builds the application context.

**8. Interview difficulty**

**Beginner**

---

### Question 13: What is the difference between static and instance members?

**1. Interview-ready answer**

A static member belongs to the class and exists once per loaded class; an instance member belongs to an object and exists once per object. Static fields are shared across all instances, static methods can be called without creating an object, and static blocks run once during class initialization. Instance fields hold per-object state and instance methods can access both instance and static members. A static method cannot directly access an instance field because there is no `this` reference in a static context. In Spring, static members are used for constants and utility helpers, while injectable beans and their state are instance members — one bean instance per scope.

**2. Detailed explanation**

Static members:
- Declared with `static`.
- One copy per classloader.
- Accessed via `ClassName.member` (accessing through an object reference compiles but is misleading).
- Static initializer blocks run in declaration order, after superclass static initialization.
- `static final` constants are inlined at compile time if they are compile-time constants.

Instance members:
- One copy per object on the heap.
- Require an object reference to call methods.
- Initialized by field initializers and constructors.

Advantages of static: memory efficiency for shared values, convenient utilities. Disadvantages: global mutable state, harder testing, cannot participate in polymorphic dispatch.

**3. Internal working**

Static fields live with the class metadata in metaspace (with the values in the class's static storage). Instance fields live in the heap object. The JVM maintains one `Class<?>` object per loaded class per classloader, so two different classloaders can produce two separate copies of the same "static" field — important in application servers and plugin systems.

**4. Practical Java/Spring Boot example — output-based**

```java
public class BookingCounters {

    private static int totalSeatsBooked;
    private final int seatsOnThisTicket;

    BookingCounters(int seatsOnThisTicket) {
        this.seatsOnThisTicket = seatsOnThisTicket;
        totalSeatsBooked += seatsOnThisTicket;
    }

    static void printTotal() {
        System.out.println("Total seats: " + totalSeatsBooked);
    }

    int seatsOnThisTicket() {
        return seatsOnThisTicket;
    }

    public static void main(String[] args) {
        BookingCounters first = new BookingCounters(2);
        BookingCounters second = new BookingCounters(3);

        System.out.println(first.totalSeatsBooked);
        System.out.println(second.totalSeatsBooked);
        System.out.println(BookingCounters.totalSeatsBooked);
        BookingCounters.printTotal();
        System.out.println(first.seatsOnThisTicket() + " and " + second.seatsOnThisTicket());
    }
}
```

Output:

```text
5
5
5
Total seats: 5
2 and 3
```

Reasoning: `totalSeatsBooked` is shared, so every reference sees `5`. `seatsOnThisTicket` is per object, so the last line prints `2 and 3`. If `printTotal()` tried to read `seatsOnThisTicket` directly, the code would not compile.

**5. Real-world use case**

A Spring Boot service uses a `private static final Logger` (or a shared `MeterRegistry` injected as a bean) for cross-cutting concerns, while each request gets its own `Order` object and `OrderService` instance. A common production bug is using a mutable `static` field to cache request data — it leaks state between users and breaks under concurrency. Use request scope or a proper cache instead.

**6. Interview follow-up questions**

- **Can a static method access an instance method?** Only through an object reference, never directly.
- **Can static methods be overridden?** No, they are hidden and resolved by reference type.
- **Can a static block be inside a non-static inner class?** No — inner classes cannot declare static members before Java 16; static nested classes can.
- **Where are static variables stored?** With class metadata in metaspace; the reference values and objects they point to follow normal heap rules.
- **Why are utility classes made final with a private constructor?** To prevent instantiation and subclassing.

**7. Common mistakes**

- Accessing a static field through an instance (`booking.totalSeatsBooked`) — legal but misleading.
- Using mutable statics as caches without synchronization.
- Assuming static final objects are immutable — only the reference is final.
- Trying to call an instance method from `main` without creating an object.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: Explain the use of `this` and `super` keywords.

**1. Interview-ready answer**

`this` is a reference to the current object. It is used to disambiguate instance fields from parameters, to chain constructors with `this(...)`, to pass the current object to another method, and to return the object for fluent APIs. `super` is a reference to the superclass portion of the current object. It is used to call a superclass constructor with `super(...)`, to invoke an overridden superclass method with `super.method()`, and to access hidden superclass fields. Both constructor calls must be the first statement in a constructor, and a single constructor cannot contain both `this(...)` and `super(...)`.

**2. Detailed explanation**

Uses of `this`:
1. `this.field = field;` when names collide.
2. `this(...)` constructor chaining.
3. Passing the current object: `validator.validate(this)`.
4. Returning `this` for fluent builders.
5. In synchronized blocks: `synchronized (this)`.

Uses of `super`:
1. `super(...)` to invoke a superclass constructor (mandatory if the parent has no no-arg constructor).
2. `super.method()` to explicitly call the parent implementation after overriding.
3. `super.field` to access a hidden superclass field (rare and usually a design smell).

Neither can be used in a static context because there is no current object.

**3. Internal working**

In bytecode, `this` is local variable 0 of every instance method. `super(...)` compiles to a special `invokespecial` call to the parent's `<init>`. `super.method()` also uses `invokespecial`, which bypasses virtual dispatch — that is why calling `super.pay()` always runs the parent implementation, while `pay()` may dispatch to an override.

**4. Practical Java/Spring Boot example**

```java
class Employee {

    private final String name;
    private final double salary;

    Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String details() {
        return name + " earns " + salary;
    }
}

class Manager extends Employee {

    private final int teamSize;

    Manager(String name, double salary, int teamSize) {
        super(name, salary);
        this.teamSize = teamSize;
    }

    @Override
    public String details() {
        return super.details() + " and manages " + this.teamSize + " people";
    }

    public static void main(String[] args) {
        Manager manager = new Manager("Priya", 180000, 8);
        System.out.println(manager.details());
    }
}
```

Output:

```text
Priya earns 180000.0 and manages 8 people
```

Important lines:
- `this.name = name` resolves the field/parameter shadowing.
- `super(name, salary)` initializes the superclass portion first.
- `super.details()` reuses the parent implementation instead of recursing into the override.
- `this.teamSize` accesses the subclass field explicitly for clarity.

**5. Real-world use case**

Spring's `@Transactional` proxies subclass your service (or implement its interface). A self-call using `this.paymentService.charge()` bypasses the proxy, which is the classic self-invocation problem covered in Chapter 12. Understanding `this` versus a proxied reference is essential for debugging transactions and AspectJ advice that silently never runs.

**6. Interview follow-up questions**

- **Can `this` be used in a static method?** No.
- **Can `this()` and `super()` both appear in one constructor?** No; both must be first.
- **Can `super.method()` call a private parent method?** No.
- **What is the difference between `this` and a reference to the same object?** They are the same value; `this` is the implicit reference.
- **Can you assign to `this`?** No — `this` is effectively final.

**7. Common mistakes**

- Forgetting `this.field = field`, accidentally assigning the parameter to itself.
- Calling an overridable method from a constructor — it dispatches to the subclass before subclass fields are initialized.
- Using `super` to access private fields.
- Thinking `super` refers to a separate parent object; it is the same object's parent part.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: Explain access modifiers in Java.

**1. Interview-ready answer**

Java has four access levels. `private` means accessible only inside the declaring class. Package-private, which is the default when you write no modifier, means accessible inside the same package. `protected` means same package plus subclasses in other packages. `public` means accessible from anywhere. For top-level classes only two levels are allowed: `public` and package-private. On overriding, you cannot reduce visibility — a `public` method must stay `public`. In Spring, private field injection works because the framework uses reflection, not normal Java access.

**2. Detailed explanation**

| Modifier | Same class | Same package | Subclass (other package) | Anywhere |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| package-private (default) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

Top-level classes: `public` (file name must match class name) or package-private. Nested classes can use all four.

`protected` is often misunderstood: a subclass in another package can access a protected member only through a reference of its own type (or subtype), not through an arbitrary parent-typed reference.

Why access control exists: it enforces encapsulation and lets library authors change internals without breaking clients. Advantage: safer evolution of APIs. Disadvantage: overuse of `public` leaks internals; overuse of `private` can make testing harder.

**3. Internal working**

Access flags are stored in the class file for each class, field, and method. The JVM verifier checks them during linking, and the runtime checks them when resolving symbolic references. Reflection can bypass them with `setAccessible(true)` unless JPMS module rules or a security manager prevents it.

**4. Practical Java/Spring Boot example**

```java
package com.example.billing;

public class Invoice {

    private final String invoiceNumber;
    protected double taxRate;
    String batchId;
    public final String currency = "INR";

    public Invoice(String invoiceNumber) {
        this.invoiceNumber = invoiceNumber;
    }

    public String getInvoiceNumber() {
        return invoiceNumber;
    }

    private double roundedTax(double amount) {
        return Math.round(amount * taxRate * 100.0) / 100.0;
    }

    public double total(double amount) {
        return amount + roundedTax(amount);
    }
}
```

Important lines:
- `private final String invoiceNumber` is accessible only inside `Invoice`.
- `protected double taxRate` is available to subclasses, including those in other packages.
- `String batchId` has package-private access — usable by classes in `com.example.billing`.
- `public final String currency` is a constant visible everywhere.

**5. Real-world use case**

A payments SDK exposes `public` methods `authorize()` and `refund()`, keeps HTTP client details `private`, and marks a `protected` hook `buildHeaders()` so clients can extend it. This keeps the supported API small and prevents callers from depending on internals that may change.

**6. Interview follow-up questions**

- **Which modifier should fields use by default?** `private`, unless there is a specific reason.
- **Can a `public` method be overridden as package-private?** No — that reduces visibility.
- **Can a top-level class be `protected`?** No.
- **Why does `protected` include same-package access?** So classes in the original package can collaborate without subclassing.
- **Does `private` stop reflection?** No; JPMS modules and security policies are needed for stronger boundaries.

**7. Common mistakes**

- Confusing default access with `public`.
- Thinking `protected` is only for subclasses.
- Making everything `public` "to make testing easier".
- Forgetting that interface fields are implicitly `public static final`.

**8. Interview difficulty**

**Beginner**

---

### Question 16: Composition vs inheritance — which should you choose?

**1. Interview-ready answer**

Composition models a "has-a" relationship: a class contains instances of other classes and delegates to them. Inheritance models an "is-a" relationship: a class is a specialized version of another. The general rule is to favor composition over inheritance because it keeps classes loosely coupled, avoids the fragile base class problem, and lets behavior change at runtime through injection. Inheritance is appropriate when there is a genuine type hierarchy and subclasses truly substitute for the parent, such as different payment methods sharing a common contract. Spring itself is built around composition — beans receive their collaborators through constructor injection instead of extending framework classes.

**2. Detailed explanation**

Inheritance advantages: automatic reuse, natural polymorphism, easy to express a stable hierarchy. Disadvantages: tight coupling to parent internals, all subclasses inherit changes (good or bad), single inheritance limits flexibility, and testing can require complex parent setup.

Composition advantages: flexible runtime wiring, easier testing with mocks, no dependency on parent internals, can combine several behaviors. Disadvantages: more delegation code, and behavior discovery is less obvious than reading a superclass.

Decision rule: ask "is this really an is-a relationship that will stay true?" If not, use composition. Also watch the Liskov Substitution Principle — if a subclass cannot honor the parent's contract, inheritance is wrong.

**3. Internal working**

With composition, the containing object holds references on the heap and calls methods through them, which is still virtual dispatch when the field type is an interface. With inheritance, the subclass object contains the superclass fields inline and method resolution walks the class hierarchy. Composition makes dependency graphs explicit, which is exactly what a DI container like Spring manages.

**4. Practical Java/Spring Boot example**

```java
interface MessageClient {
    void send(String recipient, String text);
}

final class EmailClient implements MessageClient {

    @Override
    public void send(String recipient, String text) {
        System.out.println("Email to " + recipient + ": " + text);
    }
}

final class SmsClient implements MessageClient {

    @Override
    public void send(String recipient, String text) {
        System.out.println("SMS to " + recipient + ": " + text);
    }
}

final class NotificationService {

    private final MessageClient primaryClient;
    private final MessageClient fallbackClient;

    NotificationService(MessageClient primaryClient, MessageClient fallbackClient) {
        this.primaryClient = primaryClient;
        this.fallbackClient = fallbackClient;
    }

    void notifyCustomer(String recipient, String text) {
        try {
            primaryClient.send(recipient, text);
        } catch (RuntimeException failure) {
            fallbackClient.send(recipient, "Fallback: " + text);
        }
    }

    public static void main(String[] args) {
        NotificationService service = new NotificationService(new EmailClient(), new SmsClient());
        service.notifyCustomer("customer@example.com", "Your order shipped");
    }
}
```

Output:

```text
Email to customer@example.com: Your order shipped
```

Important lines:
- `NotificationService` has clients rather than extending `EmailClient`.
- Constructor injection makes dependencies visible and testable.
- Switching to SMS as primary requires only a different constructor argument.

**5. Real-world use case**

An order service composes a `PaymentGateway`, `InventoryClient`, `NotificationService`, and `AuditLogger`. In tests you inject fakes; in production Spring injects real beans. If the team had made `OrderService extends PaymentGateway`, swapping gateways would require a new subclass for every combination and would couple order logic to payment internals.

**6. Interview follow-up questions**

- **Is inheritance ever the right choice?** Yes, for stable is-a hierarchies such as `CardPayment extends Payment` when subclasses genuinely substitute for the parent.
- **What is the fragile base class problem?** A parent change unexpectedly breaks subclasses that relied on its implementation details.
- **How does Spring encourage composition?** Through constructor injection of interface-typed collaborators.
- **What is the decorator pattern?** Composition-based alternative to subclassing for adding behavior dynamically.
- **Can you mix both?** Yes — use inheritance for type identity and composition for varying behavior.

**7. Common mistakes**

- Extending a class only to reuse one method.
- Creating deep hierarchies (four or more levels) that are hard to reason about.
- Calling superclass constructors with side effects just to "get setup for free".
- Treating every helper relationship as "is-a" because inheritance feels convenient.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: What is the difference between association, aggregation, and composition?

**1. Interview-ready answer**

All three describe relationships between objects. Association is a general "uses-a/knows-about" link — an `Order` references a `Customer`. Aggregation is a weaker whole-part relationship where parts can exist independently — a `Department` has `Employee`s, but employees can exist without the department. Composition is a strong whole-part relationship with shared lifetime — an `Order` owns `OrderItem`s; deleting the order deletes its items. In code, composition is usually expressed with final fields and cascading deletion, aggregation with references injected from outside, and association with a plain reference or method parameter.

**2. Detailed explanation**

| Relationship | Meaning | Lifetime | Java expression |
|---|---|---|---|
| Association | Objects know about each other | Independent | Field or method parameter |
| Aggregation | Whole has parts, parts are independent | Independent | Field assigned from outside, no cascade delete |
| Composition | Whole owns parts, parts cannot live alone | Same | `final` field, created in constructor, cascade delete |

Why the distinction matters: it drives database design (foreign keys, cascade rules), JSON serialization (which side is serialized), and garbage collection (whether children are retained).

Advantages of modeling these explicitly: clearer domain model, correct cascade behavior, fewer orphan rows. Disadvantage: over-modeling every relationship adds complexity.

**3. Internal working**

In memory, all three are just references. The difference is in lifecycle ownership enforced by your code and the persistence layer. In JPA, composition maps to `@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)`, while aggregation maps to a plain `@ManyToOne` without cascading removal (JPA details in Chapter 14).

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

final class Customer {
    private final String customerId;

    Customer(String customerId) {
        this.customerId = customerId;
    }

    String id() {
        return customerId;
    }
}

final class OrderItem {
    private final String sku;
    private final int quantity;

    OrderItem(String sku, int quantity) {
        this.sku = sku;
        this.quantity = quantity;
    }

    String summary() {
        return sku + " x " + quantity;
    }
}

final class Order {
    private final String orderId;
    private final Customer customer;
    private final List<OrderItem> items = new ArrayList<>();

    Order(String orderId, Customer customer) {
        this.orderId = orderId;
        this.customer = customer;
    }

    void addItem(OrderItem item) {
        items.add(item);
    }

    void printOrder() {
        System.out.println(orderId + " for " + customer.id());
        items.forEach(item -> System.out.println("  " + item.summary()));
    }
}

final class Department {
    private final String name;
    private final List<Customer> members = new ArrayList<>();

    Department(String name) {
        this.name = name;
    }

    void addMember(Customer customer) {
        members.add(customer);
    }
}

public class RelationshipDemo {

    public static void main(String[] args) {
        Customer customer = new Customer("CUST-1");
        Order order = new Order("ORD-1", customer);
        order.addItem(new OrderItem("SKU-10", 2));
        order.printOrder();

        Department department = new Department("Premium Support");
        department.addMember(customer);
    }
}
```

Output:

```text
ORD-1 for CUST-1
  SKU-10 x 2
```

Important lines:
- `Order` holding `Customer` is association — both exist independently.
- `Order` creating and owning its `OrderItem` list is composition; items are meaningless without the order.
- `Department` keeping a list of pre-existing `Customer` objects is aggregation-like; removing the department does not destroy customers.

**5. Real-world use case**

In an airline booking system, `Booking` owns `PassengerSeatAssignment` rows (composition), references an existing `Flight` (association), and a `LoyaltyProgram` contains `Member` accounts (aggregation). When a booking is cancelled, seat assignments are deleted, but the flight and members survive.

**6. Interview follow-up questions**

- **Which relationship maps to `@ManyToOne`?** Association/aggregation from the many side to the one side.
- **Which one uses `orphanRemoval`?** Composition, when children should not outlive the parent.
- **Can aggregation be bidirectional?** Yes, but JPA requires `mappedBy` on the inverse side.
- **Are these relationships visible in memory?** Only as references; lifecycle rules are enforced by code and the database.
- **Which is strongest?** Composition.

**7. Common mistakes**

- Treating aggregation as composition and accidentally deleting shared entities.
- Exposing mutable child lists directly instead of through controlled methods.
- Forgetting that serialization frameworks may recurse through bidirectional relationships.
- Confusing inheritance ("is-a") with composition ("has-a").

**8. Interview difficulty**

**Intermediate**

---

### Question 18: Explain the SOLID principles with backend examples.

**1. Interview-ready answer**

SOLID is five design principles: Single Responsibility — a class has one reason to change; Open/Closed — open for extension, closed for modification; Liskov Substitution — subclasses must be usable wherever their parent is expected; Interface Segregation — clients should not depend on methods they do not use; Dependency Inversion — high-level modules depend on abstractions, not concrete classes. In a Spring backend these show up constantly: services should not also parse HTTP and write SQL (SRP), new payment methods should be new implementations rather than edits to a giant `if-else` (OCP), injected interfaces enable substitution (LSP/DIP), and small focused interfaces keep mocks simple (ISP).

**2. Detailed explanation**

- **S — Single Responsibility Principle (SRP):** `OrderService` should orchestrate orders; `OrderValidator` validates; `OrderMapper` maps DTOs. A class that validates, persists, sends email, and formats JSON changes for four different reasons.
- **O — Open/Closed Principle (OCP):** Add a `CouponRule` implementation instead of adding another `case` to a switch.
- **L — Liskov Substitution Principle (LSP):** If `Notification` promises `send()`, every implementation must honor it. A `SmsNotification` that throws for long messages while `EmailNotification` accepts them violates expectations if callers assume uniform behavior. A classic textbook violation is `Square extends Rectangle` where setting width changes height.
- **I — Interface Segregation Principle (ISP):** Split a fat `PaymentOperations` into `PaymentReader` and `PaymentRefunder` so a read-only reporting component does not implement refund methods.
- **D — Dependency Inversion Principle (DIP):** `OrderService` depends on `PaymentGateway` interface; Spring injects `StripeGateway`.

Advantages: easier changes, focused tests, parallel development. Disadvantages: more classes and interfaces, which can feel heavy for small projects; apply judgment, not dogma.

**3. Internal working**

DIP works at runtime through polymorphism and dependency injection: the container resolves the interface to a concrete bean. OCP is realized through polymorphic collections such as `List<Rule>` and Spring's plugin-like registration. The JVM's vtable dispatch is the mechanism that lets new implementations plug in without changing callers.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

interface PaymentGateway {
    String charge(BigDecimal amount);
}

interface PaymentRefunder {
    String refund(String transactionId, BigDecimal amount);
}

final class StripeGateway implements PaymentGateway, PaymentRefunder {

    @Override
    public String charge(BigDecimal amount) {
        return "STRIPE_CHARGE:" + amount.toPlainString();
    }

    @Override
    public String refund(String transactionId, BigDecimal amount) {
        return "STRIPE_REFUND:" + transactionId + ":" + amount.toPlainString();
    }
}

final class OrderService {

    private final PaymentGateway paymentGateway;

    OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public String placeOrder(BigDecimal amount) {
        return "ORDER_PLACED|" + paymentGateway.charge(amount);
    }
}

public class SolidDemo {

    public static void main(String[] args) {
        PaymentGateway gateway = new StripeGateway();
        OrderService orderService = new OrderService(gateway);
        System.out.println(orderService.placeOrder(BigDecimal.valueOf(4500)));
    }
}
```

Output:

```text
ORDER_PLACED|STRIPE_CHARGE:4500
```

Important lines:
- `OrderService` depends only on `PaymentGateway` (DIP) and does one job (SRP).
- Interfaces are split by capability (ISP).
- Adding `PayUGateway` requires no change to `OrderService` (OCP).

**5. Real-world use case**

A product company's checkout pipeline has validation, fraud checks, payment, inventory reservation, and notification as separate beans. Each can be tested independently and replaced without redeploying the rest. When the payment provider changes, only one adapter class changes — the rest of the service stays untouched.

**6. Interview follow-up questions**

- **Can you have too many interfaces?** Yes. Split only when different clients actually need different capabilities.
- **How does SRP relate to microservices?** A service should own one business capability; it is SRP applied at the service boundary.
- **What violates LSP in Java collections?** `Arrays.asList()` returns a fixed-size list whose `add()` throws — callers expecting normal `List` behavior are surprised.
- **Is SOLID mandatory?** No. It is a tool for managing change; simple code may not need all five.
- **How do you explain DIP to a junior?** "Depend on what something does, not on the class that does it."

**7. Common mistakes**

- Reciting acronyms without a concrete example.
- Creating an interface for every single class — unnecessary indirection.
- Assuming SRP means "one method per class".
- Ignoring LSP and creating subclasses that throw `UnsupportedOperationException` for inherited methods.

**8. Interview difficulty**

**Advanced**

---

### Question 19: Output-based master challenge — static initialization, constructor order, and dynamic dispatch

**1. Interview-ready answer**

The key is to separate three rules: static initializers run when the class is initialized, superclass first; instance field initializers and constructors run superclass part first; and method calls are dynamically dispatched to the runtime type while field access is resolved at compile time from the reference type. Combining these explains why a superclass constructor can call an overridden method that sees the subclass's fields before they have been initialized — and why a superclass-typed reference reads the superclass field.

**2. Detailed explanation**

Execution order for `new Child()`:

1. Load and initialize `Base` (static block), then `Child` (static block).
2. Allocate memory with all instance fields set to default values (`0`, `null`, `false`).
3. Run `Base` instance field initializers, then the `Base` constructor.
4. Run `Child` instance field initializers, then the `Child` constructor.
5. Return the reference.

During step 3, `Child`'s fields still hold defaults, even though the method executing is `Child`'s override. This is the dangerous part of calling overridable methods from constructors.

**3. Internal working**

Static initialization is triggered by first active use (`new`, accessing a static field/method, or reflection) and is guarded by a class initialization lock in the JVM. Instance initialization compiles into each `<init>` method as field initializers followed by constructor body. Field access uses the declared type at compile time; virtual method calls resolve through the runtime type.

**4. Practical Java example — predict the output**

```java
class Base {

    static {
        System.out.println("Base static");
    }

    int value = 10;

    Base() {
        System.out.println("Base constructor: " + getValue());
    }

    int getValue() {
        return value;
    }
}

class Child extends Base {

    static {
        System.out.println("Child static");
    }

    int value = 20;

    Child() {
        System.out.println("Child constructor: " + value);
    }

    @Override
    int getValue() {
        return value;
    }
}

public class OutputChallenge {

    public static void main(String[] args) {
        System.out.println("main start");
        Base reference = new Child();
        System.out.println(reference.value + " / " + reference.getValue());
    }
}
```

Output:

```text
main start
Base static
Child static
Base constructor: 0
Child constructor: 20
10 / 20
```

Line-by-line reasoning:
- `main start` prints first because `OutputChallenge` has no static block.
- `new Child()` triggers `Base` static initialization, then `Child` static initialization.
- Allocating memory sets both `value` fields to `0`.
- `Base`'s field initializer sets `Base.value = 10`; `Base`'s constructor calls `getValue()`.
- `getValue()` is virtual and dispatches to `Child.getValue()`, which reads `Child.value` — still `0` because `Child`'s initializer has not run yet.
- `Child`'s field initializer sets `Child.value = 20`; `Child`'s constructor prints `20`.
- `reference.value` is field access through a `Base` reference, so it reads `Base.value` = `10`.
- `reference.getValue()` is virtual, so it reads `Child.value` = `20`.

**5. Real-world use case**

A Spring service's superclass constructor calls an abstract method implemented by a subclass that injects a repository. The repository field is still `null` when the parent constructor runs, causing a `NullPointerException` only in production. The fix is to move initialization to `@PostConstruct` or after-properties-set callbacks.

**6. Interview follow-up questions**

- **Why is the first constructor output `0` and not `10`?** Because the override reads the subclass field, which is still at its default value.
- **Why does `reference.value` print `10`?** Fields are not polymorphic; access is bound to the declared type.
- **When do static blocks run relative to object creation?** Once, before the first instance is fully created.
- **How do you avoid the constructor trap?** Keep constructors simple and non-virtual; use an initialization callback.
- **What if `getValue()` were `private`?** Private methods are not overridden, so `Base` would print `10`.

**7. Common mistakes**

- Assuming fields are dynamically dispatched like methods.
- Believing subclass fields are initialized before the superclass constructor.
- Calling overridable methods from constructors.
- Forgetting that static initializers of the parent run before the child's.

**8. Interview difficulty**

**Advanced**

---

### Question 20: Coding challenge — design an immutable airline booking model using abstraction, composition, and polymorphism

**1. Interview-ready answer**

I would model `Seat` as an interface with `fare()`, `seatNumber()`, and `baggageAllowanceKg()`, then provide `EconomySeat` and `BusinessSeat` implementations. `BusinessSeat` can own a `LoungeAccess` object, which is composition. A `FlightBooking` holds a `Seat` reference rather than a concrete class, so fare calculation is polymorphic, and I would make all domain objects immutable with validation in constructors. A `BookingService` creates bookings and keeps the creation logic in one place. This design is open for new seat types without modifying existing code.

**2. Detailed explanation**

**Problem statement:** Build a small domain model for flight bookings where economy and business seats have different fares and baggage allowances, a booking covers one or more passengers on one seat, and total fare is computed polymorphically. The model must be immutable and reject invalid input.

**Approach:**
1. Define a `Seat` interface for the contract (abstraction).
2. Implement concrete seat types with final fields (encapsulation + immutability).
3. Use composition for `LoungeAccess` inside `BusinessSeat`.
4. Make `FlightBooking` hold the interface type and compute the total through polymorphism.
5. Validate all inputs in constructors and return defensive values.

**3. Internal working**

No shared mutable state, so no synchronization is needed. `BigDecimal` multiplication creates a new object, preserving immutability. Polymorphic dispatch resolves `seat.fare()` to the concrete implementation at runtime.

**4. Practical Java solution**

```java
import java.math.BigDecimal;
import java.util.Objects;

interface Seat {

    String seatNumber();

    BigDecimal fare();

    int baggageAllowanceKg();
}

final class LoungeAccess {

    private final String partnerLounge;

    LoungeAccess(String partnerLounge) {
        this.partnerLounge = Objects.requireNonNull(partnerLounge, "partnerLounge is required");
    }

    String partner() {
        return partnerLounge;
    }
}

final class EconomySeat implements Seat {

    private final String seatNumber;
    private final BigDecimal fare;

    EconomySeat(String seatNumber, BigDecimal fare) {
        if (seatNumber == null || seatNumber.isBlank()) {
            throw new IllegalArgumentException("seatNumber is required");
        }
        if (fare == null || fare.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("fare must be positive");
        }
        this.seatNumber = seatNumber;
        this.fare = fare;
    }

    @Override
    public String seatNumber() {
        return seatNumber;
    }

    @Override
    public BigDecimal fare() {
        return fare;
    }

    @Override
    public int baggageAllowanceKg() {
        return 15;
    }
}

final class BusinessSeat implements Seat {

    private final String seatNumber;
    private final BigDecimal fare;
    private final LoungeAccess loungeAccess;

    BusinessSeat(String seatNumber, BigDecimal fare, LoungeAccess loungeAccess) {
        if (seatNumber == null || seatNumber.isBlank()) {
            throw new IllegalArgumentException("seatNumber is required");
        }
        if (fare == null || fare.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("fare must be positive");
        }
        this.seatNumber = seatNumber;
        this.fare = fare;
        this.loungeAccess = Objects.requireNonNull(loungeAccess, "loungeAccess is required");
    }

    @Override
    public String seatNumber() {
        return seatNumber;
    }

    @Override
    public BigDecimal fare() {
        return fare;
    }

    @Override
    public int baggageAllowanceKg() {
        return 40;
    }

    String loungePartner() {
        return loungeAccess.partner();
    }
}

final class FlightBooking {

    private final String bookingReference;
    private final Seat seat;
    private final int passengers;

    FlightBooking(String bookingReference, Seat seat, int passengers) {
        if (bookingReference == null || bookingReference.isBlank()) {
            throw new IllegalArgumentException("bookingReference is required");
        }
        this.seat = Objects.requireNonNull(seat, "seat is required");
        if (passengers <= 0) {
            throw new IllegalArgumentException("passengers must be positive");
        }
        this.bookingReference = bookingReference;
        this.passengers = passengers;
    }

    BigDecimal totalFare() {
        return seat.fare().multiply(BigDecimal.valueOf(passengers));
    }

    String summary() {
        return bookingReference + " | " + seat.seatNumber()
                + " | pax=" + passengers
                + " | total=" + totalFare().toPlainString()
                + " | baggage=" + seat.baggageAllowanceKg() + "kg";
    }
}

final class BookingService {

    FlightBooking createBooking(String reference, Seat seat, int passengers) {
        return new FlightBooking(reference, seat, passengers);
    }
}

public class BookingModelDemo {

    public static void main(String[] args) {
        BookingService bookingService = new BookingService();

        Seat economy = new EconomySeat("12A", BigDecimal.valueOf(5000));
        Seat business = new BusinessSeat("1A", BigDecimal.valueOf(12000), new LoungeAccess("Lounge-One"));

        FlightBooking first = bookingService.createBooking("BK-1", economy, 2);
        FlightBooking second = bookingService.createBooking("BK-2", business, 1);

        System.out.println(first.summary());
        System.out.println(second.summary());
    }
}
```

Output:

```text
BK-1 | 12A | pax=2 | total=10000 | baggage=15kg
BK-2 | 1A | pax=1 | total=12000 | baggage=40kg
```

**5. Dry run**

| Step | Action | State |
|---|---|---|
| 1 | Create `EconomySeat("12A", 5000)` | fare = 5000, baggage = 15 |
| 2 | Create `BusinessSeat("1A", 12000, lounge)` | fare = 12000, baggage = 40 |
| 3 | Create booking `BK-1` with 2 passengers | seat reference → economy |
| 4 | `totalFare()` on `BK-1` | `5000 × 2 = 10000` |
| 5 | Create booking `BK-2` with 1 passenger | seat reference → business |
| 6 | `totalFare()` on `BK-2` | `12000 × 1 = 12000` |

**6. Complexity**

- Booking creation: **O(1)** time, **O(1)** space.
- Fare calculation: **O(1)** time, **O(1)** extra space (one `BigDecimal` result).
- Adding a new seat type: no changes to `FlightBooking` or `BookingService`.

**7. Edge cases**

- `seatNumber` blank or `null` → `IllegalArgumentException`.
- `fare` zero, negative, or `null` → `IllegalArgumentException`.
- `passengers` zero or negative → `IllegalArgumentException`.
- `loungeAccess` null → `NullPointerException` via `requireNonNull`.
- Very large passenger counts → `BigDecimal` avoids overflow but business limits should be enforced by validation.

**8. Real-world use case**

This model maps directly to a Spring Boot booking API: `EconomySeat` and `BusinessSeat` are domain objects, `BookingService` is a `@Service`, and a REST controller converts DTOs into seats through a factory. Adding a premium economy cabin requires one new class implementing `Seat`.

**9. Interview follow-up questions**

- **Why not use `double` for fare?** Floating-point rounding errors; use `BigDecimal` or integer minor units.
- **Why is `Seat` an interface rather than an abstract class?** Seat types share no state or default behavior; an interface keeps them decoupled.
- **How would you make this thread-safe?** It already is — all fields are final and objects are immutable.
- **Where would you put a discount rule?** As another polymorphic component composed into `FlightBooking` or applied by a pricing service.
- **How would Hibernate persist this?** Using `@Embeddable`/inheritance strategies or separate tables with a discriminator — see Chapter 14.

**10. Common mistakes**

- Using mutable setters on domain objects and losing immutability.
- Hard-coding `if (seat instanceof BusinessSeat)` — that defeats polymorphism.
- Using `float`/`double` for money.
- Adding business logic to the controller instead of the domain or service layer.
- Exposing internal collections directly.

**11. Interview difficulty**

**Intermediate**

---

## Chapter 1 Verification

| Syllabus topic | Question(s) |
|---|---|
| JVM, JDK, JRE | Q1 |
| Java compilation and execution lifecycle | Q2 |
| Bytecode and platform independence | Q3 |
| Classes and objects | Q4 |
| Constructors | Q5 |
| OOP principles | Q6 |
| Encapsulation | Q7 |
| Inheritance | Q8 |
| Polymorphism | Q9 |
| Abstraction | Q10 |
| Method overloading and overriding | Q11 |
| Interfaces vs abstract classes | Q10 |
| Static and instance members | Q12, Q13 |
| `this` and `super` | Q14 |
| Access modifiers | Q15 |
| Composition vs inheritance | Q16 |
| Association, aggregation, composition | Q17 |
| SOLID principles | Q18 |
| Coding and output-based questions | Q11, Q13, Q19, Q20 |

**Total interview questions: 20** (all with the required 8-part format; coding questions include problem statement, approach, dry run, complexity, and edge cases).

**Next:** [Chapter 2 — Strings and Object Handling](02-Strings-and-Object-Handling.md). Say `NEXT CHAPTER` to continue.
