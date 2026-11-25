# Chapter 3: Exception Handling

**Chapter goal:** Answer exception-handling questions with the precision interviewers expect — hierarchy, checked vs unchecked, `finally` semantics, resource safety, exception chaining, Spring REST error handling, transaction rollback interaction, and production logging.

**Topics covered:** Exception hierarchy · checked vs unchecked · `Error` vs `Exception` · `try`/`catch`/`finally` · `throw` vs `throws` · custom exceptions · multiple and multi-catch blocks · try-with-resources · exception propagation · exception chaining and wrapping · Spring `@ControllerAdvice` · logging · Spring transaction rollback · best practices · output-based and coding questions.

**Cross-references:** `final`/`finally`/`finalize` basics are in [Chapter 2, Q10](02-Strings-and-Object-Handling.md). Transaction rollback rules are expanded in [Chapter 12](../04-Spring/12-Transactions-and-Scheduling.md). REST status codes are covered in [Chapter 11](../04-Spring/11-REST-API-Development.md).

---

### Question 1: Explain the Java exception hierarchy.

**1. Interview-ready answer**

All throwable objects inherit from `java.lang.Throwable`. It has two main branches: `Exception` and `Error`. `Exception` represents recoverable conditions — either checked exceptions such as `IOException` and `SQLException`, which the compiler forces you to handle, or unchecked exceptions such as `NullPointerException` and `IllegalArgumentException`, which extend `RuntimeException`. `Error` represents serious JVM-level problems such as `OutOfMemoryError` and `StackOverflowError` that applications normally should not catch. At the top of the RuntimeException branch sit programming errors and invalid states; at the checked branch sit external conditions a caller can reasonably recover from.

**2. Detailed explanation**

```text
Throwable
├── Exception
│   ├── RuntimeException (unchecked)
│   │   ├── NullPointerException
│   │   ├── IllegalArgumentException
│   │   ├── IllegalStateException
│   │   ├── ArithmeticException
│   │   └── ClassCastException
│   ├── IOException
│   ├── SQLException
│   └── ClassNotFoundException
└── Error
    ├── VirtualMachineError
    │   ├── OutOfMemoryError
    │   └── StackOverflowError
    ├── LinkageError
    │   ├── NoClassDefFoundError
    │   └── ExceptionInInitializerError
    └── AssertionError
```

Why the split exists: the compiler can enforce handling for recoverable external failures, but it cannot force you to handle every possible null pointer or divide-by-zero — that would make code unreadable. `Error` is separate because recovery is usually impossible.

Advantages: clear contract about what callers must handle, structured propagation with causes. Disadvantages: checked exceptions add boilerplate and can be overused, leading to `throws Exception` everywhere.

**3. Internal working**

Exceptions are ordinary Java objects allocated on the heap. When `throw` executes, the JVM captures the current stack trace by filling in a `StackTraceElement[]` — a relatively expensive operation. Control transfers to the nearest matching `catch` frame as the stack unwinds. Exception tables in each method's bytecode map instruction ranges to handler start PCs and exception types; the JVM walks these tables while unwinding.

**4. Practical Java/Spring Boot example**

```java
import java.io.IOException;
import java.sql.SQLException;

public class ExceptionHierarchyDemo {

    public static void main(String[] args) {
        Throwable[] examples = {
                new IllegalArgumentException("unchecked"),
                new IOException("checked"),
                new SQLException("checked"),
                new StackOverflowError("error")
        };

        for (Throwable example : examples) {
            System.out.println(example.getClass().getSimpleName()
                    + " -> Exception? " + (example instanceof Exception)
                    + " | RuntimeException? " + (example instanceof RuntimeException)
                    + " | Error? " + (example instanceof Error));
        }
    }
}
```

Output:

```text
IllegalArgumentException -> Exception? true | RuntimeException? true | Error? false
IOException -> Exception? true | RuntimeException? false | Error? false
SQLException -> Exception? true | RuntimeException? false | Error? false
StackOverflowError -> Exception? false | RuntimeException? false | Error? true
```

Important lines:
- `RuntimeException` is a subclass of `Exception`, so the first check is true for it.
- `Error` is a sibling of `Exception` under `Throwable`.

**5. Real-world use case**

A payment service defines `PaymentDeclinedException extends RuntimeException` for business failures and lets `IOException` from an HTTP client be caught and wrapped at the adapter boundary. The global REST handler maps each type to a distinct HTTP status. It never catches `Error`, because an `OutOfMemoryError` should trigger a restart and an alert rather than be silently converted to a 500 response.

**6. Interview follow-up questions**

- **Is `RuntimeException` a checked exception?** No — it is unchecked despite extending `Exception`.
- **Can you catch `Throwable`?** Yes, but only at top-level boundaries (e.g., thread `run()` or framework filter), never in normal business code.
- **Which exceptions does the compiler check?** `Exception` subclasses excluding `RuntimeException` and its subclasses.
- **Is `NullPointerException` an Error?** No — it is an unchecked RuntimeException.
- **Why is `ClassNotFoundException` checked but `NoClassDefFoundError` an Error?** The former is a recoverable lookup failure; the latter means a required class was available at compile time but missing at runtime — a linkage failure.

**7. Common mistakes**

- Saying all `Exception` subclasses are checked.
- Catching `Error` in business code and pretending the service is healthy.
- Throwing `Throwable` or generic `Exception` from APIs.
- Assuming stack traces are free — they are expensive under high error rates.

**8. Interview difficulty**

**Beginner**

---

### Question 2: Checked vs unchecked exceptions — how do you decide which to use?

**1. Interview-ready answer**

Checked exceptions extend `Exception` but not `RuntimeException`; the compiler requires you to catch or declare them. They represent recoverable conditions outside the caller's control, such as a file not found or a network timeout. Unchecked exceptions extend `RuntimeException`; they usually indicate programming errors or invalid state — null arguments, illegal arguments, bad configuration — and the compiler does not force handling. My rule: use unchecked exceptions for domain and programming errors that callers should fix, and checked exceptions only when the caller has a meaningful recovery action. In Spring, most service exceptions are unchecked so `@Transactional` rolls back automatically and controllers can map them centrally.

**2. Detailed explanation**

| Aspect | Checked | Unchecked |
|---|---|---|
| Base class | `Exception` (excluding RuntimeException) | `RuntimeException` |
| Compiler enforcement | Must catch or declare | None |
| Typical cause | External, recoverable | Programming/state error |
| Examples | `IOException`, `SQLException` | `NPE`, `IllegalArgumentException` |
| Spring rollback default | No rollback | Rollback |
| API impact | `throws` clauses propagate to callers | Cleaner signatures |

Why checked exceptions exist: to make failure modes explicit in method signatures. Why they are controversial: they spread boilerplate, encourage swallowed exceptions, and break lambda compatibility (Chapter 5).

**3. Internal working**

The compiler verifies checked exceptions by analyzing the `throws` clauses of called methods and the types caught in enclosing `try` statements. This is purely compile-time — bytecode has no checked/unchecked distinction. The JVM treats all throwables identically at runtime.

**4. Practical Java/Spring Boot example**

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class CheckedVsUnchecked {

    public static String readConfig(Path path) throws IOException {
        if (path == null) {
            throw new IllegalArgumentException("path must not be null");
        }
        return Files.readString(path);
    }

    public static void main(String[] args) {
        try {
            readConfig(null);
        } catch (IllegalArgumentException exception) {
            System.out.println("Programming error: " + exception.getMessage());
        }

        try {
            readConfig(Path.of("missing-config.txt"));
        } catch (IOException exception) {
            System.out.println("Recoverable error: " + exception.getClass().getSimpleName());
        }
    }
}
```

Output:

```text
Programming error: path must not be null
Recoverable error: NoSuchFileException
```

Important lines:
- `IllegalArgumentException` is unchecked — used for a caller bug (null path).
- `IOException` is checked — declared with `throws` and handled by the caller.
- `Files.readString` is Java 11+.

**5. Real-world use case**

A file-upload service treats `NoSuchFileException` as checked because it can fall back to a default template. It treats a negative file size as `IllegalArgumentException` because that indicates a bug in the caller. At the REST boundary, Spring maps both to appropriate responses without leaking stack traces to clients.

**6. Interview follow-up questions**

- **Can a checked exception be thrown from a lambda?** Not directly in `Function`/`Consumer` — you must wrap it or use a functional interface that declares `throws`.
- **Do checked exceptions affect Spring rollback?** Yes — by default `@Transactional` does not roll back on checked exceptions; use `rollbackFor`.
- **Is it bad to declare `throws Exception`?** Yes — it hides the real failure modes and forces callers to handle everything.
- **Can you catch an unchecked exception?** Yes, when you can genuinely recover or translate it.
- **Why do many frameworks prefer unchecked exceptions?** They keep APIs clean and work naturally with proxies and lambdas.

**7. Common mistakes**

- Using checked exceptions for validation errors.
- Swallowing checked exceptions with empty catch blocks just to satisfy the compiler.
- Declaring `throws Exception` on every method.
- Assuming unchecked exceptions are less serious — they often crash requests just as hard.

**8. Interview difficulty**

**Intermediate**

---

### Question 3: What is the difference between `Error` and `Exception`?

**1. Interview-ready answer**

Both extend `Throwable`, but `Exception` represents conditions an application may reasonably handle, while `Error` represents serious problems in the JVM or runtime environment that applications normally cannot recover from. Examples of `Error` are `OutOfMemoryError`, `StackOverflowError`, `NoClassDefFoundError`, and `ExceptionInInitializerError`. When an `Error` occurs, the usual response is to let it propagate, log it at the outermost boundary if possible, and allow the process to fail and be restarted by an orchestrator. Catching `OutOfMemoryError` and continuing usually leaves the JVM in an unreliable state.

**2. Detailed explanation**

Key differences:
- **Origin:** Exceptions usually arise from application or library code; Errors from the JVM (resource exhaustion, linkage failures, assertion failures).
- **Recoverability:** Exceptions are often recoverable; Errors generally are not.
- **Compiler treatment:** Both unchecked in effect — the compiler never forces you to handle `Error`.
- **Recommended handling:** Catch specific exceptions; do not catch Errors in business logic.

Subtleties: `AssertionError` is an Error but is deliberately thrown by `assert` statements. `VirtualMachineError` includes internal errors and resource exhaustion. `NoClassDefFoundError` is a `LinkageError`, often caused by a missing dependency or a failed static initializer.

**3. Internal working**

`OutOfMemoryError` is thrown when the JVM cannot allocate an object and the GC cannot free enough space. `StackOverflowError` occurs when a thread's call stack exceeds its limit — typically infinite recursion. `ExceptionInInitializerError` wraps an exception thrown from a static initializer, and later uses of that class produce `NoClassDefFoundError`.

**4. Practical Java/Spring Boot example**

```java
public class ErrorDemo {

    private static int depth = 0;

    private static void recurse() {
        depth++;
        recurse();
    }

    public static void main(String[] args) {
        try {
            recurse();
        } catch (StackOverflowError error) {
            System.out.println("Stack overflow after " + depth + " calls");
            System.out.println("Error is Exception? " + (error instanceof Exception));
        }
    }
}
```

Typical output:

```text
Stack overflow after <several thousand> calls
Error is Exception? false
```

Important lines:
- `StackOverflowError` is caught only to demonstrate the hierarchy; production code should rarely catch it.
- The exact depth depends on stack size and JVM settings.

**5. Real-world use case**

A Spring Boot service started returning `NoClassDefFoundError` after deployment. Investigation showed that a static initializer in a library threw during startup, so the class was marked erroneous. The fix was correcting a missing configuration required by that initializer — not adding a catch block. Another service had repeated `OutOfMemoryError`s; the correct response was a heap dump, restart, and a memory-leak fix (Chapter 6).

**6. Interview follow-up questions**

- **Should you catch `OutOfMemoryError` to clean up?** Generally no; attempt a heap dump and let the process die.
- **What causes `NoClassDefFoundError`?** Missing runtime dependency, failed static initialization, or classloader problems.
- **Is `AssertionError` checked?** No — all Errors are unchecked.
- **CanSpring translate an Error into an HTTP response?** It can via broad handlers, but it is usually better to let the process fail.
- **What is `ExceptionInInitializerError`?** A wrapper for exceptions thrown during static initialization.

**7. Common mistakes**

- Catching `Throwable` in business methods.
- Treating `OutOfMemoryError` as a normal request failure.
- Assuming `Error` means only memory problems.
- Ignoring the root cause wrapped inside `ExceptionInInitializerError`.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 4: Explain the execution flow of `try`, `catch`, and `finally`.

**1. Interview-ready answer**

The `try` block contains code that may throw. When an exception occurs, normal execution stops and the JVM searches for the first `catch` block whose parameter type matches the thrown object, walking from more specific to more general. If a matching catch is found, it runs; then the `finally` block always runs, whether an exception occurred or not. If no catch matches, `finally` still runs before the exception propagates up the call stack. `finally` exists for cleanup that must happen on every path, though try-with-resources is usually a better choice for `AutoCloseable` resources.

**2. Detailed explanation**

Possible paths:
1. No exception → `try` completes → `finally` runs → continue after the block.
2. Exception caught → matching `catch` runs → `finally` runs → continue after the block.
3. Exception not caught → `finally` runs → exception propagates to caller.
4. `return` in `try` or `catch` → `finally` runs before the method returns.
5. `break`/`continue` in a loop → `finally` runs before control transfers.

Why it exists: to guarantee cleanup without duplicating code in every branch. Disadvantage: `finally` can hide exceptions if it throws or returns.

**3. Internal working**

The compiler emits an exception table for each `try` region. Each entry has start/end PCs, a handler PC, and a caught type. The JVM walks this table during unwinding. The `finally` block is duplicated on all normal and exceptional paths at bytecode level (or handled via a single copy with exception tables in newer compilers).

**4. Practical Java example — output-based**

```java
public class TryCatchFinallyFlow {

    static String process() {
        try {
            System.out.println("try");
            throw new IllegalArgumentException("bad input");
        } catch (IllegalArgumentException exception) {
            System.out.println("catch: " + exception.getMessage());
            return "returned from catch";
        } finally {
            System.out.println("finally");
        }
    }

    public static void main(String[] args) {
        System.out.println(process());
        System.out.println("after process");
    }
}
```

Output:

```text
try
catch: bad input
finally
returned from catch
after process
```

Important lines:
- `finally` runs after the catch body completes but before the return value is delivered.
- The method returns the value computed in `catch`, not something from `finally`.

**5. Real-world use case**

A legacy JDBC method closed connections in `finally`. It worked but produced noisy code and occasional bugs when `close()` itself threw. Refactoring to try-with-resources made the code shorter and gave correct suppressed-exception behavior automatically.

**6. Interview follow-up questions**

- **Does `finally` run if there is no exception?** Yes.
- **Does `finally` run if `catch` rethrows?** Yes, before propagation.
- **Can you have `try` with only `finally` and no catch?** Yes — `try { ... } finally { ... }`.
- **What if `finally` throws a new exception?** It replaces the original exception; the original is lost unless handled manually.
- **What if a `return` appears in `finally`?** It overrides any previous return and swallows in-flight exceptions.

**7. Common mistakes**

- Putting business logic in `finally`.
- Throwing from `finally` and losing the real failure.
- Assuming `finally` runs on `System.exit()` — it does not.
- Catching `Exception` before a more specific subtype.

**8. Interview difficulty**

**Beginner**

---

### Question 5: When does `finally` NOT run, and what happens with `return` inside it?

**1. Interview-ready answer**

`finally` does not run if the JVM exits during the `try` or `catch` block — for example, `System.exit()`, `Runtime.halt()`, an OS-level kill, a JVM crash, or an infinite loop that never leaves the block. It also does not run if the thread is killed before reaching it, though `Thread.stop()` is deprecated. A `return` inside `finally` is dangerous: it overrides a return from `try`/`catch` and silently swallows any exception in flight. For these reasons, `finally` should only contain cleanup, and `AutoCloseable` resources should use try-with-resources.

**2. Detailed explanation**

Scenarios:
- `System.exit(1)` → shutdown hooks run, but `finally` does not.
- `OutOfMemoryError` → may prevent the JVM from executing cleanup reliably.
- Power loss or `kill -9` → nothing runs.
- Infinite loop in `try` → `finally` is never reached.
- `return` in `finally` → last word wins.

Why interviewers ask: candidates often claim "finally always runs" and then fail on follow-ups. The precise answer earns credibility.

**3. Internal working**

On `System.exit`, the JVM initiates an orderly shutdown: shutdown hooks run, but the current thread's stack is not unwound, so `finally` blocks are skipped. A `return` in `finally` compiles to an override of the return address slot used by earlier returns.

**4. Practical Java example — output-based**

```java
public class FinallyPuzzle {

    static int returnFromTry() {
        try {
            return 1;
        } finally {
            return 2;
        }
    }

    static int exceptionWithCleanup() {
        try {
            throw new IllegalStateException("failure");
        } finally {
            System.out.println("cleanup runs");
        }
    }

    static int exceptionSwallowedByFinally() {
        try {
            throw new IllegalStateException("original failure");
        } finally {
            return 99;
        }
    }

    public static void main(String[] args) {
        System.out.println(returnFromTry());

        try {
            System.out.println(exceptionWithCleanup());
        } catch (IllegalStateException exception) {
            System.out.println("caught: " + exception.getMessage());
        }

        System.out.println(exceptionSwallowedByFinally());
    }
}
```

Output:

```text
2
cleanup runs
caught: failure
99
```

Reasoning: the first method returns 2 because `finally` wins; the second propagates its exception after cleanup; the third swallows the exception entirely.

**5. Real-world use case**

A job runner had `return` inside a `finally` block that released a lock. When the job threw, the exception disappeared and the orchestrator marked the job successful — data was silently skipped. Removing the `return` and using try-with-resources restored correct failure reporting.

**6. Interview follow-up questions**

- **Do shutdown hooks run on `System.exit()`?** Yes, generally.
- **Does `finally` run on `kill -9`?** No.
- **Can `finally` hide an exception?** Yes, by throwing or returning.
- **Is `finally` the right place to close a stream?** No — use try-with-resources.
- **What is the alternative for guaranteed process-level cleanup?** Shutdown hooks plus external health checks and orchestration.

**7. Common mistakes**

- Saying "finally always executes" without exceptions to the rule.
- Returning from `finally`.
- Logging and swallowing errors in `finally`.
- Relying on `finally` for transactional commit semantics.

**8. Interview difficulty**

**Intermediate**

---

### Question 6: What is the difference between `throw` and `throws`?

**1. Interview-ready answer**

`throw` is a statement used to actually create and throw an exception object — there can be only one throwable, and execution stops immediately at that point. `throws` is a method declaration clause listing exception types the method may pass to its caller; it is a compile-time contract, not an action. You use `throw` inside a method body and `throws` in the method signature. A method that declares `throws IOException` does not necessarily throw one — it merely warns callers, who must catch it or declare it too if it is checked.

**2. Detailed explanation**

`throw` rules:
- Followed by an expression of type `Throwable` or a subclass.
- Only one exception is thrown.
- Control transfers immediately; subsequent statements are unreachable.
- Can throw checked exceptions only if declared or caught.

`throws` rules:
- Used in method/constructor declarations.
- Can list multiple types separated by commas.
- Required for checked exceptions that are not handled in the method.
- Optional for unchecked exceptions and Errors, though documenting them with `@throws` Javadoc is useful.

Why both: `throw` is the mechanism, `throws` is the contract.

**3. Internal working**

`throw` compiles to the `athrow` bytecode instruction, which unwinds the stack. `throws` is metadata in the method's class-file attributes; the compiler uses it for checked-exception verification. The JVM ignores it at runtime.

**4. Practical Java/Spring Boot example**

```java
import java.io.IOException;

public class ThrowVsThrows {

    static void validateAmount(int amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("amount must be positive");
        }
    }

    static String readRecord(boolean available) throws IOException {
        if (!available) {
            throw new IOException("record not available");
        }
        return "record";
    }

    public static void main(String[] args) {
        validateAmount(10);

        try {
            System.out.println(readRecord(false));
        } catch (IOException exception) {
            System.out.println("handled: " + exception.getMessage());
        }
    }
}
```

Output:

```text
handled: record not available
```

Important lines:
- `throw new IllegalArgumentException(...)` is an unchecked exception — no `throws` needed.
- `readRecord` declares `throws IOException` because it throws a checked exception.
- The caller must handle the checked exception.

**5. Real-world use case**

A Spring service throws a custom unchecked `OrderNotFoundException` from the repository adapter; the method signature stays clean. A file-based report exporter declares `throws IOException` because callers genuinely need to handle storage failures — retry, fallback, or user notification.

**6. Interview follow-up questions**

- **Can `throw` be used without `throws`?** Yes for unchecked exceptions.
- **Can a method declare `throws` but never throw?** Yes — legal, though usually pointless.
- **Can you throw a checked exception from a lambda?** Not with standard `Function`/`Consumer`; wrap it or define a throwing functional interface.
- **Can a constructor declare `throws`?** Yes.
- **Can you throw `null`?** `throw null` compiles but causes NPE at runtime.

**7. Common mistakes**

- Writing `throws new Exception()` — invalid syntax.
- Declaring `throws Exception` instead of specific types.
- Throwing checked exceptions from interfaces where implementations cannot reasonably recover.
- Thinking `throws` actually throws something.

**8. Interview difficulty**

**Beginner**

---

### Question 7: How do you design a good custom exception class?

**1. Interview-ready answer**

A good custom exception has a meaningful name ending in `Exception`, extends the most appropriate base — usually `RuntimeException` for domain errors in Spring — provides constructors for message, cause, and message-plus-cause, and carries structured fields such as an error code or order ID when callers need them. It should preserve the root cause via exception chaining and avoid exposing sensitive data. I also document with `@throws` where useful and map the exception to an HTTP status centrally with `@ExceptionHandler`. I avoid creating a new exception class for every trivial case; a small, coherent hierarchy is easier to maintain.

**2. Detailed explanation**

Design guidelines:
- Name: `PaymentDeclinedException`, not `MyException` or `Problem`.
- Base: `RuntimeException` for most backend domain errors; `Exception` only if callers must handle it.
- Constructors: `(String message)`, `(String message, Throwable cause)`, and optionally domain fields.
- Preserve cause: never discard the underlying exception.
- Error codes: useful for stable client contracts; HTTP status mapping belongs in the web layer.
- Serialization: if sent over RMI or stored, consider `serialVersionUID`.

Advantages: precise error semantics, easier logging and metrics, stable client contracts. Disadvantages: too many classes create noise.

**3. Internal working**

Exception chaining works through the `cause` field in `Throwable`. `initCause()` can set it once; constructors pass it to `super(message, cause)`. Printing a stack trace walks the cause chain and prints each one.

**4. Practical Java/Spring Boot example**

```java
public class PaymentDeclinedException extends RuntimeException {

    private final String declineCode;
    private final String orderId;

    public PaymentDeclinedException(String message, String declineCode, String orderId) {
        super(message);
        this.declineCode = declineCode;
        this.orderId = orderId;
    }

    public PaymentDeclinedException(String message, String declineCode, String orderId, Throwable cause) {
        super(message, cause);
        this.declineCode = declineCode;
        this.orderId = orderId;
    }

    public String declineCode() {
        return declineCode;
    }

    public String orderId() {
        return orderId;
    }
}

public class CustomExceptionDemo {

    public static void main(String[] args) {
        RuntimeException cause = new IllegalStateException("gateway timeout");
        PaymentDeclinedException exception = new PaymentDeclinedException(
                "Card declined", "INSUFFICIENT_FUNDS", "ORD-88", cause);

        System.out.println(exception.getMessage());
        System.out.println(exception.declineCode());
        System.out.println(exception.getCause().getMessage());
    }
}
```

Output:

```text
Card declined
INSUFFICIENT_FUNDS
gateway timeout
```

Important lines:
- Extending `RuntimeException` keeps service signatures clean and enables Spring rollback.
- `super(message, cause)` preserves the original failure for debugging.
- Structured fields make metrics and client responses reliable.

**5. Real-world use case**

A payment adapter translates gateway-specific errors such as `insufficient_funds` and `expired_card` into `PaymentDeclinedException` with stable decline codes. The REST layer maps these to HTTP 402 with a documented error code, while internal gateway class names never leak to clients.

**6. Interview follow-up questions**

- **Why not reuse `RuntimeException` directly?** You lose semantic distinction and structured data.
- **Should custom exceptions be checked?** Usually no in Spring services; use checked only for genuinely recoverable conditions.
- **What is `serialVersionUID`?** A version identifier for Java serialization; include it if exceptions cross serialization boundaries.
- **Should exception messages contain PII?** No — use IDs, not card numbers or tokens.
- **How do you test exceptions?** With JUnit `assertThrows` and assertions on message/code/cause.

**7. Common mistakes**

- Creating an exception for every minor validation case.
- Losing the cause by calling only `super(message)`.
- Putting stack traces or sensitive data in client-facing messages.
- Forgetting to map new exception types in the global handler.

**8. Interview difficulty**

**Intermediate**

---

### Question 8: How do multiple catch blocks and multi-catch work?

**1. Interview-ready answer**

With multiple catch blocks, the JVM checks them in source order and executes the first one whose type matches the thrown exception, so more specific types must come before broader ones. Putting `catch (Exception e)` before `catch (IOException e)` is a compile error because the second is unreachable. Java 7 added multi-catch — `catch (IOException | SQLException e)` — which lets one block handle unrelated types without duplication; the caught variable is effectively final inside that block. The alternatives in a multi-catch cannot be in a subclass relationship with each other.

**2. Detailed explanation**

Rules:
- Catch order: subclass before superclass.
- Multi-catch alternatives must be disjoint in the type hierarchy.
- The multi-catch parameter is effectively final — you cannot reassign it.
- Re-throwing from a multi-catch preserves the precise original type via precise rethrow analysis.
- Catching `Exception` last is a common safety net at boundaries, not in business logic.

Why it exists: to avoid duplicated handling code and to make failure handling explicit. Disadvantage: long catch ladders signal complex logic that may need refactoring.

**3. Internal working**

Catch matching is done by the JVM walking exception-table entries in order. For multi-catch, the compiler emits one handler entry per alternative, all pointing to the same handler code.

**4. Practical Java example — output-based**

```java
import java.io.IOException;
import java.sql.SQLException;

public class MultiCatchDemo {

    static void riskyOperation(int mode) throws IOException, SQLException {
        if (mode == 1) {
            throw new IOException("disk failure");
        }
        if (mode == 2) {
            throw new SQLException("query failure");
        }
    }

    public static void main(String[] args) {
        for (int mode = 1; mode <= 2; mode++) {
            try {
                riskyOperation(mode);
            } catch (IOException | SQLException exception) {
                System.out.println("handled " + exception.getClass().getSimpleName()
                        + ": " + exception.getMessage());
            }
        }
    }
}
```

Output:

```text
handled IOException: disk failure
handled SQLException: query failure
```

Invalid ordering example:

```java
try {
    riskyOperation(1);
} catch (Exception exception) {
    System.out.println("generic");
} catch (IOException exception) {
    System.out.println("specific");
}
```

This does not compile: `IOException` is already covered by `Exception`.

**5. Real-world use case**

A notification adapter calls two unrelated clients: an SMTP client throwing `MessagingException` and an SMS client throwing `HttpClientErrorException`. Instead of two identical log-and-wrap blocks, one multi-catch logs the common context and wraps the cause in `NotificationDeliveryException`.

**6. Interview follow-up questions**

- **Can multi-catch alternatives be `IOException` and `Exception`?** No — `IOException` is a subclass.
- **Is the multi-catch variable final?** Effectively final, so reassignment is a compile error.
- **What is precise rethrow?** The compiler infers the actual thrown types so a rethrow can keep specific `throws` clauses.
- **Can you catch `RuntimeException` and `Exception` together?** No, same hierarchy.
- **Should catch blocks log and rethrow?** Avoid double logging; log once at the boundary.

**7. Common mistakes**

- Putting the broadest catch first.
- Duplicating identical handling code across catch blocks.
- Catching `Exception` and doing nothing.
- Using multi-catch when each type needs different recovery logic.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: What is try-with-resources and how does it handle suppressed exceptions?

**1. Interview-ready answer**

Try-with-resources, introduced in Java 7, lets you declare `AutoCloseable` resources in parentheses after `try`; the JVM automatically closes them at the end of the block, even if an exception occurs. Since Java 9, an existing effectively final variable can be used directly. If the main body throws an exception and `close()` also throws, the body exception becomes the primary one and the close exception is attached as a suppressed exception, retrievable with `getSuppressed()`. Resources are closed in reverse order of declaration. This removes boilerplate and prevents resource leaks that plain `finally` blocks often introduce.

**2. Detailed explanation**

- `AutoCloseable.close() throws Exception` — the base interface.
- `Closeable extends AutoCloseable` — narrows to `IOException`.
- Multiple resources: `try (Connection c = ...; PreparedStatement s = ...)`.
- Closing order: reverse declaration order.
- Suppressed exceptions: added in order of closing, available via `Throwable.getSuppressed()`.
- Java 9 enhancement: `try (existingResource)` without redeclaration.

Why it exists: deterministic cleanup without duplicated `finally` code. Advantage: correct suppressed-exception semantics. Disadvantage: only works with `AutoCloseable`; some legacy APIs need adapters.

**3. Internal working**

The compiler desugars try-with-resources into nested `try`/`finally` blocks with generated calls to `addSuppressed(...)` on the primary exception. Bytecode contains explicit null checks so a resource that failed to initialize does not cause a misleading close error.

**4. Practical Java/Spring Boot example**

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class TryWithResourcesDemo {

    static class FailingResource implements AutoCloseable {

        @Override
        public void close() throws IOException {
            throw new IOException("close failed");
        }
    }

    static void process() throws IOException {
        try (FailingResource resource = new FailingResource()) {
            throw new IOException("primary failure");
        }
    }

    public static void main(String[] args) {
        try {
            process();
        } catch (IOException exception) {
            System.out.println("primary: " + exception.getMessage());
            for (Throwable suppressed : exception.getSuppressed()) {
                System.out.println("suppressed: " + suppressed.getMessage());
            }
        }
        System.out.println("temp file lines: " + countLines());
    }

    private static int countLines() {
        try {
            Path path = Files.createTempFile("demo", ".txt");
            Files.writeString(path, "one\ntwo\n");
            return Files.readAllLines(path).size();
        } catch (IOException exception) {
            throw new IllegalStateException(exception);
        }
    }
}
```

Output:

```text
primary: primary failure
suppressed: close failed
temp file lines: 2
```

Important lines:
- The primary exception remains the body exception.
- `getSuppressed()` exposes the close failure instead of losing it.
- `Files.readAllLines` itself opens and closes streams internally.

**5. Real-world use case**

A batch job writes files to Azure Blob Storage through an `OutputStream`. Under load, `close()` occasionally failed and hid the real write error. Switching to try-with-resources preserved both failures in logs, making the storage issue diagnosable. Spring's `JdbcTemplate` and `RestTemplate` handle their own resources, but custom clients must do the same.

**6. Interview follow-up questions**

- **What happens if `close()` throws with no primary exception?** The close exception propagates normally.
- **In what order are resources closed?** Reverse declaration order.
- **Can you use try-with-resources without a catch?** Yes — `try (...) { ... }` alone is valid.
- **Which interface must a resource implement?** `AutoCloseable` (or `Closeable`).
- **Are suppressed exceptions printed by default?** Yes, `printStackTrace` shows them.

**7. Common mistakes**

- Closing resources manually in `finally` alongside try-with-resources.
- Assuming close exceptions replace primary exceptions — they are suppressed instead.
- Putting resource creation outside the parentheses and forgetting to close it.
- Ignoring suppressed exceptions in logging.

**8. Interview difficulty**

**Intermediate**

---

### Question 10: How do exceptions propagate up the call stack?

**1. Interview-ready answer**

When a method throws an exception and does not catch it, the JVM unwinds the call stack frame by frame, looking for the nearest matching `catch` block. `finally` blocks along the way still execute. If no handler is found, the exception reaches the top-level thread handler and the thread terminates — in a Spring Boot web app, the framework's filter chain catches it and turns it into an error response. Unchecked exceptions propagate without any declaration; checked exceptions must be declared with `throws` in every method they pass through unless caught. Propagation stops when some method catches the exception.

**2. Detailed explanation**

Key behaviors:
- Propagation follows the reverse order of calls: deepest method first, then its caller, and so on.
- Runtime exceptions can travel arbitrarily far without compile-time declarations.
- Checked exceptions force each caller to handle or declare them.
- A catch block can stop propagation, rethrow the same exception, or wrap it in another exception.
- `finally` executes in every frame being unwound.

Why it exists: it separates detection from handling — low-level code detects failures, high-level code decides policy (retry, fallback, HTTP status).

**3. Internal working**

Each thread has its own stack of frames. On `athrow`, the JVM searches the current frame's exception table; if no handler matches, it pops the frame and repeats in the caller. The stack trace captured at throw time records the entire chain of call sites, which is why logs show the deepest cause first.

**4. Practical Java example — output-based**

```java
public class PropagationDemo {

    static void deepest() {
        throw new IllegalStateException("db timeout");
    }

    static void repositoryLayer() {
        System.out.println("repository calls deepest");
        deepest();
    }

    static void serviceLayer() {
        System.out.println("service calls repository");
        repositoryLayer();
    }

    public static void main(String[] args) {
        try {
            serviceLayer();
        } catch (IllegalStateException exception) {
            System.out.println("handled at controller boundary: " + exception.getMessage());
            System.out.println("top frame: " + exception.getStackTrace()[0].getMethodName());
        }
    }
}
```

Output:

```text
service calls repository
repository calls deepest
handled at controller boundary: db timeout
top frame: deepest
```

Important lines:
- Neither `serviceLayer` nor `repositoryLayer` declares the unchecked exception.
- `getStackTrace()[0]` is the throw site, not the catch site.
- The handler at the outer boundary decides the response policy.

**5. Real-world use case**

A JDBC exception occurs in a repository, is wrapped by a service into `OrderPersistenceException`, and is finally handled by `@ControllerAdvice` as HTTP 503 with a retry-after header. Each layer adds context without forcing every method to know about HTTP statuses.

**6. Interview follow-up questions**

- **Does `finally` run in every unwound frame?** Yes, if execution reaches it.
- **Can a checked exception propagate without `throws`?** No — compilation fails.
- **What happens if the main thread throws uncaught?** The thread terminates; the default handler prints the stack trace.
- **How does Spring catch exceptions from controllers?** Via `DispatcherServlet` and registered `HandlerExceptionResolver`s.
- **Does wrapping reset the stack trace?** No — the cause keeps its original trace; the wrapper adds its own.

**7. Common mistakes**

- Catching too early and losing context.
- Rethrowing without preserving the cause.
- Declaring checked exceptions through every layer "because the compiler asked".
- Assuming the stack trace starts at the catch site.

**8. Interview difficulty**

**Intermediate**

---

### Question 11: What is exception chaining? When should you wrap an exception instead of rethrowing it?

**1. Interview-ready answer**

Exception chaining means attaching the original exception as the `cause` of a new, higher-level exception. It preserves the root cause while letting each layer express the failure in its own vocabulary. I rethrow the same exception when the current layer has no additional context to add — for example, a helper that simply validates input. I wrap when translating across a boundary: a repository catching `SQLException` should throw `OrderPersistenceException("Failed to save order " + orderId, cause)` so the service and controller do not depend on JDBC details. Always pass the cause to `super(message, cause)`; never discard it.

**2. Detailed explanation**

`Throwable` provides:
- `Throwable(Throwable cause)` and `Throwable(String message, Throwable cause)`.
- `initCause(Throwable)` — can be called once if the constructor did not set it.
- `getCause()` and `printStackTrace()` which walks the chain.

When to wrap:
- Translating technology-specific exceptions (JDBC, HTTP client, Kafka) into domain exceptions.
- Adding contextual data such as order ID, tenant, or retry attempt.
- Hiding implementation details from upper layers.

When to rethrow as-is:
- The exception is already semantically correct for the caller.
- Adding a wrapper would create noise without information.

Advantages: layered abstraction plus full diagnostics. Disadvantage: chains that are too deep become hard to read; keep them shallow.

**3. Internal working**

The cause is stored in a field of `Throwable`. `printStackTrace` prints each cause below the wrapper with a "Caused by:" line. Logging frameworks walk the same chain, which is why a single `log.error("...", exception)` captures the entire sequence.

**4. Practical Java/Spring Boot example**

```java
import java.sql.SQLException;

class OrderPersistenceException extends RuntimeException {

    OrderPersistenceException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class ExceptionChainingDemo {

    static void saveOrder(String orderId) throws SQLException {
        throw new SQLException("connection reset by peer");
    }

    static void persist(String orderId) {
        try {
            saveOrder(orderId);
        } catch (SQLException cause) {
            throw new OrderPersistenceException("Failed to save order " + orderId, cause);
        }
    }

    public static void main(String[] args) {
        try {
            persist("ORD-42");
        } catch (OrderPersistenceException exception) {
            System.out.println(exception.getMessage());
            System.out.println("cause: " + exception.getCause().getMessage());
        }
    }
}
```

Output:

```text
Failed to save order ORD-42
cause: connection reset by peer
```

Important lines:
- `super(message, cause)` preserves the JDBC failure.
- The wrapper adds the order ID that the low-level SQLException did not know about.
- Upper layers see only the domain exception type.

**5. Real-world use case**

A Kafka consumer catches `JsonProcessingException` from deserialization and wraps it in `OrderEventParseException(eventId, cause)`. The dead-letter topic stores the original payload plus the root cause message, and monitoring alerts on the domain type instead of library-specific classes.

**6. Interview follow-up questions**

- **What does `initCause()` do?** Sets the cause on an exception whose constructor did not set one; only once.
- **Does wrapping reset the stack trace?** No — each exception keeps its own trace.
- **When is rethrowing better than wrapping?** When no new context or abstraction is added.
- **Can an exception have multiple causes?** Not directly; use suppressed exceptions for multiple secondary failures.
- **Should user-facing messages include the cause?** No — log it, but return a safe message.

**7. Common mistakes**

- Calling `super(message)` and dropping the cause.
- Wrapping the same exception repeatedly at every layer.
- Logging at each layer and then wrapping — duplicate logs with the same stack trace.
- Showing raw `SQLException` text in API responses.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: How do you handle checked exceptions inside lambdas and streams?

**1. Interview-ready answer**

Standard functional interfaces such as `Function`, `Consumer`, and `Supplier` do not declare checked exceptions, so you cannot directly call a method that throws `IOException` inside a lambda. The cleanest solutions are: wrap the checked exception in an unchecked domain exception inside the lambda; define your own functional interface whose method declares `throws`; or use a small generic helper that wraps and unwraps checked exceptions. Avoid "sneaky throw" tricks in production code — they make error handling invisible. In Spring services, wrapping into an unchecked exception also ensures `@Transactional` rollback works as expected.

**2. Detailed explanation**

Options:
1. **Wrap inside the lambda:** `path -> { try { return Files.readString(path); } catch (IOException e) { throw new UncheckedIOException(e); } }`. `UncheckedIOException` is a JDK-provided wrapper.
2. **Custom throwing interface:** `@FunctionalInterface interface ThrowingFunction<T,R,E extends Exception> { R apply(T t) throws E; }` plus an adapter.
3. **Helper utility:** a generic `unchecked(ThrowingSupplier<T>)` that catches and rethrows.
4. **Avoid streams for I/O-heavy work:** sometimes a plain loop is clearer.

Why it matters: checked exceptions and functional APIs were never fully reconciled in Java; interviewers use this to test practical Java 8+ experience.

**3. Internal working**

The functional interface method signature has no `throws` clause, and the compiler checks lambda bodies against it. Throwing an unchecked exception is always allowed because unchecked exceptions are not part of method signatures.

**4. Practical Java/Spring Boot example**

```java
import java.io.IOException;
import java.io.UncheckedIOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

@FunctionalInterface
interface ThrowingFunction<T, R> {
    R apply(T value) throws IOException;
}

public class LambdaExceptionDemo {

    static <T, R> java.util.function.Function<T, R> wrap(ThrowingFunction<T, R> function) {
        return value -> {
            try {
                return function.apply(value);
            } catch (IOException exception) {
                throw new UncheckedIOException(exception);
            }
        };
    }

    public static void main(String[] args) {
        List<Path> paths = List.of(Path.of("a.txt"), Path.of("b.txt"));

        paths.stream()
                .map(wrap(Files::readString))
                .map(String::toUpperCase)
                .forEach(System.out::println);
    }
}
```

If the files are missing, an `UncheckedIOException` propagates out of the stream and can be mapped centrally by Spring.

Important lines:
- `wrap` adapts a throwing function to a standard `Function`.
- `UncheckedIOException` preserves the original `IOException` as its cause.
- The method reference `Files::readString` matches the custom interface.

**5. Real-world use case**

An order export job reads thousands of files in a stream. Each read can throw `IOException`. The team wraps it as `UncheckedIOException`, catches it once at the job boundary, logs the failing file path from the cause, and marks the batch failed. Sneaky throw was rejected because it hid failures from monitoring.

**6. Interview follow-up questions**

- **Why can't `Function.apply` throw checked exceptions?** Its signature has no `throws` clause.
- **What is `UncheckedIOException`?** A JDK wrapper for IO failures with a cause.
- **Is sneaky throw safe?** It works at runtime due to type erasure but hides errors — avoid it.
- **Does `@Transactional` roll back on `UncheckedIOException`?** Yes, because it is a RuntimeException.
- **Can `Optional.map` handle checked exceptions?** No; the mapper still needs wrapping.

**7. Common mistakes**

- Empty catch blocks inside lambdas just to compile.
- Letting one bad element silently disappear from a stream.
- Using sneaky throw in shared libraries.
- Assuming the compiler will infer `throws` for lambdas.

**8. Interview difficulty**

**Advanced**

---

### Question 13: How do you handle exceptions globally in a Spring Boot REST API?

**1. Interview-ready answer**

I use a `@RestControllerAdvice` class with `@ExceptionHandler` methods for each exception family. Each handler builds a stable error response — code, message, path, timestamp, and optionally a trace ID — and returns an appropriate `ResponseEntity` with the right HTTP status. This keeps controllers clean, avoids duplicated try/catch blocks, and prevents stack traces from leaking to clients. I handle domain exceptions explicitly, add a fallback handler for unexpected `Exception`, and rely on Spring's built-in handlers for validation and type-mismatch errors, customizing them through `ResponseEntityExceptionHandler` when needed.

**2. Detailed explanation**

Key annotations:
- `@ControllerAdvice` — shared advice across controllers; `@RestControllerAdvice` adds `@ResponseBody`.
- `@ExceptionHandler(SomeException.class)` — maps a specific exception type.
- `@ResponseStatus(HttpStatus.NOT_FOUND)` — alternative for simple cases.
- `ResponseEntityExceptionHandler` — base class with handlers for Spring MVC exceptions such as `MethodArgumentNotValidException`.
- `@Order` — controls advice precedence when multiple advices exist.

Design an error contract:
- `code`: stable machine-readable string.
- `message`: safe, human-readable text.
- `path`: request URI.
- `timestamp`: `Instant`.
- `traceId`: correlation ID for logs.

Advantages: central policy, consistent API errors, easier client handling. Disadvantages: a giant advice class can become a dumping ground; keep handlers focused.

**3. Internal working**

When a controller throws, `DispatcherServlet` asks the configured `HandlerExceptionResolver`s to resolve it. `ExceptionHandlerExceptionResolver` finds `@ExceptionHandler` methods in advices and controllers, choosing the most specific match. The returned `ResponseEntity` is written through `HttpMessageConverter`s as JSON.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;

record ErrorResponse(String code, String message, String path, Instant timestamp) {
}

class OrderNotFoundException extends RuntimeException {
    OrderNotFoundException(String orderId) {
        super("Order not found: " + orderId);
    }
}

class OrderConflictException extends RuntimeException {
    OrderConflictException(String message) {
        super(message);
    }
}

@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    ResponseEntity<ErrorResponse> handleNotFound(OrderNotFoundException exception,
                                                   jakarta.servlet.http.HttpServletRequest request) {
        ErrorResponse body = new ErrorResponse("ORDER_NOT_FOUND", exception.getMessage(),
                request.getRequestURI(), Instant.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }

    @ExceptionHandler(OrderConflictException.class)
    ResponseEntity<ErrorResponse> handleConflict(OrderConflictException exception,
                                                  jakarta.servlet.http.HttpServletRequest request) {
        ErrorResponse body = new ErrorResponse("ORDER_CONFLICT", exception.getMessage(),
                request.getRequestURI(), Instant.now());
        return ResponseEntity.status(HttpStatus.CONFLICT).body(body);
    }

    @ExceptionHandler(Exception.class)
    ResponseEntity<ErrorResponse> handleUnexpected(Exception exception,
                                                    jakarta.servlet.http.HttpServletRequest request) {
        ErrorResponse body = new ErrorResponse("INTERNAL_ERROR", "Unexpected error",
                request.getRequestURI(), Instant.now());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(body);
    }
}

@RestController
@RequestMapping("/api/orders")
class OrderController {

    @GetMapping("/{orderId}")
    String getOrder(@PathVariable String orderId) {
        if ("missing".equals(orderId)) {
            throw new OrderNotFoundException(orderId);
        }
        return orderId;
    }
}
```

A request to `/api/orders/missing` returns HTTP 404 with a JSON body containing `ORDER_NOT_FOUND`.

Important lines:
- `@RestControllerAdvice` applies to all controllers.
- Specific handlers precede the generic `Exception` handler.
- The fallback message is intentionally generic to avoid leaking internals.

**5. Real-world use case**

A payment API maps `PaymentDeclinedException` to 402, `DuplicatePaymentException` to 409, validation failures to 400, and everything else to 500 with a trace ID. The frontend switches on the stable `code` field, not on message text, so wording changes never break clients.

**6. Interview follow-up questions**

- **What is the difference between `@ControllerAdvice` and `@RestControllerAdvice`?** The latter includes `@ResponseBody` so returned objects are serialized as JSON.
- **Which handler wins if two match?** The most specific exception type; use `@Order` across advices.
- **How do you handle validation errors?** Extend `ResponseEntityExceptionHandler` or handle `MethodArgumentNotValidException`.
- **Should you log in every handler?** Log once; handlers focused on mapping are cleaner.
- **How do you hide internal details?** Return generic messages and log the exception server-side with a trace ID.

**7. Common mistakes**

- Returning stack traces in API responses.
- Using only a generic `Exception` handler.
- Duplicating try/catch inside every controller method.
- Forgetting to map new custom exceptions.
- Depending on message text in clients.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: What are `@ResponseStatus`, `ResponseStatusException`, and Spring Boot's error handling mechanism?

**1. Interview-ready answer**

`@ResponseStatus` is an annotation you can put on an exception class or handler method to set the HTTP status automatically. `ResponseStatusException` is a convenient programmatic option: throw `new ResponseStatusException(HttpStatus.NOT_FOUND, "Order missing", cause)` from a controller or service without defining a custom exception class. For framework-level errors — 404 for unknown paths, 405 for wrong methods, 500 for uncaught failures — Spring Boot uses `/error`, `ErrorAttributes`, and `BasicErrorController`. I use custom exceptions plus `@ControllerAdvice` for domain errors, `ResponseStatusException` for simple controller-level cases, and customize `ErrorAttributes` or implement `ErrorController` only when I need to change the default error body globally.

**2. Detailed explanation**

Options compared:
- `@ResponseStatus` on exception: simple, but couples the exception class to HTTP.
- `@ResponseStatus` on handler method: sets status for a specific handler.
- `ResponseStatusException`: flexible, no extra class; good for quick cases.
- `@ControllerAdvice`: centralized, structured responses — best for production APIs.
- `ErrorController`/`ErrorAttributes`: controls the default `/error` response.

Boot properties: `server.error.include-message`, `include-stacktrace`, `include-exception`, `whitelabel.enabled` — tune these so production never leaks internals.

**3. Internal working**

`DispatcherServlet` calls `HandlerExceptionResolver`s. `ResponseStatusExceptionResolver` reads `@ResponseStatus` or `ResponseStatusException` status codes. If nothing resolves the error, the servlet container forwards to `/error`, where `BasicErrorController` uses `ErrorAttributes` to build the default map.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CustomerController {

    @GetMapping("/customers/{id}")
    public String findCustomer(@PathVariable String id) {
        if (!"CUST-1".equals(id)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Customer not found: " + id);
        }
        return "Customer-1";
    }
}
```

Invalid id → HTTP 404 with a Boot error body. In production, set `server.error.include-message=never` and return a controlled body through an advice instead.

**5. Real-world use case**

A small internal admin API used `ResponseStatusException` for quick 404s. As the API grew and clients needed stable error codes, the team migrated to domain exceptions plus `@ControllerAdvice`, keeping `ResponseStatusException` only for trivial path-not-found checks.

**6. Interview follow-up questions**

- **Can you put `@ResponseStatus` on an exception class?** Yes.
- **Does `ResponseStatusException` log by default?** No — it is handled by the resolver; log explicitly if needed.
- **What is `/error`?** Boot's default error endpoint used by the container for unresolved errors.
- **How do you disable the whitelabel error page?** `server.error.whitelabel.enabled=false`.
- **Should service layers throw `ResponseStatusException`?** Generally no — keep HTTP concerns in the web layer.

**7. Common mistakes**

- Coupling domain exceptions to HTTP statuses.
- Exposing stack traces with `server.error.include-stacktrace=always` in production.
- Returning raw `Map` error bodies with unstable fields.
- Using `ResponseStatusException` for business errors that need error codes.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: How do exceptions interact with Spring `@Transactional` rollback?

**1. Interview-ready answer**

By default, Spring rolls back a transaction when an unchecked exception — a `RuntimeException` or `Error` — escapes a `@Transactional` method. Checked exceptions do **not** trigger rollback by default because Spring assumes they may be recoverable. To roll back on a checked exception, specify `@Transactional(rollbackFor = PaymentException.class)`. Conversely, you can prevent rollback for specific unchecked exceptions with `noRollbackFor`. Rollback happens when the exception leaves the proxied method, so catching the exception inside the same method and not rethrowing it will cause a commit. This is one of the most common production bugs in Spring applications.

**2. Detailed explanation**

Default rules:
- RuntimeException/Error → rollback.
- Checked Exception → commit.
- `rollbackFor` adds types (including subtypes).
- `noRollbackFor` excludes types.
- Rollback only occurs if the exception crosses the transactional proxy boundary.

Also important: the transaction proxy must be involved — self-invocation bypasses it (Chapter 12). Exceptions thrown from a `@Scheduled` method have no transaction unless the method itself is proxied.

**3. Internal working**

`TransactionInterceptor` wraps the target method. When the invocation throws, it consults the `TransactionAttribute`'s rollback rules; if matched, it calls `PlatformTransactionManager.rollback()`. Otherwise it commits before rethrowing.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class PaymentService {

    @Transactional(rollbackFor = PaymentProcessingException.class)
    public void processPayment(String paymentId) throws PaymentProcessingException {
        try {
            savePayment(paymentId);
            callGateway(paymentId);
        } catch (GatewayException cause) {
            throw new PaymentProcessingException("Payment failed: " + paymentId, cause);
        }
    }

    @Transactional
    public void unsafeMethod(String paymentId) {
        try {
            callGateway(paymentId);
        } catch (GatewayException cause) {
            throw new PaymentProcessingException("Payment failed", cause);
        }
    }

    private void savePayment(String paymentId) {
    }

    private void callGateway(String paymentId) throws GatewayException {
    }
}

class GatewayException extends Exception {
}

class PaymentProcessingException extends Exception {

    PaymentProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

`processPayment` rolls back because of `rollbackFor`. `unsafeMethod` commits — the checked exception is not configured for rollback.

Important lines:
- `throws PaymentProcessingException` forces callers to handle a checked failure.
- Without `rollbackFor`, partial database writes would remain.
- Catching the exception and returning normally would also cause a commit.

**5. Real-world use case**

A booking service reserved a seat, then the ticket-issuing call failed with a checked `TicketIssuanceException`. Because the method lacked `rollbackFor`, the seat stayed reserved with no ticket. Adding `rollbackFor = TicketIssuanceException.class` and converting the domain exception to unchecked fixed the inconsistency.

**6. Interview follow-up questions**

- **Do checked exceptions roll back by default?** No.
- **What if I catch the exception inside the transactional method?** No rollback unless you rethrow or mark rollback-only.
- **How do I force rollback programmatically?** `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`.
- **Does `rollbackFor` include subclasses?** Yes.
- **Does `@Transactional` work on private methods?** No — proxies only intercept public methods.

**7. Common mistakes**

- Assuming all exceptions roll back.
- Catching exceptions inside `@Transactional` methods and returning normally.
- Using checked business exceptions without `rollbackFor`.
- Expecting transactions to roll back Kafka or Redis writes — they do not.

**8. Interview difficulty**

**Advanced**

---

### Question 16: What are the rules for logging exceptions in production?

**1. Interview-ready answer**

Log an exception once, at the boundary that has enough context to decide what to do, and pass the exception object itself so the stack trace and cause chain are captured — `log.error("Payment failed for order {}", orderId, exception)`. Never log only `exception.getMessage()` because the stack trace is lost. Avoid logging and rethrowing at every layer, which creates duplicate noise. Use parameterized logging rather than string concatenation, never log secrets or full request bodies, include a correlation/trace ID, and choose levels carefully: `error` for failures needing attention, `warn` for recoverable degradation, `info` for expected business events. For expected business failures, often no stack trace is needed — log the code and message instead.

**2. Detailed explanation**

Good practices:
- `log.error("Failed to charge order {}", orderId, exception);`
- Include trace IDs from MDC (Micrometer Tracing / Sleuth successor).
- Log the cause chain automatically by passing the throwable.
- Alert on rates of specific exception types, not just log text.

Bad practices:
- `log.error("Failed: " + e.getMessage())` — no stack.
- `e.printStackTrace()` — bypasses logging configuration.
- Logging in every layer of a wrapped chain.
- Logging PII, tokens, card numbers, or full payloads.

**3. Internal working**

SLF4J treats the last argument specially when it is a `Throwable`: it is rendered as a stack trace. Logback/Log4j2 then walk `getCause()` and `getSuppressed()` to print the full chain. MDC values are attached to the logging event and appear in each line.

**4. Practical Java/Spring Boot example**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.stereotype.Service;

@Service
public class CheckoutService {

    private static final Logger log = LoggerFactory.getLogger(CheckoutService.class);

    public void checkout(String orderId) {
        MDC.put("traceId", orderId);
        try {
            charge(orderId);
        } catch (PaymentDeclinedException exception) {
            log.warn("Payment declined code={} orderId={}", exception.declineCode(), orderId);
            throw exception;
        } catch (RuntimeException exception) {
            log.error("Checkout failed for order {}", orderId, exception);
            throw exception;
        } finally {
            MDC.remove("traceId");
        }
    }

    private void charge(String orderId) {
        throw new IllegalStateException("gateway unreachable");
    }
}

class PaymentDeclinedException extends RuntimeException {

    private final String declineCode;

    PaymentDeclinedException(String declineCode) {
        this.declineCode = declineCode;
    }

    String declineCode() {
        return declineCode;
    }
}
```

Important lines:
- `log.error("...", orderId, exception)` includes the stack trace.
- Business declines use `warn` without a stack trace because they are expected.
- `MDC.put`/`remove` scopes the trace ID to this operation.

**5. Real-world use case**

A payment service emitted 40,000 identical error lines per hour because every layer logged and rethrew. Centralizing logging in the controller advice and adding a metric on `PaymentDeclinedException` reduced log volume dramatically and made alerting meaningful.

**6. Interview follow-up questions**

- **Why is `e.printStackTrace()` bad?** It bypasses log configuration, has no levels or trace IDs, and writes to stderr.
- **Should expected business errors log stack traces?** Usually no — log code and context.
- **What is MDC?** Mapped Diagnostic Context — thread-local key/value pairs added to log events.
- **How do you avoid duplicate logs?** Log only where you add context or at the outermost boundary.
- **How do you alert on exceptions?** Export logger/metrics counts to monitoring and alert on rates.

**7. Common mistakes**

- Logging only the message.
- Concatenating strings in log statements instead of using placeholders.
- Logging sensitive data.
- Catching, logging, and rethrowing at multiple layers.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: What are the most important exception-handling best practices?

**1. Interview-ready answer**

My rules are: never swallow exceptions silently; catch only what you can handle; use specific exception types; always preserve the cause when wrapping; don't use exceptions for normal control flow; keep `try` blocks small; clean up with try-with-resources; log once with context and a trace ID; map exceptions to stable API errors at the boundary; and configure `@Transactional` rollback explicitly for checked exceptions. Also, avoid returning null as an error signal — prefer `Optional` or a typed result. These practices make failures visible, diagnosable, and safe.

**2. Detailed explanation**

Expanded checklist:
1. **Don't swallow:** an empty `catch` hides bugs and corrupts data.
2. **Catch narrowly:** catching `Exception` everywhere makes intent unclear.
3. **Throw early, catch late:** validate at boundaries; handle at the layer that owns the policy.
4. **Preserve cause:** `super(message, cause)`.
5. **No exceptions for flow control:** don't throw to end a loop or signal "not found" in hot paths; use `Optional` or result types.
6. **Small try blocks:** the larger the block, the harder to know what threw.
7. **Automatic cleanup:** try-with-resources.
8. **Stable error contract:** codes, status, trace ID; no internals.
9. **Transactional awareness:** rollback rules and proxy boundaries.
10. **Meaningful messages:** include IDs and operation, not secrets.

Advantages: maintainability, reliable monitoring, safer data. Disadvantage: requires discipline and sometimes a bit more code.

**3. Internal working**

Each thrown exception captures a stack trace — expensive relative to normal branching. That is another reason not to use exceptions for expected control flow in high-throughput paths.

**4. Practical Java/Spring Boot example**

```java
import java.util.Optional;

public class BestPracticesDemo {

    static Optional<String> findDiscountCode(String customerId) {
        if ("VIP".equals(customerId)) {
            return Optional.of("VIP10");
        }
        return Optional.empty();
    }

    static String applyDiscount(String customerId) {
        return findDiscountCode(customerId).orElse("NO_DISCOUNT");
    }

    public static void main(String[] args) {
        System.out.println(applyDiscount("VIP"));
        System.out.println(applyDiscount("REGULAR"));
    }
}
```

Output:

```text
VIP10
NO_DISCOUNT
```

Important lines:
- `Optional` models "no discount" as a normal outcome, not an exception.
- Exceptions are reserved for genuine failures such as a repository outage.

**5. Real-world use case**

A checkout service originally threw `NoDiscountException` for customers without a coupon — thousands of stack traces per minute under normal traffic. Replacing it with `Optional<String>` removed the noise and improved throughput; real failures still throw and alert.

**6. Interview follow-up questions**

- **When is an empty catch acceptable?** Almost never; at most with a comment and a metric in truly best-effort paths.
- **Why not use exceptions for "not found"?** It is an expected outcome and stack traces are costly.
- **Should every method declare checked exceptions?** No — only when callers can recover.
- **What is fail-fast?** Validate inputs early and throw immediately rather than proceeding with invalid state.
- **How do you document exceptions?** Javadoc `@throws` and a documented error-code catalog.

**7. Common mistakes**

- `catch (Exception ignored) {}`.
- Throwing `new Exception("error")` with no type or context.
- Returning `null` to signal failure.
- Large try blocks spanning unrelated operations.

**8. Interview difficulty**

**Intermediate**

---

### Question 18: Output-based master challenge — chaining, suppression, and `finally`

**1. Interview-ready answer**

The output is determined by three rules: the exception thrown in the try body is primary; exceptions from resource `close()` are attached as suppressed; and a `catch` handler runs before `finally`, with any return value delivered after `finally`. The primary exception is caught, its suppressed array contains the close failure, `finally` prints last among the blocks, and the method returns the catch's value.

**2. Detailed explanation**

Order of events for try-with-resources:
1. Body throws primary exception.
2. Resource `close()` is invoked automatically.
3. Close exception is added to the primary via `addSuppressed`.
4. Matching catch runs.
5. `finally` runs.
6. Return value is delivered.

**3. Internal working**

The compiler-generated `addSuppressed` calls are why the close failure does not replace the primary failure. `getSuppressed()` returns them in closing order.

**4. Practical Java example — predict the output**

```java
class PaymentException extends Exception {

    PaymentException(String message) {
        super(message);
    }
}

class GatewayResource implements AutoCloseable {

    @Override
    public void close() throws PaymentException {
        throw new PaymentException("close failed");
    }
}

public class ExceptionMasterChallenge {

    static int execute() throws PaymentException {
        try (GatewayResource resource = new GatewayResource()) {
            throw new PaymentException("primary failure");
        } catch (PaymentException exception) {
            System.out.println("caught: " + exception.getMessage());
            System.out.println("suppressed count: " + exception.getSuppressed().length);
            System.out.println("suppressed: " + exception.getSuppressed()[0].getMessage());
            return 7;
        } finally {
            System.out.println("finally runs");
        }
    }

    public static void main(String[] args) throws PaymentException {
        System.out.println("result: " + execute());
    }
}
```

Output:

```text
caught: primary failure
suppressed count: 1
suppressed: close failed
finally runs
result: 7
```

Reasoning: the body exception is primary; the close exception is suppressed; the catch executes before `finally`; and the return value from the catch survives because `finally` does not return.

**5. Real-world use case**

A database client's `close()` failed during a payment write. Because the service used try-with-resources, logs showed both the real SQL failure and the close failure, allowing the team to fix the connection-pool configuration quickly.

**6. Interview follow-up questions**

- **Which exception would `printStackTrace` show first?** The primary; suppressed ones appear below.
- **What if the body does not throw?** The close exception propagates normally.
- **What if `finally` returned a value?** It would swallow the primary exception and override the return.
- **Can suppressed exceptions be disabled?** Yes with `addSuppressed` suppression control in `Throwable`, rarely used.
- **How many resources can a try-with-resources declare?** Any number, separated by semicolons.

**7. Common mistakes**

- Assuming the close exception replaces the primary one.
- Forgetting that `finally` runs after catch.
- Returning from `finally`.
- Ignoring `getSuppressed()` in diagnostics.

**8. Interview difficulty**

**Advanced**

---

### Question 19: Coding challenge — build an exception hierarchy and REST error contract for a payment API

**1. Interview-ready answer**

I would create a small hierarchy rooted at an unchecked `PaymentException`, with `PaymentDeclinedException`, `PaymentNotFoundException`, and `DuplicatePaymentException` for specific cases. A record `ApiError` gives a stable JSON shape. A `@RestControllerAdvice` maps each exception to a documented HTTP status and error code, while a fallback handler maps unexpected failures to 500 with a generic message. The service layer throws these domain exceptions with causes preserved, and the controller remains free of try/catch blocks.

**2. Detailed explanation**

**Problem statement:** Design error handling for `POST /api/payments/{orderId}/charge` covering decline, not-found, duplicate request, validation failure, and unexpected error.

**Approach:**
1. Define base and specific unchecked exceptions with structured fields.
2. Define a record for the error response.
3. Implement a global handler with specific handlers plus a fallback.
4. Keep HTTP concerns out of the service layer.

**3. Internal working**

`DispatcherServlet` routes thrown exceptions to `ExceptionHandlerExceptionResolver`, which selects the most specific `@ExceptionHandler` method. The returned `ResponseEntity<ApiError>` is serialized by Jackson.

**4. Practical Java/Spring Boot solution**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;

record ApiError(String code, String message, String path, Instant timestamp) {
}

class PaymentException extends RuntimeException {

    PaymentException(String message, Throwable cause) {
        super(message, cause);
    }
}

final class PaymentDeclinedException extends PaymentException {

    private final String declineCode;

    PaymentDeclinedException(String declineCode, String orderId) {
        super("Payment declined for order " + orderId + ": " + declineCode, null);
        this.declineCode = declineCode;
    }

    String declineCode() {
        return declineCode;
    }
}

final class PaymentNotFoundException extends PaymentException {

    PaymentNotFoundException(String orderId) {
        super("No payment found for order " + orderId, null);
    }
}

final class DuplicatePaymentException extends PaymentException {

    DuplicatePaymentException(String idempotencyKey) {
        super("Duplicate payment request for key " + idempotencyKey, null);
        this.idempotencyKey = idempotencyKey;
    }

    private final String idempotencyKey;

    String idempotencyKey() {
        return idempotencyKey;
    }
}

@RestControllerAdvice
class PaymentExceptionHandler {

    @ExceptionHandler(PaymentDeclinedException.class)
    ResponseEntity<ApiError> handleDeclined(PaymentDeclinedException exception,
                                              jakarta.servlet.http.HttpServletRequest request) {
        return build(HttpStatus.PAYMENT_REQUIRED, "PAYMENT_DECLINED", exception.getMessage(), request);
    }

    @ExceptionHandler(PaymentNotFoundException.class)
    ResponseEntity<ApiError> handleMissing(PaymentNotFoundException exception,
                                            jakarta.servlet.http.HttpServletRequest request) {
        return build(HttpStatus.NOT_FOUND, "PAYMENT_NOT_FOUND", exception.getMessage(), request);
    }

    @ExceptionHandler(DuplicatePaymentException.class)
    ResponseEntity<ApiError> handleDuplicate(DuplicatePaymentException exception,
                                              jakarta.servlet.http.HttpServletRequest request) {
        return build(HttpStatus.CONFLICT, "DUPLICATE_PAYMENT", exception.getMessage(), request);
    }

    @ExceptionHandler(Exception.class)
    ResponseEntity<ApiError> handleUnexpected(Exception exception,
                                               jakarta.servlet.http.HttpServletRequest request) {
        return build(HttpStatus.INTERNAL_SERVER_ERROR, "INTERNAL_ERROR", "Unexpected error", request);
    }

    private ResponseEntity<ApiError> build(HttpStatus status, String code, String message,
                                           jakarta.servlet.http.HttpServletRequest request) {
        ApiError body = new ApiError(code, message, request.getRequestURI(), Instant.now());
        return ResponseEntity.status(status).body(body);
    }
}

@RestController
@RequestMapping("/api/payments")
class PaymentController {

    private final PaymentService paymentService;

    PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping("/{orderId}/charge")
    String charge(@PathVariable String orderId, @RequestHeader("Idempotency-Key") String key) {
        paymentService.charge(orderId, key);
        return "CHARGED";
    }
}

class PaymentService {

    void charge(String orderId, String idempotencyKey) {
        if ("missing".equals(orderId)) {
            throw new PaymentNotFoundException(orderId);
        }
        if ("duplicate".equals(idempotencyKey)) {
            throw new DuplicatePaymentException(idempotencyKey);
        }
        if ("declined".equals(orderId)) {
            throw new PaymentDeclinedException("INSUFFICIENT_FUNDS", orderId);
        }
    }
}
```

Important lines:
- All domain exceptions extend `RuntimeException` via `PaymentException`, so `@Transactional` rollback works.
- The fallback handler returns a generic message — no stack trace or internal detail.
- The `build` helper keeps response construction consistent.
- The controller contains no try/catch.

**5. Dry run**

| Request | Exception | Status | `code` |
|---|---|---|---|
| `/api/payments/missing/charge` | `PaymentNotFoundException` | 404 | `PAYMENT_NOT_FOUND` |
| header `Idempotency-Key: duplicate` | `DuplicatePaymentException` | 409 | `DUPLICATE_PAYMENT` |
| `/api/payments/declined/charge` | `PaymentDeclinedException` | 402 | `PAYMENT_DECLINED` |
| service throws NPE | fallback | 500 | `INTERNAL_ERROR` |

**6. Complexity**

- Exception mapping: O(1) per request apart from JSON serialization.
- Stack trace capture is the dominant cost, so expected cases should avoid exceptions (use `Optional` or result objects) where practical.

**7. Edge cases**

- Missing `Idempotency-Key` header → Spring's `MissingRequestHeaderException` should be mapped to 400 by a dedicated handler.
- Null message in an exception → handler still returns a valid body.
- Unknown exception types → fallback prevents leaking details.
- Validation failures → add a `MethodArgumentNotValidException` handler to the advice.

**8. Real-world use case**

This is the error contract used by an e-commerce checkout API. Clients switch on `code`, retry on 409 and 503, and show a friendly message on 402. Operations dashboards alert on the rate of `INTERNAL_ERROR` responses.

**9. Interview follow-up questions**

- **Why unchecked exceptions here?** Clean service signatures and default transactional rollback.
- **How do you add validation handling?** Extend `ResponseEntityExceptionHandler` or add `@ExceptionHandler(MethodArgumentNotValidException.class)`.
- **Should the error body include a stack trace?** Never in production; use a trace ID instead.
- **How do you version the error contract?** Include a schema version or use API versioning (Chapter 11).
- **How do you test the advice?** `@WebMvcTest` with `MockMvc` and assertions on status and JSON fields.

**10. Common mistakes**

- One generic 500 handler for everything.
- Returning exception class names to clients.
- Throwing checked exceptions and forgetting rollback configuration.
- Duplicating error mapping in every controller.

**11. Interview difficulty**

**Advanced**

---

### Question 20: How do you handle exceptions in async, `CompletableFuture`, and scheduled jobs?

**1. Interview-ready answer**

Exceptions in async code do not reach the caller's normal try/catch because they occur on another thread. With `CompletableFuture`, I attach `exceptionally`, `handle`, or `whenComplete` to translate failures, and I always join with a bounded timeout so a stuck future cannot hang the request. With Spring `@Async`, exceptions from `void` methods go to an `AsyncUncaughtExceptionHandler`; exceptions from methods returning `Future`/`CompletableFuture` are delivered through the future. For `@Scheduled` jobs, the framework logs an uncaught exception and continues with the next run, so I wrap the job body in try/catch, log with context, emit metrics, and alert — otherwise failures are invisible.

**2. Detailed explanation**

Key rules:
- `CompletableFuture.exceptionally(Function<Throwable, T>)` recovers with a fallback value.
- `handle(BiFunction<T, Throwable, R>)` sees both success and failure.
- `whenComplete` observes without changing the result.
- `get()` throws checked `ExecutionException`/`InterruptedException`; `join()` wraps in `CompletionException`.
- `@Async void` needs a custom `AsyncConfigurer` to handle exceptions.
- Scheduled jobs should never rely on Spring MVC's `@ControllerAdvice`.

**3. Internal working**

`CompletableFuture` stores the exception in its internal result and completes exceptionally; dependent stages receive the throwable. On `@Async`, Spring submits a task to the `TaskExecutor`; any thrown exception is caught by the executor's wrapper and routed to the uncaught handler.

**4. Practical Java/Spring Boot example**

```java
import java.time.Duration;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class AsyncExceptionDemo {

    static CompletableFuture<String> callGateway(boolean fail) {
        return CompletableFuture.supplyAsync(() -> {
            if (fail) {
                throw new IllegalStateException("gateway down");
            }
            return "OK";
        });
    }

    static String safeCall(boolean fail) {
        return callGateway(fail)
                .exceptionally(throwable -> {
                    Throwable cause = throwable instanceof CompletionException
                            ? throwable.getCause() : throwable;
                    System.out.println("handled async failure: " + cause.getMessage());
                    return "FALLBACK";
                })
                .orTimeout(2, TimeUnit.SECONDS)
                .join();
    }

    public static void main(String[] args) {
        System.out.println(safeCall(false));
        System.out.println(safeCall(true));
    }
}
```

Output:

```text
OK
handled async failure: gateway down
FALLBACK
```

Important lines:
- `exceptionally` converts failure into a fallback result.
- `CompletionException` wraps the original cause — unwrap it before logging.
- `orTimeout` prevents indefinite waiting; `join()` then completes with a `TimeoutException` if needed.

**5. Real-world use case**

A notification service fans out to three providers with `CompletableFuture`. One provider times out; `exceptionally` returns a fallback, and the service still succeeds using the other two. A nightly `@Scheduled` reconciliation job wraps its body in try/catch and increments a `job_failures_total` metric so monitoring pages the on-call engineer.

**6. Interview follow-up questions**

- **What is the difference between `get()` and `join()`?** `get()` throws checked exceptions; `join()` throws unchecked `CompletionException`.
- **Can `@ControllerAdvice` handle `@Async` exceptions?** No — they happen on a different thread.
- **How do you configure an async exception handler?** Implement `AsyncConfigurer` and provide an `AsyncUncaughtExceptionHandler`.
- **What happens when a scheduled job throws?** Spring logs it and schedules the next run.
- **How do you stop cascading async failures?** Timeouts, bulkheads, circuit breakers — Chapter 16.

**7. Common mistakes**

- Assuming try/catch around `supplyAsync(...)` catches work done inside the lambda.
- Forgetting timeouts on `join()`.
- Logging `CompletionException` instead of its cause.
- Letting scheduled jobs fail silently with no metrics.

**8. Interview difficulty**

**Advanced**

---

## Chapter 3 Verification

| Syllabus topic | Question(s) |
|---|---|
| Exception hierarchy | Q1 |
| Checked vs unchecked exceptions | Q2, Q12 |
| `Error` vs `Exception` | Q3 |
| `try`, `catch`, `finally` | Q4, Q5, Q18 |
| `throw` vs `throws` | Q6 |
| Custom exceptions | Q7, Q19 |
| Multiple catch / multi-catch | Q8 |
| Try-with-resources | Q9, Q18 |
| Exception propagation | Q10 |
| Chaining and wrapping | Q11, Q19 |
| Best practices | Q2, Q16, Q17 |
| Spring Boot exception handling | Q13, Q14, Q19 |
| Transaction rollback | Q15 |
| Async exceptions | Q20 |
| Output-based and coding questions | Q4, Q5, Q8, Q10, Q18, Q19 |

**Total interview questions: 20** (all with the required 8-part format; coding questions include problem statement, approach, dry run, complexity, and edge cases).

**Next:** [Chapter 4 — Generics and Advanced Java](04-Generics-and-Advanced-Java.md). Say `NEXT CHAPTER` to continue.
