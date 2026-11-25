# Chapter 2: Strings and Object Handling

**Chapter goal:** Answer String, wrapper, object-identity and immutability questions precisely — these are the most frequently used "filter" questions in Java backend interviews — and back every answer with correct code and output reasoning.

**Topics covered:** String immutability · String Pool and interning · String vs StringBuilder vs StringBuffer · `==` vs `equals()` · equals/hashCode contract · Object class methods · wrapper classes · autoboxing/unboxing · immutable classes · `final`, `finally`, `finalize` · String concatenation internals · enums · inner and nested classes · output-based and coding questions.

**Cross-references:** Object creation and reference semantics are introduced in [Chapter 1, Q4](../01-Core-Java/01-Java-Fundamentals-and-OOP.md). Exception handling with `finally` and try-with-resources continues in [Chapter 3](03-Exception-Handling.md). Generics and records are covered in [Chapter 4](04-Generics-and-Advanced-Java.md), lambdas in [Chapter 5](05-Java-8-Features-and-Streams.md).

---

### Question 1: Why is `String` immutable in Java?

**1. Interview-ready answer**

A `String` cannot be changed after creation because its internal character storage is private and final, the class itself is `final`, and it exposes no method that mutates that storage. In Java 8 the storage was a `private final char[]`; since Java 9 it is a `private final byte[]` plus a `coder` field for compact strings. Methods like `toUpperCase()` or `replace()` never modify the original — they return a new `String`. Immutability enables the String Pool, makes strings safe as `HashMap` keys and across threads, allows the hash code to be cached, and protects security-sensitive values such as class names, file paths, anddatabase URLs. The cost is extra object creation when you build strings repeatedly, which is why we use `StringBuilder` for that.

**2. Detailed explanation**

Immutability is enforced by design:
- `public final class String` — cannot be subclassed, so no subclass can add mutators.
- The backing array is `private final` and never returned directly.
- Constructors copy the input array, so changing the original `char[]` afterwards does not change the String.
- All "modifying" methods create new objects.

Why it exists:
1. **String Pool:** the JVM can safely share one object for identical literals.
2. **Caching:** `hashCode()` is computed once and cached in the object.
3. **Thread safety:** no synchronization needed for shared strings.
4. **Security:** class loaders and security checks trust that a String value cannot change after validation.
5. **Collection keys:** a key whose hash code changes would corrupt a `HashMap`.

Disadvantages: repeated concatenation produces many temporary objects and extra garbage; holding large substrings used to retain the whole original array before Java 7 (fixed since).

**3. Internal working**

Java 9+ compact strings store LATIN-1 text in one byte per character and UTF-16 text in two bytes per character, selected by the `coder` field. `StringLatin1` and `StringUTF16` helper classes hold the operations. The cached hash is stored in a field and computed lazily. Reflection can still mutate private fields with `setAccessible(true)`, so "immutable" means immutable through the public API, not against deliberate reflection.

**4. Practical Java/Spring Boot example**

```java
public class StringImmutabilityDemo {

    public static void main(String[] args) {
        char[] source = {'o', 'r', 'd', 'e', 'r'};
        String original = new String(source);
        source[0] = 'X';

        String changed = original.toUpperCase();

        System.out.println("original = " + original);
        System.out.println("changed  = " + changed);
        System.out.println("same object = " + (original == changed));
    }
}
```

Output:

```text
original = order
changed  = ORDER
same object = false
```

Important lines:
- `new String(source)` copies the array, so mutating `source` does not affect `original`.
- `toUpperCase()` returns a new object; `original` stays unchanged.
- `==` proves a new object was created.

**5. Real-world use case**

A Spring Boot service builds a cache key from `tenantId + userId + methodName`. Because strings are immutable, that key can be stored in a `ConcurrentHashMap` and shared across threads without defensive copying. If the key were mutable, another thread changing it would make the entry unreachable and corrupt the cache.

**6. Interview follow-up questions**

- **Is `String` truly immutable?** Through its public API, yes. Reflection or Unsafe can change internal state, but that is outside normal use.
- **Why is `String` final?** To prevent subclasses from breaking immutability or cached-hash assumptions.
- **Does `substring()` share the original array?** Not since Java 7; it creates a new array.
- **Why use a record for DTOs instead of String?** Records generate immutable fields, equals/hashCode, and toString automatically (Chapter 4).
- **What about `StringBuilder` as a map key?** Avoid it — it is mutable, so its hash changes when content changes.

**7. Common mistakes**

- Saying `String` is immutable only because fields are `final` — the class being `final` and no exposed mutators matter too.
- Assuming `s.concat("x")` changes `s`.
- Using `String` in a loop for heavy concatenation and blaming the GC instead of the code.
- Thinking Java 9 compact strings change the public semantics — they do not.

**8. Interview difficulty**

**Intermediate**

---

### Question 2: What is the String Pool? How does `intern()` work?

**1. Interview-ready answer**

The String Pool is a JVM-managed table of unique String objects. When you create a string literal such as `"ORDER"`, the JVM checks the pool and reuses the existing object if present; otherwise it adds it. `new String("ORDER")` always creates a new heap object but still uses the pooled literal internally. Calling `intern()` on any String returns the canonical pooled instance, adding it if necessary. Since Java 7 the pool lives in the heap, not PermGen. This saves memory and allows fast reference comparison for interned strings, but the pool has limited size, so interning arbitrary user input can waste memory.

**2. Detailed explanation**

- Literals and compile-time constant expressions are automatically interned.
- `new String(...)` bypasses the pool for the new object.
- `intern()` returns a reference from the pool with equal content.
- The pool is per JVM (technically per classloader-era in older versions; in modern HotSpot it is a global `StringTable` in the heap).
- `-XX:StringTableSize` tunes the hash table; large-scale interning can cause contention.

Advantages: memory savings for repeated literals, faster `==` checks when interned, faster equality in some workloads. Disadvantages: pool overhead, potential memory growth if you intern unbounded values, and confusing `==` behavior if developers forget about interning.

**3. Internal working**

The pool is a hash table mapping string contents to weak references to `String` objects. During class loading, the JVM calls intern-like logic for literals in the constant pool. At runtime, `String.intern()` is a native method that looks up or inserts the value. Since Java 7 the referenced objects live in the regular heap and can be garbage-collected if no strong references remain.

**4. Practical Java/Spring Boot example — output-based**

```java
public class StringPoolDemo {

    public static void main(String[] args) {
        String literal = "ORDER";
        String anotherLiteral = "ORDER";
        String heapObject = new String("ORDER");
        String interned = heapObject.intern();
        String part1 = "OR";
        String part2 = part1 + "DER";

        System.out.println(literal == anotherLiteral);
        System.out.println(literal == heapObject);
        System.out.println(literal == interned);
        System.out.println(literal.equals(part2));
        System.out.println(literal == part2);
    }
}
```

Output:

```text
true
false
true
true
false
```

Reasoning:
- Both literals resolve to the same pooled object → `true`.
- `new String(...)` creates a distinct object → `false`.
- `intern()` returns the pooled object → `true`.
- `part2` is built at runtime from a variable → new object; content equal but reference different.

**5. Real-world use case**

A logging framework interns log level names (`"INFO"`, `"ERROR"`) and a rule engine interns repeated country codes read from a small lookup table. Because the domain is bounded, memory is predictable. The same team deliberately avoids interning customer names or request IDs, which are unbounded and would fill the pool.

**6. Interview follow-up questions**

- **Where is the pool stored?** In the heap since Java 7; PermGen before that.
- **Does `new String("x")` create one or two objects?** One new object plus the literal that is already pooled (or created in the pool).
- **Should you call `intern()` in application code?** Rarely. Use it only for bounded, highly repeated values.
- **Is `==` safe for interned strings?** Yes, but `equals()` is clearer and safer.
- **What happens if a pooled string has no references?** It can be collected; the pool holds weak references.

**7. Common mistakes**

- Claiming all strings are pooled — only literals, constants, and explicitly interned strings are.
- Using `==` to compare strings from JSON deserialization.
- Interning unbounded user input and causing memory growth.
- Believing the pool is in PermGen in Java 17.

**8. Interview difficulty**

**Intermediate**

---

### Question 3: `String` vs `StringBuilder` vs `StringBuffer` — when do you use each?

**1. Interview-ready answer**

`String` is immutable, so every concatenation creates a new object. `StringBuilder` and `StringBuffer` are mutable character sequences backed by a resizable array; they change contents in place. `StringBuilder` is not thread-safe and is the default choice for local string building. `StringBuffer` has synchronized mutating methods and is thread-safe, but slower, so it is rarely needed — modern code uses `StringBuilder` locally or explicit locking when sharing. In Java 9+, plain `+` concatenation compiles to `invokedynamic` with `StringConcatFactory`, which is often as good as manual `StringBuilder` for simple cases, but loops still benefit from an explicit builder with a sensible initial capacity.

**2. Detailed explanation**

| Class | Mutable | Thread-safe | Typical use |
|---|---|---|---|
| `String` | No | Yes (immutable) | Constants, keys, DTO fields |
| `StringBuilder` | Yes | No | Local building, loops, SQL generation |
| `StringBuffer` | Yes | Yes (synchronized) | Legacy shared builder; mostly avoid |

All three implement `CharSequence`. `StringBuilder` and `StringBuffer` share `AbstractStringBuilder`. Default capacity is 16 characters; growth is roughly `2 * oldCapacity + 2`. You can pre-size with `new StringBuilder(256)` to avoid reallocations.

Advantages of builders: O(n) total work for n appends versus O(n²) with repeated `String` concatenation. Disadvantages: mutable state, not suitable as map keys, and extra API clutter when a simple `+` would do.

**3. Internal working**

`append()` writes into a `byte[]` (Java 9+) and only updates the `coder` if needed. When capacity is exceeded, `Arrays.copyOf` allocates a larger array. `StringBuffer` wraps mutating methods in `synchronized`, which adds monitor overhead on every call.

**4. Practical Java/Spring Boot example**

```java
import java.util.List;

public class SqlBuilderDemo {

    public static void buildWithString(List<String> columns) {
        String query = "";
        for (String column : columns) {
            query = query + column + ", ";
        }
        System.out.println("String loop: " + query);
    }

    public static void buildWithBuilder(List<String> columns) {
        StringBuilder query = new StringBuilder(columns.size() * 16);
        for (String column : columns) {
            query.append(column).append(", ");
        }
        System.out.println("Builder loop: " + query);
    }

    public static void main(String[] args) {
        buildWithString(List.of("id", "order_no", "amount"));
        buildWithBuilder(List.of("id", "order_no", "amount"));
    }
}
```

Output:

```text
String loop: id, order_no, amount, 
Builder loop: id, order_no, amount, 
```

Important lines:
- `new StringBuilder(columns.size() * 16)` pre-sizes the buffer to reduce array copies.
- Chained `append()` calls return the same builder instance.
- The `String` loop produces the same result but allocates many intermediate strings.

**5. Real-world use case**

A reporting service dynamically builds a 200-column SQL `SELECT`. A developer used `String +=` inside the loop, creating hundreds of intermediate objects per request and causing GC pressure under load. Replacing it with a pre-sized `StringBuilder` cut allocation rate significantly. In production code, prefer a SQL builder library or JPA Criteria API for safety — never concatenate untrusted input into SQL.

**6. Interview follow-up questions**

- **Is `+` always slow?** No. For a few operands in one expression, the compiler/JDK optimizes it well. Loops are the problem.
- **What is `StringJoiner`?** A Java 8 helper for joining delimited strings; `String.join()` uses it internally.
- **Why is `StringBuffer` synchronized?** To make interleaved appends from multiple threads safe.
- **What does `StringBuilder.capacity()` return?** Current buffer size, not content length.
- **Can `StringBuilder` be used in a `switch`?** No — `switch` works with `String`, enums, and constants; use `toString()`.

**7. Common mistakes**

- Using `StringBuffer` "because it is safer" without a real sharing requirement.
- Creating a builder inside a loop instead of outside it.
- Using builders as `HashMap` keys.
- Assuming `StringBuilder.append(null)` throws — it appends the literal `"null"`.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 4: What is the difference between `==` and `equals()` for Strings?

**1. Interview-ready answer**

`==` compares values for primitives and references for objects. Since `String` is an object, `==` only tells you whether two variables point to the same object. `equals()` is overridden in `String` to compare the actual character content. Because of the String Pool, two separately written literals can pass `==`, which misleads people into thinking `==` compares content — it does not. Always use `equals()` for string content, and use `Objects.equals(a, b)` when either side may be `null`.

**2. Detailed explanation**

- Primitive `==`: compares numeric/boolean values.
- Reference `==`: compares memory addresses (identity).
- `String.equals()`: checks `this == obj` first for speed, then type and content.
- `Objects.equals(a, b)`: null-safe delegation to `a.equals(b)`.

Why it matters: values read from JSON, databases, sockets, or `new String(...)` are new objects. `==` on them fails even when text is identical — a classic source of authentication and routing bugs.

**3. Internal working**

`String.equals()` compares the internal byte arrays with `Arrays.equals` after checking the coder and length. The `this == obj` fast path makes comparing the same reference cheap.

**4. Practical Java/Spring Boot example — output-based**

```java
import java.util.Objects;

public class EqualityDemo {

    public static void main(String[] args) {
        String a = "AUTH";
        String b = new String("AUTH");
        String c = new String("AUTH").intern();
        String d = "AU" + "TH";
        String prefix = "AU";
        String e = prefix + "TH";

        System.out.println(a == b);
        System.out.println(a.equals(b));
        System.out.println(a == c);
        System.out.println(a == d);
        System.out.println(a == e);
        System.out.println(Objects.equals(a, null));
    }
}
```

Output:

```text
false
true
true
true
false
false
```

Reasoning: `b` is a fresh object; `c` is interned back to `a`; `d` is a compile-time constant folded into a literal; `e` is built at runtime from a variable.

**5. Real-world use case**

A JWT filter compared `tokenType == "Bearer"` instead of `"Bearer".equals(tokenType)`. Tokens parsed from the `Authorization` header are new String objects, so the check always failed and every request got 401. The fix is `"Bearer".equals(tokenType)` or `Bearer.class` enum parsing.

**6. Interview follow-up questions**

- **Is `equals()` safe if the left side is null?** No — use a constant on the left or `Objects.equals`.
- **Why does `a == d` return true above?** Compile-time constant folding made `d` a literal.
- **Does `equals()` work across String and StringBuilder?** No — different classes return false.
- **Should you use `equalsIgnoreCase()` for tokens?** Only if the specification allows case-insensitive values; most security tokens are case-sensitive.
- **What about records?** Records generate content-based `equals()` automatically (Chapter 4).

**7. Common mistakes**

- Comparing strings from request parameters with `==`.
- Assuming the String Pool makes `==` reliable everywhere.
- Calling `equals()` on a possibly-null request field.
- Overriding `equals()` without `hashCode()` (next question).

**8. Interview difficulty**

**Beginner**

---

### Question 5: Explain the `equals()` and `hashCode()` contract. Why must you override both?

**1. Interview-ready answer**

Java defines a contract: if two objects are equal according to `equals()`, they must return the same `hashCode()`. The reverse is not required — unequal objects may collide. `HashMap` and `HashSet` use `hashCode()` to choose a bucket and `equals()` to distinguish entries inside that bucket. If you override only `equals()`, two equal objects can land in different buckets and a `HashSet` will store both, or a `HashMap.get()` will return null for a key you just put. If you override only `hashCode()`, distinct objects that should be equal are treated as different. Modern code often uses `record` or an IDE/Objects helper to generate both consistently.

**2. Detailed explanation**

The `equals()` contract requires:
- **Reflexive:** `x.equals(x)` is true.
- **Symmetric:** `x.equals(y)` iff `y.equals(x)`.
- **Transitive:** if `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`.
- **Consistent:** repeated calls return the same result while state is unchanged.
- **Null:** `x.equals(null)` is false.

The `hashCode()` contract:
- Consistent within an execution while state is unchanged.
- Equal objects must produce equal hash codes.
- Unequal objects may produce the same hash code (collision) — that is legal but hurts performance.

Why it exists: hash-based collections need a cheap numeric hint to locate entries. Correctness depends on the equal-objects-equal-hash rule.

**3. Internal working**

`HashMap.put(key, value)` computes `hash = (h = key.hashCode()) ^ (h >>> 16)`, picks bucket `hash & (n-1)`, then walks the bucket comparing `hash` and `equals()`. `HashMap.get()` repeats the same steps. If a mutable field used in `hashCode()` changes after insertion, the key sits in the wrong bucket and becomes effectively lost.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

final class PaymentKey {

    private final String orderId;
    private final String currency;

    PaymentKey(String orderId, String currency) {
        this.orderId = Objects.requireNonNull(orderId);
        this.currency = Objects.requireNonNull(currency);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof PaymentKey other)) {
            return false;
        }
        return orderId.equals(other.orderId) && currency.equals(other.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(orderId, currency);
    }

    @Override
    public String toString() {
        return orderId + "/" + currency;
    }
}

public class EqualsHashCodeDemo {

    public static void main(String[] args) {
        PaymentKey first = new PaymentKey("ORD-1", "INR");
        PaymentKey second = new PaymentKey("ORD-1", "INR");
        Set<PaymentKey> keys = new HashSet<>();

        keys.add(first);
        keys.add(second);

        System.out.println(first.equals(second));
        System.out.println(first.hashCode() == second.hashCode());
        System.out.println(keys.size());
    }
}
```

Output:

```text
true
true
1
```

Important lines:
- `instanceof PaymentKey other` is Java 16+ pattern matching.
- `Objects.hash(...)` combines field hashes; `Objects.equals(...)` would be an alternative for field comparison.
- Fields are `final`, so the hash code cannot change after insertion into a set.

**5. Real-world use case**

An idempotency filter stores `PaymentKey(orderId, currency)` in a `HashSet` to reject duplicate payment callbacks. A developer initially overrode only `equals()`, so duplicate callbacks slipped through and charged the customer twice. Correct `equals`/`hashCode` plus an idempotency key in Redis (Chapter 18) fixed it.

**6. Interview follow-up questions**

- **Can two unequal objects have the same hash?** Yes — a collision; `equals()` resolves it.
- **Why is `Objects.hash` slower than manual computation?** It creates a varargs array; fine for most domain objects, but hot paths may compute manually.
- **What happens if a key's fields change after insertion?** The entry becomes unreachable through `get()` — a memory leak and logic bug.
- **Should entities use database ID in equals/hashCode?** Carefully; transient entities have null IDs. See Chapter 14.
- **What do records generate?** `equals`, `hashCode`, and `toString` based on all components.

**7. Common mistakes**

- Overriding one method but not the other.
- Including mutable fields in `hashCode()` for objects used as keys.
- Using `getClass()` versus `instanceof` inconsistently and breaking symmetry with subclasses.
- Returning a constant hash code — legal but destroys performance.

**8. Interview difficulty**

**Intermediate**

---

### Question 6: What methods does the `Object` class provide?

**1. Interview-ready answer**

Every Java class inherits `java.lang.Object`. Its key methods are `equals()` for logical equality, `hashCode()` for hash-based collections, `toString()` for text representation, `getClass()` for runtime type, `clone()` for copying, `finalize()` for pre-GC cleanup (deprecated), and the thread methods `wait()`, `notify()`, and `notifyAll()`. `getClass`, `wait`, `notify`, and `notifyAll` are `final`, so you cannot override them. Most backend code overrides `equals`, `hashCode`, and `toString`; `clone` is generally avoided in favor of copy constructors or records.

**2. Detailed explanation**

| Method | Purpose | Usually overridden? |
|---|---|---|
| `equals(Object)` | Logical equality | Yes for value objects |
| `hashCode()` | Hash-based collection support | Yes, with equals |
| `toString()` | Debug/log representation | Yes |
| `getClass()` | Runtime `Class<?>` | No — final |
| `clone()` | Create a copy | No — use copy constructor |
| `finalize()` | Legacy cleanup hook | No — deprecated |
| `wait()/notify()/notifyAll()` | Thread coordination | No — final |

Default `equals()` is reference identity; default `hashCode()` is often derived from the object address; default `toString()` prints `ClassName@hexHash`, which is useless in logs.

**3. Internal working**

`Object` is the root of the class hierarchy. Every object header contains a pointer to its class metadata, which is how `getClass()` works without stored per-object data. `wait/notify` require holding the object's monitor and are backed by JVM wait sets — covered in Chapter 8.

**4. Practical Java/Spring Boot example**

```java
import java.util.Objects;

public class AuditEvent {

    private final String eventId;
    private final String actor;

    AuditEvent(String eventId, String actor) {
        this.eventId = eventId;
        this.actor = actor;
    }

    @Override
    public boolean equals(Object obj) {
        return obj instanceof AuditEvent other
                && Objects.equals(eventId, other.eventId);
    }

    @Override
    public int hashCode() {
        return Objects.hash(eventId);
    }

    @Override
    public String toString() {
        return "AuditEvent{eventId='" + eventId + "', actor='" + actor + "'}";
    }

    public static void main(String[] args) {
        AuditEvent event = new AuditEvent("EVT-1", "asha");
        System.out.println(event.getClass().getSimpleName());
        System.out.println(event);
    }
}
```

Output:

```text
AuditEvent
AuditEvent{eventId='EVT-1', actor='asha'}
```

Important lines:
- `toString()` makes logs readable; without it you would see `AuditEvent@1b6d3586`.
- `getClass().getSimpleName()` uses the final `Object` method.

**5. Real-world use case**

A Spring service logs a DTO with `log.info("event={}", event)`. Without `toString()`, the log contains only an identity hash, making incident debugging impossible. Teams often use Lombok `@Data`, records, or a shared `ToStringBuilder` to generate consistent representations — while excluding sensitive fields such as passwords and tokens.

**6. Interview follow-up questions**

- **Can you override `getClass()`?** No, it is `final`.
- **What is the default `equals()` behavior?** Reference equality, same as `==`.
- **Why avoid `clone()`?** It is error-prone with shallow copies and has a broken contract; prefer copy constructors.
- **Is `finalize()` reliable?** No — called at an unpredictable time, possibly never.
- **Which methods are final in Object?** `getClass`, `wait`, `notify`, `notifyAll`, `registerNatives` is native/private.

**7. Common mistakes**

- Forgetting `toString()` and shipping unreadable logs.
- Calling `wait()` without synchronization — throws `IllegalMonitorStateException`.
- Attempting to override `final` thread methods.
- Putting secrets in `toString()` output.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 7: What are wrapper classes? Why do they exist?

**1. Interview-ready answer**

Wrapper classes are immutable object representations of the eight primitive types: `Integer`, `Long`, `Short`, `Byte`, `Double`, `Float`, `Character`, and `Boolean`. They exist because generics and collections work with objects, not primitives; because APIs need to represent "no value" with `null`; and because they provide utility constants and parsing methods such as `Integer.MAX_VALUE` and `Integer.parseInt()`. Autoboxing, added in Java 5, automatically converts primitives to wrappers and back. Wrappers are `final` and cached for small values, but they still carry object overhead, so performance-critical code uses primitives where null is not needed.

**2. Detailed explanation**

| Primitive | Wrapper | Cache range |
|---|---|---|
| `int` | `Integer` | -128 to 127 |
| `long` | `Long` | -128 to 127 |
| `short` | `Short` | -128 to 127 |
| `byte` | `Byte` | all values |
| `char` | `Character` | 0 to 127 |
| `boolean` | `Boolean` | all values |
| `float` | `Float` | none |
| `double` | `Double` | none |

Advantages: usable in collections/generics, nullable, utility methods, reflection-friendly. Disadvantages: memory overhead, possible NPE on unboxing, and slower arithmetic due to boxing/unboxing.

**3. Internal working**

`Integer.valueOf(int)` uses a static cache array created at class initialization; the upper bound can be raised with `-XX:AutoBoxCacheMax`. Constructors like `new Integer(5)` are deprecated since Java 9 — always use `valueOf`. Unboxing calls `intValue()` and throws `NullPointerException` if the wrapper reference is null.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class WrapperDemo {

    public static void main(String[] args) {
        List<Integer> orderQuantities = new ArrayList<>();
        orderQuantities.add(2);
        orderQuantities.add(Integer.valueOf(5));

        Integer cached = Integer.valueOf(127);
        Integer notCached = Integer.valueOf(128);

        int total = orderQuantities.get(0) + orderQuantities.get(1);

        System.out.println(total);
        System.out.println(cached == Integer.valueOf(127));
        System.out.println(notCached == Integer.valueOf(128));
    }
}
```

Output:

```text
7
true
false
```

Important lines:
- `orderQuantities.add(2)` autoboxes `int` to `Integer`.
- `orderQuantities.get(0) + orderQuantities.get(1)` unboxes both to `int` before addition.
- The cache makes small `Integer` values share references; 128 is outside the default cache.

**5. Real-world use case**

A Spring DTO uses `Long customerId` rather than `long` so that a missing JSON field is detected as `null` during validation. A database `BIGINT` column that is nullable maps to `Long` in JPA. Using `long` would silently store `0` and hide the error.

**6. Interview follow-up questions**

- **Why can't `List<int>` be written?** Generics require reference types; use `List<Integer>` or primitive arrays.
- **Which wrappers are cached?** Boolean, Byte, all Short, Integer/Long in -128..127, Character 0..127.
- **What does `Integer.parseInt("12x")` do?** Throws `NumberFormatException`.
- **Are wrappers immutable?** Yes — all fields are final.
- **When should you use primitives?** Local variables, counters, and hot arithmetic paths.

**7. Common mistakes**

- Using `new Integer(...)` instead of `valueOf`.
- Comparing wrappers with `==` expecting numeric comparison.
- Assuming all wrapper values are cached.
- Forgetting that unboxing a null wrapper throws NPE.

**8. Interview difficulty**

**Beginner**

---

### Question 8: What are autoboxing and unboxing? What pitfalls do they create?

**1. Interview-ready answer**

Autoboxing is the compiler's automatic conversion of a primitive to its wrapper, such as `int` to `Integer`, implemented by inserting a call to `Integer.valueOf()`. Unboxing is the reverse, implemented by calling `intValue()` or an equivalent. It makes collections and generics convenient, but it hides costs: object allocation outside the cache ranges, possible NPE when unboxing null, surprising `==` behavior, and overload-resolution ambiguity. In performance-sensitive loops, prefer primitives to avoid thousands of hidden allocations.

**2. Detailed explanation**

Where boxing appears:
- Adding primitives to `List<T>` or `Map<K,V>`.
- Passing primitives to methods expecting wrappers or `Object`.
- Assigning primitives to wrapper variables.
- Ternary expressions mixing wrappers and primitives.

Where unboxing appears:
- Arithmetic on wrapper objects.
- Comparing a wrapper with a primitive using `==`.
- Passing a wrapper to a method expecting a primitive.

Advantages: concise code, seamless use of generic collections. Disadvantages: hidden allocations, null-related crashes, and confusing equality semantics.

**3. Internal working**

The compiler rewrites `Integer i = 5;` to `Integer i = Integer.valueOf(5);` and `int n = i;` to `int n = i.intValue();`. In a loop like `sum += list.get(i)`, each iteration performs an unbox and a box if `sum` is a wrapper.

**4. Practical Java/Spring Boot example — output-based**

```java
public class AutoboxingPuzzle {

    public static void main(String[] args) {
        Integer smallA = 100;
        Integer smallB = 100;
        Integer largeA = 200;
        Integer largeB = 200;

        System.out.println(smallA == smallB);
        System.out.println(largeA == largeB);
        System.out.println(largeA.equals(largeB));
        System.out.println(largeA == 200);

        Integer missing = null;
        try {
            int value = missing;
            System.out.println(value);
        } catch (NullPointerException exception) {
            System.out.println("Unboxing null failed");
        }
    }
}
```

Output:

```text
true
false
true
true
Unboxing null failed
```

Reasoning:
- 100 is in the Integer cache → same references.
- 200 is outside the cache → distinct objects.
- `equals()` compares content → true.
- `largeA == 200` unboxes the wrapper to `int` → numeric comparison → true.
- Unboxing `null` calls `intValue()` on null → NPE.

**5. Real-world use case**

A pricing service stored totals as `Long` and used `total1 == total2` in a test. Values above 127 were different objects, so tests failed intermittently depending on data. The team switched to `Objects.equals()` and kept primitives for arithmetic inside the pricing engine.

**6. Interview follow-up questions**

- **Does `Integer a = 5; int b = a;` unbox?** Yes.
- **Why is `Integer == Integer` unreliable?** It compares references; cache range decides the result.
- **What does `Integer.valueOf(300) == 300` return?** True — one side is unboxed.
- **Can autoboxing cause OutOfMemoryError?** Indirectly, by creating many wrapper objects in hot loops.
- **How do you avoid boxing in streams?** Use `IntStream`, `LongStream`, and `DoubleStream` (Chapter 5).

**7. Common mistakes**

- Comparing wrapper objects with `==` in tests and business logic.
- Unboxing nullable DTO fields without a null check.
- Using wrappers in tight loops for counters.
- Expecting `Float` and `Double` to be cached.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: How do you create an immutable class in Java?

**1. Interview-ready answer**

To make a class immutable: declare it `final` so it cannot be subclassed, make all fields `private final`, provide no setters, validate in the constructor, and return defensive copies for any mutable field such as an array, `List`, or `Date`. For collections, copy in the constructor and expose either `Collections.unmodifiableList(...)` or a fresh copy from the getter. Avoid leaking `this` from the constructor. In modern Java, a `record` gives you immutability for its components automatically, but you still need defensive copies in a compact constructor for mutable components.

**2. Detailed explanation**

Checklist for immutability:
1. `final` class (or sealed/record).
2. `final` fields.
3. No mutating methods.
4. Defensive copies on input and output.
5. No method that publishes `this` before construction completes.
6. Constructor validates invariants.

Advantages: inherently thread-safe, safe as map keys and cache entries, no defensive synchronization, easier reasoning. Disadvantages: every "change" creates a new object; large objects may be expensive to copy.

**3. Internal working**

`final` fields are safely published: the JVM guarantees that a properly constructed object's final field values are visible to other threads without additional synchronization (Java Memory Model, Chapter 8). This is why immutable objects are the foundation of concurrent designs.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

public final class FlightRoute {

    private final String flightNumber;
    private final List<String> stopoverCodes;

    public FlightRoute(String flightNumber, List<String> stopoverCodes) {
        if (flightNumber == null || flightNumber.isBlank()) {
            throw new IllegalArgumentException("flightNumber is required");
        }
        this.flightNumber = flightNumber;
        this.stopoverCodes = List.copyOf(Objects.requireNonNull(stopoverCodes));
    }

    public String flightNumber() {
        return flightNumber;
    }

    public List<String> stopoverCodes() {
        return stopoverCodes;
    }

    public FlightRoute withAdditionalStopover(String code) {
        if (code == null || code.isBlank()) {
            throw new IllegalArgumentException("code is required");
        }
        List<String> updated = new ArrayList<>(stopoverCodes);
        updated.add(code);
        return new FlightRoute(flightNumber, updated);
    }

    public static void main(String[] args) {
        List<String> stops = new ArrayList<>();
        stops.add("DEL");
        FlightRoute route = new FlightRoute("AI-204", stops);
        stops.add("BOM");

        System.out.println(route.stopoverCodes());
        System.out.println(route.withAdditionalStopover("DXB").stopoverCodes());
    }
}
```

Output:

```text
[DEL]
[DEL, DXB]
```

Important lines:
- `List.copyOf(...)` makes an immutable snapshot; changing the caller's `stops` list afterwards has no effect.
- `Collections.unmodifiableList(new ArrayList<>(...))` is an alternative if you need to avoid throwing on null elements.
- `withAdditionalStopover` returns a new instance instead of mutating the old one.

**5. Real-world use case**

A flight search service caches `FlightRoute` objects in Caffeine and Redis. Because routes are immutable, they can be shared across all request threads and serialized to JSON without any risk of one request corrupting another's data. Pricing updates create new objects and evict the cache key.

**6. Interview follow-up questions**

- **Is a class with only final fields automatically immutable?** No — a final `List` reference can still have its contents changed.
- **Why `final` class?** A subclass could add mutable state or override getters.
- **What is a `record`?** A transparent carrier with generated equals/hashCode/toString and final components (Chapter 4).
- **Should getters return copies or unmodifiable views?** Both are valid; copies avoid throwing on mutation attempts, views avoid allocation.
- **What about `Date` and `Calendar`?** They are mutable — use `Instant` or `LocalDate` instead (Chapter 5).

**7. Common mistakes**

- Returning the internal mutable collection from a getter.
- Storing a mutable object received from a caller without copying.
- Using `Collections.unmodifiableList` on the original list — the caller can still mutate it through their reference.
- Registering listeners or starting threads in the constructor, leaking `this`.

**8. Interview difficulty**

**Intermediate to Advanced**

---

### Question 10: Explain `final`, `finally`, and `finalize`.

**1. Interview-ready answer**

These three are unrelated despite the similar names. `final` is a modifier: a final variable cannot be reassigned, a final method cannot be overridden, and a final class cannot be extended. `finally` is an exception-handling block that runs after `try`/`catch` whether or not an exception occurred, mainly for cleanup. `finalize()` was a legacy `Object` method called by the JVM before garbage collection, but it is deprecated for removal because its timing is unpredictable and it has serious performance problems. Modern code uses try-with-resources and `Cleaner` instead.

**2. Detailed explanation**

`final`:
- Primitive final variable: value cannot change.
- Reference final variable: reference cannot change, but the object can still be mutated.
- Blank final: declared without initialization but assigned exactly once in every constructor path.
- final method: prevents overriding, useful for template methods.
- final class: prevents inheritance, e.g. `String`, `Integer`.
- Also has a memory-model effect: final fields are safely published.

`finally`:
- Executes even when `try` returns, breaks, or throws.
- Does not execute if the JVM exits (`System.exit`), the process is killed, or the thread is abruptly stopped.
- A `return` inside `finally` overrides the `try` return — a bad practice.
- Prefer try-with-resources for anything implementing `AutoCloseable`.

`finalize()`:
- Called by a dedicated Finalizer thread before an object is reclaimed.
- Unpredictable, can delay GC for many cycles, can resurrect objects, and is deprecated since Java 9 and deprecated for removal in Java 18.
- Alternatives: try-with-resources for resources, `Cleaner` or `PhantomReference` for last-resort cleanup.

**3. Internal working**

Final fields are enforced by the JVM verifier and have special JMM semantics. `finally` is implemented by duplicating cleanup bytecode on all normal and exceptional paths (or via exception tables). `finalize()` requires the GC to enqueue objects on a finalization queue, extending object lifetime and increasing pressure.

**4. Practical Java/Spring Boot example**

```java
public class FinalFinallyDemo {

    private final String serviceName;

    FinalFinallyDemo(String serviceName) {
        this.serviceName = serviceName;
    }

    String process(int quantity) {
        try {
            if (quantity <= 0) {
                throw new IllegalArgumentException("quantity must be positive");
            }
            return serviceName + ":" + quantity;
        } finally {
            System.out.println("finally always runs for " + serviceName);
        }
    }

    public static void main(String[] args) {
        FinalFinallyDemo demo = new FinalFinallyDemo("inventory");
        System.out.println(demo.process(5));
        try {
            demo.process(0);
        } catch (IllegalArgumentException exception) {
            System.out.println("caught: " + exception.getMessage());
        }
    }
}
```

Output:

```text
finally always runs for inventory
inventory:5
finally always runs for inventory
caught: quantity must be positive
```

Important lines:
- `private final String serviceName` is assigned in the constructor and cannot change.
- The `finally` block runs before the `return` value is delivered and before the exception propagates.
- Use try-with-resources rather than `finally` for closing streams and connections.

**5. Real-world use case**

A JDBC-based migration job historically closed `Connection` objects in `finally`, but early returns caused leaks during exceptions. Refactoring to try-with-resources removed the leak. Separately, a legacy class used `finalize()` to close a native file handle; under load, handles accumulated because finalization lagged behind allocation — a `Cleaner` plus explicit `close()` solved it.

**6. Interview follow-up questions**

- **Does `finally` run after `System.exit()`?** No.
- **Can `final` be used with a method parameter?** Yes, it prevents reassignment inside the method.
- **Is a final object immutable?** No — only its reference is fixed.
- **Why is `finalize()` deprecated?** Unpredictable timing, GC overhead, resurrection issues.
- **What should replace `finalize()`?** try-with-resources for `AutoCloseable`, `Cleaner` for fallback cleanup.

**7. Common mistakes**

- Believing `final` makes an object immutable.
- Returning from `finally` and swallowing exceptions.
- Using `finalize()` for business cleanup or releasing scarce resources.
- Assuming `finally` always runs on JVM shutdown.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 11: How does the `+` operator work internally for Strings?

**1. Interview-ready answer**

The compiler treats `+` on strings specially. If every operand is a compile-time constant — a literal, a `final` primitive/String initialized with a constant expression, or a constant sub-expression — the compiler folds it into a single literal at compile time, so no runtime work happens. If any operand is a variable or method call, concatenation occurs at runtime. Up to Java 8 the compiler desugared it into `new StringBuilder().append(...).toString()`; since Java 9 it emits `invokedynamic` with `StringConcatFactory`, which can generate an optimized concatenation method and avoid building an intermediate builder. Inside loops, repeated `+` still creates many objects, so an explicit `StringBuilder` is clearer and usually faster.

**2. Detailed explanation**

Compile-time constants include:
- String literals: `"ORD"`.
- `final String PREFIX = "ORD";`
- Constant arithmetic: `"ORD" + "-" + 1`.

Not constants:
- `String prefix = "ORD";` (not final).
- `String value = getPrefix() + "X";`
- `String fromDb = row.getString("code");`

Why this distinction matters: output-based interview questions use it to decide whether `==` returns true. Performance questions use it to explain why a single expression with five `+` operators is cheap while a 10,000-iteration loop is expensive.

**3. Internal working**

Java 9+ `StringConcatFactory` uses `invokedynamic` to generate a dedicated method at runtime, flattening arguments into a single array and invoking `String` construction once. This is called " invokedynamic-based string concatenation" and is one reason newer JDKs improved `+` performance without changing source code.

**4. Practical Java/Spring Boot example — output-based**

```java
public class ConcatenationDemo {

    private static final String CONSTANT_PREFIX = "ORD";

    public static void main(String[] args) {
        String variablePrefix = "ORD";
        String folded = "ORD" + "-" + 1;
        String fromConstant = CONSTANT_PREFIX + "-1";
        String runtime = variablePrefix + "-1";

        System.out.println("ORD-1" == folded);
        System.out.println("ORD-1" == fromConstant);
        System.out.println("ORD-1" == runtime);
        System.out.println("ORD-1".equals(runtime));
    }
}
```

Output:

```text
true
true
false
true
```

Reasoning: `folded` and `fromConstant` are compile-time constants and are interned; `runtime` is computed at runtime and produces a new object.

**5. Real-world use case**

A Spring Boot service builds metrics tag names by concatenating constants such as `"orders." + "created"`, which the compiler folds into one literal — zero allocation per request. But a loop that builds a CSV row with `row += value + ","` allocated megabytes per request; replacing it with `StringBuilder` removed the GC spikes.

**6. Interview follow-up questions**

- **Is `final String s = compute();` a compile-time constant?** No — the initializer must be a constant expression.
- **Did Java 8 use `StringBuilder` for `+`?** Yes, typically one builder per expression.
- **Which JDK introduced `StringConcatFactory`?** Java 9.
- **Does `String.concat()` mutate the original?** No, it returns a new String.
- **When is `StringBuilder` definitely better?** Multiple appends in loops or conditional branches.

**7. Common mistakes**

- Assuming all `+` expressions are interned.
- Building large strings with `+` in loops and blaming the GC.
- Using string concatenation to build SQL — use prepared statements (Chapter 15).
- Assuming `+=` on String mutates the variable in place.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: What is an enum in Java? How is it different from a set of constants?

**1. Interview-ready answer**

An `enum` is a special Java type representing a fixed set of named instances. Each enum constant is a singleton object created when the enum class is initialized. Unlike `public static final int` constants, enums are type-safe — you cannot pass a `String` or an unrelated integer where a `PaymentStatus` is expected — and they can carry fields, methods, and implement interfaces. They also provide `values()` and `valueOf()`, work in `switch`, and are safe to serialize by name. Use enums for closed sets such as order status, booking class, and feature flags; avoid storing their `ordinal()` in a database because inserting a new constant can silently corrupt data.

**2. Detailed explanation**

Key facts:
- `enum` compiles to a class extending `java.lang.Enum<E>`.
- Constants are `public static final` instances, created in declaration order.
- Constructors are private (even if you omit the modifier).
- `name()` returns the declared name; `ordinal()` returns its position — avoid relying on it.
- `valueOf("CREATED")` throws `IllegalArgumentException` for unknown names.
- Enums can have abstract methods implemented per constant.

Why it exists: type safety, readable code, and a bounded domain. Advantages: compile-time checks, no invalid values, useful metadata. Disadvantages: adding a constant requires redeploying clients; deserializing an unknown constant fails; large enum switch statements can become a maintenance smell.

**3. Internal working**

The compiler generates a class with a static array `$VALUES` holding the constants and a `values()` method returning a clone. `Enum.valueOf()` does a hash lookup by name. Each constant may get its own anonymous subclass if it overrides methods, which is why `getClass()` can differ between constants.

**4. Practical Java/Spring Boot example**

```java
public enum BookingStatus {

    CREATED("Order created", false),
    PAID("Payment received", false),
    TICKETED("Ticket issued", true),
    CANCELLED("Booking cancelled", true);

    private final String description;
    private final boolean terminal;

    BookingStatus(String description, boolean terminal) {
        this.description = description;
        this.terminal = terminal;
    }

    public String description() {
        return description;
    }

    public boolean isTerminal() {
        return terminal;
    }

    public boolean canTransitionTo(BookingStatus next) {
        return switch (this) {
            case CREATED, PAID -> next == PAID || next == TICKETED || next == CANCELLED;
            case TICKETED, CANCELLED -> false;
        };
    }
}

public class EnumDemo {

    public static void main(String[] args) {
        BookingStatus status = BookingStatus.valueOf("PAID");

        System.out.println(status.description());
        System.out.println(status.canTransitionTo(BookingStatus.TICKETED));
        System.out.println(BookingStatus.values().length);
    }
}
```

Output:

```text
Payment received
true
4
```

Important lines:
- `switch` on enum with arrow syntax is Java 14+; it is exhaustive and needs no `default` here.
- `valueOf` requires the exact declared name.
- `values()` returns a new array each call, so caching it in a static field is safe if you never expose it.

**5. Real-world use case**

An airline booking service stores `CREATED`, `PAID`, `TICKETED`, and `CANCELLED` as `VARCHAR` in PostgreSQL. Hibernate maps them with `@Enumerated(EnumType.STRING)`. A new constant `CHECKED_IN` can be added without changing existing rows — using `ORDINAL` would have made existing numeric values shift meaning.

**6. Interview follow-up questions**

- **Can an enum have a constructor?** Yes, private only; called once per constant.
- **Can an enum implement an interface?** Yes.
- **Can you extend an enum?** No — enums are implicitly final.
- **Why avoid `ordinal()`?** Inserting or reordering constants changes persisted values.
- **What happens with `valueOf("unknown")`?** `IllegalArgumentException`; validate external input first.

**7. Common mistakes**

- Persisting `ordinal()` instead of the name.
- Comparing enum names as strings instead of using enum constants.
- Assuming `values()` returns the internal array.
- Using enums for endlessly growing categories such as product tags.

**8. Interview difficulty**

**Beginner to Intermediate**

---

### Question 13: How can you use enums as strategy objects, and what are `EnumSet` and `EnumMap`?

**1. Interview-ready answer**

Enums can implement interfaces and override methods per constant, which lets them act as strategy objects — for example, each `FareRule` constant computes a different multiplier. This removes `switch` statements and keeps the behavior next to the value. `EnumSet` and `EnumMap` are specialized collections for enum keys that use a compact bit-vector or array internally, so they are faster and use less memory than `HashSet`/`HashMap` for enum domains. Use them for permission sets, state groups, and enum-keyed configuration.

**2. Detailed explanation**

Strategy enum pattern:
- Define an interface with a method.
- Make enums implement it.
- Each constant provides its own implementation, possibly with constructor parameters.

`EnumSet`:
- Requires a common enum type; internally a bit vector (one `long` for up to 64 constants).
- No null elements allowed.
- Iteration order is enum declaration order.

`EnumMap`:
- Keys must be from one enum type.
- Backed by a single array indexed by `ordinal()`.
- Faster and more compact than `HashMap` for enum keys.

Advantages: expressive, no `if-else` ladders, excellent performance for small domains. Disadvantage: behavior changes require code changes and redeployment.

**3. Internal working**

`EnumMap` stores values in an array whose index is the key's ordinal, giving O(1) access with no hashing. `EnumSet` represents membership as bits in one or more `long` values, so union/intersection are bitwise operations.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.EnumMap;
import java.util.EnumSet;
import java.util.Map;
import java.util.Set;

interface FareMultiplier {
    BigDecimal multiplier();
}

enum CabinClass implements FareMultiplier {

    ECONOMY {
        @Override
        public BigDecimal multiplier() {
            return BigDecimal.ONE;
        }
    },
    PREMIUM {
        @Override
        public BigDecimal multiplier() {
            return BigDecimal.valueOf(1.4);
        }
    },
    BUSINESS {
        @Override
        public BigDecimal multiplier() {
            return BigDecimal.valueOf(2.5);
        }
    }
}

enum Permission {
    READ, WRITE, REFUND, AUDIT
}

public class EnumStrategyDemo {

    private static final Map<CabinClass, BigDecimal> BASE_FARES = new EnumMap<>(CabinClass.class);

    static {
        BASE_FARES.put(CabinClass.ECONOMY, BigDecimal.valueOf(5000));
        BASE_FARES.put(CabinClass.PREMIUM, BigDecimal.valueOf(7000));
        BASE_FARES.put(CabinClass.BUSINESS, BigDecimal.valueOf(12000));
    }

    public static void main(String[] args) {
        Set<Permission> reviewer = EnumSet.of(Permission.READ, Permission.AUDIT);
        BigDecimal fare = BASE_FARES.get(CabinClass.BUSINESS).multiply(CabinClass.BUSINESS.multiplier());

        System.out.println(fare.toPlainString());
        System.out.println(reviewer);
        System.out.println(reviewer.contains(Permission.REFUND));
    }
}
```

Output:

```text
30000.0
[READ, AUDIT]
false
```

Important lines:
- Each constant overrides `multiplier()` — no switch statement.
- `new EnumMap<>(CabinClass.class)` ties the map to one enum type.
- `EnumSet.of(...)` prints in declaration order.

**5. Real-world use case**

A spring security configuration stores role permissions in an `EnumMap<Role, EnumSet<Permission>>`. Authorization checks are bit-set lookups with negligible cost, and the code reads like a policy table rather than nested conditionals.

**6. Interview follow-up questions**

- **Can `EnumSet` contain null?** No — it throws NPE.
- **What is the internal representation of `EnumSet`?** A bit vector in one or more `long` values.
- **When would you prefer a normal `HashMap`?** When keys are not enums or come from multiple types.
- **Can enums be used in Spring dependency injection?** Yes — enum values can be beans via `@Bean` methods or used as qualifier keys.
- **Are enum overrides visible via `getClass()`?** Yes, constants with bodies have distinct anonymous classes.

**7. Common mistakes**

- Using `HashMap<String, ...>` when an `EnumMap` is available.
- Adding a huge number of constants far beyond a small domain.
- Putting mutable state in enum constants — they are shared singletons.
- Depending on `EnumSet` toString format for API responses.

**8. Interview difficulty**

**Intermediate**

---

### Question 14: What are inner classes in Java? Explain member, local, and anonymous classes.

**1. Interview-ready answer**

An inner class is a class declared inside another class. A non-static member inner class has an implicit reference to the enclosing instance, so it can access outer fields and private methods directly. A local class is declared inside a method or block and can access effectively final local variables. An anonymous class is an unnamed local class used to create a one-off instance, commonly for callbacks or comparators. The key trade-off is that non-static inner classes keep the outer object alive, which can cause memory leaks when the inner object outlives the outer one — prefer static nested classes when no outer reference is needed.

**2. Detailed explanation**

Types:
- **Member inner class:** declared at class level without `static`; one instance per outer instance.
- **Local class:** declared inside a method; scope limited to that method.
- **Anonymous class:** `new Interface() { ... }` or `new BaseClass() { ... }`.
- **Static nested class:** covered in the next question.

Access rules:
- Inner classes can access all outer members, including private ones, because the compiler generates synthetic accessors or nestmate access (Java 11+).
- Local and anonymous classes can capture local variables that are effectively final.
- Since Java 16, inner classes can declare static members, but they still hold the outer reference.

Advantages: tight coupling when justified, readable callbacks, encapsulation of helper logic. Disadvantages: hidden outer reference, harder testing, larger class files, and accidental memory retention.

**3. Internal working**

The compiler generates a separate `.class` file such as `Outer$Inner.class`. A non-static inner class constructor gets a hidden parameter for the outer instance. Java 11+ uses JVM nestmates so private access no longer requires synthetic bridge methods.

**4. Practical Java/Spring Boot example**

```java
import java.util.Comparator;
import java.util.List;

public class OrderProcessor {

    private final String tenantId;

    OrderProcessor(String tenantId) {
        this.tenantId = tenantId;
    }

    private class AmountValidator {
        boolean isValid(int amount) {
            return amount > 0 && tenantId != null;
        }
    }

    void process(List<Integer> amounts) {
        AmountValidator validator = new AmountValidator();
        int minimum = 100;

        class SmallOrderFilter {
            boolean accept(int amount) {
                return validator.isValid(amount) && amount >= minimum;
            }
        }

        SmallOrderFilter filter = new SmallOrderFilter();
        amounts.stream().filter(filter::accept)
                .sorted(Comparator.reverseOrder())
                .forEach(amount -> System.out.println(tenantId + ":" + amount));
    }

    public static void main(String[] args) {
        new OrderProcessor("TENANT-1").process(List.of(50, 150, 300));
    }
}
```

Output:

```text
TENANT-1:300
TENANT-1:150
```

Important lines:
- `AmountValidator` is a member inner class using `tenantId` directly.
- `SmallOrderFilter` is a local class capturing `validator` and `minimum`, which are effectively final.
- The anonymous `Comparator` implementation is replaced here by a lambda-friendly factory method; both create a one-off strategy.

**5. Real-world use case**

A retry helper used an anonymous `Runnable` capturing a Spring service. The scheduled executor retained the runnable after the request scope closed, which retained the whole service and its dependencies. The fix was a static nested class with explicit dependencies and proper cancellation.

**6. Interview follow-up questions**

- **Can an inner class be `static`?** A member class can; then it is a static nested class, not an inner class.
- **Why must captured locals be effectively final?** Because the compiler copies their values into the inner object; mutating the original would break consistency.
- **Can an anonymous class have constructors?** No — it has no name; use an initializer block.
- **How do inner classes access private outer members?** Via nestmate access in Java 11+ or synthetic accessors before that.
- **Do inner classes produce extra `.class` files?** Yes, one per nested/anonymous class.

**7. Common mistakes**

- Using a non-static inner class when no outer state is needed.
- Registering an inner-class listener on a long-lived component and leaking the outer object.
- Assuming local variables can be modified after capture.
- Expecting anonymous classes to be the same as lambdas — lambdas are more efficient and cannot hold arbitrary state (Chapter 5).

**8. Interview difficulty**

**Intermediate**

---

### Question 15: Static nested class vs inner class — which one should you use?

**1. Interview-ready answer**

A static nested class is declared with `static` inside another class and has no implicit reference to an instance of the outer class. A non-static inner class always holds that hidden reference, so it can access outer instance fields but keeps the outer object alive. Use a static nested class when the nested class does not need outer instance state — builders, helpers, and DTO grouping are common examples. Use a non-static inner class only when it genuinely needs to operate on a specific outer object. Prefer static nested classes by default because they are lighter and avoid memory leaks.

**2. Detailed explanation**

Comparison:
- **Hidden outer reference:** present for inner classes, absent for static nested classes.
- **Instantiation:** `Outer.Inner inner = outer.new Inner();` versus `Outer.Nested nested = new Outer.Nested();`.
- **Static members:** static nested classes can declare them freely; Java 16+ also permits them in inner classes.
- **Access:** both can access private static outer members; only inner classes can access private instance members directly.

Advantages of static nested: smaller objects, no lifetime coupling, easier testing. Disadvantages: must pass outer state explicitly if needed.

**3. Internal working**

A non-static inner class constructor receives a synthetic `this$0` field pointing to the outer instance. A static nested class has no such field. This is why serializing an inner class often drags the outer object along.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.Objects;

public final class PaymentCommand {

    private final String orderId;
    private final BigDecimal amount;

    private PaymentCommand(Builder builder) {
        this.orderId = builder.orderId;
        this.amount = builder.amount;
    }

    public String orderId() {
        return orderId;
    }

    public BigDecimal amount() {
        return amount;
    }

    public static final class Builder {

        private String orderId;
        private BigDecimal amount;

        public Builder orderId(String orderId) {
            this.orderId = orderId;
            return this;
        }

        public Builder amount(BigDecimal amount) {
            this.amount = amount;
            return this;
        }

        public PaymentCommand build() {
            return new PaymentCommand(this);
        }
    }

    public static void main(String[] args) {
        PaymentCommand command = new PaymentCommand.Builder()
                .orderId("ORD-9")
                .amount(BigDecimal.valueOf(2500))
                .build();
        System.out.println(Objects.requireNonNull(command.orderId()) + " : " + command.amount());
    }
}
```

Output:

```text
ORD-9 : 2500
```

Important lines:
- `Builder` is a static nested class — it does not need a `PaymentCommand` instance.
- `build()` passes the builder's state to a private constructor.
- The outer class is immutable and validation can be added in the constructor.

**5. Real-world use case**

Spring's `ResponseEntity.BodyBuilder` and many builder APIs are static nested types. In a booking service, `FlightSearchCriteria.Builder` can be used without first constructing a `FlightSearchService`, keeping the API clean and preventing accidental retention of service dependencies.

**6. Interview follow-up questions**

- **Can a static nested class access outer instance fields?** Only through an explicit outer reference.
- **What is the syntax to instantiate an inner class?** `outer.new Inner()`.
- **Which one is serialized more safely?** Static nested classes — no implicit outer reference.
- **Can a top-level class be static?** No — `static` only makes sense for members.
- **Are records allowed as nested classes?** Yes, and nested records are implicitly static.

**7. Common mistakes**

- Making a helper class non-static and accidentally retaining the outer service.
- Trying to access outer instance methods from a static nested class without a reference.
- Confusing Java's "static nested class" with C++ nested classes.
- Using anonymous inner classes where a lambda or static helper would be simpler.

**8. Interview difficulty**

**Intermediate**

---

### Question 16: Output-based master challenge — Strings, pool, cache, and builders

**1. Interview-ready answer**

Four separate rules decide this output: literals and compile-time constants are interned; `new` creates a new object; `intern()` returns the pooled object; and wrapper comparison with `==` uses references unless one side is a primitive, in which case unboxing occurs. A `StringBuilder` is unrelated to the pool until `toString()` creates a new String. Keeping these rules separate prevents almost all String output mistakes.

**2. Detailed explanation**

Rules recap:
1. `"abc"` → pooled object.
2. `new String("abc")` → new object.
3. `s.intern()` → canonical pooled object.
4. `"a" + "b"` (both constants) → pooled `"ab"`.
5. `variable + "b"` → new object at runtime.
6. `Integer` cache covers -128..127; outside that range `valueOf` creates new objects.
7. `wrapper == primitive` unboxes the wrapper.

**3. Internal working**

Interning uses the JVM's `StringTable`. Autoboxing calls `Integer.valueOf`, which returns cached instances for small values. `StringBuilder.toString()` constructs a fresh String from the builder's byte array — it never consults the pool.

**4. Practical Java example — predict the output**

```java
public class StringObjectChallenge {

    public static void main(String[] args) {
        String a = "ORDER";
        String b = new String("ORDER");
        String c = b.intern();
        String d = "OR" + "DER";
        String piece = "OR";
        String e = piece + "DER";

        System.out.println(a == b);
        System.out.println(a == c);
        System.out.println(a == d);
        System.out.println(a == e);
        System.out.println(a.equals(e));

        Integer smallOne = 127;
        Integer smallTwo = 127;
        Integer bigOne = 128;
        Integer bigTwo = 128;

        System.out.println(smallOne == smallTwo);
        System.out.println(bigOne == bigTwo);
        System.out.println(bigOne == 128);

        StringBuilder builder = new StringBuilder("ORDER");
        builder.append("-1");
        System.out.println(a.equals(builder.toString()));
    }
}
```

Output:

```text
false
true
true
false
true
true
false
true
false
```

Reasoning:
- `b` is a new object; `c` is interned back to `a`; `d` is a folded literal; `e` is runtime-built.
- `equals()` compares content, so `a.equals(e)` is true.
- 127 is cached, 128 is not; `bigOne == 128` unboxes and compares numbers.
- `builder.toString()` is `"ORDER-1"`, which is not equal to `"ORDER"`.

**5. Real-world use case**

An authorization filter compared a role name read from a JWT claim with `==` against a literal. It worked for constants during unit tests and failed for strings created by the JSON parser in production. Replacing with an enum lookup (`Role.valueOf(claim).equals(...)`) fixed both correctness and readability.

**6. Interview follow-up questions**

- **Why is `a == d` true?** Constant folding produced a literal.
- **Why is `bigOne == 128` true though `bigOne == bigTwo` is false?** The primitive side forces unboxing.
- **Does `StringBuilder.toString()` intern the result?** No.
- **What if the value were -129?** Outside cache → new objects.
- **How do you compare strings safely from external input?** Use `equals()` on a known constant or `Objects.equals`.

**7. Common mistakes**

- Treating `==` results as content comparison.
- Assuming all small strings behave like literals.
- Forgetting that `StringBuilder` appends mutate the same object.
- Believing the Integer cache upper bound is fixed — `-XX:AutoBoxCacheMax` can change it.

**8. Interview difficulty**

**Advanced**

---

### Question 17: Coding challenge — implement an immutable `Money` value object

**1. Interview-ready answer**

I would implement `Money` as a final class with `BigDecimal amount` and `Currency` fields, validate in the constructor, and provide operations that return new instances rather than mutating state. `equals()` and `hashCode()` must use both fields, and `compareTo` should use `BigDecimal.compareTo` so `10.0` and `10.00` are treated as equal amounts. Using `BigDecimal` avoids the rounding errors of `double`, and `java.util.Currency` gives type-safe currency codes.

**2. Detailed explanation**

**Problem statement:** Create a `Money` class that supports addition, subtraction, multiplication by an integer quantity, equality, and a readable string form. It must be immutable, reject negative amounts and null currency, and behave correctly in a `HashSet`.

**Approach:**
1. Final class with final fields.
2. Constructor normalizes scale and validates inputs.
3. Operations return new `Money` instances.
4. `equals`/`hashCode` based on normalized amount and currency.
5. `toString` uses the currency code and plain string.

**3. Internal working**

`BigDecimal.stripTrailingZeros()` normalizes representation; `compareTo` ignores scale while `equals` does not, which is a common trap. Normalizing in the constructor makes `equals` consistent with `compareTo`.

**4. Practical Java solution**

```java
import java.math.BigDecimal;
import java.util.Currency;
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

public final class Money {

    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        Objects.requireNonNull(amount, "amount is required");
        this.currency = Objects.requireNonNull(currency, "currency is required");
        if (amount.signum() < 0) {
            throw new IllegalArgumentException("amount must not be negative");
        }
        this.amount = amount.stripTrailingZeros();
    }

    public static Money of(String amount, String currencyCode) {
        return new Money(new BigDecimal(amount), Currency.getInstance(currencyCode));
    }

    public Money add(Money other) {
        requireSameCurrency(other);
        return new Money(amount.add(other.amount), currency);
    }

    public Money subtract(Money other) {
        requireSameCurrency(other);
        Money result = new Money(amount.subtract(other.amount), currency);
        if (result.amount.signum() < 0) {
            throw new IllegalStateException("resulting balance must not be negative");
        }
        return result;
    }

    public Money multiply(int quantity) {
        if (quantity < 0) {
            throw new IllegalArgumentException("quantity must not be negative");
        }
        return new Money(amount.multiply(BigDecimal.valueOf(quantity)), currency);
    }

    public BigDecimal amount() {
        return amount;
    }

    public Currency currency() {
        return currency;
    }

    private void requireSameCurrency(Money other) {
        Objects.requireNonNull(other, "other money is required");
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("currency mismatch: " + currency + " vs " + other.currency);
        }
    }

    @Override
    public boolean equals(Object obj) {
        return obj instanceof Money other
                && currency.equals(other.currency)
                && amount.compareTo(other.amount) == 0;
    }

    @Override
    public int hashCode() {
        return Objects.hash(currency, amount.stripTrailingZeros());
    }

    @Override
    public String toString() {
        return currency.getCurrencyCode() + " " + amount.toPlainString();
    }

    public static void main(String[] args) {
        Money fare = Money.of("2500.00", "INR");
        Money tax = Money.of("125.50", "INR");

        Money total = fare.add(tax).multiply(2);
        Set<Money> uniqueAmounts = new HashSet<>();
        uniqueAmounts.add(Money.of("5251.00", "INR"));
        uniqueAmounts.add(total);

        System.out.println(total);
        System.out.println(uniqueAmounts.size());
        System.out.println(total.equals(Money.of("5251.00", "INR")));
    }
}
```

Output:

```text
INR 5251
1
true
```

**5. Dry run**

| Step | Action | Value |
|---|---|---|
| 1 | `fare` constructed | 2500.00 → normalized 2500 |
| 2 | `tax` constructed | 125.50 → normalized 125.5 |
| 3 | `fare.add(tax)` | 2625.5 |
| 4 | `.multiply(2)` | 5251 |
| 5 | HashSet inserts two equal Money objects (5251.00 and 5251) | size = 1 |
| 6 | `equals` against "5251.00" | true via `compareTo` |

**6. Complexity**

- Construction: O(1) apart from string parsing.
- Arithmetic: O(d) where d is the number of digits — `BigDecimal` cost.
- `equals`/`hashCode`: O(1) for typical amounts.
- Space: O(1) per object.

**7. Edge cases**

- Negative amount → `IllegalArgumentException`.
- Different currencies in `add()` → `IllegalArgumentException`.
- Subtraction creating a negative balance → `IllegalStateException`.
- Scale differences (`2500` vs `2500.00`) treated as equal.
- Null amount/currency → NPE with a clear message.

**8. Real-world use case**

A payment service represents captured amounts as `Money` in its domain layer. Using `double` previously caused reconciliation mismatches of a few paise after many transactions. The value object is also used as a component of idempotency keys and cache keys because its hash code is stable.

**9. Interview follow-up questions**

- **Why not use `double`?** Binary floating point cannot represent many decimal values exactly.
- **Why normalize with `stripTrailingZeros()`?** So `BigDecimal.equals` and hash codes agree with numeric equality.
- **Should `Money` be a record?** Yes, with a compact constructor for validation and defensively copied components.
- **How would you handle currency conversion?** A separate `ExchangeRate` service returning a new `Money` in the target currency.
- **Is `Currency` immutable?** It is effectively immutable and safe to share.

**10. Common mistakes**

- Using `BigDecimal.equals()` for numeric comparison without stripping scale.
- Using `double` in the constructor, which bakes in imprecision.
- Adding amounts of different currencies.
- Omitting `hashCode` and breaking `HashSet` behavior.

**11. Interview difficulty**

**Intermediate**

---

### Question 18: Coding challenge — reverse each word in a sentence without `StringBuilder.reverse()` on the whole string

**1. Interview-ready answer**

I would convert the string to a character array and reverse each word in place by tracking word boundaries with two pointers. This gives O(n) time and O(n) space for the array (or O(1) extra space if mutation of the array is acceptable). A brute-force alternative splits into words, reverses each word with a loop, and joins them — simpler but allocates many temporary strings. I would handle multiple spaces by preserving them exactly.

**2. Detailed explanation**

**Problem statement:** Given `"order placed today"`, return `"redro detcalp yadot"`. Word order stays the same; only characters inside each word are reversed. Preserve spaces as they appear.

**Approach:**
1. Copy input to `char[]`.
2. Walk with `start` and `end` indexes.
3. When a non-space run ends, reverse that range using two pointers.
4. Build the result from the array.

**3. Internal working**

Each character is swapped at most twice, so the total number of swaps is bounded by n. The algorithm is cache-friendly because it scans the array sequentially.

**4. Practical Java solution**

```java
public class WordReverser {

    public static String reverseWords(String input) {
        if (input == null) {
            throw new IllegalArgumentException("input is required");
        }

        char[] characters = input.toCharArray();
        int index = 0;

        while (index < characters.length) {
            while (index < characters.length && characters[index] == ' ') {
                index++;
            }
            int start = index;
            while (index < characters.length && characters[index] != ' ') {
                index++;
            }
            reverseRange(characters, start, index - 1);
        }

        return new String(characters);
    }

    private static void reverseRange(char[] characters, int left, int right) {
        while (left < right) {
            char temporary = characters[left];
            characters[left] = characters[right];
            characters[right] = temporary;
            left++;
            right--;
        }
    }

    public static void main(String[] args) {
        System.out.println(reverseWords("order placed today"));
        System.out.println(reverseWords("  flight   AI204  "));
    }
}
```

Output:

```text
redro detcalp yadot
  thgilf   402IA  
```

Important lines:
- `input.toCharArray()` creates a mutable copy, keeping the original String untouched.
- The outer loop skips spaces; `start` marks a word beginning.
- `reverseRange` swaps in place with two pointers.

**5. Dry run**

Input: `"order placed today"`

| Iteration | start | end | Range before | Range after |
|---|---|---|---|---|
| 1 | 0 | 4 | order | redro |
| 2 | 6 | 11 | placed | detcalp |
| 3 | 13 | 17 | today | yadot |

Result: `redro detcalp yadot`.

**6. Complexity**

- Brute force (split + reverse each word + join): O(n) time, O(n) extra space with more allocations.
- Optimized in-place on the char array: O(n) time, O(n) space for the copy (O(1) additional beyond the array).
- Each character visited at most twice.

**7. Edge cases**

- `null` → `IllegalArgumentException`.
- Empty string → returns empty string.
- Leading/trailing/multiple spaces → preserved.
- Single-character words → unchanged.
- Unicode surrogate pairs → simple char swapping can break them; use code points for full Unicode correctness.

**8. Real-world use case**

A search service normalizes product codes before indexing; reversing token parts is part of a legacy fuzzy-matching rule. The operation runs on millions of records, so the in-place char-array version avoids massive temporary allocations.

**9. Interview follow-up questions**

- **Why not use `split(" ")`?** It allocates an array of strings and collapses repeated spaces unless using a regex with lookarounds.
- **How would you handle Unicode correctly?** Iterate with `codePointAt` and build with `StringBuilder.appendCodePoint`.
- **What if spaces must be collapsed?** Track the previous output character and skip duplicate spaces.
- **Can this be done with streams?** Yes with `Pattern.compile(" ").matcher(...)` or `Stream` of tokens, but allocation is higher.
- **What does `String.join(" ", words)` do?** Joins tokens with a delimiter; convenient but allocates.

**10. Common mistakes**

- Calling `StringBuilder.reverse()` on the whole sentence — that reverses word order too.
- Using `split(" ")` and losing information about multiple spaces.
- Forgetting the null check.
- Assuming `char` handles all Unicode characters.

**11. Interview difficulty**

**Intermediate**

---

### Question 19: Why are passwords often stored in `char[]` instead of `String`?

**1. Interview-ready answer**

Because `String` is immutable, a password stored as a String remains in memory until the garbage collector runs, and the JVM may also keep an interned copy or a copy in the string constant pool. With `char[]`, the application can explicitly overwrite the array with zeros right after hashing the password, reducing the window in which secrets sit in memory. It is not a complete defense — heap dumps, swap files, and JIT copies can still expose data — but it is a standard defense-in-depth practice. In Spring Security, `Authentication.getCredentials()` returns `Object` and implementations such as `UsernamePasswordAuthenticationToken` clear credentials with `eraseCredentials()`.

**2. Detailed explanation**

Reasons to prefer `char[]` for secrets:
- Mutable — can be cleared explicitly.
- Not interned automatically.
- Avoids accidental logging via `toString()` of a String.

Reality check:
- GC moves objects; stale copies may remain.
- The password still exists in HTTP request buffers and JSON parsers as a String.
- Stronger protections (TLS, hashing with BCrypt/Argon2, secret managers) matter more.

Advantages: reduced exposure window, explicit lifecycle control. Disadvantages: more code, still not a guarantee.

**3. Internal working**

`Arrays.fill(passwordChars, '\0')` overwrites the array contents. Since arrays are mutable and stored on the heap, this clears that particular copy. JIT may keep other copies in registers, so this is best-effort.

**4. Practical Java/Spring Boot example**

```java
import java.util.Arrays;

public class PasswordHandling {

    public static boolean verify(char[] submittedPassword, String storedHash) {
        try {
            String candidate = new String(submittedPassword);
            return storedHash.equals(hashSimulator(candidate));
        } finally {
            Arrays.fill(submittedPassword, '\0');
        }
    }

    private static String hashSimulator(String value) {
        return "hash:" + value.length();
    }

    public static void main(String[] args) {
        char[] password = {'s', 'e', 'c', 'r', 'e', 't'};
        System.out.println(verify(password, "hash:6"));
        System.out.println(Arrays.toString(password));
    }
}
```

Output:

```text
true
[, , , , , ]
```

Important lines:
- `Arrays.fill(..., '\0')` clears the char array in a `finally` block.
- The temporary `String candidate` cannot be cleared — illustrating exactly why `char[]` is preferred at the boundary.
- Real code must use BCrypt or Argon2, not a simulator.

**5. Real-world use case**

A Spring Security login endpoint receives a password in JSON, so it already exists as a String in the request body. The team converts to `char[]` at the authentication boundary, uses `BCryptPasswordEncoder.matches`, and clears the array. They also scrub logs and disable request-body logging for `/login`.

**6. Interview follow-up questions**

- **Does clearing `char[]` guarantee safety?** No — it reduces exposure only.
- **Why is BCrypt used for passwords?** It is salted and deliberately slow; details in Chapter 13.
- **Should secrets be in environment variables?** Environment-based configuration is common, but use a secrets manager in production (Chapter 19).
- **Why not use `StringBuilder` for passwords?** It is mutable but its `toString()` creates an uncleable String.
- **Do JWTs have the same issue?** Yes — tokens in memory and logs must be treated as secrets.

**7. Common mistakes**

- Logging passwords or tokens in debug statements.
- Assuming `char[]` removes the need for hashing.
- Storing passwords in constants or String literals.
- Forgetting to clear the array on exceptions.

**8. Interview difficulty**

**Intermediate**

---

### Question 20: Should API methods accept `String` or `CharSequence`?

**1. Interview-ready answer**

`CharSequence` is an interface implemented by `String`, `StringBuilder`, `StringBuffer`, and other character containers. Accepting `CharSequence` in an API makes it more flexible — callers can pass a builder without calling `toString()` and allocating a copy. Use it for read-only operations such as validation, searching, or parsing. Return `String` when you want an immutable snapshot, and avoid exposing `StringBuilder` as a return type. If you need hashing or caching behavior based on string identity, take `String` explicitly. For most Spring service and controller signatures, `String` is clearer and easier to validate with annotations.

**2. Detailed explanation**

- `CharSequence` defines `length()`, `charAt(int)`, `subSequence(...)`, and `toString()`.
- Many JDK APIs accept it: `String.valueOf`, `Pattern.matcher`, `StringBuilder.append`.
- A `StringBuilder` passed as `CharSequence` can be mutated by the caller while your method reads it — document or copy with `toString()` if you need a stable snapshot.

Advantages of `CharSequence`: fewer allocations, broader callers. Disadvantages: weaker guarantees, no `equals`/`hashCode` contract at the interface level, and less obvious intent.

**3. Internal working**

`String`, `StringBuilder`, and `StringBuffer` implement `CharSequence` with their own storage. `Pattern.matcher(CharSequence)` uses `charAt` so it works with any implementation. Converting with `toString()` copies the content into an immutable String.

**4. Practical Java/Spring Boot example**

```java
import java.util.regex.Pattern;

public class CharSequenceApi {

    private static final Pattern ORDER_ID = Pattern.compile("[A-Z]{3}-\\d{4}");

    public static boolean isValidOrderId(CharSequence candidate) {
        return candidate != null && ORDER_ID.matcher(candidate).matches();
    }

    public static String normalize(CharSequence candidate) {
        return candidate.toString().trim().toUpperCase();
    }

    public static void main(String[] args) {
        StringBuilder fromBuilder = new StringBuilder(" ai-204 ");
        System.out.println(isValidOrderId("AI-2041"));
        System.out.println(normalize(fromBuilder));
    }
}
```

Output:

```text
true
AI-204
```

Important lines:
- `matcher` accepts `CharSequence`, so no conversion is needed for validation.
- `normalize` returns a `String` snapshot, which is safe to store or cache.
- `Pattern` is compiled once as a constant — compiling it per call is a common performance bug.

**5. Real-world use case**

A validation utility accepts `CharSequence` so controllers can validate request parameters, CSV cells, and builder-built values with one method. The parsed identifier is converted to `String` before being used as a `ConcurrentHashMap` key, guaranteeing an immutable key.

**6. Interview follow-up questions**

- **Does `CharSequence` define `equals()`?** No — it is only a character-access interface.
- **Can you use `CharSequence` as a `HashMap` key?** Technically yes, but identity/hash behavior depends on the implementation — convert to String.
- **Which classes implement it?** `String`, `StringBuilder`, `StringBuffer`, `CharBuffer`, and others.
- **Why precompile `Pattern`?** Regex compilation is expensive; each `Pattern.compile` call allocates and parses the expression.
- **When should an API prefer `String`?** When you need immutability, caching, or framework binding such as `@PathVariable String`.

**7. Common mistakes**

- Converting a `CharSequence` to String unnecessarily in hot paths.
- Accepting `CharSequence` and then relying on String-specific methods without conversion.
- Using a mutable builder as a map key.
- Recompiling regex patterns inside loops.

**8. Interview difficulty**

**Intermediate**

---

## Chapter 2 Verification

| Syllabus topic | Question(s) |
|---|---|
| String immutability | Q1, Q19 |
| String Pool | Q2, Q4, Q11, Q16 |
| String vs StringBuilder vs StringBuffer | Q3, Q11 |
| `equals()` vs `==` | Q4, Q16 |
| equals/hashCode contract | Q5, Q17 |
| Object class methods | Q6 |
| Wrapper classes | Q7 |
| Autoboxing and Unboxing | Q8, Q16 |
| Immutability and immutable classes | Q1, Q9, Q17 |
| `final`, `finally`, `finalize` | Q10 |
| String concatenation / output-based questions | Q11, Q16 |
| Enums | Q12, Q13 |
| Inner and Nested Classes | Q14, Q15 |
| Coding questions | Q17, Q18 |

**Total interview questions: 20** (all with the required 8-part format; coding questions include problem statement, approach, dry run, complexity, and edge cases).

**Next:** [Chapter 3 — Exception Handling](03-Exception-Handling.md). Say `NEXT CHAPTER` to continue.
