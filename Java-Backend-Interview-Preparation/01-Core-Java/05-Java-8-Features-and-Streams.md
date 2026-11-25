# Chapter 5: Java 8+ Features and Streams

**Chapter goal:** Answer lambda, functional interface, Optional, Date/Time, and Stream questions confidently, and solve the 22 practical Stream coding problems in this chapter under interview conditions.

**Topics covered:** Lambda expressions · functional interfaces · `Predicate`, `Function`, `Consumer`, `Supplier` · method references · default and static interface methods · `Optional` · Date and Time API · Stream fundamentals · `map`, `filter`, `reduce`, `flatMap` · sorting and grouping · collectors · parallel streams · stream performance and limitations · 22 stream coding questions.

**Cross-references:** Generics and wildcards used in stream APIs are in [Chapter 4](04-Generics-and-Advanced-Java.md). Collections internals are in [Chapter 7](../02-Collections/07-Collections-Framework.md). ForkJoinPool and concurrency are in [Chapter 8](../03-Concurrency/08-Multithreading-and-Concurrency.md).

---

## Shared domain model used in this chapter

Every coding question below uses these records and sample data. They are defined once here to keep each answer focused.

```java
import java.math.BigDecimal;
import java.util.List;

record Employee(Long id, String name, String department, BigDecimal salary, boolean active) {
}

record Department(String name, String head, List<Employee> members) {
}

record Product(Long id, String name, BigDecimal price, int quantity) {
}

record OrderItem(String sku, int quantity, BigDecimal unitPrice) {

    BigDecimal lineTotal() {
        return unitPrice.multiply(BigDecimal.valueOf(quantity));
    }
}

record Order(Long id, String customerId, List<OrderItem> items, String status) {
}

record Transaction(String id, String customerId, BigDecimal amount, String status) {
}

final class SampleData {

    static final List<Employee> EMPLOYEES = List.of(
            new Employee(1L, "Asha", "IT", BigDecimal.valueOf(90000), true),
            new Employee(2L, "Ravi", "IT", BigDecimal.valueOf(60000), true),
            new Employee(3L, "Meera", "HR", BigDecimal.valueOf(75000), true),
            new Employee(4L, "Karan", "HR", BigDecimal.valueOf(45000), false),
            new Employee(5L, "Amit", "IT", BigDecimal.valueOf(120000), true));

    static final List<Product> PRODUCTS = List.of(
            new Product(1L, "Laptop", BigDecimal.valueOf(65000), 4),
            new Product(2L, "Mouse", BigDecimal.valueOf(800), 25),
            new Product(3L, "Monitor", BigDecimal.valueOf(12000), 8),
            new Product(4L, "Laptop", BigDecimal.valueOf(65000), 2));

    static final List<Order> ORDERS = List.of(
            new Order(1L, "CUST-1", List.of(
                    new OrderItem("SKU-1", 2, BigDecimal.valueOf(1000)),
                    new OrderItem("SKU-2", 1, BigDecimal.valueOf(500))), "PAID"),
            new Order(2L, "CUST-1", List.of(
                    new OrderItem("SKU-3", 3, BigDecimal.valueOf(300))), "PENDING"),
            new Order(3L, "CUST-2", List.of(
                    new OrderItem("SKU-1", 1, BigDecimal.valueOf(1000))), "PAID"));

    static final List<Transaction> TRANSACTIONS = List.of(
            new Transaction("TX-1", "CUST-1", BigDecimal.valueOf(2500), "SUCCESS"),
            new Transaction("TX-2", "CUST-2", BigDecimal.valueOf(1500), "FAILED"),
            new Transaction("TX-1", "CUST-1", BigDecimal.valueOf(2500), "SUCCESS"),
            new Transaction("TX-3", "CUST-3", BigDecimal.valueOf(9000), "SUCCESS"));

    private SampleData() {
    }
}
```

---

## Part A — Conceptual questions

### Question 1: What is a lambda expression in Java?

**1. Interview-ready answer**

A lambda expression is a compact way to represent an anonymous function — a block of code you can pass around as a value. It has the form `(parameters) -> expression` or `(parameters) -> { statements; }` and can only be used where a functional interface is expected. The compiler infers parameter types from the target type, so `(a, b) -> a + b` works for `BinaryOperator<Integer>` or `IntBinaryOperator`. Lambdas are not anonymous class instances with their own `this`; inside a lambda, `this` refers to the enclosing object. They also can only capture local variables that are final or effectively final.

**2. Detailed explanation**

A lambda has three parts: parameter list, arrow, and body. The body can be an expression (implicitly returned) or a block (requires explicit `return`). Examples:

```java
Runnable task = () -> System.out.println("running");
Function<Integer, Integer> doubleIt = value -> value * 2;
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

Why it exists: before Java 8, passing behavior required anonymous inner classes with a lot of boilerplate. Lambdas make APIs like `list.forEach(...)` and `stream.filter(...)` readable.

Advantages: concise code, enables functional-style APIs and better parallelism. Disadvantages: harder to debug stack traces, easy to hide side effects, and not suitable for every problem.

**3. Internal working**

The compiler uses `invokedynamic` and `LambdaMetafactory` to create the functional interface instance at runtime, instead of generating an anonymous class file per lambda. The lambda body is compiled into a private synthetic method of the enclosing class. Captured variables are passed as arguments to the lambda factory. This avoids class-file bloat and lets the JIT optimize well.

**4. Practical Java/Spring Boot example**

```java
import java.util.List;
import java.util.function.Function;

public class LambdaBasics {

    public static void main(String[] args) {
        int taxRate = 18;

        Function<Integer, Integer> withTax = amount -> amount + (amount * taxRate / 100);

        List.of(1000, 2000, 3000).stream()
                .map(withTax)
                .forEach(System.out::println);
    }
}
```

Output:

```text
1180
2360
3540
```

Important lines:
- `amount -> ...` has its parameter type inferred from `Function<Integer, Integer>`.
- `taxRate` is effectively final — it is never reassigned.
- `System.out::println` is a method reference (Question 5).

**5. Real-world use case**

A Spring service passes behavior to a retry helper: `retryTemplate.execute(ctx -> paymentGateway.charge(request))`. The helper owns retry policy, while the lambda owns the actual operation.

**6. Interview follow-up questions**

- **Can a lambda be used anywhere?** Only where the target type is a functional interface.
- **Can lambda parameters be `var`?** Yes since Java 10: `(var a, var b) -> a + b`.
- **Does `this` inside a lambda refer to the lambda object?** No — to the enclosing instance.
- **Can a lambda throw checked exceptions?** Only if the functional interface method declares them.
- **Are lambdas garbage-collected like objects?** The functional interface instance is an object and can be collected when unreachable.

**7. Common mistakes**

- Modifying captured local variables inside a lambda.
- Using lambdas where a named method would be clearer.
- Adding side effects inside `map`.
- Expecting a lambda to have its own identity or `this`.

**8. Interview difficulty**

**Beginner**

---

### Question 2: What is a functional interface? Why is `@FunctionalInterface` used?

**1. Interview-ready answer**

A functional interface is an interface with exactly one abstract method. It may also contain `default` and `static` methods, which do not count toward the single abstract method. Because it has one abstract method, an instance can be created with a lambda or method reference. `@FunctionalInterface` is an optional compile-time annotation that verifies the interface still has exactly one abstract method — it prevents someone from accidentally adding a second method and breaking all lambdas that target it. JDK examples include `Runnable`, `Comparator<T>`, `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, and `Supplier<T>`.

**2. Detailed explanation**

Valid functional interface:

```java
@FunctionalInterface
interface PriceCalculator {
    BigDecimal calculate(BigDecimal base);

    default BigDecimal withTax(BigDecimal base) {
        return calculate(base).multiply(BigDecimal.valueOf(1.18));
    }
}
```

Invalid: adding another abstract method such as `BigDecimal discount()`.

Why it exists: to define a contract for behavior that can be passed as a value. Advantages: clean APIs, compile-time safety with the annotation. Disadvantage: the single-method restriction can lead to many small interfaces.

**3. Internal working**

The compiler checks the abstract-method count. At runtime, a lambda targeting the interface produces an instance whose single method is implemented by the generated lambda body. Default methods are inherited normally.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;

@FunctionalInterface
interface FareRule {

    BigDecimal apply(BigDecimal fare);

    default FareRule then(FareRule next) {
        return amount -> next.apply(apply(amount));
    }
}

public class FunctionalInterfaceDemo {

    public static void main(String[] args) {
        FareRule memberDiscount = fare -> fare.multiply(BigDecimal.valueOf(0.9));
        FareRule gst = fare -> fare.multiply(BigDecimal.valueOf(1.18));

        System.out.println(memberDiscount.then(gst).apply(BigDecimal.valueOf(1000)));
    }
}
```

Output:

```text
1062.000
```

Important lines:
- `FareRule` has one abstract method plus a default combinator.
- `then` returns another lambda, composing rules functionally.

**5. Real-world use case**

A pricing engine composes rules with `FareRule.then(...)`, letting business users add new rules without editing a central switch statement.

**6. Interview follow-up questions**

- **Do default methods count as abstract methods?** No.
- **Can a functional interface extend another interface?** Yes, as long as it still has one abstract method.
- **What happens if you add a second abstract method?** Compilation fails if `@FunctionalInterface` is present.
- **Is `Comparator` functional?** Yes — `compare(T, T)` is abstract; the rest are defaults/statics.
- **Can an abstract class be functional?** No — lambdas target interfaces only.

**7. Common mistakes**

- Forgetting the annotation and accidentally adding methods.
- Thinking `Object` methods count as abstract methods — they do not.
- Creating too many nearly identical custom interfaces instead of using JDK ones.

**8. Interview difficulty**

**Beginner**

---

### Question 3: Explain `Predicate`, `Consumer`, and `Supplier` with examples.

**1. Interview-ready answer**

These are three core JDK functional interfaces. `Predicate<T>` takes a `T` and returns `boolean` — used for filtering and validation. `Consumer<T>` takes a `T` and returns nothing — used for side effects such as logging or populating a DTO. `Supplier<T>` takes nothing and returns a `T` — used for lazy creation, defaults, and factories. Each has useful combinators: `Predicate.and/or/negate`, `Consumer.andThen`, and `Supplier` used by `Optional.orElseGet`.

**2. Detailed explanation**

| Interface | Method | Purpose |
|---|---|---|
| `Predicate<T>` | `boolean test(T)` | yes/no decision |
| `Consumer<T>` | `void accept(T)` | perform an action |
| `Supplier<T>` | `T get()` | produce a value |

Variants: `IntPredicate`, `BiPredicate<T,U>`, `BiConsumer<T,U>`, `LongSupplier`, etc., avoid boxing.

**3. Internal working**

All are plain interfaces with one abstract method and default composition methods. Streams accept them as parameter objects and call them during pipeline execution.

**4. Practical Java example**

```java
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class BuiltInInterfacesDemo {

    public static void main(String[] args) {
        Predicate<Employee> highPaid = employee -> employee.salary().compareTo(java.math.BigDecimal.valueOf(80000)) > 0;
        Predicate<Employee> active = employee -> employee.active();
        Consumer<Employee> log = employee -> System.out.println(employee.name());
        Supplier<String> defaultDepartment = () -> "UNASSIGNED";

        SampleData.EMPLOYEES.stream()
                .filter(highPaid.and(active))
                .forEach(log);

        System.out.println(defaultDepartment.get());
    }
}
```

Output:

```text
Asha
Amit
UNASSIGNED
```

Important lines:
- `highPaid.and(active)` combines predicates without nested if-statements.
- `forEach(log)` applies the consumer to each element.
- `Supplier` produces a value only when `get()` is called.

**5. Real-world use case**

A Spring validation component builds `Predicate<Order>` rules for fraud checks, a notification component uses `Consumer<Order>` to publish events, and a cache loader uses `Supplier<Order>` with `Cache.get(key, supplier)` so the value is computed only on a miss.

**6. Interview follow-up questions**

- **What is `BiPredicate`?** A predicate with two arguments.
- **Why is `Consumer.andThen` useful?** It chains side effects in order.
- **How does `Supplier` differ from a factory method?** It is an object you can pass around; `get()` may produce a new value each call.
- **Which functional interface does `Optional.orElseGet` take?** `Supplier<? extends T>`.
- **What does `Predicate.isEqual(x)` do?** Returns a predicate testing equality with x.

**7. Common mistakes**

- Using `Consumer` where a `Function` is needed and losing the return value.
- Calling a `Supplier` eagerly when lazy evaluation was intended.
- Putting expensive side effects in `filter` predicates.
- Using boxed predicates in hot numeric loops.

**8. Interview difficulty**

**Beginner**

---

### Question 4: Explain `Function`, `BiFunction`, and function composition.

**1. Interview-ready answer**

`Function<T,R>` takes a `T` and returns an `R`; it is the standard interface for transformations such as DTO mapping. `BiFunction<T,U,R>` takes two arguments and returns one. Both provide `andThen(Function)` — apply this function, then the next — and `andThen` on `Function` plus `compose(Function)` — apply the argument function first. `UnaryOperator<T>` is a `Function<T,T>` with matching input and output. In backend code, combining mappers with `andThen` keeps conversion pipelines explicit and reusable.

**2. Detailed explanation**

```java
Function<Employee, String> toName = Employee::name;
Function<String, String> upper = String::toUpperCase;
Function<Employee, String> display = toName.andThen(upper);

Function<Integer, Integer> timesTwo = value -> value * 2;
Function<Integer, Integer> plusThree = value -> value + 3;
Function<Integer, Integer> composed = timesTwo.andThen(plusThree);
```

`composed.apply(5)` = `(5 * 2) + 3 = 13`. Using `timesTwo.compose(plusThree)` would give `(5 + 3) * 2 = 16` — the order matters.

**3. Internal working**

`andThen` and `compose` return new function instances that call the original functions in sequence. No new lambda class is compiled per composition — the returned object is another lambda capturing both functions.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.function.BiFunction;
import java.util.function.Function;

public class FunctionCompositionDemo {

    public static void main(String[] args) {
        Function<BigDecimal, BigDecimal> discount = amount -> amount.multiply(BigDecimal.valueOf(0.9));
        Function<BigDecimal, BigDecimal> tax = amount -> amount.multiply(BigDecimal.valueOf(1.18));
        Function<BigDecimal, BigDecimal> finalPrice = discount.andThen(tax);

        BiFunction<BigDecimal, Integer, BigDecimal> total = (price, quantity) ->
                price.multiply(BigDecimal.valueOf(quantity));

        System.out.println(finalPrice.apply(BigDecimal.valueOf(1000)).setScale(2));
        System.out.println(total.apply(finalPrice.apply(BigDecimal.valueOf(1000)), 3).setScale(2));
    }
}
```

Output:

```text
1062.00
3186.00
```

Important lines:
- `discount.andThen(tax)` applies discount first, then tax.
- `BiFunction` combines two inputs into one result.
- `setScale(2)` formats the value for display.

**5. Real-world use case**

A checkout pipeline composes `Function<Cart, Cart>` steps — apply coupon, reserve inventory, calculate tax — and runs them in order. Each step is unit-testable in isolation.

**6. Interview follow-up questions**

- **What is `UnaryOperator<T>`?** `Function<T,T>` for same-type transformations.
- **What is `BinaryOperator<T>`?** `BiFunction<T,T,T>`, useful for `reduce`.
- **What does `Function.identity()` return?** A function that returns its argument.
- **How do you compose three functions?** Chain `andThen` calls.
- **Can functions be stored in collections?** Yes — they are ordinary objects.

**7. Common mistakes**

- Confusing `andThen` with `compose` order.
- Mutating input objects inside a `Function` — prefer pure transformations.
- Using `BiFunction` when a curried `Function` chain would be clearer.
- Ignoring unchecked exceptions thrown inside a function.

**8. Interview difficulty**

**Intermediate**

---

### Question 5: What are method references? Explain the four kinds.

**1. Interview-ready answer**

A method reference is a shorthand for a lambda that simply calls an existing method: `String::toUpperCase` instead of `s -> s.toUpperCase()`. There are four kinds: reference to a static method (`Math::abs`), an instance method of a particular object (`orderService::validate`), an instance method of an arbitrary object of a type (`String::length`), and a constructor reference (`ArrayList::new`). The compiler picks the right overload from the target functional interface. Method references make stream pipelines shorter and more readable when the lambda body does nothing but delegate.

**2. Detailed explanation**

| Kind | Syntax | Equivalent lambda |
|---|---|---|
| Static | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| Bound instance | `printer::print` | `value -> printer.print(value)` |
| Unbound instance | `String::toUpperCase` | `s -> s.toUpperCase()` |
| Constructor | `Employee::new` | `args -> new Employee(args)` |

Array constructor: `int[]::new`.

**3. Internal working**

Method references compile the same way as lambdas — via `invokedynamic` and `LambdaMetafactory`. The target method is captured either as a direct method handle or as a bound receiver.

**4. Practical Java/Spring Boot example**

```java
import java.util.List;
import java.util.function.Function;
import java.util.stream.Collectors;

public class MethodReferenceDemo {

    static String describe(Employee employee) {
        return employee.name() + " (" + employee.department() + ")";
    }

    public static void main(String[] args) {
        System.out.println(SampleData.EMPLOYEES.stream()
                .map(MethodReferenceDemo::describe)
                .collect(Collectors.joining(", ")));

        Function<String, Integer> parser = Integer::parseInt;
        System.out.println(parser.apply("42"));

        System.out.println(SampleData.EMPLOYEES.stream()
                .map(Employee::name)
                .map(String::toUpperCase)
                .findFirst()
                .orElse(""));
    }
}
```

Output:

```text
Asha (IT), Ravi (IT), Meera (HR), Karan (HR), Amit (IT)
42
ASHA
```

Important lines:
- `MethodReferenceDemo::describe` is a static method reference.
- `Employee::name` is an unbound instance method reference on a record accessor.
- `String::toUpperCase` is also unbound.

**5. Real-world use case**

A Spring service uses `orderRepository::findById` as a `Function<Long, Optional<Order>>` inside a batch lookup, and `OrderDto::fromEntity` as the mapper in a stream.

**6. Interview follow-up questions**

- **Can you use a method reference with overloaded methods?** Yes, the compiler resolves from the target type.
- **What is a bound instance method reference?** `object::method` — the receiver is fixed.
- **How do you reference a constructor?** `ClassName::new`.
- **Are method references faster than lambdas?** Usually equivalent — both use invokedynamic.
- **Can you reference a private method?** Yes, from within the same class.

**7. Common mistakes**

- Using parentheses incorrectly: `String::toUpperCase()` is invalid.
- Confusing `ClassName::instanceMethod` with `object::instanceMethod`.
- Overusing references when a lambda is clearer.
- Expecting a method reference to capture state — only bound references do.

**8. Interview difficulty**

**Beginner**

---

### Question 6: What are default and static interface methods? What happens in a diamond conflict?

**1. Interview-ready answer**

Java 8 allowed interfaces to contain concrete `default` methods and `static` methods. A default method provides a fallback implementation that implementing classes inherit; a static method is utility code called on the interface itself. Default methods let the JDK add methods such as `Stream.takeWhile` (Java 9) or `Collection.removeIf` without breaking existing implementations. If two interfaces provide the same default method, a class implementing both must override it or the compiler reports a conflict. A class method always wins over an interface default. Default methods cannot override `Object` methods such as `equals` or `hashCode`.

**2. Detailed explanation**

Why they exist: binary compatibility. Adding an abstract method to a widely implemented interface would break every implementer. Default methods provide a compatible body.

Rules:
- `default` methods are public; you cannot use `final` or `static` with `default`.
- Static interface methods are not inherited by implementing classes.
- If a superclass has a concrete method with the same signature, it wins.
- Conflict resolution syntax: `InterfaceA.super.method()`.

Advantages: evolvable APIs, shared behavior without abstract classes. Disadvantages: multiple inheritance of behavior returns, which can create ambiguity.

**3. Internal working**

Default methods are compiled as regular interface methods with a body. Invocation uses `invokeinterface`; the JVM resolves to the most specific implementation. Class methods take precedence during resolution.

**4. Practical Java/Spring Boot example**

```java
interface Auditable {

    default String auditLabel() {
        return "AUDIT";
    }
}

interface TenantScoped {

    default String auditLabel() {
        return "TENANT";
    }
}

class PaymentAudit implements Auditable, TenantScoped {

    @Override
    public String auditLabel() {
        return Auditable.super.auditLabel() + "-" + TenantScoped.super.auditLabel();
    }

    static String version() {
        return "1.0";
    }

    public static void main(String[] args) {
        System.out.println(new PaymentAudit().auditLabel());
        System.out.println(PaymentAudit.version());
    }
}
```

Output:

```text
AUDIT-TENANT
1.0
```

Important lines:
- Both interfaces define `auditLabel`, so the class must override.
- `Auditable.super.auditLabel()` explicitly selects one implementation.
- `static String version()` here is a class static method; an interface static method would be called as `InterfaceName.method()`.

**5. Real-world use case**

Spring Data repositories rely on default methods to add behavior such as `findById(...).orElse(...)` helpers without changing every repository implementation. `Comparator` and `Predicate` default combinators are other everyday examples.

**6. Interview follow-up questions**

- **Can an interface default method be `final`?** No.
- **Are static interface methods inherited?** No — call them via the interface name.
- **Which wins: class method or default method?** Class method.
- **Can a default method override `toString()`?** No — compilation error.
- **Why were default methods added?** To evolve interfaces without breaking implementers.

**7. Common mistakes**

- Forgetting to resolve diamond conflicts.
- Trying to call an interface static method through an implementing class.
- Putting mutable state in interfaces — fields are still `public static final`.
- Using default methods for behavior that belongs in a class.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: What is `Optional` and how should it be used correctly?

**1. Interview-ready answer**

`Optional<T>` is a container that explicitly models "a value may be absent". It forces callers to handle the empty case instead of dereferencing null. You create it with `Optional.of(value)` for non-null values, `Optional.ofNullable(value)` when null is possible, or `Optional.empty()`. Read it with `map`, `flatMap`, `filter`, `orElse`, `orElseGet`, or `orElseThrow`. It is intended mainly for return types, not fields, constructor parameters, or collection elements. `orElse` evaluates its argument eagerly, while `orElseGet` is lazy — important when the fallback is expensive.

**2. Detailed explanation**

Common methods:
- `of`/`ofNullable`/`empty`.
- `isPresent`, `isEmpty` (Java 11).
- `map(Function)` — transform if present; wraps result in Optional.
- `flatMap(Function<T, Optional<R>>)` — avoids `Optional<Optional<R>>`.
- `filter(Predicate)`.
- `orElse(default)` — eager; `orElseGet(Supplier)` — lazy.
- `orElseThrow(Supplier<? extends X>)`.

Why it exists: to make absence visible in API signatures. Advantages: fewer NPEs, expressive pipelines. Disadvantages: can be misused as a field type or overused for trivial checks; it is not Serializable.

**3. Internal working**

`Optional` is a final value-based class holding a single `T` reference or null internally. It is not meant for identity comparisons. `map` delegates to `ofNullable(mapper.apply(value))`.

**4. Practical Java/Spring Boot example**

```java
import java.util.Optional;

public class OptionalDemo {

    static Optional<Employee> findEmployee(Long id) {
        return SampleData.EMPLOYEES.stream()
                .filter(employee -> employee.id().equals(id))
                .findFirst();
    }

    static String departmentOf(Long id) {
        return findEmployee(id)
                .map(Employee::department)
                .filter(department -> !department.isBlank())
                .orElseGet(() -> "UNASSIGNED");
    }

    public static void main(String[] args) {
        System.out.println(departmentOf(1L));
        System.out.println(departmentOf(99L));
    }
}
```

Output:

```text
IT
UNASSIGNED
```

Important lines:
- `map(Employee::department)` unwraps the employee and rewraps the department.
- `orElseGet` computes the fallback only when needed.
- Repository methods in Spring Data return `Optional<T>` for single-result lookups.

**5. Real-world use case**

A Spring Data repository returns `Optional<Customer>` from `findById`. The service uses `orElseThrow(() -> new CustomerNotFoundException(id))` so the REST layer can map it to 404. Using `orElse(new Customer())` would silently create a fake entity.

**6. Interview follow-up questions**

- **What is the difference between `map` and `flatMap`?** `map` wraps the result; `flatMap` expects an Optional and flattens it.
- **Why not use Optional as a field?** It is not Serializable and adds noise; use nullable field plus accessor or a domain construct.
- **`orElse` vs `orElseGet`?** `orElse` evaluates immediately; `orElseGet` lazily.
- **Is Optional meant for method parameters?** Generally no — it complicates overloads and null checks.
- **Does Optional prevent all NPEs?** No — the value inside can still be null if created carelessly (`map` returning null yields empty).

**7. Common mistakes**

- Calling `get()` without checking — throws `NoSuchElementException`.
- Using `Optional.of(null)` — throws NPE; use `ofNullable`.
- Using Optional in JPA entity fields.
- Using `isPresent()` followed by `get()` instead of `ifPresent`/`map`.

**8. Interview difficulty**

**Intermediate**

---

### Question 8: What is the Java Date and Time API? How is it different from `java.util.Date`?

**1. Interview-ready answer**

The `java.time` API, introduced in Java 8, is immutable, thread-safe, and explicit about time zones. `LocalDate` is a date without time, `LocalDateTime` adds time without zone, `Instant` is a machine timestamp in UTC, `ZonedDateTime` includes a time zone, `Duration` measures time between instants, and `Period` measures date-based amounts. `DateTimeFormatter` is thread-safe. In contrast, `java.util.Date` and `Calendar` are mutable, have confusing zero-based months, are not thread-safe, and mix date, time, and timezone concerns. Backend code should persist `Instant` or `LocalDateTime` and convert to a zone only at the presentation boundary.

**2. Detailed explanation**

Key types:
- `LocalDate.now()`, `LocalDate.of(2026, 11, 19)`.
- `LocalDateTime`, `LocalTime`.
- `Instant.now()` — UTC timeline point, ideal for createdAt/updatedAt.
- `ZonedDateTime.now(ZoneId.of("Asia/Kolkata"))`.
- `Duration.between(start, end)` — seconds/nanos.
- `Period.between(startDate, endDate)` — years/months/days.
- `DateTimeFormatter.ISO_LOCAL_DATE_TIME` or custom patterns.

Advantages: immutability, clear semantics, thread safety. Disadvantage: migrating legacy Date/Calendar code takes effort.

**3. Internal working**

`java.time` classes are value objects backed by primitive fields; all operations return new instances. `Instant` stores seconds and nanoseconds from the epoch. Zone rules come from the tzdb bundled with the JDK.

**4. Practical Java/Spring Boot example**

```java
import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;

public class DateTimeDemo {

    public static void main(String[] args) {
        Instant createdAt = Instant.now();
        LocalDate travelDate = LocalDate.of(2026, 12, 15);
        ZonedDateTime departure = travelDate.atTime(9, 30)
                .atZone(ZoneId.of("Asia/Kolkata"));
        Instant departureUtc = departure.toInstant();

        System.out.println(createdAt);
        System.out.println(departure.format(DateTimeFormatter.ISO_OFFSET_DATE_TIME));
        System.out.println(departureUtc);
        System.out.println(Duration.between(createdAt, departureUtc).toHours());
    }
}
```

Output will vary with the current time; it prints the instant, formatted zoned departure, UTC instant, and hours until departure.

Important lines:
- `Instant` is the right type for database timestamps and audit fields.
- `ZonedDateTime` converts a local departure time into an absolute instant.
- `DateTimeFormatter` is immutable and safe to store in a `static final` field.

**5. Real-world use case**

An airline booking service stores departure time as `Instant` in PostgreSQL, displays it in the user's zone with `ZonedDateTime`, and computes check-in windows with `Duration`. Using `Date` previously caused off-by-one-month bugs and timezone drift.

**6. Interview follow-up questions**

- **When to use `LocalDateTime` vs `Instant`?** LocalDateTime for wall-clock times without zone; Instant for absolute moments.
- **Is `DateTimeFormatter` thread-safe?** Yes, unlike `SimpleDateFormat`.
- **How do you parse a string?** `LocalDate.parse("2026-12-15")` or `DateTimeFormatter.ofPattern(...)`.
- **What is the difference between `Duration` and `Period`?** Duration is time-based; Period is date-based.
- **How do you add days?** `localDate.plusDays(1)` — returns a new object.

**7. Common mistakes**

- Storing `LocalDateTime` without documenting the zone.
- Using `SimpleDateFormat` shared across threads.
- Using `Date` for new code.
- Assuming `Period.ofMonths(1).getDays()` equals 30.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: What is the Stream API and how does a stream pipeline work?

**1. Interview-ready answer**

A stream is a sequence of elements supporting functional-style operations, not a data structure and not a place to store elements. A pipeline has a source (collection, array, generator), zero or more intermediate operations such as `filter` and `map`, and one terminal operation such as `collect`, `forEach`, or `count`. Intermediate operations are lazy — they do nothing until the terminal operation starts. Streams are one-time use; after a terminal operation the stream is consumed. They do not modify the source and are naturally suited to transformations, aggregations, and grouping.

**2. Detailed explanation**

Intermediate operations: `filter`, `map`, `flatMap`, `distinct`, `sorted`, `limit`, `skip`, `peek`, `takeWhile`, `dropWhile`.
Terminal operations: `collect`, `forEach`, `count`, `reduce`, `findFirst`, `anyMatch`, `allMatch`, `noneMatch`, `min`, `max`, `toArray`.

Why it exists: to express data-processing intent declaratively and enable parallel execution without manual thread code.

Advantages: readable pipelines, composability, lazy short-circuiting. Disadvantages: one-time use, harder debugging, overhead for tiny collections, and easy to hide side effects.

**3. Internal working**

Each intermediate operation returns a new stream that wraps the previous one, forming a chain of `Sink` objects. When the terminal operation runs, elements are pushed through the chain one at a time. Short-circuiting operations such as `findFirst` can stop early. This is why a `limit(1)` after an expensive `map` only maps one element.

**4. Practical Java example**

```java
import java.util.List;
import java.util.stream.Collectors;

public class StreamFundamentals {

    public static void main(String[] args) {
        List<String> names = SampleData.EMPLOYEES.stream()
                .filter(Employee::active)
                .map(Employee::name)
                .sorted()
                .collect(Collectors.toList());

        System.out.println(names);

        long count = SampleData.EMPLOYEES.stream()
                .filter(employee -> employee.salary().compareTo(java.math.BigDecimal.valueOf(70000)) > 0)
                .count();
        System.out.println(count);
    }
}
```

Output:

```text
[Amit, Asha, Meera, Ravi]
3
```

Important lines:
- `filter` and `map` are lazy; `collect` triggers execution.
- The source list is unchanged.
- `toList()` (Java 16+) is a shorter immutable alternative to `collect(Collectors.toList())`.

**5. Real-world use case**

A reporting endpoint builds a pipeline over orders: filter PAID, map to line items, group by SKU, sum quantities. The same pipeline works over an in-memory list or a parallel stream without changing business logic.

**6. Interview follow-up questions**

- **Can a stream be reused?** No — create a new one from the source.
- **Does `filter` mutate the collection?** No.
- **What is short-circuiting?** Operations like `findFirst` or `limit` stop processing early.
- **What does `peek` do?** An intermediate hook mainly for debugging.
- **When does sorting happen?** `sorted` is a stateful intermediate operation; it must see all elements before emitting.

**7. Common mistakes**

- Reusing a consumed stream.
- Mutating the source collection during streaming.
- Using `forEach` when `collect` is appropriate.
- Assuming intermediate operations execute immediately.

**8. Interview difficulty**

**Intermediate**

---

### Question 10: What are parallel streams and when should you use them?

**1. Interview-ready answer**

Calling `parallelStream()` or `stream().parallel()` splits the source into chunks and processes them on the common `ForkJoinPool`, whose default size is usually processor count minus one. Parallel streams can speed up CPU-bound, large, easily splittable workloads with stateless operations. They are often a poor choice for small collections, I/O-bound work, operations with shared mutable state, or ordered stateful operations, and they can interfere with other tasks sharing the common pool. In Spring Boot services, I prefer explicit executors or async methods for I/O parallelism and use parallel streams only after measuring a real gain.

**2. Detailed explanation**

How splitting works: the source is recursively divided (ArrayList splits cheaply; LinkedList poorly), workers process chunks, and results are combined. Ordered sources preserve encounter order for ordered terminal operations unless `unordered()` is used.

When to use:
- Large datasets (typically tens of thousands+ elements).
- CPU-bound pure transformations.
- No shared mutable state.
- Splittable source such as array or ArrayList.

When to avoid:
- Small datasets — coordination overhead dominates.
- Blocking I/O — use an executor instead.
- Shared mutable state or non-thread-safe collectors.
- Stateful ordered operations like `sorted` with distinct semantics.

**3. Internal working**

The common ForkJoinPool uses work-stealing. Its parallelism can be changed with `-Djava.util.concurrent.ForkJoinPool.common.parallelism=N`. Custom pools can be used by submitting the parallel stream task to your own ForkJoinPool.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.List;

public class ParallelStreamDemo {

    static boolean isPrime(int number) {
        if (number < 2) {
            return false;
        }
        for (int divisor = 2; divisor * divisor <= number; divisor++) {
            if (number % divisor == 0) {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        List<Integer> candidates = java.util.stream.IntStream.rangeClosed(1, 20000)
                .boxed().toList();

        long sequential = candidates.stream().filter(ParallelStreamDemo::isPrime).count();
        long parallel = candidates.parallelStream().filter(ParallelStreamDemo::isPrime).count();

        System.out.println(sequential + " / " + parallel);
    }
}
```

Output is the prime count printed twice; both must match.

Important lines:
- The predicate is pure and stateless — safe for parallelism.
- `boxed()` introduces wrappers; `IntStream` directly would avoid that cost.

**5. Real-world use case**

A fraud-scoring job evaluates thousands of independent rules per transaction. A parallel stream cut processing time, but only after the team removed a shared `SimpleDateFormat` (now `DateTimeFormatter`) and moved logging out of the hot path.

**6. Interview follow-up questions**

- **Which pool do parallel streams use?** The common ForkJoinPool by default.
- **Do parallel streams preserve order?** For ordered sources and ordered terminals, yes — at a cost.
- **Can you set a custom pool?** Yes, submit the task to a custom ForkJoinPool.
- **Why might a parallel stream be slower?** Small data, boxing, poor splitting, I/O, shared state.
- **Are parallel streams safe with `Collectors.toList()`?** Yes; collectors handle combining, but the accumulator must be side-effect-free.

**7. Common mistakes**

- Adding `.parallel()` everywhere assuming it is faster.
- Modifying shared lists or maps inside a parallel stream.
- Using non-thread-safe formatters or random generators inside the pipeline.
- Blocking on I/O inside parallel workers.

**8. Interview difficulty**

**Advanced**

---

### Question 11: What are the performance costs and limitations of streams?

**1. Interview-ready answer**

Streams have setup costs, allocate pipeline objects, and often box primitives, so for very small collections a simple loop is faster. Large pipelines with `sorted`, `distinct`, or grouping allocate more and require seeing all elements. Side effects inside `map` or `filter` are dangerous because the execution order is unspecified, especially in parallel streams. Streams are also hard to debug — stack traces show lambda bodies rather than readable steps — and a stream can be consumed only once. I use streams for clarity in transformations and aggregations, primitive streams for numeric hot paths, and loops when I need tight control over iteration, early exit, or mutation.

**2. Detailed explanation**

Costs:
- Pipeline object allocation per chain.
- Boxing with `Stream<Integer>` — use `IntStream`/`LongStream`.
- Stateful operations buffer elements (`sorted`, `distinct`, `groupingBy`).
- Lambdas capture variables — small allocation unless non-capturing.
- Debugging and profiling complexity.

Limitations:
- No checked exceptions in standard functional interfaces.
- One-time consumption.
- No direct access to index.
- Not ideal for algorithms requiring mutation or complex control flow.

**3. Internal working**

The JIT can inline many lambda calls and eliminate some allocations, but it cannot remove the logical cost of buffering for stateful operations. Parallel streams add fork/join task overhead.

**4. Practical Java example**

```java
import java.util.List;
import java.util.stream.IntStream;

public class StreamPerformanceDemo {

    public static void main(String[] args) {
        List<Integer> values = IntStream.rangeClosed(1, 1000).boxed().toList();

        int boxedSum = values.stream().mapToInt(Integer::intValue).sum();
        int primitiveSum = IntStream.rangeClosed(1, 1000).sum();

        System.out.println(boxedSum);
        System.out.println(primitiveSum);
    }
}
```

Output:

```text
500500
500500
```

Important lines:
- The boxed version unboxes each element with `mapToInt`.
- The primitive version avoids wrapper objects entirely.

**5. Real-world use case**

A metrics computation used `Stream<Double>` over millions of values and caused heavy GC. Replacing it with `DoubleStream` and pre-sized arrays reduced allocation and smoothed latency.

**6. Interview follow-up questions**

- **Are streams always slower than loops?** Not always — for complex pipelines they can be comparable or clearer. Measure.
- **Why avoid side effects in streams?** Execution order and threading are unspecified.
- **Which operations are stateful?** `sorted`, `distinct`, `limit` (in ordered parallel), `groupingBy`.
- **How do you debug a stream?** Use `peek` temporarily or break the pipeline into named steps.
- **How do you avoid boxing?** Use `IntStream`, `LongStream`, `DoubleStream`.

**7. Common mistakes**

- Using streams for a two-element list.
- Mutating a counter inside `forEach` — especially wrong in parallel.
- Chaining too many operations without readability.
- Assuming `map` preserves order in parallel — it does for ordered sources but costs more.

**8. Interview difficulty**

**Advanced**

---

### Question 12: When should you prefer a loop over a stream in backend code?

**1. Interview-ready answer**

I prefer a loop when the operation needs mutation of external state, early exit with complex conditions, index-based access, checked exceptions, or careful error handling per element. Loops are also clearer for tiny collections and critical hot paths where allocation matters. Streams win for declarative transformations, filtering, grouping, and aggregation where readability and composability matter. In backend code I often use streams at the service/reporting layer and plain loops in low-level parsers or performance-sensitive utilities.

**2. Detailed explanation**

Prefer streams for:
- `filter` + `map` + `collect` pipelines.
- Grouping, partitioning, summarizing.
- Composable rules and predicates.

Prefer loops for:
- Building objects with multiple side effects.
- Breaking out on the first complex condition.
- Indexed algorithms and in-place updates.
- Checked exception handling without wrapping.

**3. Internal working**

A loop executes directly in method bytecode; a stream builds a pipeline of sinks and invokes lambdas. The JIT narrows the gap but cannot remove all structural differences.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class LoopVsStreamDemo {

    static List<String> withStream() {
        return SampleData.EMPLOYEES.stream()
                .filter(Employee::active)
                .map(Employee::name)
                .toList();
    }

    static List<String> withLoop() {
        List<String> names = new ArrayList<>();
        for (Employee employee : SampleData.EMPLOYEES) {
            if (!employee.active()) {
                continue;
            }
            if (employee.salary().compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalStateException("Invalid salary for " + employee.id());
            }
            names.add(employee.name());
        }
        return names;
    }

    public static void main(String[] args) {
        System.out.println(withStream());
        System.out.println(withLoop());
    }
}
```

Output:

```text
[Asha, Ravi, Meera, Amit]
[Asha, Ravi, Meera, Amit]
```

Important lines:
- The stream version is concise for a simple filter/map.
- The loop version handles a per-element exception clearly and could break early.

**5. Real-world use case**

A CSV importer uses loops because each row needs row-number-aware error reporting and partial-commit handling. A dashboard service uses streams because its work is pure grouping and aggregation.

**6. Interview follow-up questions**

- **Are streams thread-safe?** They are not shared by design; parallel streams handle splitting, but your lambdas must be stateless.
- **Can you use `break` in a stream?** No — use `findFirst`/`anyMatch` or a loop.
- **How do you handle checked exceptions in streams?** Wrap in a functional interface that declares throws.
- **Which is more testable?** Neither inherently; small pure functions used by either are most testable.
- **Do streams work with databases?** Spring Data streams exist but need careful resource handling; pagination is often safer.

**7. Common mistakes**

- Using streams purely to look modern.
- Hiding exceptions with `try/catch` inside every lambda.
- Believing loops are always faster without measuring.
- Mixing stream and loop styles confusingly in one method.

**8. Interview difficulty**

**Intermediate**

---

---

## Part B — Stream coding questions

### Question 13: Get active employees with salary above 80,000, sorted by name

**1. Interview-ready answer**

I filter twice — once on `active` and once on salary — then sort by name with a comparator and collect to a list. I use `BigDecimal.compareTo` rather than `>` for money.

**2. Detailed explanation**

**Problem statement:** From `SampleData.EMPLOYEES`, return names of active employees earning more than 80,000, alphabetically.

**Approach:** `filter` → `filter` → `sorted` → `map` → `toList`.

**3. Internal working**

Predicates are evaluated element by element; `sorted` buffers the surviving elements before emitting them in order.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.List;

public class Q13HighPaidActive {

    public static List<String> solve() {
        return SampleData.EMPLOYEES.stream()
                .filter(Employee::active)
                .filter(employee -> employee.salary().compareTo(BigDecimal.valueOf(80000)) > 0)
                .sorted(Comparator.comparing(Employee::name))
                .map(Employee::name)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[Amit, Asha]
```

**5. Real-world use case**

An HR dashboard lists eligible employees for a bonus program.

**6. Interview follow-up questions**

- Why compare BigDecimal with `compareTo`? `>` is not defined for objects; `equals` compares scale.
- Can I combine the filters? Yes with `Predicate.and`.
- Is sorting stable? Yes.
- What if salary is null? Filter nulls first or fix the data model.

**7. Common mistakes**

- Using `salary > 80000` on BigDecimal.
- Sorting before filtering — extra work.
- Forgetting the active check.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** Asha (90000, active) ✓; Ravi (60000) ✗; Meera (75000) ✗; Karan inactive ✗; Amit (120000, active) ✓ → sorted `[Amit, Asha]`.

**10. Complexity:** Time O(n log n) due to sorting; space O(n).

**11. Edge cases:** empty input, null salary, all inactive, duplicate names.

---

### Question 14: Count employees per department

**1. Interview-ready answer**

Use `Collectors.groupingBy` with `counting`. I add a `TreeMap` supplier when I want deterministic alphabetical output.

**2. Detailed explanation**

**Problem statement:** Return a map of department name → employee count.

**Approach:** Group by `Employee::department`, downstream `Collectors.counting()`.

**3. Internal working**

`groupingBy` builds a map of maps: the classifier chooses a key, and the downstream collector aggregates values under that key.

**4. Practical Java example**

```java
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Q14DepartmentCounts {

    public static Map<String, Long> solve() {
        return SampleData.EMPLOYEES.stream()
                .collect(Collectors.groupingBy(
                        Employee::department,
                        TreeMap::new,
                        Collectors.counting()));
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
{HR=2, IT=3}
```

**5. Real-world use case**

An admin API returns headcount per department for an organization chart.

**6. Interview follow-up questions**

- How do you count only active employees? Add `filter(Employee::active)`.
- What collector gives a `Map<String, List<Employee>>`? `groupingBy` without a downstream collector.
- Why TreeMap? Deterministic order.
- How do you get `Integer` counts? Downstream `Collectors.summingInt(e -> 1)`.

**7. Common mistakes**

- Using `toMap` with a merge function incorrectly — `groupingBy` is clearer.
- Null department keys cause NPE; handle them explicitly.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** IT → Asha, Ravi, Amit = 3; HR → Meera, Karan = 2.

**10. Complexity:** Time O(n), space O(k) where k = number of departments.

**11. Edge cases:** null department, empty list, one department.

---

### Question 15: Find the top 3 highest-paid employees

**1. Interview-ready answer**

Sort by salary descending and call `limit(3)`. For very large datasets where you only need the top k, a PriorityQueue is more efficient than sorting everything.

**2. Detailed explanation**

**Problem statement:** Return the three employees with the highest salary.

**Approach:** `sorted(comparing(salary).reversed())` then `limit(3)`.

**3. Internal working**

`sorted` is stateful — it consumes all elements before emitting. `limit` then short-circuits.

**4. Practical Java example**

```java
import java.util.Comparator;
import java.util.List;

public class Q15TopPaid {

    public static List<Employee> solve() {
        return SampleData.EMPLOYEES.stream()
                .sorted(Comparator.comparing(Employee::salary).reversed())
                .limit(3)
                .toList();
    }

    public static void main(String[] args) {
        solve().forEach(employee -> System.out.println(employee.name() + " " + employee.salary()));
    }
}
```

Output:

```text
Amit 120000
Asha 90000
Meera 75000
```

**5. Real-world use case**

A leaderboard of top earners or top-selling products.

**6. Interview follow-up questions**

- How do you do this with a heap? Keep a min-heap of size 3.
- Why not `max` three times? O(3n) and repeated work.
- Does `limit` preserve order? Yes for ordered streams.
- How do you break ties? Add `thenComparing(Employee::name)`.

**7. Common mistakes**

- Forgetting `.reversed()`.
- Using `Integer.compare` on BigDecimal.
- Sorting before filtering when both are needed.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** Sorted salaries: 120000, 90000, 75000, 60000, 45000 → first three.

**10. Complexity:** Time O(n log n); space O(n). Heap alternative O(n log k), space O(k).

**11. Edge cases:** fewer than 3 employees, equal salaries, null salaries.

---

### Question 16: Compute total salary using `reduce`

**1. Interview-ready answer**

Map each employee to salary and reduce with `BigDecimal.ZERO` as identity and `BigDecimal::add` as accumulator. Using the two-argument reduce is safe for empty streams and works correctly with parallel streams because BigDecimal is associative.

**2. Detailed explanation**

**Problem statement:** Sum all employee salaries.

**Approach:** `map(Employee::salary).reduce(BigDecimal.ZERO, BigDecimal::add)`.

**3. Internal working**

Reduce combines elements into a single result. With an identity and an associative combiner, parallel reduction can split and merge partial sums.

**4. Practical Java example**

```java
import java.math.BigDecimal;

public class Q16TotalSalary {

    public static BigDecimal solve() {
        return SampleData.EMPLOYEES.stream()
                .map(Employee::salary)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
390000
```

**5. Real-world use case**

Payroll totals and order value aggregation in reports.

**6. Interview follow-up questions**

- What does one-argument `reduce` return? `Optional<BigDecimal>`.
- Why provide an identity? Avoids Optional and handles empty streams.
- Is BigDecimal addition associative? Yes.
- What about `Collectors.reducing`? Useful as a downstream collector.

**7. Common mistakes**

- Using `BigDecimal::add` without identity on an empty stream — returns Optional.
- Using mutable accumulation in parallel streams.
- Using `double` for money.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** 90000+60000+75000+45000+120000 = 390000.

**10. Complexity:** Time O(n), space O(1) sequential.

**11. Edge cases:** empty stream → 0; null salary → NPE.

---

### Question 17: Combine predicates to filter names starting with "A" and active

**1. Interview-ready answer**

Define two `Predicate<Employee>` variables and combine them with `and`, then map to names. This keeps the business rules named and reusable.

**2. Detailed explanation**

**Problem statement:** Return names of active employees whose name starts with "A".

**Approach:** `Predicate.and`, `filter`, `map`, `toList`.

**3. Internal working**

`and` returns a composed predicate that short-circuits: if the first fails, the second is not evaluated.

**4. Practical Java example**

```java
import java.util.List;
import java.util.function.Predicate;

public class Q17PredicateCombo {

    public static List<String> solve() {
        Predicate<Employee> startsWithA = employee -> employee.name().startsWith("A");
        Predicate<Employee> isActive = Employee::active;

        return SampleData.EMPLOYEES.stream()
                .filter(startsWithA.and(isActive))
                .map(Employee::name)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[Asha, Amit]
```

**5. Real-world use case**

Composable fraud rules: predicate per rule, combined with `and`/`or`.

**6. Interview follow-up questions**

- What does `negate()` do? Inverts the predicate.
- Is evaluation short-circuit? Yes.
- How do you combine with OR? `predicate.or(other)`.
- Can predicates be stored in a list? Yes — use `reduce(Predicate::and)`.

**7. Common mistakes**

- Putting side effects in predicates.
- Forgetting that `startsWith` is case-sensitive.
- Null names causing NPE.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** Asha ✓, Ravi ✗, Meera ✗, Karan ✗, Amit ✓.

**10. Complexity:** Time O(n), space O(k) for results.

**11. Edge cases:** null names, empty names, case-insensitive requirement.

---

### Question 18: Build a map of employee ID to name with duplicate handling

**1. Interview-ready answer**

Use `Collectors.toMap` with key `Employee::id`, value `Employee::name`, and a merge function that keeps the first value. A `LinkedHashMap` supplier preserves encounter order.

**2. Detailed explanation**

**Problem statement:** Convert a list of employees into `Map<Long, String>`.

**Approach:** `toMap(keyMapper, valueMapper, mergeFunction, mapSupplier)`.

**3. Internal working**

`toMap` throws `IllegalStateException` on duplicate keys unless a merge function is supplied. The merge function receives the existing and new values.

**4. Practical Java example**

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.stream.Collectors;

public class Q18EmployeeMap {

    public static Map<Long, String> solve() {
        return SampleData.EMPLOYEES.stream()
                .collect(Collectors.toMap(
                        Employee::id,
                        Employee::name,
                        (existing, incoming) -> existing,
                        LinkedHashMap::new));
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
{1=Asha, 2=Ravi, 3=Meera, 4=Karan, 5=Amit}
```

**5. Real-world use case**

Building a lookup map for DTO enrichment without repeated linear scans.

**6. Interview follow-up questions**

- What happens without a merge function and with duplicate keys? `IllegalStateException`.
- How do you keep the last value? `(existing, incoming) -> incoming`.
- Why LinkedHashMap? Predictable order for tests and responses.
- How do you filter null values before mapping? `filter(e -> e.name() != null)`.

**7. Common mistakes**

- Ignoring duplicate keys.
- Using `HashMap` when deterministic order matters.
- Using null keys — `toMap` rejects them.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** Each ID unique → map gets five entries in stream order.

**10. Complexity:** Time O(n), space O(n).

**11. Edge cases:** duplicate IDs, null keys/values, empty stream.

---

### Question 19: Flatten all order items into one list with `flatMap`

**1. Interview-ready answer**

`flatMap` replaces each order with its stream of items and concatenates the results. Use it whenever a nested collection needs to become one sequence.

**2. Detailed explanation**

**Problem statement:** Collect every `OrderItem` from all orders.

**Approach:** `flatMap(order -> order.items().stream())`.

**3. Internal working**

`flatMap` applies a function returning a stream per element and then flattens those streams into the main pipeline. `map` would produce `Stream<List<OrderItem>>` instead.

**4. Practical Java example**

```java
import java.util.List;

public class Q19FlatMapItems {

    public static List<OrderItem> solve() {
        return SampleData.ORDERS.stream()
                .flatMap(order -> order.items().stream())
                .toList();
    }

    public static void main(String[] args) {
        solve().forEach(item -> System.out.println(item.sku() + ":" + item.quantity()));
    }
}
```

Output:

```text
SKU-1:2
SKU-2:1
SKU-3:3
SKU-1:1
```

**5. Real-world use case**

Aggregating all line items across a customer's orders for a spend report.

**6. Interview follow-up questions**

- What is the difference between `map` and `flatMap`? map wraps; flatMap flattens.
- How do you sum line totals after flattening? `map(OrderItem::lineTotal).reduce(ZERO, add)`.
- What if `items()` is null? Filter or default to `List.of()`.
- Can flatMap be used with Optional? Yes — `Optional::stream` flattens optionals.

**7. Common mistakes**

- Using `map` and expecting a flat list.
- Returning null from the flatMap function.
- Losing the parent context when you need order ID — map to a record containing both.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** Order1 → 2 items; Order2 → 1; Order3 → 1 → 4 total.

**10. Complexity:** Time O(total items), space O(total items).

**11. Edge cases:** empty orders, null item lists, duplicate SKUs.

---

### Question 20: Find departments with more than two employees

**1. Interview-ready answer**

Group employees into a count map, then stream the map entries and filter values greater than two. This is a common two-stage pattern: aggregate, then query the aggregation.

**2. Detailed explanation**

**Problem statement:** Return department names with headcount > 2.

**Approach:** `groupingBy` + `counting`, then entry-set stream.

**3. Internal working**

The first stream produces a `Map`; the second stream operates on `Map.Entry` objects.

**4. Practical Java example**

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Q20LargeDepartments {

    public static List<String> solve() {
        Map<String, Long> counts = SampleData.EMPLOYEES.stream()
                .collect(Collectors.groupingBy(Employee::department, Collectors.counting()));

        return counts.entrySet().stream()
                .filter(entry -> entry.getValue() > 2)
                .map(Map.Entry::getKey)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[IT]
```

**5. Real-world use case**

Finding business units that exceed a threshold in an HR dashboard.

**6. Interview follow-up questions**

- How do you include inactive employees only? Filter in the first stream.
- Can this be done in one stream? Not cleanly — a second pass over the map is clearer.
- How do you sort results? Add `.sorted()`.
- What about `Collectors.counting()` vs `summingInt`? counting is simpler.

**7. Common mistakes**

- Filtering before grouping by employee attribute instead of group size.
- Using `==` on Long counts — unbox or use `longValue() > 2`.
- Forgetting `Map.Entry::getKey`.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** counts = {IT=3, HR=2} → filter → IT.

**10. Complexity:** Time O(n + k), space O(k).

**11. Edge cases:** all departments small, empty input, null department.

---

### Question 21: Partition employees by salary threshold

**1. Interview-ready answer**

`Collectors.partitioningBy` is a specialized `groupingBy` with a boolean predicate — it always produces two keys, `true` and `false`, even if one group is empty.

**2. Detailed explanation**

**Problem statement:** Split employees into salary ≥ 80,000 and < 80,000.

**Approach:** `partitioningBy(predicate)`.

**3. Internal working**

The classifier is a `Predicate`, and the map is a `Map<Boolean, List<T>>` (or a downstream result per key).

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Q21PartitionEmployees {

    public static Map<Boolean, List<String>> solve() {
        return SampleData.EMPLOYEES.stream()
                .collect(Collectors.partitioningBy(
                        employee -> employee.salary().compareTo(BigDecimal.valueOf(80000)) >= 0,
                        Collectors.mapping(Employee::name, Collectors.toList())));
    }

    public static void main(String[] args) {
        solve().forEach((key, value) -> System.out.println(key + " -> " + value));
    }
}
```

Output:

```text
false -> [Ravi, Meera, Karan]
true -> [Asha, Amit]
```

**5. Real-world use case**

Splitting transactions into successful and failed groups, or orders into domestic and international.

**6. Interview follow-up questions**

- How is partitioningBy different from groupingBy? Boolean keys; both keys always present.
- Can you partition into more than two groups? No — use groupingBy.
- Can you count per partition? Downstream `Collectors.counting()`.
- Is output order guaranteed? For ordered sources with list collector, yes.

**7. Common mistakes**

- Using groupingBy with a Boolean classifier when partitioningBy fits.
- Assuming missing groups are absent — both keys exist.
- Comparing BigDecimal with `>=`.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** true: Asha 90000, Amit 120000; false: Ravi 60000, Meera 75000, Karan 45000.

**10. Complexity:** Time O(n), space O(n).

**11. Edge cases:** all employees in one partition, empty stream → two empty lists.

---

### Question 22: Find the second-highest distinct salary

**1. Interview-ready answer**

Map to salaries, remove duplicates, sort descending, skip one, and take the first. Using `distinct()` before sorting is important so equal salaries do not occupy two positions.

**2. Detailed explanation**

**Problem statement:** Return the second-highest distinct salary as `Optional<BigDecimal>`.

**Approach:** `map → distinct → sorted(reverse) → skip(1) → findFirst`.

**3. Internal working**

`distinct` uses a HashSet statefully; `sorted` buffers; `findFirst` short-circuits after skip.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.Optional;

public class Q22SecondHighest {

    public static Optional<BigDecimal> solve() {
        return SampleData.EMPLOYEES.stream()
                .map(Employee::salary)
                .distinct()
                .sorted(Comparator.reverseOrder())
                .skip(1)
                .findFirst();
    }

    public static void main(String[] args) {
        System.out.println(solve().orElse(BigDecimal.ZERO));
    }
}
```

Output:

```text
90000
```

**5. Real-world use case**

Finding second-best bid or runner-up score in a ranking system.

**6. Interview follow-up questions**

- What if there is only one distinct salary? Optional is empty.
- Why distinct before sorting? To avoid duplicates occupying top slots.
- Can you do it in one pass? Yes — track top two values in a loop.
- How do you get the nth? `skip(n-1)`.

**7. Common mistakes**

- Forgetting `distinct`.
- Using `findAny` — order matters here.
- Assuming `Optional.get()` is safe.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** distinct sorted = [120000, 90000, 75000, 60000, 45000] → skip 1 → 90000.

**10. Complexity:** Time O(n log n), space O(n).

**11. Edge cases:** fewer than two distinct salaries, duplicates, empty input.

---

### Question 23: Average salary per department

**1. Interview-ready answer**

Use `groupingBy` with the `averagingDouble` downstream collector. For money I would normally sum and divide as BigDecimal, but averagingDouble is fine for reporting statistics.

**2. Detailed explanation**

**Problem statement:** Return department → average salary.

**Approach:** `groupingBy(Employee::department, averagingDouble(salary::doubleValue))`.

**3. Internal working**

The downstream collector maintains a running sum and count per key, then divides in the finisher.

**4. Practical Java example**

```java
import java.util.Map;
import java.util.stream.Collectors;

public class Q23AverageSalary {

    public static Map<String, Double> solve() {
        return SampleData.EMPLOYEES.stream()
                .collect(Collectors.groupingBy(
                        Employee::department,
                        Collectors.averagingDouble(employee -> employee.salary().doubleValue())));
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
{HR=60000.0, IT=90000.0}
```

**5. Real-world use case**

HR analytics and compensation benchmarking dashboards.

**6. Interview follow-up questions**

- How do you get sum, count, min, max, and average together? `summarizingDouble`.
- Why not use double for payroll? It is only for statistics here, not money storage.
- How do you round the output? `Math.round(value * 100.0) / 100.0`.
- Can you average BigDecimal? Use a custom reduce with count.

**7. Common mistakes**

- Using averagingInt and truncating decimals unexpectedly.
- Forgetting doubleValue conversion.
- Including inactive employees unintentionally.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** IT = (90000+60000+120000)/3 = 90000; HR = (75000+45000)/2 = 60000.

**10. Complexity:** Time O(n), space O(k).

**11. Edge cases:** empty department groups cannot occur via groupingBy; null salaries.

---

### Question 24: Filter products by price range and sort by price descending

**1. Interview-ready answer**

Filter with two BigDecimal comparisons and sort with `Comparator.comparing(Product::price).reversed()`. Then map to names or collect products directly.

**2. Detailed explanation**

**Problem statement:** Return products priced between 1,000 and 70,000 inclusive, most expensive first.

**Approach:** `filter` twice → `sorted(reversed)` → `toList`.

**3. Internal working**

Both comparisons use `compareTo`; sorting buffers matching elements.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.List;

public class Q24ProductsInRange {

    public static List<String> solve() {
        BigDecimal min = BigDecimal.valueOf(1000);
        BigDecimal max = BigDecimal.valueOf(70000);

        return SampleData.PRODUCTS.stream()
                .filter(product -> product.price().compareTo(min) >= 0)
                .filter(product -> product.price().compareTo(max) <= 0)
                .sorted(Comparator.comparing(Product::price).reversed())
                .map(Product::name)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[Laptop, Laptop, Monitor]
```

**5. Real-world use case**

Catalog filtering and price sorting in an e-commerce API.

**6. Interview follow-up questions**

- How do you remove duplicate products? `distinct()` on records or by ID with toMap.
- How do you combine range checks in one predicate? Use `and`.
- How do you paginate after sorting? `skip(page * size).limit(size)`.
- Why not use `Double` for price? Precision issues.

**7. Common mistakes**

- Using `>` instead of `>=` for inclusive bounds.
- Sorting ascending by mistake.
- Using `Product::price` with natural order when descending is required.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** Laptop 65000 ✓, Mouse 800 ✗, Monitor 12000 ✓, Laptop 65000 ✓ → sorted desc.

**10. Complexity:** Time O(n log n), space O(n).

**11. Edge cases:** empty range, equal prices, null price.

---

### Question 25: Compute total order value per customer

**1. Interview-ready answer**

I group orders by `customerId` and use a downstream `reducing` collector over each order's `totalValue()`. Mapping each order to its BigDecimal total first keeps the accumulator simple and associative, so it also works with parallel streams.

**2. Detailed explanation**

**Problem statement:** Return `Map<String, BigDecimal>` with the total value of all orders placed by each customer.

**Approach:**
1. `groupingBy(Order::customerId, TreeMap::new, downstream)`.
2. Downstream: `mapping(Order::totalValue, reducing(BigDecimal.ZERO, BigDecimal::add))`.
3. Print each entry for verification.

**3. Internal working**

`groupingBy` partitions orders by customer. For each partition, `mapping` converts the orders to BigDecimal values and `reducing` combines them with addition. The identity `ZERO` handles missing values and keeps parallel combination correct.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Q25OrderTotals {

    public static Map<String, BigDecimal> solve() {
        return SampleData.ORDERS.stream()
                .collect(Collectors.groupingBy(
                        Order::customerId,
                        TreeMap::new,
                        Collectors.mapping(
                                Order::totalValue,
                                Collectors.reducing(BigDecimal.ZERO, BigDecimal::add))));
    }

    public static void main(String[] args) {
        solve().forEach((customer, total) -> System.out.println(customer + "=" + total));
    }
}
```

Output:

```text
CUST-1=3400
CUST-2=1000
```

**5. Real-world use case**

A billing service aggregates order totals per customer for invoices and credit-limit checks.

**6. Interview follow-up questions**

- How do you count only PAID orders? Add `filter(order -> "PAID".equals(order.status()))`.
- What collector gives a list of orders per customer? `groupingBy` without a downstream collector.
- Why not use `double` for totals? BigDecimal avoids floating-point rounding errors.
- How do you round to two decimals? `total.setScale(2, RoundingMode.HALF_UP)`.

**7. Common mistakes**

- Using product price instead of `OrderItem::lineTotal`.
- Using `reduce` without an identity on an empty group.
- Forgetting that `groupingBy` with `TreeMap` requires comparable keys — String is fine.

**8. Interview difficulty**

**Intermediate**

**9. Dry run**

| Order | Customer | Line totals | Order total |
|---|---|---:|---:|
| 1 | CUST-1 | 2000 + 500 | 2500 |
| 2 | CUST-1 | 900 | 900 |
| 3 | CUST-2 | 1000 | 1000 |

CUST-1 = 2500 + 900 = 3400; CUST-2 = 1000.

**10. Complexity:** Time O(n × itemsPerOrder); space O(k), where k is the number of customers.

**11. Edge cases:** empty order list, null customer ID, empty item list, duplicate orders.

---

### Question 26: Find duplicate transaction IDs

**1. Interview-ready answer**

Group transactions by ID with `counting()`, then filter groups whose count is greater than one. This is the standard streams solution for finding duplicates; for huge datasets I would stream to a set and report duplicates as they appear to avoid holding a full count map.

**2. Detailed explanation**

**Problem statement:** Return IDs that occur more than once in `SampleData.TRANSACTIONS`.

**Approach:** `groupingBy(id, counting)` → entry-set stream → `filter(count > 1)` → `map(key)`.

**3. Internal working**

The first stream builds `Map<String, Long>`. The second stream queries that map. Using `TreeMap` gives sorted, deterministic output.

**4. Practical Java example**

```java
import java.util.List;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Q26DuplicateTransactions {

    public static List<String> solve() {
        Map<String, Long> counts = SampleData.TRANSACTIONS.stream()
                .collect(Collectors.groupingBy(
                        Transaction::id,
                        TreeMap::new,
                        Collectors.counting()));

        return counts.entrySet().stream()
                .filter(entry -> entry.getValue() > 1)
                .map(Map.Entry::getKey)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[TX-1]
```

**5. Real-world use case**

Payment systems detect duplicate transaction IDs before posting to the ledger.

**6. Interview follow-up questions**

- How do you return the duplicate records, not just IDs? Collect to `Map<String, List<Transaction>>` and filter list size.
- Can you do it in one pass with a Set? Yes: `stream().filter(t -> !seen.add(t.id())).toList()`.
- Why is the Set approach order-dependent? It reports the second and later occurrences only.
- How do you make it thread-safe? Use a concurrent set or synchronize the collector.

**7. Common mistakes**

- Comparing Long counts with `==` — use `> 1` after unboxing.
- Reporting IDs that occur exactly twice only — use `> 1`.
- Using `distinct()` — it removes duplicates instead of finding them.

**8. Interview difficulty**

**Intermediate**

**9. Dry run**

| Transaction ID | Count |
|---|---:|
| TX-1 | 2 |
| TX-2 | 1 |
| TX-3 | 1 |

Only TX-1 passes the filter.

**10. Complexity:** Time O(n + k); space O(k).

**11. Edge cases:** no duplicates, all duplicates, null IDs, empty stream.

---

### Question 27: Find the first transaction above 5,000

**1. Interview-ready answer**

Filter by amount with `BigDecimal.compareTo` and call `findFirst()`. `findFirst` is a short-circuiting terminal operation, so it stops as soon as a match is found in encounter order.

**2. Detailed explanation**

**Problem statement:** Return the first `Transaction` whose amount is greater than 5,000 as an `Optional<Transaction>`.

**Approach:** `filter` → `findFirst`.

**3. Internal working**

`filter` is stateless. `findFirst` short-circuits; on a parallel stream it still returns the first element in encounter order, though it may evaluate more elements than a sequential stream.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Optional;

public class Q27FirstLargeTransaction {

    public static Optional<Transaction> solve() {
        return SampleData.TRANSACTIONS.stream()
                .filter(transaction -> transaction.amount().compareTo(BigDecimal.valueOf(5000)) > 0)
                .findFirst();
    }

    public static void main(String[] args) {
        solve().ifPresent(transaction -> System.out.println(
                transaction.id() + " " + transaction.amount() + " " + transaction.status()));
    }
}
```

Output:

```text
TX-3 9000 SUCCESS
```

**5. Real-world use case**

Fraud pipelines flag the first transaction above a threshold in a stream of events.

**6. Interview follow-up questions**

- How is `findFirst` different from `findAny`? findFirst respects encounter order; findAny may pick any.
- What does an empty Optional mean here? No transaction exceeded the threshold.
- How do you get the last match? Iterate in reverse or reduce keeping the latest.
- Why not `min`/`max`? We want the first qualifying event, not the largest.

**7. Common mistakes**

- Using `get()` without `isPresent`.
- Using `equals` for BigDecimal comparison.
- Assuming parallel streams make findFirst faster.

**8. Interview difficulty**

**Beginner**

**9. Dry run**

| Transaction | Amount | Result |
|---|---:|---|
| TX-1 | 2500 | ✗ |
| TX-2 | 1500 | ✗ |
| TX-1 | 2500 | ✗ |
| TX-3 | 9000 | ✓ stop |

**10. Complexity:** Time O(n) worst case, O(1) best case; space O(1).

**11. Edge cases:** no match, empty stream, null amount, exact boundary value 5000.

---

### Question 28: Count transactions by status

**1. Interview-ready answer**

Use `Collectors.groupingBy(Transaction::status, Collectors.counting())`. It produces a compact status histogram that can be fed directly into a dashboard response.

**2. Detailed explanation**

**Problem statement:** Count how many transactions are SUCCESS and how many are FAILED.

**Approach:** Group by status with a counting downstream collector.

**3. Internal working**

Each transaction's status classifier selects a bucket; the downstream adder increments the count for that bucket.

**4. Practical Java example**

```java
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Q28TransactionStatusCounts {

    public static Map<String, Long> solve() {
        return SampleData.TRANSACTIONS.stream()
                .collect(Collectors.groupingBy(
                        Transaction::status,
                        TreeMap::new,
                        Collectors.counting()));
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
{FAILED=1, SUCCESS=3}
```

**5. Real-world use case**

A payment dashboard shows success/failure ratios and alerts when failure counts spike.

**6. Interview follow-up questions**

- How do you percentage-normalize this map? Divide each count by the total count.
- How do you handle null status? `groupingBy` rejects null keys; filter or map null to "UNKNOWN".
- How do you count only success? `filter(...).count()`.
- What collector returns counts as Integer? `summingInt(t -> 1)`.

**7. Common mistakes**

- Using `count()` inside groupingBy incorrectly — `count()` is terminal.
- Forgetting TreeMap when deterministic output matters.
- Treating status as case-insensitive without normalizing.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** SUCCESS → TX-1, TX-1, TX-3 = 3; FAILED → TX-2 = 1.

**10. Complexity:** Time O(n), space O(k).

**11. Edge cases:** empty stream, null status, new status values.

---

### Question 29: Convert names to uppercase, remove duplicates, and sort

**1. Interview-ready answer**

Map each employee name to uppercase, call `distinct()`, then `sorted()`. Order matters: uppercase before distinct so "asha" and "Asha" collapse to one value.

**2. Detailed explanation**

**Problem statement:** Return a sorted list of unique uppercase employee names.

**Approach:** `map(String::toUpperCase)` → `distinct` → `sorted` → `toList`.

**3. Internal working**

`distinct` uses a HashSet-backed stateful filter; `sorted` buffers the distinct values. For a locale-safe version, use `toUpperCase(Locale.ROOT)`.

**4. Practical Java example**

```java
import java.util.List;
import java.util.Locale;

public class Q29UniqueUpperNames {

    public static List<String> solve() {
        return SampleData.EMPLOYEES.stream()
                .map(Employee::name)
                .map(name -> name.toUpperCase(Locale.ROOT))
                .distinct()
                .sorted()
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[AMIT, ASHA, KARAN, MEERA, RAVI]
```

**5. Real-world use case**

Normalizing tags, SKUs, or usernames before storing them in a search index.

**6. Interview follow-up questions**

- Why `Locale.ROOT`? Avoids locale-specific conversions such as Turkish `i`.
- What if distinct() comes before uppercase? Case variants survive.
- How do you sort case-insensitively? `sorted(String.CASE_INSENSITIVE_ORDER)`.
- Is distinct() stable for ordered streams? Yes.

**7. Common mistakes**

- Using default locale in production normalization.
- Calling `sorted` before `distinct` — more work.
- Forgetting null checks on names.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** names → uppercase → unique set → sorted alphabetically.

**10. Complexity:** Time O(n log n) due to sorting; space O(n).

**11. Edge cases:** null names, same names in different case, empty input.

---

### Question 30: Filter employees safely when department can be null

**1. Interview-ready answer**

I remove null elements with `filter(Objects::nonNull)` before dereferencing, then filter out null departments explicitly. Streams do not make code null-safe automatically — a null element still causes an NPE inside a lambda.

**2. Detailed explanation**

**Problem statement:** From a mixed list containing null employees and employees with null departments, return the names of employees that have a department.

**Approach:** `Stream.of(...)` → `filter(Objects::nonNull)` → `filter(department != null)` → `map(name)`.

**3. Internal working**

Each predicate runs in order, so the null check must come before `employee.department()`. Predicate ordering is part of correctness, not just style.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.List;
import java.util.Objects;
import java.util.stream.Stream;

public class Q30NullSafeFiltering {

    public static List<String> solve() {
        return Stream.of(
                        new Employee(6L, "Nina", null, BigDecimal.valueOf(50000), true),
                        null,
                        new Employee(7L, "Omar", "IT", BigDecimal.valueOf(55000), true))
                .filter(Objects::nonNull)
                .filter(employee -> employee.department() != null)
                .map(Employee::name)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve());
    }
}
```

Output:

```text
[Omar]
```

**5. Real-world use case**

Cleaning partially populated records from a CSV import before sending them to validation.

**6. Interview follow-up questions**

- What happens if you swap the two filters? NPE on the null employee.
- How do you handle null names too? Add `filter(employee -> employee.name() != null)`.
- Why not use `Optional.ofNullable` inside map? It hides bad data; filter first.
- How do you default null department to "UNASSIGNED"? `map` to a new Employee or DTO.

**7. Common mistakes**

- Assuming streams skip nulls — they do not.
- Dereferencing before null checks.
- Using `Optional` as a field just to avoid validation.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** Nina → department null ✗; null element ✗; Omar → IT ✓.

**10. Complexity:** Time O(n), space O(k).

**11. Edge cases:** all null elements, all null departments, empty stream.

---

### Question 31: Merge two product lists and deduplicate by product ID

**1. Interview-ready answer**

Concatenate both streams and collect with `toMap`. The key is `Product::id`, the value mapper is `Function.identity()`, and the merge function uses `BinaryOperator.maxBy` on quantity so the record with the larger stock wins.

**2. Detailed explanation**

**Problem statement:** Merge `SampleData.PRODUCTS` with an incoming list and keep one product per ID, preferring the higher quantity.

**Approach:** `Stream.concat` → `Collectors.toMap(id, identity, maxBy(quantity), LinkedHashMap::new)`.

**3. Internal working**

`toMap` calls the merge function whenever a key already exists. `maxBy` wraps a comparator in a `BinaryOperator`, returning the larger of the existing and incoming products. The map supplier preserves encounter order.

**4. Practical Java example**

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class Q31MergeProducts {

    public static Map<Long, Product> solve() {
        return Stream.concat(
                        SampleData.PRODUCTS.stream(),
                        Stream.of(
                                new Product(1L, "Laptop", BigDecimal.valueOf(65000), 10),
                                new Product(5L, "Keyboard", BigDecimal.valueOf(2500), 20)))
                .collect(Collectors.toMap(
                        Product::id,
                        Function.identity(),
                        BinaryOperator.maxBy(Comparator.comparingInt(Product::quantity)),
                        LinkedHashMap::new));
    }

    public static void main(String[] args) {
        solve().forEach((id, product) ->
                System.out.println(id + "=" + product.name() + ":" + product.quantity()));
    }
}
```

Output:

```text
1=Laptop:10
2=Mouse:25
3=Monitor:8
4=Laptop:2
5=Keyboard:20
```

**5. Real-world use case**

Merging inventory feeds where the same SKU appears from multiple warehouses.

**6. Interview follow-up questions**

- How do you keep the latest record instead? Add a timestamp field and compare by it.
- What happens without a merge function? `IllegalStateException` on duplicate ID.
- Why `Function.identity()`? The product itself is the value.
- Why LinkedHashMap? Stable order for tests and API output.

**7. Common mistakes**

- Using `distinct()` — records differ by quantity, so duplicates remain.
- Picking the first value when business rule wants the latest.
- Using null IDs — toMap rejects them.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** ID 1 appears twice with quantities 4 and 10 → keep 10; ID 5 is new → inserted.

**10. Complexity:** Time O(n), space O(k).

**11. Edge cases:** duplicate IDs with equal quantity, null IDs, empty incoming list.

---

### Question 32: Sum and summarize product quantities

**1. Interview-ready answer**

For a single total, `mapToInt(Product::quantity).sum()` is the cleanest. For count, min, max, average, and sum together, use `Collectors.summarizingInt`.

**2. Detailed explanation**

**Problem statement:** Compute total units in inventory and a full numeric summary of product quantities.

**Approach:** `mapToInt` + `sum`, and `summarizingInt`.

**3. Internal working**

`mapToInt` avoids boxing compared with `map` to `Integer`. `IntSummaryStatistics` maintains running counters and recomputes the average from sum/count.

**4. Practical Java example**

```java
import java.util.IntSummaryStatistics;
import java.util.stream.Collectors;

public class Q32QuantityStats {

    public static int totalUnits() {
        return SampleData.PRODUCTS.stream()
                .mapToInt(Product::quantity)
                .sum();
    }

    public static IntSummaryStatistics stats() {
        return SampleData.PRODUCTS.stream()
                .collect(Collectors.summarizingInt(Product::quantity));
    }

    public static void main(String[] args) {
        System.out.println("total=" + totalUnits());
        System.out.println("count=" + stats().getCount()
                + " sum=" + stats().getSum()
                + " min=" + stats().getMin()
                + " max=" + stats().getMax()
                + " average=" + stats().getAverage());
    }
}
```

Output:

```text
total=39
count=4 sum=39 min=2 max=25 average=9.75
```

**5. Real-world use case**

Inventory reports show total stock, largest stocked item, and average quantity per SKU.

**6. Interview follow-up questions**

- Why mapToInt instead of map? Avoids boxing overhead.
- What collector summarizes BigDecimal? No built-in equivalent; use a custom accumulator.
- How do you sum line totals? `map(OrderItem::lineTotal).reduce(ZERO, BigDecimal::add)`.
- What does getMin() return for empty stats? Integer.MAX_VALUE — check count first.

**7. Common mistakes**

- Using `Integer` accumulation in a mutable external variable.
- Ignoring empty statistics.
- Using `average()` on boxed Stream<Integer> — mapToInt is clearer.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** quantities = 4, 25, 8, 2 → sum 39, min 2, max 25, average 39/4 = 9.75.

**10. Complexity:** Time O(n), space O(1).

**11. Edge cases:** empty products, negative quantities, single product.

---

### Question 33: Find the highest-paid employee in each department

**1. Interview-ready answer**

Group by department and use `toMap` with a `maxBy` merge function comparing salaries. This is the natural collector-based version of a SQL `GROUP BY` with a per-group maximum.

**2. Detailed explanation**

**Problem statement:** Return `Map<String, Employee>` where each department maps to its highest-paid employee.

**Approach:** `Collectors.toMap(department, identity, maxBy(salary), TreeMap::new)`.

**3. Internal working**

The first employee for a department is inserted. For later employees with the same department, the merge function compares the existing and incoming salary and keeps the larger.

**4. Practical Java example**

```java
import java.util.Comparator;
import java.util.Map;
import java.util.TreeMap;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.stream.Collectors;

public class Q33TopPerDepartment {

    public static Map<String, Employee> solve() {
        return SampleData.EMPLOYEES.stream()
                .collect(Collectors.toMap(
                        Employee::department,
                        Function.identity(),
                        BinaryOperator.maxBy(Comparator.comparing(Employee::salary)),
                        TreeMap::new));
    }

    public static void main(String[] args) {
        solve().forEach((department, employee) ->
                System.out.println(department + "=" + employee.name() + ":" + employee.salary()));
    }
}
```

Output:

```text
HR=Meera:75000
IT=Amit:120000
```

**5. Real-world use case**

A compensation report shows the top earner per business unit.

**6. Interview follow-up questions**

- How do you keep all ties? Use `groupingBy` to lists, then filter by max salary.
- How do you restrict to active employees? `filter(Employee::active)` before collect.
- Can you use `groupingBy` with `maxBy`? Yes — `groupingBy(dept, maxBy(comparing(salary)))` returns `Map<String, Optional<Employee>>`.
- Why TreeMap? Alphabetical output.

**7. Common mistakes**

- Using `minBy` accidentally.
- Forgetting the merge function — duplicate departments throw.
- Assuming maxBy returns Optional here — in toMap it returns an Employee.

**8. Interview difficulty**

**Intermediate**

**9. Dry run:** IT: Asha 90000 vs Ravi 60000 → Asha; vs Amit 120000 → Amit. HR: Meera 75000 vs Karan 45000 → Meera.

**10. Complexity:** Time O(n), space O(k).

**11. Edge cases:** empty stream, equal salaries, null department, null salary.

---

### Question 34: Paginate employees with `skip` and `limit`

**1. Interview-ready answer**

Sort by a stable key such as ID first, then `skip((long) page * size)` and `limit(size)`. Sorting before pagination is essential — skipping an unsorted stream gives non-repeatable pages.

**2. Detailed explanation**

**Problem statement:** Return page 1 (zero-based) of employees with page size 2, ordered by ID.

**Approach:** `sorted(id)` → `skip` → `limit` → `map(name)`.

**3. Internal working**

`sorted` buffers all elements. `skip` discards the first offset elements, and `limit` caps output. Casting to `long` prevents int overflow for large page offsets.

**4. Practical Java example**

```java
import java.util.Comparator;
import java.util.List;

public class Q34Pagination {

    public static List<String> solve(int page, int size) {
        if (page < 0) {
            throw new IllegalArgumentException("page must not be negative");
        }
        if (size <= 0) {
            throw new IllegalArgumentException("size must be positive");
        }

        return SampleData.EMPLOYEES.stream()
                .sorted(Comparator.comparing(Employee::id))
                .skip((long) page * size)
                .limit(size)
                .map(Employee::name)
                .toList();
    }

    public static void main(String[] args) {
        System.out.println(solve(1, 2));
    }
}
```

Output:

```text
[Meera, Karan]
```

**5. Real-world use case**

REST endpoints with `?page=1&size=2`; in production, push pagination into SQL with LIMIT/OFFSET rather than loading all rows.

**6. Interview follow-up questions**

- Why sort first? Without a deterministic order, pages overlap or omit rows.
- What happens past the last page? Empty list.
- How do you return total elements too? Count separately or use a database count query.
- Is skip efficient on JDBC streams? No — prefer keyset pagination for deep pages.

**7. Common mistakes**

- Using `(page * size)` as int — overflow risk.
- Forgetting validation.
- Using an unsorted source.
- Confusing page numbers: page 1 skips the first page.

**8. Interview difficulty**

**Beginner**

**9. Dry run:** sorted names are Asha(1), Ravi(2), Meera(3), Karan(4), Amit(5). Page 1 with size 2 skips 2 → Meera, Karan.

**10. Complexity:** Time O(n log n), space O(n) due to sorting; skip/limit themselves are O(offset + size).

**11. Edge cases:** page 0, page beyond end, negative page, zero size, unsorted source.

---

---

## Chapter 5 Verification

| Syllabus topic | Question(s) |
|---|---|
| Lambda expressions and functional interfaces | Q1, Q2, Q3, Q4, Q5, Q17 |
| Default/static methods and diamond conflict | Q6 |
| Optional | Q7 |
| Date and Time API | Q8 |
| Stream fundamentals and pipelines | Q9, Q12 |
| Parallel streams, performance, and limitations | Q10, Q11 |
| `map`, `filter`, `reduce`, `flatMap` | Q9, Q13, Q16, Q17, Q19, Q24, Q29, Q32 |
| Sorting, grouping, and partitioning | Q13, Q14, Q15, Q18, Q20, Q21, Q23, Q25, Q26, Q28, Q30, Q33 |
| Collectors and `toMap` | Q14, Q18, Q21, Q23, Q25, Q26, Q28, Q31, Q33 |
| Employee/Department/Product/Order/Transaction coding | Q13–Q34 |
| Pagination and ordering | Q15, Q24, Q29, Q34 |
| Edge cases and interview trade-offs | Q7, Q11, Q12, Q13–Q34 |

**Total interview questions: 34** — 12 conceptual + 22 Stream coding questions. All follow the required 8-part format; coding questions include problem statement, approach, Java solution, dry run, time complexity, space complexity, and edge cases.

**Next:** [Chapter 6 — JVM, Memory and Garbage Collection](06-JVM-Memory-and-Garbage-Collection.md). Say `NEXT CHAPTER` to continue.
