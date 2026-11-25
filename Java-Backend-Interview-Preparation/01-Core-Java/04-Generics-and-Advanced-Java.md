# Chapter 4: Generics and Advanced Java Concepts

**Chapter goal:** Master generic type safety, erasure, wildcards, ordering, reflection, annotations, serialization, cloning, and copy semantics — all common "advanced Java" filter topics for backend interviews.

**Topics covered:** Generics · generic classes and methods · bounded type parameters · wildcards (`?`, `? extends`, `? super`) · PECS · type erasure · generic arrays and varargs · Comparable vs Comparator · Reflection API · annotations · Java serialization · cloning · shallow vs deep copy · output-based and coding questions.

**Cross-references:** Immutability and value objects are in [Chapter 2, Q9 and Q17](02-Strings-and-Object-Handling.md). Records are introduced here and used throughout the book. Streams and collectors that heavily use generics are in [Chapter 5](05-Java-8-Features-and-Streams.md); Spring's use of reflection and annotations is in [Chapter 9](../04-Spring/09-Spring-Framework.md).

---

### Question 1: What are generics in Java and why do they exist?

**1. Interview-ready answer**

Generics let you parameterize a class, interface, or method with a type, so the same code can work with many types while retaining compile-time type safety. Before Java 5, collections stored `Object`, so every retrieval needed a cast and a wrong cast failed at runtime. With `List<String>`, the compiler knows the element type, inserts no visible casts in source, and reports mistakes at compile time. Generics were added mainly for collections, but they now underpin APIs such as `Optional<T>`, `CompletableFuture<T>`, Spring's `ResponseEntity<T>`, and repositories like `JpaRepository<Order, Long>`. The main caveat is type erasure: most type information is removed at runtime.

**2. Detailed explanation**

Benefits:
- **Compile-time type safety:** `list.add(42)` fails for `List<String>`.
- **No explicit casts:** `String s = list.get(0)`.
- **Reusability:** one `Box<T>` for `Box<String>`, `Box<Integer>`, etc.
- **Better API contracts:** the signature documents what goes in and comes out.

Costs:
- Type erasure limits runtime checks.
- Generics are invariant — `List<String>` is not a `List<Object>`.
- Cannot use primitives directly; use wrappers.

Why it exists: to move type errors from runtime to compile time without duplicating code for every type — the same motivation as templates in C++, but implemented differently.

**3. Internal working**

The compiler erases type parameters: `T` becomes `Object`, or its first bound if bounded, such as `Number`. It inserts casts where needed at call sites. The class file contains generic signature metadata in the `Signature` attribute, which reflection and tools can read, but the JVM itself mostly sees erased types. A classic example: `class Box<T> { T value; }` becomes `class Box { Object value; }` at runtime.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class GenericsIntro {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>();
        orderIds.add("ORD-1");
        String first = orderIds.get(0);

        for (String id : orderIds) {
            System.out.println(id.toUpperCase());
        }
    }
}
```

Output:

```text
ORD-1
```

Important lines:
- `List<String>` prevents adding an `Integer`.
- `orderIds.get(0)` needs no cast.
- The enhanced for loop is itself generic-safe.

**5. Real-world use case**

A Spring controller returns `ResponseEntity<OrderDto>`; Jackson serializes the DTO based on that type. A JPA repository declared as `JpaRepository<Order, Long>` guarantees that `findById` accepts a `Long` and returns an `Optional<Order>`, catching type mistakes at compile time rather than in production.

**6. Interview follow-up questions**

- **Are generics available at runtime?** Mostly no — erased, except for signature metadata.
- **Why can't you use `List<int>`?** Type arguments must be reference types; use `List<Integer>`.
- **What is a raw type?** Using `List` without a type argument; it disables type checks and produces warnings.
- **Do generics improve performance?** Not directly; they avoid casts, but the JIT removes most cast costs anyway.
- **How do generics help with Spring repositories?** They bind domain type and ID type at compile time.

**7. Common mistakes**

- Using raw types in new code.
- Suppressing unchecked warnings instead of fixing the type design.
- Assuming `List<String>` and `List<Integer>` share runtime type information.
- Expecting `new T()` to compile.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 2: What is type erasure? What consequences does it have?

**1. Interview-ready answer**

Type erasure is how Java implements generics: the compiler removes type parameters and replaces them with their bounds — `Object` for unbounded parameters, the first bound for bounded ones — and inserts casts where needed. This keeps bytecode compatible with pre-generics JVMs but means the runtime does not know whether a `List` is a `List<String>` or `List<Integer>`. Consequences: you cannot use `instanceof List<String>`, cannot create generic arrays like `new T[10]`, cannot overload methods whose only difference is the generic type argument, and generic varargs cause unchecked warnings. The compiler may generate bridge methods to preserve polymorphism when a generic method is overridden with a concrete type.

**2. Detailed explanation**

Erasure rules:
- `T` → `Object` when unbounded.
- `T extends Number` → `Number`.
- `T extends Number & Comparable<T>` → `Number` (first bound).
- Type parameters in method signatures are erased; return types too.
- Generic type arguments in fields are erased; metadata remains only in `Signature` attributes.

Consequences in detail:
1. `list instanceof List<String>` is a compile error; only `list instanceof List` is allowed.
2. `T[] array = new T[10]` is illegal — use `Object[]` and cast, or `Array.newInstance`.
3. `void add(List<String> l)` and `void add(List<Integer> l)` have the same erased signature — compile error.
4. `@Override` of `T get()` with `String get()` generates a bridge method returning `Object` and casting to `String`.
5. Generic varargs like `List<String>... lists` produce "unchecked generic array creation" warnings.

**3. Internal working**

Consider:

```java
class Box<T> {
    private T value;
    void set(T value) { this.value = value; }
    T get() { return value; }
}
```

Erased form conceptually:

```java
class Box {
    private Object value;
    void set(Object value) { this.value = value; }
    Object get() { return value; }
}
```

At the call site `String s = box.get();`, the compiler inserts a checkcast to `String`.

**4. Practical Java example — output-based**

```java
import java.util.ArrayList;
import java.util.List;

public class ErasureDemo {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        List<Integer> numbers = new ArrayList<>();

        System.out.println(strings.getClass() == numbers.getClass());
        System.out.println(strings instanceof List);

        Class<?> listClass = strings.getClass();
        System.out.println(listClass.getTypeParameters().length);
    }
}
```

Output:

```text
true
true
0
```

Reasoning: both lists are `ArrayList` at runtime; `getTypeParameters()` on the runtime class returns 0 because `ArrayList.class` is raw — generic parameters belong to the declaration, not the instance.

**5. Real-world use case**

A framework tried to inspect a field declared as `List<OrderDto>` with reflection, expecting to discover `OrderDto`. It could only read generic metadata through `Field.getGenericType()`, not `field.getType()`. Spring uses exactly that richer metadata for dependency resolution and JSON mapping.

**6. Interview follow-up questions**

- **What is a bridge method?** A compiler-generated method that preserves overriding after erasure changes signatures.
- **Why can't you overload on generic type arguments?** They erase to the same signature.
- **How do you create a generic array safely?** Use `Array.newInstance(componentType, size)` or `toArray(new T[0])` with a Class token.
- **Does `List<String>` carry any runtime type info?** Only in signature metadata; the object itself does not.
- **How does Gson/Jackson know the element type?** Via `TypeToken`/`ParameterizedTypeReference` capturing generic metadata.

**7. Common mistakes**

- Using `instanceof List<String>`.
- Assuming runtime type checks validate generic arguments.
- Creating generic arrays with casts and then wondering about `ArrayStoreException`.
- Overloading methods only by type argument.

**8. Interview difficulty**

**Advanced**

---

### Question 3: How do you write generic classes and generic methods?

**1. Interview-ready answer**

A generic class declares type parameters in angle brackets after the class name, such as `class ApiResponse<T>`, and uses `T` in fields and method signatures. A generic method declares its own type parameter before the return type, such as `<T> T first(List<T> items)`. The compiler infers the type from arguments or the assignment target. Static methods cannot use a class's type parameter, so they must declare their own. Generic classes are ideal for containers and wrappers; generic methods are ideal for utilities that operate across unrelated types.

**2. Detailed explanation**

Naming conventions: `T` type, `E` element, `K` key, `V` value, `R` result. Use meaningful names when several parameters appear.

Generic class:
```java
class Box<T> { private T value; ... }
```

Generic method:
```java
static <T> List<T> singleton(T value) { ... }
```

Type inference uses the target type (Java 8+) and argument types. If inference fails, supply explicit type arguments: `Collections.<String>emptyList()`.

**3. Internal working**

Type parameters are erased as described in Q2. At the call site, the compiler chooses the inferred type and inserts casts. Generic methods can be recursive and bounded, and their type parameters are local to the method.

**4. Practical Java/Spring Boot example**

```java
import java.util.List;

final class ApiResponse<T> {

    private final boolean success;
    private final T data;
    private final String message;

    private ApiResponse(boolean success, T data, String message) {
        this.success = success;
        this.data = data;
        this.message = message;
    }

    static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>(true, data, null);
    }

    static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, null, message);
    }

    boolean isSuccess() {
        return success;
    }

    T data() {
        return data;
    }

    String message() {
        return message;
    }
}

public class GenericMethodsDemo {

    static <T> T firstOrNull(List<T> values) {
        return values.isEmpty() ? null : values.get(0);
    }

    public static void main(String[] args) {
        ApiResponse<String> response = ApiResponse.ok("ORD-1");
        System.out.println(response.isSuccess() + " : " + response.data());

        ApiResponse<String> failure = ApiResponse.error("not found");
        System.out.println(failure.isSuccess() + " : " + failure.message());

        System.out.println(firstOrNull(List.of(10, 20)));
    }
}
```

Output:

```text
true : ORD-1
false : not found
10
```

Important lines:
- `ApiResponse.<String>ok(...)` type is inferred from the assignment target.
- `error` has no data argument, so the type comes from the target variable.
- `firstOrNull` works for any `List<T>`.

**5. Real-world use case**

A Spring service wraps every controller result in `ApiResponse<T>` so clients get a consistent envelope with `success`, `data`, and `message`. The same class carries `OrderDto`, `List<FlightDto>`, or `PaymentReceipt` without duplicating response classes.

**6. Interview follow-up questions**

- **Can a static method use the class's `T`?** No — `T` belongs to instances.
- **Can you have multiple type parameters?** Yes: `class Pair<K, V>`.
- **What is diamond inference?** `new ApiResponse<>(...)` lets Java 7+ infer arguments from context.
- **Can a generic method be overloaded?** Yes, if erased parameters differ.
- **When should you avoid adding a type parameter?** When one `Object` or a sealed set of types is enough.

**7. Common mistakes**

- Declaring `static T ...` inside a generic class.
- Using raw types in helper methods.
- Overcomplicating utilities with type parameters that add no safety.
- Forgetting that `null` can be a valid `T`.

**8. Interview difficulty**

**Intermediate**

---

### Question 4: What are bounded type parameters?

**1. Interview-ready answer**

A bounded type parameter restricts what types can be supplied: `<T extends Number>` means `T` must be `Number` or a subclass. You can also require multiple bounds with `<T extends Number & Comparable<T>>`, where a class bound must come first and interfaces follow. Bounds matter because erasure replaces `T` with the first bound, so bounded methods can call methods declared by that bound — for example `value.doubleValue()` when `T extends Number`. In backend code, bounds are common in generic services such as `abstract class BaseService<T extends BaseEntity>`.

**2. Detailed explanation**

Kinds of bounds:
- Upper bound: `<T extends Number>`.
- Multiple bounds: `<T extends Number & Comparable<T>>`.
- Interface bound: `<T extends Closeable>`.
- Recursive bound: `<T extends Comparable<T>>` (used by `Collections.sort`).

Why it exists: to express constraints while keeping code generic. Without bounds, the only methods you can call on `T` are those on `Object`.

Advantages: compile-time constraints, access to bound methods, better erasure type. Disadvantage: complex bounds reduce readability.

**3. Internal working**

Erasure uses the leftmost bound. With `<T extends Number & Comparable<T>>`, erased `T` is `Number`; the compiler inserts casts to `Comparable` where needed. Multiple bounds are why the class bound must be listed first — the JVM supports only one erased supertype.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.List;

abstract class BaseEntity {
    private final Long id;

    BaseEntity(Long id) {
        this.id = id;
    }

    Long id() {
        return id;
    }
}

final class Order extends BaseEntity {
    Order(Long id) {
        super(id);
    }
}

abstract class BaseService<T extends BaseEntity> {

    abstract List<T> findAll();

    void logIds() {
        findAll().forEach(entity -> System.out.println("Entity id: " + entity.id()));
    }
}

final class OrderService extends BaseService<Order> {

    @Override
    List<Order> findAll() {
        return List.of(new Order(1L), new Order(2L));
    }
}

public class BoundedTypesDemo {

    static <T extends Number> double sumAsDouble(List<T> numbers) {
        return numbers.stream().mapToDouble(Number::doubleValue).sum();
    }

    public static void main(String[] args) {
        new OrderService().logIds();
        System.out.println(sumAsDouble(List.of(10, 20.5, BigDecimal.valueOf(5))));
    }
}
```

Output:

```text
Entity id: 1
Entity id: 2
35.5
```

Important lines:
- `T extends BaseEntity` lets the base service call `id()` without knowing the concrete entity.
- `Number::doubleValue` is legal because the bound guarantees the method exists.
- `BigDecimal` and `Double` are both `Number` subclasses.

**5. Real-world use case**

A Spring admin framework defines `CrudService<T extends BaseEntity, ID extends Serializable>` so all services share id-based lookup and audit logic. Trying to instantiate it with a DTO that lacks an ID is a compile-time error.

**6. Interview follow-up questions**

- **What is the erasure of `T extends Number & Comparable<T>`?** `Number`.
- **Can a bound be a final class like `String`?** Technically yes, but then the generic is not flexible — just use `String`.
- **Can you use `super` in a type parameter bound?** No; `super` is for wildcards, not type parameters.
- **Why must the class bound come first?** JVM erasure supports a single supertype.
- **What is a recursive bound?** `<T extends Comparable<T>>`, requiring `T` to be comparable to itself.

**7. Common mistakes**

- Trying to call methods not declared by the bound.
- Putting an interface before the class in multiple bounds.
- Using `T super Number` — not allowed.
- Overusing bounds where a plain interface dependency would be clearer.

**8. Interview difficulty**

**Intermediate**

---

### Question 5: Explain wildcard types and the PECS rule.

**1. Interview-ready answer**

Wildcards let a generic type be unknown or constrained. `List<?>` means a list of some unknown type. `List<? extends Number>` is a producer: you can read `Number` values but cannot add anything except `null`, because the actual type might be `Integer` or `Double`. `List<? super Integer>` is a consumer: you can safely add `Integer` values but can only read `Object`. The rule is **PECS — Producer Extends, Consumer Super**: use `extends` when you only read from the parameter, `super` when you only write to it, and an exact type `List<T>` when you do both.

**2. Detailed explanation**

| Type | Can read as | Can add |
|---|---|---|
| `List<?>` | `Object` | only `null` |
| `List<? extends Number>` | `Number` | only `null` |
| `List<? super Integer>` | `Object` | `Integer` and subtypes |
| `List<Number>` | `Number` | `Number` and subtypes |

Why PECS exists: it preserves type safety. A `List<? extends Number>` could actually be a `List<Double>`; adding an `Integer` would corrupt it. A `List<? super Integer>` could be a `List<Object>` or `List<Number>`, both of which accept an `Integer`.

**3. Internal working**

Wildcards are compile-time constructs and are erased like other generic types. The compiler tracks capture conversion: reading from `List<? extends Number>` yields a value of type `Number`. There is no runtime wildcard object.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class WildcardDemo {

    static double sum(List<? extends Number> numbers) {
        return numbers.stream().mapToDouble(Number::doubleValue).sum();
    }

    static void addIntegers(List<? super Integer> destination) {
        destination.add(10);
        destination.add(20);
    }

    static void copy(List<? extends Number> source, List<? super Number> destination) {
        source.forEach(destination::add);
    }

    public static void main(String[] args) {
        System.out.println(sum(List.of(10, 20.5, BigDecimal.valueOf(5))));

        List<Object> objects = new ArrayList<>();
        addIntegers(objects);
        System.out.println(objects);

        List<Number> numbers = new ArrayList<>();
        copy(List.of(1, 2.5), numbers);
        System.out.println(numbers);
    }
}
```

Output:

```text
35.5
[10, 20]
[1, 2.5]
```

Important lines:
- `sum` only reads → `? extends Number`.
- `addIntegers` only writes → `? super Integer`.
- `copy` reads Number and writes to something that accepts Number.

**5. Real-world use case**

A reporting utility accepts `List<? extends OrderBase>` so it can process `List<Order>` and `List<ReturnOrder>` without mutating them. A mapper method accepts `List<? super OrderDto>` so it can write results into `List<Object>` in a generic pipeline.

**6. Interview follow-up questions**

- **What does `List<?>` mean?** A list of unknown type; read as Object, add only null.
- **Why can't you add to `List<? extends Number>`?** The actual type could be `Double` or `BigDecimal`.
- **When do you use an exact `List<T>`?** When reading and writing.
- **Can wildcards be used in method return types?** Yes, but usually avoid them — prefer a concrete type parameter.
- **How does PECS appear in the JDK?** `Collections.copy(List<? super T>, List<? extends T>)`.

**7. Common mistakes**

- Using `? extends` when you need to add elements.
- Using `? super` when you need to read a specific type.
- Returning wildcard types from APIs, making callers fight capture conversion.
- Confusing `List<?>` with `List<Object>` — they are different.

**8. Interview difficulty**

**Advanced**

---

### Question 6: What is the difference between `List<?>`, `List<Object>`, and `<T> void method(List<T>)`?

**1. Interview-ready answer**

`List<?>` is a list of some unknown type — you can read elements as `Object` but cannot add anything except `null`. `List<Object>` is a concrete list that can hold any object, so you can add anything. A generic method `<T> void method(List<T> list)` ties the type to the argument and stays consistent across parameters, letting you read and write elements of type `T` safely. Use `List<?>` when the type genuinely does not matter (e.g., `size()` or `clear()`), `List<Object>` when you intentionally want a heterogeneous container, and a generic method when the type matters but can vary.

**2. Detailed explanation**

Comparison:
- `List<?>` — unknown but fixed type; type-safe reading as Object.
- `List<Object>` — known type Object; accepts all references.
- `<T> void m(List<T> a, List<T> b)` — same T across both arguments; compiler rejects mismatched types.
- `void m(List<?> a, List<?> b)` — each list may have a different unknown type.

Why it matters: interviewers use this to test whether you understand generics are invariant and wildcards represent *unknown*, not *any*.

**3. Internal working**

All three erase to `List`. The difference exists only in the compiler's type checker. Wildcard capture assigns a fresh type variable internally when needed.

**4. Practical Java example**

```java
import java.util.ArrayList;
import java.util.List;

public class WildcardVsGenericMethod {

    static void printAll(List<?> values) {
        values.forEach(value -> System.out.println(value.getClass().getSimpleName()));
    }

    static <T> void moveLast(List<T> source, List<T> destination) {
        if (!source.isEmpty()) {
            destination.add(source.remove(source.size() - 1));
        }
    }

    static void addToUnknown(List<?> values) {
        values.add(null);
    }

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>(List.of("ORD-1"));
        List<String> target = new ArrayList<>();

        printAll(strings);
        moveLast(strings, target);
        System.out.println(target);
        addToUnknown(strings);
        System.out.println(strings.size());
    }
}
```

Output:

```text
String
[ORD-1]
1
```

Important lines:
- `moveLast` requires both lists to share the same `T`.
- `addToUnknown` can add only `null` — adding a String would not compile.
- `strings` still has one element because `null` was appended after the print of `target`.

Actually after `moveLast`, `strings` is empty; then `addToUnknown` adds null, so size is 1. Correct.

**5. Real-world use case**

A validation framework has `void validate(List<?> values)` because it only checks sizes and nulls. A mapping framework uses `<S, T> List<T> map(List<S> source, Function<? super S, ? extends T> mapper)` so source and result types stay correlated.

**6. Interview follow-up questions**

- **Is `List<?>` a supertype of `List<String>`?** Yes — `List<String>` can be assigned to `List<?>`.
- **Is `List<Object>` a supertype of `List<String>`?** No — generics are invariant.
- **Can you add null to `List<?>`?** Yes.
- **Why use a generic method rather than `List<?>` when copying?** To guarantee compatible element types.
- **What does `? super T` allow you to read?** Only `Object`.

**7. Common mistakes**

- Treating `List<?>` as "list of anything you can add to".
- Using `List<Object>` to accept any generic list — it will not compile.
- Returning `List<?>` from public APIs without documenting it.
- Trying to call `T`-specific methods on a `?` capture.

**8. Interview difficulty**

**Advanced**

---

### Question 7: Why can't you create generic arrays, and how do you work around it?

**1. Interview-ready answer**

Java arrays are covariant and carry their component type at runtime, while generics are erased. Creating `new T[10]` would compile to `new Object[10]`, which could then be misused as a `String[]` and cause an `ArrayStoreException` at runtime — so the compiler forbids it. Similarly, `new List<String>[10]` is illegal. Workarounds include using a `Class<T>` token with `Array.newInstance`, using `List<T>` instead of arrays, or using collection `toArray(new T[0])` where the runtime component type is supplied. Generic varargs produce unchecked warnings for the same reason.

**2. Detailed explanation**

Arrays are covariant: `String[]` is an `Object[]`. Generics are invariant: `List<String>` is not `List<Object>`. This mismatch is the root of the problem.

If generic arrays were allowed:

```java
List<String>[] arrays = new List<String>[1];
Object[] raw = arrays;
raw[0] = List.of(42);
String value = arrays[0].get(0);
```

Erasure would make this type-unsafe and fail only at runtime.

Safe approaches:
- `List<T>` instead of `T[]`.
- `T[] array = (T[]) Array.newInstance(componentType, size);` with a `Class<T>` token.
- `list.toArray(new String[0])` for concrete types.

**3. Internal working**

`Array.newInstance(Class<?> componentType, int length)` creates an array with the correct runtime component type. The cast to `T[]` is unchecked but safe if the token matches `T`.

**4. Practical Java/Spring Boot example**

```java
import java.lang.reflect.Array;
import java.util.List;

public class GenericArrayDemo {

    static <T> T[] toArray(List<T> values, Class<T> componentType) {
        @SuppressWarnings("unchecked")
        T[] array = (T[]) Array.newInstance(componentType, values.size());
        return values.toArray(array);
    }

    public static void main(String[] args) {
        String[] result = toArray(List.of("ORD-1", "ORD-2"), String.class);
        System.out.println(result[0] + " / " + result.length);
    }
}
```

Output:

```text
ORD-1 / 2
```

Important lines:
- `Class<T>` supplies the runtime component type erased generics cannot provide.
- `values.toArray(array)` fills the correctly typed array.
- The suppression is justified because the token guarantees the type.

**5. Real-world use case**

A generic Excel export utility needs a typed array for a third-party library that only accepts arrays. It passes `OrderDto.class` from the caller so the framework can build the correct array without heap pollution.

**6. Interview follow-up questions**

- **Why are arrays covariant but generics invariant?** Arrays were designed before generics; covariance was chosen for polymorphism, creating this mismatch.
- **What is heap pollution?** A variable of a parameterized type referring to an object of a different type, possible with unchecked casts/varargs.
- **What does `@SafeVarargs` do?** Suppresses warnings for methods whose varargs are safe; allowed on final/static/private methods.
- **Can you use `Object[]` internally and cast on read?** Yes, with care and usually a suppression.
- **Why prefer `List<T>`?** It is type-safe at compile time and flexible at runtime.

**7. Common mistakes**

- Casting `Object[]` to `String[]` without ensuring the actual component type.
- Ignoring unchecked warnings on generic varargs.
- Believing `new ArrayList<String>[5]` works with a cast.
- Using arrays when a list would be simpler.

**8. Interview difficulty**

**Advanced**

---

### Question 8: Why is `List<String>` not assignable to `List<Object>`?

**1. Interview-ready answer**

Because generics are invariant: `String` being a subclass of `Object` does not make `List<String>` a subtype of `List<Object>`. If it were, you could add an `Integer` through the `List<Object>` reference and break the original `List<String>`, causing a runtime failure when reading. Java chose compile-time safety over this convenience. To accept both, use `List<?>` for unknown types, `List<? extends Object>` (equivalent to `List<?>`) for reading, or `List<? super String>` when you only add strings and subtypes.

**2. Detailed explanation**

This is the generics version of the Liskov Substitution Principle: a subtype must not allow operations that violate the supertype's contract. `List<String>` cannot honor `List<Object>`'s promise to accept any object.

Comparison:
- Arrays: `String[]` **is** an `Object[]` (covariant, but can throw `ArrayStoreException`).
- Generics: `List<String>` **is not** a `List<Object>` (invariant, type safe).

**3. Internal working**

At runtime both are `ArrayList`, but the compiler rejects the assignment. The type safety is entirely compile-time.

**4. Practical Java example**

```java
import java.util.ArrayList;
import java.util.List;

public class InvarianceDemo {

    static void printUnknown(List<?> values) {
        values.forEach(System.out::println);
    }

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>();
        orderIds.add("ORD-1");

        printUnknown(orderIds);

        Object[] array = orderIds.toArray();
        System.out.println(array[0]);

        System.out.println(orderIds instanceof List);
    }
}
```

Output:

```text
ORD-1
ORD-1
true
```

Important lines:
- `printUnknown` accepts `List<String>` because `List<?>` is a supertype.
- `toArray()` returns `Object[]`, showing array covariance in action.
- The attempted assignment `List<Object> objects = orderIds;` would not compile.

**5. Real-world use case**

A utility method originally declared `void log(List<Object> items)` could not be called with `List<OrderDto>`. Changing it to `void log(List<?> items)` fixed the API while keeping it read-only.

**6. Interview follow-up questions**

- **Is `List<? extends Object>` the same as `List<?>`?** Yes.
- **Can you assign `List<String>` to `Collection<Object>`?** No; `Collection<?>` works.
- **Why not make generics covariant like arrays?** It would reintroduce runtime type errors.
- **What is use-site variance?** Wildcards declared where the type is used.
- **Do Kotlin/Scala differ?** Yes — they support declaration-site variance (`out`/`in`).

**7. Common mistakes**

- Trying to "fix" invariance with raw types.
- Using `List<Object>` as a catch-all parameter.
- Assuming subclass relationships carry over to generic containers.
- Forgetting arrays behave differently.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: What is the difference between `Comparable` and `Comparator`?

**1. Interview-ready answer**

`Comparable<T>` defines a class's natural ordering through `compareTo(T other)` — a class can have only one natural order, and sorting calls such as `Collections.sort(list)` use it. `Comparator<T>` defines an external ordering through `compare(T a, T b)`, so you can have many comparators for the same class — by fare, by departure time, by name — and pass them to `list.sort(comparator)`. I use `Comparable` when there is one obvious default order, such as `LocalDate` or `BigDecimal`, and `Comparator` for everything else, especially domain objects where ordering depends on the use case.

**2. Detailed explanation**

| Aspect | Comparable | Comparator |
|---|---|---|
| Method | `compareTo(T)` | `compare(T, T)` |
| Location | Inside the class | Separate class/lambda |
| Number of orderings | One natural | Many |
| Package | `java.lang` | `java.util` |
| Sorting call | `Collections.sort(list)` | `list.sort(comparator)` |
| Contract | negative/zero/positive | same |

Why both: `Comparable` gives a default order for sorted collections like `TreeSet`; `Comparator` keeps ordering policies out of the domain class when several are needed.

**3. Internal working**

`List.sort` calls `Arrays.sort` with the comparator (or natural ordering when null). TimSort is the underlying algorithm — O(n log n) worst case, stable, and very fast on partially sorted data. Comparators are used repeatedly during sorting, so they should be cheap and stateless.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

final class Flight implements Comparable<Flight> {

    private final String flightNumber;
    private final BigDecimal fare;
    private final int departureHour;

    Flight(String flightNumber, BigDecimal fare, int departureHour) {
        this.flightNumber = flightNumber;
        this.fare = fare;
        this.departureHour = departureHour;
    }

    @Override
    public int compareTo(Flight other) {
        return this.fare.compareTo(other.fare);
    }

    String flightNumber() {
        return flightNumber;
    }

    BigDecimal fare() {
        return fare;
    }

    int departureHour() {
        return departureHour;
    }

    @Override
    public String toString() {
        return flightNumber + "(" + fare.toPlainString() + "," + departureHour + ")";
    }
}

public class ComparableComparatorDemo {

    public static void main(String[] args) {
        List<Flight> flights = new ArrayList<>(List.of(
                new Flight("AI204", BigDecimal.valueOf(9000), 18),
                new Flight("6E123", BigDecimal.valueOf(5000), 8),
                new Flight("UK99", BigDecimal.valueOf(5000), 6)));

        flights.sort(null);
        System.out.println(flights);

        flights.sort(Comparator.comparingInt(Flight::departureHour)
                .thenComparing(Flight::fare)
                .thenComparing(Flight::flightNumber));
        System.out.println(flights);
    }
}
```

Output:

```text
[6E123(5000,8), UK99(5000,6), AI204(9000,18)]
[UK99(5000,6), 6E123(5000,8), AI204(9000,18)]
```

Important lines:
- `compareTo` defines fare as the natural order; equal fares keep insertion order because TimSort is stable.
- `thenComparing` chains tie-breakers.
- `Comparator.comparingInt` avoids boxing.

**5. Real-world use case**

A flight search page sorts by cheapest by default (`Comparable`), but users can choose departure time, duration, or rating (`Comparator`). A `TreeSet<Flight>` with no comparator uses natural fare order and would treat two different flights with the same fare as duplicates — a subtle bug the team avoided by using a comparator that includes flight number.

**6. Interview follow-up questions**

- **When should a class implement Comparable?** When there is one obvious natural order.
- **What does `compareTo` returning 0 mean?** Objects are considered equal for ordering — important for `TreeSet`.
- **Is `Collections.sort` stable?** Yes; equal elements keep relative order.
- **Can you sort with both?** `list.sort(comparator)` overrides natural order.
- **Should `compareTo` be consistent with `equals`?** Ideally yes for sorted sets/maps; otherwise document the discrepancy.

**7. Common mistakes**

- Returning `a - b` for int comparison — can overflow; use `Integer.compare`.
- Using `double` subtraction for BigDecimal.
- Adding expensive logic inside comparators called O(n log n) times.
- Using `TreeSet` with an inconsistent comparator and losing elements.

**8. Interview difficulty**

**Intermediate**

---

### Question 10: How do you build advanced comparators with Java 8+ APIs?

**1. Interview-ready answer**

Java 8 added static and default methods on `Comparator` that make ordering declarative: `Comparator.comparing(Flight::fare)`, `thenComparing(...)`, `reversed()`, `nullsLast(...)`, and `comparingInt/Long/Double` to avoid boxing. You can chain them to express multi-level sorts, and `Comparator.naturalOrder()` or `reverseOrder()` cover common cases. Because comparators are stateless and cheap, they are safe to store in `static final` constants and reuse across requests. I also make sure null handling is explicit when sorting data from databases or external APIs.

**2. Detailed explanation**

Common methods:
- `comparing(Function)` / `comparingInt`, `comparingLong`, `comparingDouble`.
- `thenComparing(...)`, `thenComparingInt`, etc.
- `reversed()` — reverses the whole chain built so far, so order of calls matters.
- `nullsFirst`/`nullsLast` — wrap a comparator to handle nullable keys.
- `Comparator.naturalOrder()`, `Comparator.reverseOrder()`.

Why it exists: to replace verbose anonymous comparator classes with readable pipelines.

**3. Internal working**

Each comparator combinator returns a new comparator that delegates to the previous ones. During sorting, the final comparator is invoked O(n log n) times, so capturing a key extractor in a local variable is fine but avoid recomputing expensive values.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.List;

record Hotel(String name, BigDecimal price, Integer rating, String city) {
}

public class AdvancedComparatorDemo {

    static final Comparator<Hotel> RECOMMENDED_ORDER =
            Comparator.comparing(Hotel::city, Comparator.nullsLast(String::compareToIgnoreCase))
                    .thenComparing(Hotel::rating, Comparator.nullsLast(Comparator.reverseOrder()))
                    .thenComparing(Hotel::price)
                    .thenComparing(Hotel::name);

    public static void main(String[] args) {
        List<Hotel> hotels = List.of(
                new Hotel("Taj", BigDecimal.valueOf(12000), 5, "Delhi"),
                new Hotel("Ibis", BigDecimal.valueOf(5000), 4, "Delhi"),
                new Hotel("Unknown", BigDecimal.valueOf(3000), null, "Mumbai"));

        hotels.stream().sorted(RECOMMENDED_ORDER)
                .forEach(hotel -> System.out.println(hotel.name() + " " + hotel.city()));
    }
}
```

Output:

```text
Taj Delhi
Ibis Delhi
Unknown Mumbai
```

Important lines:
- `nullsLast` prevents NPE on nullable rating/city.
- `Comparator.reverseOrder()` gives highest rating first.
- `static final` reuse avoids rebuilding the comparator per request.

**5. Real-world use case**

A hotel search API exposes sort options `price_asc`, `rating_desc`, and `recommended`. The service holds a `Map<String, Comparator<Hotel>>` built once at startup, so each request selects a ready-made comparator instead of constructing one — cleaner and cheaper under load.

**6. Interview follow-up questions**

- **What does `reversed()` apply to?** The comparator chain built before the call.
- **Why use `comparingInt` over `comparing`?** Avoids boxing for primitive keys.
- **How do you sort nulls last?** `Comparator.nullsLast(naturalOrder())`.
- **Can comparators be stored in a Map?** Yes, they are stateless objects.
- **How do you handle multiple fields with mixed directions?** Apply `reversed()` on individual field comparators before chaining.

**7. Common mistakes**

- Placing `reversed()` after the full chain when only one field should be reversed.
- Forgetting null handling on external data.
- Using subtraction for numeric comparison.
- Building comparators inside tight loops.

**8. Interview difficulty**

**Intermediate**

---

### Question 11: What is the Reflection API and how is it used?

**1. Interview-ready answer**

Reflection lets Java code inspect and manipulate classes, fields, methods, and constructors at runtime. The entry point is `java.lang.Class`; from it you can obtain `Field`, `Method`, `Constructor`, and annotation objects, read modifiers and generic signatures, and invoke members dynamically with `Method.invoke()` or instantiate objects via `Constructor.newInstance()`. Spring, Hibernate, Jackson, and JUnit rely on reflection for dependency injection, ORM mapping, JSON binding, and test discovery. The trade-offs are slower calls than direct code, weaker compile-time safety, and restrictions under JPMS modules or security managers.

**2. Detailed explanation**

Common reflection operations:
- Obtain `Class<?>`: `obj.getClass()`, `ClassName.class`, `Class.forName("com.acme.Order")`.
- Inspect: `getDeclaredFields()`, `getMethods()`, `getConstructors()`, `getSuperclass()`, `getInterfaces()`.
- Access: `setAccessible(true)` to bypass access checks for private members.
- Invoke: `Method.invoke(target, args)`, `Constructor.newInstance(args)`, `Field.get/set`.
- Annotations: `getAnnotation(Class)`, `getDeclaredAnnotations()`.

`getDeclaredX` returns members declared in the class; `getX` also includes inherited public members.

**3. Internal working**

Class metadata lives in metaspace as `java.lang.Class` instances, one per loaded class per classloader. Reflection calls go through access checks and are harder for the JIT to inline; modern JVMs cache some metadata and use generated accessors, but direct calls remain faster. `Constructor.newInstance` replaced the deprecated `Class.newInstance` in Java 9 because it supports checked exceptions and arbitrary constructors. JPMS strong encapsulation can block `setAccessible` unless the module is opened.

**4. Practical Java/Spring Boot example**

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.math.BigDecimal;

class Order {

    private final String orderId;
    private final BigDecimal amount;

    Order(String orderId, BigDecimal amount) {
        this.orderId = orderId;
        this.amount = amount;
    }

    public String summary() {
        return "order:" + orderId + ":" + amount.toPlainString();
    }
}

public class ReflectionDemo {

    public static void main(String[] args) throws Exception {
        Class<?> orderClass = Class.forName("Order");

        Constructor<?> constructor = orderClass.getDeclaredConstructor(String.class, BigDecimal.class);
        constructor.setAccessible(true);
        Object order = constructor.newInstance("ORD-1", BigDecimal.TEN);

        Field idField = orderClass.getDeclaredField("orderId");
        idField.setAccessible(true);
        System.out.println(idField.get(order));

        Method summary = orderClass.getDeclaredMethod("summary");
        System.out.println(summary.invoke(order));
    }
}
```

Output:

```text
ORD-1
order:ORD-1:10
```

Important lines:
- `getDeclaredConstructor(String.class, BigDecimal.class)` selects an exact signature.
- `setAccessible(true)` permits access to private members.
- `Method.invoke` returns `Object`, so the result may need a cast.

**5. Real-world use case**

Spring creates beans by calling constructors reflectively, injects values into private fields annotated with `@Autowired`, and proxies methods. Hibernate reads `@Entity` metadata and maps private fields to columns. Jackson uses reflection to call getters and setters when converting JSON to DTOs.

**6. Interview follow-up questions**

- **Which is faster: reflection or direct call?** Direct call — reflection adds access checks and inhibits inlining.
- **What is `setAccessible(true)`?** It suppresses Java language access checks for that member.
- **What is `Class.forName` used for?** Loading a class by name and running its static initializer.
- **Can reflection read generic type arguments?** Yes, via `getGenericType()` and `ParameterizedType`, not `getType()`.
- **How do modules affect reflection?** Strong encapsulation can deny `setAccessible` unless packages are opened.

**7. Common mistakes**

- Using `Class.newInstance()` — deprecated since Java 9.
- Assuming reflection sees erased runtime types for `List<String>` without generic metadata APIs.
- Ignoring `InvocationTargetException` — the real exception is inside it.
- Using reflection in hot loops without caching metadata.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: What are the risks and performance costs of reflection? How do frameworks mitigate them?

**1. Interview-ready answer**

Reflection is slower than direct calls because of access checks, dynamic argument arrays, and reduced JIT inlining; it also loses compile-time safety, so mistakes appear at runtime. Under JPMS or a security manager, `setAccessible` can fail. Frameworks mitigate this by caching `Class`, `Field`, and `Method` metadata at startup, generating bytecode or using `MethodHandle`/LambdaMetafactory for hot paths, and validating configuration early. Spring, for example, scans and caches bean metadata once, and Jackson caches serializers/deserializers per type. My rule is to use reflection at framework boundaries, not in per-request hot code.

**2. Detailed explanation**

Costs:
- Each `Method.invoke` boxes primitives into `Object[]` and may unbox results.
- Access checks occur unless cached.
- JIT cannot always inline reflective calls.
- Errors surface as `InvocationTargetException`, `IllegalAccessException`, or `InaccessibleObjectException`.

Mitigations:
- Cache `Method`/`Field` objects in static maps or class metadata.
- Use `MethodHandle`, which the JVM can optimize better.
- Use generated accessors (ASM, Byte Buddy) — Spring and Hibernate do this.
- Prefer interfaces and direct calls in application code.
- Validate reflective configuration during startup (fail fast).

**3. Internal working**

`MethodHandle` is a typed, directly executable reference to a method with invocation semantics closer to bytecode. LambdaMetafactory can generate lambdas that call a method directly. Since Java 9, `VarHandle` provides similar optimized access to fields.

**4. Practical Java/Spring Boot example**

```java
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;
import java.lang.reflect.Method;

public class ReflectionCostDemo {

    static String direct(OrderDto dto) {
        return dto.orderId();
    }

    static String reflective(OrderDto dto) throws Exception {
        Method method = OrderDto.class.getDeclaredMethod("orderId");
        return (String) method.invoke(dto);
    }

    static String withMethodHandle(OrderDto dto, MethodHandle handle) throws Throwable {
        return (String) handle.invokeExact(dto);
    }

    public static void main(String[] args) throws Throwable {
        OrderDto dto = new OrderDto("ORD-1");
        MethodHandle handle = MethodHandles.lookup()
                .findVirtual(OrderDto.class, "orderId",
                        MethodType.methodType(String.class));

        System.out.println(direct(dto));
        System.out.println(reflective(dto));
        System.out.println(withMethodHandle(dto, handle));
    }
}

record OrderDto(String orderId) {
}
```

Important lines:
- `Method.invoke` requires checked exception handling and a cast.
- `MethodHandle.invokeExact` is strongly typed and more JIT-friendly.
- In real code, cache the `Method` instead of looking it up per call.

**5. Real-world use case**

An in-house mapper looked up getters with reflection for every row of a million-row import job, taking minutes. Caching `Method` objects in a `ClassValue` reduced the time dramatically. Spring Data and Jackson already do this caching internally.

**6. Interview follow-up questions**

- **What is a `MethodHandle`?** A typed, directly invocable reference to a method with better optimization potential.
- **What is `ClassValue`?** A utility for associating cached metadata with classes without leaks.
- **What exception wraps the real thrown exception in reflection?** `InvocationTargetException`.
- **Can reflection break encapsulation?** Yes, via `setAccessible`, subject to module rules.
- **Why do frameworks prefer startup-time scanning?** Fail fast and avoid per-request cost.

**7. Common mistakes**

- Looking up methods inside loops.
- Ignoring reflective exception wrapping.
- Using reflection where an interface would be clearer.
- Assuming reflection works identically in modular and classpath applications.

**8. Interview difficulty**

**Advanced**

---

### Question 13: What are annotations? How do you create and read a custom annotation?

**1. Interview-ready answer**

Annotations are metadata attached to classes, methods, fields, parameters, or packages. They do not change program behavior by themselves; tools or frameworks read them. Java has built-in annotations such as `@Override`, `@Deprecated`, and `@SuppressWarnings`, and lets you define your own with `@interface`. Two meta-annotations matter most: `@Retention` decides whether the annotation exists in source only, in the class file, or at runtime; `@Target` restricts where it can be applied. Spring reads runtime annotations to wire beans, map URLs, validate input, and apply transactions.

**2. Detailed explanation**

Retention policies:
- `SOURCE` — discarded by compiler; `@Override`.
- `CLASS` — stored in class file but not visible via reflection by default; used by bytecode tools.
- `RUNTIME` — available via reflection; used by Spring and Hibernate.

Targets include `TYPE`, `METHOD`, `FIELD`, `PARAMETER`, `CONSTRUCTOR`, `LOCAL_VARIABLE`, `TYPE_USE`, and more.

Other meta-annotations: `@Documented`, `@Inherited`, `@Repeatable`.

Why they exist: declarative configuration — describe intent next to the code instead of in an external XML file. Advantages: concise, discoverable, tool-friendly. Disadvantage: overuse can hide control flow and create "magic".

**3. Internal working**

Annotations are stored as attributes in the class file. At runtime, reflection returns proxy objects implementing the annotation interface. Reading them is relatively expensive, so frameworks cache parsed metadata.

**4. Practical Java/Spring Boot example**

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Method;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Audited {

    String action();
}

class PaymentService {

    @Audited(action = "CHARGE")
    public void charge(String orderId) {
        System.out.println("Charging " + orderId);
    }
}

public class AnnotationDemo {

    public static void main(String[] args) throws Exception {
        Method method = PaymentService.class.getMethod("charge", String.class);
        Audited audited = method.getAnnotation(Audited.class);

        System.out.println(audited.action());
        method.invoke(new PaymentService(), "ORD-1");
    }
}
```

Output:

```text
CHARGE
Charging ORD-1
```

Important lines:
- `RetentionPolicy.RUNTIME` is required for reflection to see the annotation.
- `getAnnotation` returns a proxy implementing `Audited`.
- In a real Spring app, an aspect would read this metadata and write an audit record.

**5. Real-world use case**

A Spring service annotates sensitive operations with `@Audited(action = "REFUND")`. An aspect reads the annotation, writes an audit event with the actor and order ID, and forwards it to Kafka. Because retention is RUNTIME, the aspect can discover it at startup and cache the metadata.

**6. Interview follow-up questions**

- **What happens if retention is SOURCE?** The annotation is gone after compilation; reflection cannot see it.
- **Can annotations have methods with parameters?** No — members are parameterless and may have defaults.
- **Can an annotation be inherited?** Only with `@Inherited`, and only for classes, not methods.
- **What is a repeatable annotation?** One annotated with `@Repeatable` holding a container annotation.
- **Why is annotation reading cached?** Reflection is slow; frameworks parse once at startup.

**7. Common mistakes**

- Forgetting `RUNTIME` retention and wondering why reflection returns null.
- Adding behavior to annotations themselves — they are passive metadata.
- Using annotations for logic that should be explicit.
- Not specifying `@Target`, allowing accidental misuse.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: How does Spring use annotations and meta-annotations?

**1. Interview-ready answer**

Spring scans classpath metadata at startup, not by instantiating every class; it reads annotation metadata to discover components, configuration, mappings, transactions, and validation rules. Many Spring annotations are meta-annotated — `@Service` is annotated with `@Component`, `@RestController` combines `@Controller` and `@ResponseBody`, and `@GetMapping` is meta-annotated with `@RequestMapping(method = GET)`. This composition lets Spring treat custom annotations as first-class constructs. You can create your own composed annotation by annotating it with `@Service` or `@RestController`, and Spring's `AnnotatedElementUtils` will find the merged annotation attributes.

**2. Detailed explanation**

Examples of composed annotations:
- `@RestController = @Controller + @ResponseBody`.
- `@GetMapping = @RequestMapping(method = GET)`.
- `@SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan`.

Spring uses `AnnotationMetadata` and ASM-based scanning before classes are loaded, then caches metadata. `MergedAnnotations` API searches meta-annotations and resolves attribute overrides.

Advantages: declarative configuration, reusable stereotypes, less boilerplate. Disadvantage: debugging "where did this behavior come from?" requires understanding meta-annotations.

**3. Internal working**

`ClassPathScanningCandidateComponentProvider` scans `.class` files with ASM, reading annotations without loading classes. Later, `ConfigurationClassParser` and bean post-processors use `MergedAnnotations`/`AnnotatedElementUtils` to resolve composed annotations, including attributes overridden on the custom annotation.

**4. Practical Java/Spring Boot example**

```java
import org.springframework.core.annotation.AnnotatedElementUtils;
import org.springframework.stereotype.Service;
import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Service
public @interface BookingServiceComponent {

    String value() default "";
}

@BookingServiceComponent("flight-booking")
public class FlightBookingService {
}

public class MetaAnnotationDemo {

    public static void main(String[] args) {
        Service service = AnnotatedElementUtils.findMergedAnnotation(
                FlightBookingService.class, Service.class);

        BookingServiceComponent custom = FlightBookingService.class
                .getAnnotation(BookingServiceComponent.class);

        System.out.println(service != null);
        System.out.println(custom.value());
    }
}
```

Output:

```text
true
flight-booking
```

Important lines:
- `@BookingServiceComponent` is meta-annotated with `@Service`, so Spring treats classes marked with it as beans.
- `findMergedAnnotation` discovers the meta-annotation.
- This requires Spring on the classpath; in pure Java you would need manual traversal.

**5. Real-world use case**

A company defines `@TenantService` combining `@Service` and a custom `@RequiresTenant` aspect annotation. Every tenant-aware service uses one annotation, and adding a new cross-cutting behavior requires changing only the composed annotation.

**6. Interview follow-up questions**

- **Is `@Service` functionally different from `@Component`?** No — it is a specialization used for clarity.
- **How does Spring scan without loading every class?** It reads class-file metadata with ASM.
- **What is `AnnotatedElementUtils`?** Spring's utility for finding merged/meta annotations.
- **Can custom annotations override attribute values?** Yes, via aliasFor and merged annotations.
- **Does annotation order matter?** Usually not, except for ordered interceptors and advice.

**7. Common mistakes**

- Assuming annotations execute code by themselves.
- Using `@Component` and `@Bean` interchangeably — they target different scenarios.
- Forgetting `RUNTIME` retention for framework-discovered annotations.
- Creating too many layers of meta-annotations.

**8. Interview difficulty**

**Advanced**

---

### Question 15: What is Java serialization? How do `serialVersionUID` and `transient` work?

**1. Interview-ready answer**

Java serialization converts an object graph into a byte stream using `ObjectOutputStream`, and deserialization reconstructs it with `ObjectInputStream`. A class must implement the marker interface `Serializable`. `serialVersionUID` is a version identifier; if the serialized bytes and the local class have different UIDs, deserialization throws `InvalidClassException`. Fields marked `transient` are skipped, which is how you avoid serializing passwords, sockets, or caches. You can customize the format with `writeObject`/`readObject` or use `readResolve` to preserve singletons. Java serialization is legacy and unsafe for untrusted input, so modern services prefer JSON, Protobuf, or Avro.

**2. Detailed explanation**

Key points:
- `Serializable` has no methods — it is a marker.
- All non-transient fields are serialized recursively; the whole reachable graph must be serializable or you get `NotSerializableException`.
- Static fields are never serialized — they belong to the class.
- `serialVersionUID` should be declared explicitly to keep versions compatible.
- `externalizable` allows fully custom format via `writeExternal`/`readExternal`.

Advantages: easy object persistence/RPC in legacy systems. Disadvantages: verbose binary format, versioning fragility, security vulnerabilities with gadget chains, and tight coupling to Java class structure.

**3. Internal working**

`ObjectOutputStream` writes class metadata and field values. On deserialization the JVM allocates objects without calling constructors, then populates fields. That is why constructors and field initializers do not run — a common surprise. `readObject` lets you validate or restore transient state.

**4. Practical Java/Spring Boot example**

```java
import java.io.*;

class PaymentSession implements Serializable {

    private static final long serialVersionUID = 1L;

    private final String sessionId;
    private transient String authToken;

    PaymentSession(String sessionId, String authToken) {
        this.sessionId = sessionId;
        this.authToken = authToken;
    }

    public String toString() {
        return sessionId + " / token=" + authToken;
    }
}

public class SerializationDemo {

    public static void main(String[] args) throws Exception {
        PaymentSession original = new PaymentSession("SESS-1", "secret-token");

        ByteArrayOutputStream bytes = new ByteArrayOutputStream();
        try (ObjectOutputStream out = new ObjectOutputStream(bytes)) {
            out.writeObject(original);
        }

        try (ObjectInputStream in = new ObjectInputStream(
                new ByteArrayInputStream(bytes.toByteArray()))) {
            PaymentSession restored = (PaymentSession) in.readObject();
            System.out.println(original);
            System.out.println(restored);
        }
    }
}
```

Output:

```text
SESS-1 / token=secret-token
SESS-1 / token=null
```

Important lines:
- `transient String authToken` is not serialized, so it is null after restore.
- `serialVersionUID` stabilizes the class version.
- The constructor does not run during deserialization.

**5. Real-world use case**

A legacy session store serialized `HttpSession` attributes into Redis. Adding a field without a stable `serialVersionUID` caused `InvalidClassException` after deployment and logged everyone out. The team declared explicit UIDs first, then migrated to JSON with Jackson so sessions could be inspected and evolved safely.

**6. Interview follow-up questions**

- **What happens if `serialVersionUID` changes?** `InvalidClassException` on deserialization.
- **Are static fields serialized?** No.
- **Are constructors called during deserialization?** No — objects are allocated directly.
- **What is a transient field used for?** Excluding derived state, secrets, or non-serializable resources.
- **Why avoid Java serialization for APIs?** Language coupling, versioning issues, and security risks.

**7. Common mistakes**

- Forgetting `serialVersionUID` in serializable classes.
- Serializing objects containing sockets, locks, or lambdas.
- Believing `transient` protects secrets from all threats — JSON serializers may still see getters.
- Deserializing untrusted byte streams.

**8. Interview difficulty**

**Intermediate**

---

### Question 16: What are the risks of Java serialization, and what should you use instead?

**1. Interview-ready answer**

Java native serialization is risky because deserializing untrusted data can instantiate unexpected classes and trigger gadget-chain attacks; it also couples stored bytes to Java class structure, breaks when fields change, and is verbose and slow. Modern backend systems use JSON for human-readable APIs, Protobuf or Avro for compact, schema-driven service-to-service messages, and database-native formats for storage. Records work well with JSON libraries because they are immutable and have clear component names. If native serialization is unavoidable, validate the object filter with `ObjectInputFilter`, declare stable `serialVersionUID`s, and never deserialize untrusted input.

**2. Detailed explanation**

Risks:
- **Security:** malicious streams can invoke methods during deserialization.
- **Versioning:** field changes require careful UID and `readObject` management.
- **Performance/size:** binary format includes class metadata; JSON is often comparable and easier to debug.
- **Language lock-in:** non-Java consumers cannot read it.

Alternatives:
- **JSON (Jackson):** REST APIs, config, logs.
- **Protobuf:** typed schemas, efficient binary, gRPC.
- **Avro:** schema evolution friendly, Kafka ecosystems.
- **Kryo/FST:** faster Java serialization where trusted.

Mitigation if stuck with Java serialization: `ObjectInputFilter` allowlists classes, limits graph depth/arrays, and rejects unknown types.

**3. Internal working**

Deserialization builds objects by reflection and can call `readObject` hooks before your application validates anything. That is why filters must be configured on the `ObjectInputStream` before reading.

**4. Practical Java/Spring Boot example**

```java
import java.io.*;

public class SafeDeserializationDemo {

    public static void main(String[] args) throws Exception {
        ByteArrayOutputStream bytes = new ByteArrayOutputStream();
        try (ObjectOutputStream out = new ObjectOutputStream(bytes)) {
            out.writeObject("ORD-1");
        }

        ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
                "java.lang.String;java.lang.Number;!*");

        try (ObjectInputStream in = new ObjectInputStream(
                new ByteArrayInputStream(bytes.toByteArray()))) {
            in.setObjectInputFilter(filter);
            Object value = in.readObject();
            System.out.println(value);
        }
    }
}
```

Output:

```text
ORD-1
```

Important lines:
- The filter allowlists `String` and `Number`, rejecting everything else with `!*`.
- `setObjectInputFilter` must be called before `readObject`.
- In Spring Boot REST APIs, Jackson binding replaces this mechanism entirely.

**5. Real-world use case**

A Kafka topic originally carried serialized Java `Order` objects. Consumers in other languages could not read them, and schema changes broke deployments. The team migrated to Avro with a schema registry, gaining safe evolution and cross-language support.

**6. Interview follow-up questions**

- **What is a deserialization gadget chain?** A sequence of classes whose `readObject` hooks can be abused for code execution.
- **How does Protobuf help?** Schema-driven, language-neutral, compact binary with explicit evolution rules.
- **Are records serializable?** Only if they implement `Serializable`; they are commonly mapped with JSON instead.
- **What is `ObjectInputFilter`?** A JEP 290 mechanism to restrict classes/graphs during deserialization.
- **Why is JSON easier to debug?** It is text and human-readable.

**7. Common mistakes**

- Deserializing data from untrusted sockets or queues.
- Assuming "binary" automatically means faster or safer.
- Skipping schema design for Kafka events.
- Using Java serialization for cross-service contracts.

**8. Interview difficulty**

**Advanced**

---

### Question 17: What is cloning in Java? Why is `Object.clone()` considered problematic?

**1. Interview-ready answer**

Cloning creates a copy of an object. `Object.clone()` performs a field-by-field copy, which is shallow by default: primitive values are copied, but object references still point to the same objects. To use it, a class must implement the marker interface `Cloneable` and usually override `clone()` as public. The design is widely criticized — `Cloneable` does not define `clone()`, the method is protected in `Object`, constructors are bypassed, final fields are awkward, and exceptions are odd. Most modern Java code uses copy constructors, copy factories, or records instead.

**2. Detailed explanation**

Requirements for cloning:
- Implement `Cloneable`, otherwise `Object.clone()` throws `CloneNotSupportedException`.
- Override `clone()` with public visibility and call `super.clone()`.
- For deep copies, clone or copy each mutable field explicitly.

Problems:
- `Cloneable` is a marker with no methods — easy to forget.
- `Object.clone()` is protected.
- It bypasses constructors and validation.
- Checked `CloneNotSupportedException` is awkward.
- Subclassing complicates clone contracts.
- Shallow copying is a common source of shared-state bugs.

**3. Internal working**

`Object.clone()` is a native method that allocates a new object of the same runtime class and copies the bit pattern of fields from the original — no constructor invocation.

**4. Practical Java/Spring Boot example**

```java
class SeatMap implements Cloneable {

    private int bookedSeats;

    SeatMap(int bookedSeats) {
        this.bookedSeats = bookedSeats;
    }

    void bookSeat() {
        bookedSeats++;
    }

    int bookedSeats() {
        return bookedSeats;
    }

    @Override
    public SeatMap clone() throws CloneNotSupportedException {
        return (SeatMap) super.clone();
    }

    public static void main(String[] args) throws Exception {
        SeatMap original = new SeatMap(10);
        SeatMap copy = original.clone();
        copy.bookSeat();

        System.out.println(original.bookedSeats());
        System.out.println(copy.bookedSeats());
    }
}
```

Output:

```text
10
11
```

Important lines:
- `super.clone()` copies the field value, so the objects are independent for primitives.
- `Cloneable` must be implemented or `CloneNotSupportedException` is thrown.
- For a class with a `List` field, this same code would produce a shallow copy and shared state.

**5. Real-world use case**

A reservation engine cloned a `BookingDraft` containing a mutable `List<Passenger>`. Passengers added to the copy also appeared in the original, producing duplicate tickets. Replacing `clone()` with a copy constructor that used `List.copyOf(...)` fixed the bug.

**6. Interview follow-up questions**

- **What happens if a class does not implement `Cloneable`?** `CloneNotSupportedException`.
- **Is `clone()` shallow or deep by default?** Shallow.
- **What is the preferred alternative?** Copy constructors or static copy factories.
- **Why is `Cloneable` called a broken interface?** It has no methods and does not declare `clone()`.
- **Do records support cloning?** No need — they are immutable; create a new instance.

**7. Common mistakes**

- Assuming `clone()` produces a deep copy.
- Forgetting to implement `Cloneable`.
- Calling `clone()` on a subclass without considering new fields.
- Using cloning when an immutable object would eliminate the need.

**8. Interview difficulty**

**Intermediate**

---

### Question 18: What is the difference between shallow copy and deep copy?

**1. Interview-ready answer**

A shallow copy copies an object's field values, so references to nested mutable objects still point to the same nested objects — the two copies share internal state. A deep copy recursively copies mutable nested objects, so each copy is fully independent. Primitive and immutable fields (`int`, `String`, `Instant`, `BigDecimal`) need no special handling; mutable fields (`List`, `Map`, arrays, custom mutable objects) must be copied explicitly. In modern Java I prefer immutable designs with `List.copyOf` and copy constructors, which make deep-copy bugs much less likely.

**2. Detailed explanation**

Shallow copy:
- Copies references, not referred objects.
- Fast but risky.
- `Object.clone()` default, `ArrayList`'s copy constructor (copies elements but not nested objects), `BeanUtils.copyProperties`.

Deep copy:
- Copies objects recursively.
- Safer for isolation, caching, thread safety.
- More expensive; cyclic graphs require identity maps.
- Achieved with copy constructors, serialization (avoid), or mappers.

**3. Internal working**

Consider `Booking -> List<Passenger>`. Shallow copy gives both bookings the same `ArrayList` reference. Deep copy creates a new list and new `Passenger` objects (or reuses immutable ones).

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

final class Passenger {

    private final String name;

    Passenger(String name) {
        this.name = name;
    }

    String name() {
        return name;
    }

    @Override
    public String toString() {
        return name;
    }
}

final class Booking {

    private final String bookingId;
    private final List<Passenger> passengers;

    Booking(String bookingId, List<Passenger> passengers) {
        this.bookingId = bookingId;
        this.passengers = new ArrayList<>(passengers);
    }

    static Booking shallowCopy(Booking source) {
        return new Booking(source.bookingId, source.passengers);
    }

    static Booking deepCopy(Booking source) {
        List<Passenger> copied = source.passengers.stream()
                .map(passenger -> new Passenger(passenger.name()))
                .toList();
        return new Booking(source.bookingId, copied);
    }

    void addPassenger(String name) {
        passengers.add(new Passenger(name));
    }

    @Override
    public String toString() {
        return bookingId + passengers;
    }
}

public class CopyDemo {

    public static void main(String[] args) {
        Booking original = new Booking("BK-1", new ArrayList<>(List.of(new Passenger("Asha"))));

        Booking shallow = Booking.shallowCopy(original);
        shallow.addPassenger("Ravi");

        Booking deep = Booking.deepCopy(original);
        deep.addPassenger("Meera");

        System.out.println(original);
        System.out.println(shallow);
        System.out.println(deep);
    }
}
```

Output:

```text
BK-1[Asha, Ravi]
BK-1[Asha, Ravi]
BK-1[Asha, Meera]
```

Important lines:
- `shallowCopy` passes the same list; the constructor's `new ArrayList<>(...)` copies the list itself, so the original list object is not shared — but its `Passenger` elements are immutable, so this example's "shallow" is already safe for elements. To demonstrate mutation sharing precisely, the nested object must be mutable; with immutable `Passenger`, the practical difference disappears.
- `deepCopy` creates new `Passenger` objects as well as a new list.
- Immutable elements make deep copying unnecessary — the preferred design.

**5. Real-world use case**

A pricing engine cached `Booking` objects. A shallow copy let one request add a passenger and corrupt another request's price. The team made `Passenger` immutable and stored `List.copyOf(...)`, so sharing the list became safe and deep copying was no longer needed.

**6. Interview follow-up questions**

- **Which fields need deep copying?** Mutable ones only.
- **Is `String` safe to share?** Yes — immutable.
- **How do you deep copy a cyclic graph?** Track visited objects with an identity map.
- **Is serialization a good deep-copy trick?** No — slow and fragile.
- **How do immutable collections help?** They remove the possibility of external mutation.

**7. Common mistakes**

- Assuming copying a collection copies its elements.
- Deep copying immutable objects unnecessarily.
- Forgetting arrays in nested objects.
- Using shallow copies for cache values that are mutated later.

**8. Interview difficulty**

**Intermediate**

---

### Question 19: Output-based master challenge — generics, ordering, and reflection

**1. Interview-ready answer**

Each line follows from one rule: generics are erased so runtime classes are identical; `Comparable` defines natural order; `Comparator` chains add tie-breakers and `reversed()` reverses the whole chain built so far; wildcard lists can be read as their upper bound but not written to; and reflection reports the runtime class, not generic arguments.

**2. Detailed explanation**

- `Collections.sort(list)` uses `compareTo`.
- `list.sort(comparator)` overrides natural order.
- `Comparator.comparing(Flight::toString).reversed()` sorts by the string representation descending.
- `List<? extends Number>` yields `Number` on read.
- `getClass().getSimpleName()` returns the erased runtime class.

**3. Internal working**

TimSort calls the comparator O(n log n) times; `Comparator.reversed()` wraps the existing comparator. Reflection reads class metadata from metaspace.

**4. Practical Java example — predict the output**

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

final class Flight implements Comparable<Flight> {

    private final String code;
    private final BigDecimal fare;

    Flight(String code, BigDecimal fare) {
        this.code = code;
        this.fare = fare;
    }

    @Override
    public int compareTo(Flight other) {
        return this.fare.compareTo(other.fare);
    }

    @Override
    public String toString() {
        return code + ":" + fare.toPlainString();
    }
}

public class AdvancedJavaChallenge {

    public static void main(String[] args) {
        List<Flight> flights = new ArrayList<>(List.of(
                new Flight("AI", BigDecimal.valueOf(5000)),
                new Flight("6E", BigDecimal.valueOf(3000))));

        Collections.sort(flights);
        System.out.println(flights);

        flights.sort(Comparator.comparing(Flight::toString).reversed());
        System.out.println(flights);

        List<? extends Number> values = List.of(1, 2.5);
        System.out.println(values.get(0).getClass().getSimpleName());

        System.out.println(flights.get(0).getClass().getSimpleName());
    }
}
```

Output:

```text
[6E:3000, AI:5000]
[AI:5000, 6E:3000]
Integer
Flight
```

Reasoning: natural fare order puts 6E first. In ascending string order `"6E:3000"` comes before `"AI:5000"` because `'6'` (code point 54) is less than `'A'` (65), so `reversed()` produces `"AI:5000"` then `"6E:3000"`. The wildcard read returns the actual element type `Integer`, and reflection returns the erased class `Flight`.

**5. Real-world use case**

A flight-results page sorted by fare and then by flight code. A candidate's comparator used string subtraction and produced inconsistent results for equal fares; understanding `compareTo` versus `Comparator` avoids that bug.

**6. Interview follow-up questions**

- **Why can't you `values.add(3)`?** `? extends Number` could be a `List<Double>`.
- **What would erasure report for `List<Flight>`?** `ArrayList`.
- **Which sort is stable?** TimSort used by `List.sort`.
- **What does `compareTo` returning 0 mean for a `TreeSet`?** Duplicate element — the new one is discarded.
- **How would you sort by fare descending?** `flights.sort(Comparator.comparing(Flight::fare).reversed())`.

**7. Common mistakes**

- Misreading `reversed()` scope.
- Assuming wildcard lists allow writes.
- Using `==` on BigDecimal.
- Expecting reflection to reveal generic arguments.

**8. Interview difficulty**

**Advanced**

---

### Question 20: Coding challenge — implement a generic, immutable `PagedResult<T>`

**1. Interview-ready answer**

I would implement `PagedResult<T>` as a final generic class holding an immutable snapshot of content plus page metadata. The constructor copies the input list with `List.copyOf`, validates page/size/totals, and exposes `map(Function<? super T, ? extends R>)` and `sorted(Comparator<? super T>)` that return new instances. Bounded or flexible function types let the API work with subclasses without breaking type safety. This mirrors Spring Data's `Page<T>` but shows the underlying generic design clearly.

**2. Detailed explanation**

**Problem statement:** Build a reusable pagination result object for REST APIs that supports mapping to a DTO type and sorting without mutating the original.

**Approach:**
1. Final class, final fields, defensive copies.
2. Generic `map` and `sorted` methods.
3. Compute `totalPages` safely.
4. Validate inputs.

**3. Internal working**

`List.copyOf` creates an unmodifiable snapshot. `Function<? super T, ? extends R>` uses PECS so a `Function<Order, OrderDto>` can be used where `? super T` is needed. Sorting creates a new list instead of mutating the original.

**4. Practical Java solution**

```java
import java.util.Comparator;
import java.util.List;
import java.util.Objects;
import java.util.function.Function;

public final class PagedResult<T> {

    private final List<T> content;
    private final int page;
    private final int size;
    private final long totalElements;

    public PagedResult(List<T> content, int page, int size, long totalElements) {
        if (page < 0) {
            throw new IllegalArgumentException("page must not be negative");
        }
        if (size <= 0) {
            throw new IllegalArgumentException("size must be positive");
        }
        if (totalElements < 0) {
            throw new IllegalArgumentException("totalElements must not be negative");
        }
        this.content = List.copyOf(Objects.requireNonNull(content, "content is required"));
        this.page = page;
        this.size = size;
        this.totalElements = totalElements;
    }

    public List<T> content() {
        return content;
    }

    public int page() {
        return page;
    }

    public int size() {
        return size;
    }

    public long totalElements() {
        return totalElements;
    }

    public int totalPages() {
        return (int) Math.ceil((double) totalElements / size);
    }

    public <R> PagedResult<R> map(Function<? super T, ? extends R> mapper) {
        Objects.requireNonNull(mapper, "mapper is required");
        return new PagedResult<>(content.stream().map(mapper).toList(), page, size, totalElements);
    }

    public PagedResult<T> sorted(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator, "comparator is required");
        return new PagedResult<>(content.stream().sorted(comparator).toList(), page, size, totalElements);
    }

    @Override
    public String toString() {
        return "page=" + page + ", size=" + size + ", total=" + totalElements + ", content=" + content;
    }

    public static void main(String[] args) {
        PagedResult<String> orders = new PagedResult<>(List.of("ORD-2", "ORD-1", "ORD-3"), 0, 3, 7);

        PagedResult<String> sorted = orders.sorted(String::compareTo);
        PagedResult<Integer> lengths = sorted.map(String::length);

        System.out.println(orders);
        System.out.println(sorted);
        System.out.println(lengths);
        System.out.println(lengths.totalPages());
    }
}
```

Output:

```text
page=0, size=3, total=7, content=[ORD-2, ORD-1, ORD-3]
page=0, size=3, total=7, content=[ORD-1, ORD-2, ORD-3]
page=0, size=3, total=7, content=[5, 5, 5]
3
```

Important lines:
- `List.copyOf` prevents callers from mutating content.
- `Function<? super T, ? extends R>` is PECS-correct.
- `sorted` and `map` return new objects, preserving immutability.

**5. Dry run**

| Step | Operation | Result |
|---|---|---|
| 1 | Construct with 3 orders, total 7 | content unchanged, totalPages = 3 |
| 2 | `sorted(String::compareTo)` | new list `[ORD-1, ORD-2, ORD-3]` |
| 3 | `map(String::length)` | new list `[5, 5, 5]` |
| 4 | Original `orders` | still unsorted |

**6. Complexity**

- Construction: O(n) to copy the list.
- `map`: O(n) time and O(n) space.
- `sorted`: O(n log n) time, O(n) space.
- `totalPages`: O(1).

**7. Edge cases**

- Null content → NPE with message.
- Negative page or total → `IllegalArgumentException`.
- Zero size → `IllegalArgumentException`.
- Empty content → valid, totalPages 0.
- Null mapper/comparator → NPE before processing.
- `totalElements` much larger than content — normal for a page.

**8. Real-world use case**

Spring Data JPA returns `Page<Order>`, but APIs often need `PagedResult<OrderDto>`. This pattern maps entities to DTOs while keeping page metadata, and the immutability makes it safe to cache or pass between threads.

**9. Interview follow-up questions**

- **Why not use Spring Data's `Page` directly in the API?** It couples the web contract to persistence and can serialize unwanted metadata.
- **Why `? super T` in the comparator?** It allows a `Comparator<Object>` to sort `T` values.
- **Why is `List.copyOf` better than `Collections.unmodifiableList`?** It snapshots the input; an unmodifiable wrapper still reflects later changes to the source.
- **How would you add pagination links?** Add a `links` map computed from page/size/total.
- **What if mapping throws?** The exception propagates; no partially built object is exposed.

**10. Common mistakes**

- Exposing the mutable input list directly.
- Using `Comparator<T>` instead of `Comparator<? super T>`.
- Mutating content in `sorted`.
- Forgetting to validate page/size.

**11. Interview difficulty**

**Intermediate**

---

## Chapter 4 Verification

| Syllabus topic | Question(s) |
|---|---|
| Generics | Q1, Q2, Q6, Q8 |
| Generic classes and methods | Q3 |
| Bounded type parameters | Q4 |
| Wildcards / PECS | Q5, Q6, Q8 |
| Type erasure and generic arrays | Q2, Q7, Q19 |
| Comparable vs Comparator | Q9, Q10, Q19 |
| Reflection API | Q11, Q12, Q19 |
| Annotations | Q13, Q14 |
| Java serialization | Q15, Q16 |
| Cloning | Q17 |
| Shallow vs deep copy | Q18 |
| Coding and output-based questions | Q2, Q6, Q19, Q20 |

**Total interview questions: 20** (all with the required 8-part format; coding questions include problem statement, approach, dry run, complexity, and edge cases).

**Next:** [Chapter 5 — Java 8+ Features and Streams](05-Java-8-Features-and-Streams.md). Say `NEXT CHAPTER` to continue.
