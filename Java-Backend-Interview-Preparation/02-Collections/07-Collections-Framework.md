ne# Chapter 7: Collections Framework

**Chapter goal:** Answer collection hierarchy, implementation-internals, complexity, iteration, and real-world selection questions confidently — especially `HashMap`, `ConcurrentHashMap`, `LinkedHashMap`, and fail-fast behavior.

**Topics covered:** Collections hierarchy · `List`, `Set`, `Queue`, `Map` · `ArrayList`, `LinkedList`, `ArrayDeque` · `HashSet`, `LinkedHashSet`, `TreeSet` · `HashMap` internals, hashing, collisions, resizing, treeification · `Hashtable`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap` · `PriorityQueue` · iterators, `ListIterator`, `Spliterator` · fail-fast vs weakly consistent · complexity and choosing the right collection.

**Cross-references:** `equals`/`hashCode` rules are in [Chapter 2](../01-Core-Java/02-Strings-and-Object-Handling.md). Stream operations over collections are in [Chapter 5](../01-Core-Java/05-Java-8-Features-and-Streams.md). Concurrent collections and locks are in [Chapter 8](../03-Concurrency/08-Multithreading-and-Concurrency.md).

---

## Part A — Hierarchy and core interfaces

### Question 1: Explain the Java Collections Framework hierarchy

**1. Interview-ready answer**

The framework has two top-level families: `Collection` for groups of elements and `Map` for key-value pairs. `Collection` extends `Iterable` and branches into `List` (ordered, duplicates), `Set` (unique elements), and `Queue` (ordered processing, with `Deque` for two-ended access). `Map` is not a `Collection`; it has its own hierarchy. Utilities such as `Collections` and `Arrays` provide algorithms and wrappers.

**2. Detailed explanation**

```text
Iterable
└── Collection
    ├── List: ArrayList, LinkedList, Vector
    ├── Set: HashSet, LinkedHashSet, TreeSet, EnumSet
    └── Queue: PriorityQueue, ArrayDeque, LinkedList
        └── Deque: ArrayDeque, LinkedList

Map
├── HashMap
├── LinkedHashMap
├── TreeMap
├── Hashtable
└── ConcurrentHashMap
```

| Interface | Allows duplicates | Ordering | Typical implementations |
|---|---|---|---|
| `List` | Yes | Insertion/index order | `ArrayList`, `LinkedList` |
| `Set` | No | Depends on implementation | `HashSet`, `LinkedHashSet`, `TreeSet` |
| `Queue` | Yes | Queue policy | `PriorityQueue`, `ArrayDeque` |
| `Map` | Unique keys | Depends on implementation | `HashMap`, `LinkedHashMap`, `TreeMap` |

**3. Internal working**

Interfaces define contracts, not data structures. For example, `Set` promises uniqueness but does not require a specific order. Implementations choose trade-offs: `HashSet` uses a hash table, `LinkedHashSet` adds insertion order, and `TreeSet` uses a red-black tree.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Deque;
import java.util.HashMap;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class CollectionsHierarchyDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>(List.of("ORD-1", "ORD-2"));
        Set<String> uniqueOrderIds = new LinkedHashSet<>(List.of("ORD-1", "ORD-1", "ORD-2"));
        Map<String, Integer> quantities = new HashMap<>();
        Deque<String> retryQueue = new ArrayDeque<>();

        quantities.put("ORD-1", 2);
        retryQueue.add("ORD-1");

        System.out.println(orderIds);
        System.out.println(uniqueOrderIds);
        System.out.println(quantities);
        System.out.println(retryQueue.remove());
    }
}
```

Output:

```text
[ORD-1, ORD-2]
[ORD-1, ORD-2]
{ORD-1=2}
ORD-1
```

**5. Real-world use case**

A Spring controller accepts a `List<OrderRequest>`, a validation layer converts tags to a `Set<String>`, and a service caches computed prices in a `Map<String, BigDecimal>`.

**6. Interview follow-up questions**

- **Is `Map` a `Collection`?** No.
- **Which interface does `HashSet` implement?** `Set`, backed by a `HashMap`.
- **Why program to interfaces?** You can swap implementations without changing callers.
- **Are collections serializable?** Most standard implementations are, but their stored elements must be too.
- **What is `Iterable` for?** It enables the enhanced for-loop and provides an `Iterator`.

**7. Common mistakes**

- Declaring concrete types everywhere.
- Assuming all `Set`s are sorted.
- Thinking `Map` can be iterated like a `Collection` directly.
- Using `Vector`/`Hashtable` in new code.

**8. Interview difficulty**

**Beginner**

---

### Question 2: How do you choose between `List`, `Set`, `Queue`, and `Map`?

**1. Interview-ready answer**

Use a `List` when position and duplicates matter, a `Set` when uniqueness matters, a `Queue` when elements must be processed in a defined order, and a `Map` when you need lookup by key. Then pick the implementation based on ordering, thread safety, and access patterns.

**2. Detailed explanation**

| Need | Choose | Why |
|---|---|---|
| Preserve request order, allow duplicates | `ArrayList` | Indexed, compact, fast iteration |
| Unique tags, no order needed | `HashSet` | O(1) average add/contains |
| Unique tags in user-visible order | `LinkedHashSet` | Insertion order preserved |
| Sorted unique codes | `TreeSet` | Navigable ordering |
| FIFO job processing | `ArrayDeque` | Efficient queue operations |
| Priority-based processing | `PriorityQueue` | Heap ordering |
| Key → value lookup | `HashMap` | O(1) average lookup |
| LRU cache | `LinkedHashMap` | Access-order iteration |
| Concurrent lookup table | `ConcurrentHashMap` | Thread-safe, scalable |

**3. Internal working**

The wrong abstraction causes bugs even when code compiles: using a `List` for unique IDs allows duplicates; using a `HashSet` for ordered output loses order; using a `HashMap` from multiple threads without synchronization corrupts state.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

record Order(String id, String customerId, BigDecimal amount) {
}

public class CollectionChoiceDemo {

    public static void main(String[] args) {
        List<Order> orders = new ArrayList<>();
        orders.add(new Order("ORD-1", "CUST-1", BigDecimal.valueOf(100)));
        orders.add(new Order("ORD-2", "CUST-1", BigDecimal.valueOf(200)));

        Set<String> customerIds = new LinkedHashSet<>();
        Map<String, BigDecimal> totals = new HashMap<>();

        for (Order order : orders) {
            customerIds.add(order.customerId());
            totals.merge(order.customerId(), order.amount(), BigDecimal::add);
        }

        System.out.println(customerIds);
        System.out.println(totals);
    }
}
```

Output:

```text
[CUST-1]
{CUST-1=300}
```

**5. Real-world use case**

An order aggregation service uses `List` for the incoming page, `Set` to deduplicate customer IDs, and `Map.merge` to compute totals in one pass.

**6. Interview follow-up questions**

- **When would you use `EnumSet`?** For enums; it is a compact bitset with very fast operations.
- **Why not `LinkedList` for a queue?** `ArrayDeque` is usually faster and uses fewer allocations.
- **Can a `Set` preserve order?** `LinkedHashSet` and `TreeSet` do; `HashSet` does not.
- **Which map preserves insertion order?** `LinkedHashMap`.
- **Which map gives sorted keys?** `TreeMap`.

**7. Common mistakes**

- Using `List.contains` in a loop — O(n) per lookup.
- Using `HashSet` when deterministic response order is required.
- Using `HashMap` in a shared singleton service without synchronization.
- Choosing `TreeMap` when sorting is not needed.

**8. Interview difficulty**

**Beginner**

---

### Question 3: How does `ArrayList` work internally?

**1. Interview-ready answer**

`ArrayList` wraps a dynamic `Object[]` called `elementData` and an integer `size`. `add` appends at `size` and grows the array by roughly 1.5× when full using `Arrays.copyOf`. Random access is O(1), amortized append is O(1), and insertion/removal in the middle is O(n) because elements must shift.

**2. Detailed explanation**

Important fields conceptually:

```text
Object[] elementData
int size
int modCount
```

- `DEFAULT_CAPACITY` is 10 when no capacity is specified.
- `ensureCapacity` checks whether a new element fits.
- `grow` creates a larger array and copies existing references.
- `remove(index)` uses `System.arraycopy` to shift left and nulls the last slot for GC.
- `modCount` is incremented on structural changes so iterators can detect concurrent modification.

**3. Internal working**

Appending is cheap because it writes one slot, but growth costs O(n). Because growth happens rarely, amortized append is constant time. Middle insertion must shift all following elements, so it is linear even though the actual insertion is one assignment. Access by index directly reads `elementData[index]`.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListGrowthDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>(4);

        orderIds.add("ORD-1");
        orderIds.add("ORD-2");
        orderIds.add("ORD-3");
        orderIds.add("ORD-4");
        orderIds.add("ORD-5");

        orderIds.remove("ORD-2");

        System.out.println(orderIds);
        System.out.println(orderIds.get(2));
    }
}
```

Output:

```text
[ORD-1, ORD-3, ORD-4, ORD-5]
ORD-4
```

Providing an initial capacity avoids growth when the approximate size is known.

**5. Real-world use case**

A batch importer reads 100,000 rows. `new ArrayList<>(rows.size())` avoids repeated array copies during loading.

**6. Interview follow-up questions**

- **What is ArrayList's growth factor?** About 1.5× in HotSpot.
- **Is ArrayList thread-safe?** No.
- **What happens to removed references?** The last slot is nulled so the object can be collected.
- **Why is random access fast?** Contiguous array and index arithmetic.
- **Can ArrayList hold nulls?** Yes, multiple nulls.

**7. Common mistakes**

- Deleting from an `ArrayList` inside a forward index loop.
- Assuming `remove(Object)` is O(1) — it must search first.
- Not sizing large lists.
- Using `ArrayList` for frequent middle insertions.

**8. Interview difficulty**

**Beginner**

---

### Question 4: When would you use `ArrayList` versus `LinkedList`?

**1. Interview-ready answer**

Use `ArrayList` for almost everything: indexed access, iteration, and append-heavy workloads benefit from contiguous memory and cache locality. `LinkedList` wins only when you already hold a node/iterator position and need constant-time insert/remove there, or when you need both stack and queue behavior — though `ArrayDeque` is usually better for that.

**2. Detailed explanation**

| Operation | `ArrayList` | `LinkedList` |
|---|---|---|
| `get(index)` | O(1) | O(n) |
| Append | O(1) amortized | O(1) |
| Insert at known iterator position | O(n) shift | O(1) |
| Insert by index | O(n) | O(n) to find + O(1) to link |
| Iteration | Cache-friendly | Pointer chasing |
| Memory per element | One reference | Node with prev/next/data |

**3. Internal working**

`LinkedList` is a doubly linked list of `Node` objects. Each node has three references. Even if insertion at a known node is O(1), finding that node by index is O(n). `ArrayList` stores references contiguously, which modern CPUs prefetch efficiently.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Deque;
import java.util.List;

public class ListChoiceDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>();
        orderIds.add("ORD-1");
        orderIds.add("ORD-2");

        Deque<String> retryQueue = new ArrayDeque<>();
        retryQueue.addLast("ORD-1");
        retryQueue.addLast("ORD-2");

        System.out.println(orderIds.get(0));
        System.out.println(retryQueue.removeFirst());
    }
}
```

Output:

```text
ORD-1
ORD-1
```

**5. Real-world use case**

A retry worker uses `ArrayDeque` for FIFO jobs and an `ArrayList` for the response page. A linked list would allocate a node per job without improving the access pattern.

**6. Interview follow-up questions**

- **Which is better as a stack?** `ArrayDeque`, not `Stack` or `LinkedList`.
- **Is LinkedList always O(1) for insert?** Only when the position is already known.
- **Why is ArrayList cache-friendly?** Contiguous array storage.
- **Does LinkedList use less memory for few elements?** Usually no; node overhead dominates.
- **Is LinkedList deprecated?** No, but often unnecessary.

**7. Common mistakes**

- Choosing LinkedList for frequent `get(i)`.
- Using legacy `Stack`.
- Assuming node-based structures are always faster for inserts.
- Ignoring memory overhead.

**8. Interview difficulty**

**Beginner**

---

### Question 5: Compare `HashSet`, `LinkedHashSet`, and `TreeSet`

**1. Interview-ready answer**

`HashSet` stores unique elements in a hash table with O(1) average add/contains and no order guarantee. `LinkedHashSet` adds a linked list to preserve insertion order with slightly more memory. `TreeSet` uses a red-black tree, guarantees sorted order, and provides O(log n) operations plus navigational methods.

**2. Detailed explanation**

| Set | Backing structure | Order | Average complexity | Nulls |
|---|---|---|---|---|
| `HashSet` | `HashMap` | None | O(1) | One null |
| `LinkedHashSet` | `LinkedHashMap` | Insertion | O(1) | One null |
| `TreeSet` | `TreeMap` | Natural/Comparator | O(log n) | None by default |

**3. Internal working**

`HashSet.add(e)` delegates to `HashMap.put(e, PRESENT)`. `LinkedHashSet` maintains before/after links around each entry. `TreeSet` compares elements with `Comparable` or a `Comparator`; comparison must be consistent with `equals` if you care about uniqueness semantics.

**4. Practical Java/Spring Boot example**

```java
import java.util.Comparator;
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;
import java.util.TreeSet;

public class SetChoiceDemo {

    public static void main(String[] args) {
        Set<String> unique = new HashSet<>(java.util.List.of("ORD-2", "ORD-1", "ORD-2"));
        Set<String> insertionOrdered = new LinkedHashSet<>(unique);
        Set<String> sorted = new TreeSet<>(Comparator.naturalOrder());
        sorted.addAll(unique);

        System.out.println(unique);
        System.out.println(insertionOrdered);
        System.out.println(sorted);
    }
}
```

Output may be:

```text
[ORD-1, ORD-2]
[ORD-1, ORD-2]
[ORD-1, ORD-2]
```

`HashSet` order is implementation-dependent; `LinkedHashSet` and `TreeSet` orders are guaranteed.

**5. Real-world use case**

A notification service deduplicates recipient IDs with `HashSet`, preserves user-visible tag order with `LinkedHashSet`, and shows sorted audit codes with `TreeSet`.

**6. Interview follow-up questions**

- **Why does HashSet allow only one null?** A map can have one null key.
- **How does TreeSet decide duplicates?** By compare result `0`.
- **Which set is fastest?** `HashSet` typically.
- **Which set supports `floor`/`ceiling`?** `TreeSet`.
- **Can you make HashSet sorted?** No; use TreeSet or sort when reading.

**7. Common mistakes**

- Relying on HashSet iteration order.
- Using mutable objects as set elements.
- Forgetting Comparator in TreeSet for custom classes.
- Using TreeSet where O(1) lookup is needed.

**8. Interview difficulty**

**Beginner**

---

---

## Part B — Hash-based collections

### Question 6: Explain `HashMap` internals

**1. Interview-ready answer**

`HashMap` stores entries in an array of buckets. It applies an improved hash function `h ^ (h >>> 16)` and computes the bucket index with `(n - 1) & hash`. Collisions are handled by chaining nodes in a linked list; Java 8 converts long chains to red-black trees when the bucket has at least 8 nodes and the table has at least 64 buckets. Lookup compares hash first, then `equals`.

**2. Detailed explanation**

Key fields conceptually:

```text
Node<K,V>[] table
int size
int threshold
float loadFactor
int modCount
```

Steps for `put(key, value)`:

1. Compute the spread hash.
2. If the table is null, initialize it.
3. Compute index with `(table.length - 1) & hash`.
4. If the bucket is empty, place a new node there.
5. If the first node matches, replace its value.
6. If the bucket is a tree, use tree insertion.
7. Otherwise walk the chain, replacing an equal key or appending a new node.
8. Increment `size` and resize if `size > threshold`.

**3. Internal working**

The spread hash mixes high bits into low bits so a power-of-two table does not suffer from poor low-bit distributions. `equals` is still required because distinct keys can share a hash. Treeification changes worst-case lookup from O(n) in a bucket to O(log n), but it costs more memory per node.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashMap;
import java.util.Map;

record OrderKey(String tenantId, String orderId) {
}

public class HashMapInternalsDemo {

    public static void main(String[] args) {
        Map<OrderKey, Integer> quantities = new HashMap<>();

        quantities.put(new OrderKey("T1", "ORD-1"), 2);
        quantities.put(new OrderKey("T1", "ORD-1"), 5);
        quantities.put(new OrderKey("T2", "ORD-1"), 1);

        System.out.println(quantities.get(new OrderKey("T1", "ORD-1")));
        System.out.println(quantities.size());
    }
}
```

Output:

```text
5
2
```

Records implement `equals` and `hashCode` from all components, making them safe keys when components are immutable.

**5. Real-world use case**

A pricing service keys cached prices by `(tenantId, productId)` to avoid cross-tenant data leakage.

**6. Interview follow-up questions**

- **Why is table size a power of two?** Bitmask indexing is fast and distributes spreads evenly.
- **Can two unequal keys have the same hash?** Yes — that is a collision.
- **What happens on equal hash but unequal keys?** `equals` decides; chain/tree traversal continues.
- **Why treeify?** To defend against many colliding keys and hash-flooding.
- **Are HashMap keys stored sorted?** No.

**7. Common mistakes**

- Using mutable fields in keys.
- Implementing `equals` without `hashCode`.
- Assuming hash collisions mean equal keys.
- Using `HashMap` concurrently without protection.

**8. Interview difficulty**

**Intermediate**

---

### Question 7: Explain `HashMap` resizing, load factor, and treeification

**1. Interview-ready answer**

`HashMap` resizes when `size` exceeds `capacity × loadFactor`. The default capacity is 16 and load factor 0.75, so the first resize happens after the 12th entry. Resizing doubles the bucket array and redistributes nodes; Java 8 splits each chain into low and high lists based on a bit of the hash rather than recomputing every index from scratch. Treeification occurs at 8 nodes in a bucket only when the table has at least 64 buckets.

**2. Detailed explanation**

| Concept | Detail |
|---|---|
| Default capacity | 16 |
| Default load factor | 0.75 |
| Initial threshold | 12 |
| Resize | Doubles capacity |
| Treeify threshold | 8 nodes in a bucket and table size ≥ 64 |
| Untreeify threshold | 6 nodes |
| Table max capacity | 1 << 30 |

Load factor trades memory against collisions: lower values reduce collisions but waste memory; higher values save memory but lengthen chains.

**3. Internal working**

During resize, a node with hash bit `0` stays in the same index; a node with bit `1` moves to `oldIndex + oldCapacity`. This preserves relative order and avoids rehashing each key. Tree nodes may be split into two trees or converted back to lists if they become small.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapCapacityDemo {

    public static void main(String[] args) {
        int expectedEntries = 1000;
        int initialCapacity = (int) (expectedEntries / 0.75f) + 1;

        Map<String, String> cache = new HashMap<>(initialCapacity);

        for (int i = 0; i < expectedEntries; i++) {
            cache.put("SKU-" + i, "price-" + i);
        }

        System.out.println(cache.size());
    }
}
```

Output:

```text
1000
```

Pre-sizing avoids repeated rehash work during startup cache warming.

**5. Real-world use case**

An application caches 50,000 SKU prices on startup. Pre-sizing the map makes startup faster and reduces temporary garbage.

**6. Interview follow-up questions**

- **Why 0.75 default?** A good compromise between space and collision probability.
- **What happens if initial capacity is 1000?** HashMap rounds up to the next power of two.
- **Does treeification happen at 8 nodes in a small table?** No; the table is resized first.
- **Can load factor be greater than 1?** Yes, but collisions increase.
- **Is resize thread-safe?** No.

**7. Common mistakes**

- Sizing to expected entries instead of expected / 0.75.
- Setting load factor to 1 to “save memory.”
- Assuming resize only copies values.
- Using `HashMap` in a concurrent cache without synchronization.

**8. Interview difficulty**

**Intermediate**

---

### Question 8: Compare `HashMap`, `Hashtable`, `Collections.synchronizedMap`, and `ConcurrentHashMap`

**1. Interview-ready answer**

`HashMap` is fast and unsynchronized, allowing one null key and many null values. `Hashtable` is a legacy synchronized map that rejects nulls and locks the whole map. `Collections.synchronizedMap` wraps any map with a single coarse lock and requires manual synchronization during iteration. `ConcurrentHashMap` is the modern concurrent choice: it rejects nulls, uses fine-grained synchronization, and provides weakly consistent iterators.

**2. Detailed explanation**

| Map | Thread-safe | Nulls | Locking | Iteration |
|---|---|---|---|---|
| `HashMap` | No | One key, many values | None | Fail-fast |
| `Hashtable` | Yes | None | Whole map | Fail-fast |
| `synchronizedMap` | Yes | Depends on backing map | Whole map wrapper | Must synchronize manually |
| `ConcurrentHashMap` | Yes | None | Bin-level CAS/synchronized | Weakly consistent |

**3. Internal working**

`Hashtable` methods are `synchronized`, so concurrent writes serialize. `ConcurrentHashMap` allows concurrent reads and updates to different bins, dramatically improving throughput. Weakly consistent iterators never throw `ConcurrentModificationException` and may or may not reflect concurrent updates.

**4. Practical Java/Spring Boot example**

```java
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class MapThreadSafetyDemo {

    public static void main(String[] args) {
        Map<String, Integer> synchronizedMap =
                Collections.synchronizedMap(new HashMap<>());

        synchronized (synchronizedMap) {
            synchronizedMap.put("ORD-1", 1);
            for (Map.Entry<String, Integer> entry : synchronizedMap.entrySet()) {
                System.out.println(entry.getKey() + "=" + entry.getValue());
            }
        }

        ConcurrentMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("ORD-1", 1);
        concurrentMap.computeIfPresent("ORD-1", (key, value) -> value + 1);

        System.out.println(concurrentMap.get("ORD-1"));
    }
}
```

Output:

```text
ORD-1=1
2
```

**5. Real-world use case**

A singleton metrics registry shared by request threads should use `ConcurrentHashMap`; a legacy synchronized wrapper around `HashMap` becomes a throughput bottleneck.

**6. Interview follow-up questions**

- **Why does ConcurrentHashMap reject nulls?** Null would be ambiguous with “absent” in concurrent `get`/`compute`.
- **Is ConcurrentHashMap iterator strongly consistent?** No, weakly consistent.
- **Is `Collections.synchronizedMap` enough for compound actions?** No; synchronize externally.
- **Is Hashtable deprecated?** Not deprecated, but legacy and generally avoided.
- **Which map for read-heavy local cache?** `ConcurrentHashMap` or Caffeine.

**7. Common mistakes**

- Putting nulls in a `ConcurrentHashMap`.
- Iterating a synchronizedMap without external synchronization.
- Assuming ConcurrentHashMap makes multi-step logic atomic.
- Using Hashtable in new code.

**8. Interview difficulty**

**Intermediate**

---

### Question 9: How does `ConcurrentHashMap` work internally?

**1. Interview-ready answer**

In Java 8+, `ConcurrentHashMap` uses a `Node[]` table similar to `HashMap`. Empty-bin insertion uses CAS; non-empty bins are updated by synchronizing on the first node. During resize, forwarding nodes help other threads migrate bins. Size accounting uses base count plus `CounterCell`s, similar to `LongAdder`, to reduce contention.

**2. Detailed explanation**

Java 7 used a fixed number of segments, each effectively a separate HashMap. Java 8 removed segments and made synchronization finer-grained per bin, allowing better scalability.

Key mechanisms:

- **CAS:** Atomically installs a node into an empty bucket.
- **`synchronized` first node:** Serializes updates within one bucket only.
- **`ForwardingNode`:** Marks a migrated bucket during resize and helps transfers.
- **`sizeCnt` / `CounterCell`:** Distributes size increments across cells.
- **Weakly consistent iterators:** Traverse a snapshot-like view without locking.

**3. Internal working**

`putVal` computes the spread hash and loops. If the bin is empty, `casTabAt` installs the node. If a resize is in progress, the thread may help move bins. If the bin has nodes, it synchronizes on the first node and performs chain/tree insertion. `computeIfAbsent` holds the bin lock while computing the mapping function, so the function must be short and must not recursively update the same map.

**4. Practical Java/Spring Boot example**

```java
import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.stream.IntStream;

public class ConcurrentHashMapDemo {

    public static void main(String[] args) {
        ConcurrentMap<String, BigDecimal> totals = new ConcurrentHashMap<>();
        List<String> customerIds = IntStream.rangeClosed(1, 1000)
                .mapToObj(i -> "CUST-" + (i % 10))
                .toList();

        customerIds.parallelStream().forEach(customerId ->
                totals.merge(customerId, BigDecimal.ONE, BigDecimal::add));

        System.out.println(totals.get("CUST-1"));
    }
}
```

Output:

```text
100
```

`merge` is atomic per key and avoids a check-then-act race.

**5. Real-world use case**

A rate-limiter filter stores per-IP counters in a `ConcurrentHashMap` shared by all request threads.

**6. Interview follow-up questions**

- **Is ConcurrentHashMap lock-free?** No; it mixes CAS and short synchronized sections.
- **Can iterators throw ConcurrentModificationException?** No.
- **Why must mapping functions be short?** They run while a bin is locked.
- **Does `size()` need a global lock?** No; it sums base count and cells.
- **How do you atomically update only if present?** `computeIfPresent` or `compute`.

**7. Common mistakes**

- Using `get` followed by `put` as if atomic.
- Recursive `computeIfAbsent` on the same map.
- Storing null values.
- Assuming iteration reflects a single point in time.

**8. Interview difficulty**

**Advanced**

---

### Question 10: How do you build an LRU cache with `LinkedHashMap`?

**1. Interview-ready answer**

`LinkedHashMap` supports `accessOrder = true`, which moves an entry to the tail whenever `get` is called. Override `removeEldestEntry` to evict the head once size exceeds the limit. This gives a simple, single-threaded LRU cache; for concurrent use, wrap it with a lock or use Caffeine.

**2. Detailed explanation**

Constructor: `new LinkedHashMap<>(initialCapacity, loadFactor, accessOrder)`.

- `accessOrder = false`: iteration follows insertion order.
- `accessOrder = true`: iteration follows least-recently-accessed to most-recently-accessed.
- `removeEldestEntry` is called after each insertion.
- Returning `true` removes the eldest entry.

**3. Internal working**

Each `LinkedHashMap.Entry` has `before` and `after` references forming a doubly linked list. With access order enabled, `get` calls `afterNodeAccess`, which moves the entry to the tail. Eviction removes the head entry from both the hash table and linked list.

**4. Practical Java/Spring Boot example**

```java
import java.util.LinkedHashMap;
import java.util.Map;

public final class LruPriceCache<K, V> extends LinkedHashMap<K, V> {

    private final int maxEntries;

    public LruPriceCache(int maxEntries) {
        super(maxEntries + 1, 0.75f, true);
        if (maxEntries <= 0) {
            throw new IllegalArgumentException("maxEntries must be positive");
        }
        this.maxEntries = maxEntries;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxEntries;
    }

    public static void main(String[] args) {
        LruPriceCache<String, String> cache = new LruPriceCache<>(2);

        cache.put("SKU-1", "100");
        cache.put("SKU-2", "200");
        cache.get("SKU-1");
        cache.put("SKU-3", "300");

        System.out.println(cache);
    }
}
```

Output:

```text
{SKU-1=100, SKU-3=300}
```

Accessing SKU-1 made SKU-2 the least recently used entry, so SKU-2 was evicted.

**5. Real-world use case**

A local cache stores the last 500 calculated product prices before falling back to Redis or the database.

**6. Interview follow-up questions**

- **Is this thread-safe?** No; synchronize access or use Caffeine.
- **Why capacity `maxEntries + 1`?** The extra slot lets insertion happen before eviction.
- **What is the difference from FIFO?** FIFO ignores reads; LRU updates on access.
- **Can you use it for sessions?** Only for small single-node caches.
- **How do you add TTL?** Track timestamps or use a cache library.

**7. Common mistakes**

- Forgetting `accessOrder = true`.
- Returning `size() >= maxEntries` — evicts too early.
- Using it concurrently without synchronization.
- Putting large objects with no expiration policy.

**8. Interview difficulty**

**Intermediate**

---

---

## Part C — Sorted, priority, and iteration behavior

### Question 11: How does `TreeMap` work internally?

**1. Interview-ready answer**

`TreeMap` is a red-black tree implementation of `NavigableMap`. Keys must be comparable through `Comparable` or a supplied `Comparator`. Operations such as `get`, `put`, and `remove` are O(log n). It supports ordered navigation with `firstKey`, `lastKey`, `floorKey`, `ceilingKey`, `subMap`, `headMap`, and `tailMap`.

**2. Detailed explanation**

A red-black tree is a self-balancing binary search tree with invariants that keep height O(log n):

- Every node is red or black.
- The root is black.
- Red nodes have black children.
- Every path from a node to null children has the same number of black nodes.

These rules prevent the tree from degenerating into a linked list for sorted input.

**3. Internal working**

Comparison, not hashing, determines the position of a key. A compare result of `0` means the same key for the map, even if `equals` would return false. That is why comparator consistency with `equals` matters when a `TreeMap` is used as a set/map of domain keys.

**4. Practical Java/Spring Boot example**

```java
import java.util.Map;
import java.util.NavigableMap;
import java.util.TreeMap;

public class TreeMapDemo {

    public static void main(String[] args) {
        NavigableMap<Integer, String> schedule = new TreeMap<>();
        schedule.put(1, "validate");
        schedule.put(3, "settle");
        schedule.put(5, "notify");

        Map.Entry<Integer, String> next = schedule.ceilingEntry(2);
        NavigableMap<Integer, String> window = schedule.subMap(1, true, 5, false);

        System.out.println(next);
        System.out.println(window);
        System.out.println(schedule.pollFirstEntry());
    }
}
```

Output:

```text
3=settle
{1=validate, 3=settle}
1=validate
```

**5. Real-world use case**

A scheduler stores retry tasks by delay in milliseconds and repeatedly polls the earliest task with `pollFirstEntry`.

**6. Interview follow-up questions**

- **What if key has no Comparable?** `TreeMap` throws `ClassCastException` at insertion unless a Comparator is given.
- **Is TreeMap thread-safe?** No.
- **What is complexity of iteration?** O(n) in sorted order.
- **Which is faster, HashMap or TreeMap?** HashMap is usually O(1) vs O(log n).
- **How do you reverse order?** `new TreeMap<>(Comparator.reverseOrder())`.

**7. Common mistakes**

- Comparator inconsistent with equals.
- Using mutable key fields that change ordering after insertion.
- Assuming TreeMap allows null keys with natural ordering.
- Using TreeMap when no ordering is required.

**8. Interview difficulty**

**Intermediate**

---

### Question 12: How does `PriorityQueue` work and when should you use it?

**1. Interview-ready answer**

`PriorityQueue` is an unbounded min-heap backed by a dynamically resizing array. `peek` is O(1), and `offer`/`poll` are O(log n). Iteration is not sorted. Use it when you repeatedly need the smallest or largest element according to a comparator, such as top-k results or priority scheduling. Use `PriorityBlockingQueue` for concurrent producers and consumers.

**2. Detailed explanation**

The heap is a complete binary tree stored in an array:

```text
index 0 -> root (smallest by natural/comparator order)
parent = (i - 1) / 2
left = 2i + 1
right = 2i + 2
```

`offer` appends at the end and sifts up. `poll` removes the root, moves the last element to the root, and sifts down.

**3. Internal working**

No array slot is permanently tied to a sorted position. Only the heap property — parent less than or equal to children — is maintained. To get a largest-first queue, pass `Comparator.reverseOrder()`.

**4. Practical Java/Spring Boot example**

```java
import java.util.Comparator;
import java.util.PriorityQueue;
import java.util.Queue;

record Job(int priority, String id) {
}

public class PriorityQueueDemo {

    public static void main(String[] args) {
        Queue<Job> jobs = new PriorityQueue<>(Comparator.comparingInt(Job::priority));

        jobs.offer(new Job(3, "email"));
        jobs.offer(new Job(1, "payment"));
        jobs.offer(new Job(2, "audit"));

        while (!jobs.isEmpty()) {
            System.out.println(jobs.poll().id());
        }
    }
}
```

Output:

```text
payment
audit
email
```

**5. Real-world use case**

A notification dispatcher processes high-priority alerts before bulk emails, using a `PriorityBlockingQueue` shared by worker threads.

**6. Interview follow-up questions**

- **Is PriorityQueue sorted?** No; only the head is guaranteed.
- **Does it allow nulls?** No.
- **Does it allow duplicates?** Yes.
- **How do you get top k largest?** Keep a min-heap of size k or use reverse comparator.
- **Is it thread-safe?** No; use `PriorityBlockingQueue`.

**7. Common mistakes**

- Iterating and assuming sorted output.
- Using natural order when a comparator is required.
- Using it for FIFO — use `ArrayDeque`.
- Blocking on `poll` expecting waiting behavior; use the blocking variant.

**8. Interview difficulty**

**Beginner**

---

### Question 13: Why prefer `ArrayDeque` over `Stack` or `LinkedList`?

**1. Interview-ready answer**

`ArrayDeque` is a resizable circular array implementing `Deque`, with O(1) amortized add/remove at both ends and good cache locality. `Stack` is a legacy class extending `Vector`, so its methods are synchronized and it carries old design baggage. `LinkedList` allocates a node per element and has poor locality. For stacks and queues, `ArrayDeque` is usually the best default.

**2. Detailed explanation**

| Use | Avoid | Prefer |
|---|---|---|
| Stack | `Stack` | `ArrayDeque` with `push`/`pop` |
| FIFO queue | `LinkedList` | `ArrayDeque` with `add`/`remove` |
| Deque | `LinkedList` | `ArrayDeque` |
| Indexed list | `LinkedList` | `ArrayList` |

`ArrayDeque` rejects null elements, which helps avoid ambiguity with `poll()` returning null when empty.

**3. Internal working**

A circular array keeps `head` and `tail` indexes. When an end reaches the array boundary, it wraps to index 0. Growth doubles the array and copies elements in order. No node allocation occurs per operation.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeDemo {

    public static void main(String[] args) {
        Deque<String> stack = new ArrayDeque<>();
        stack.push("ORD-1");
        stack.push("ORD-2");

        Deque<String> queue = new ArrayDeque<>();
        queue.offer("job-1");
        queue.offer("job-2");

        System.out.println(stack.pop());
        System.out.println(queue.poll());
    }
}
```

Output:

```text
ORD-2
job-1
```

**5. Real-world use case**

A recursive category traversal uses an explicit `ArrayDeque` stack to avoid deep Java recursion.

**6. Interview follow-up questions**

- **Does ArrayDeque allow null?** No.
- **Is it thread-safe?** No; use `ConcurrentLinkedDeque` or a blocking deque.
- **Can it be used as a queue?** Yes, `Deque` extends `Queue`.
- **Why is it faster than LinkedList?** Contiguous array, fewer allocations.
- **Is `Stack` deprecated?** Not formally deprecated, but legacy.

**7. Common mistakes**

- Using `Stack` in new code.
- Using `LinkedList` just because it implements `Queue`.
- Adding nulls to `ArrayDeque`.
- Sharing an `ArrayDeque` across threads.

**8. Interview difficulty**

**Beginner**

---

### Question 14: Compare `Iterator`, `ListIterator`, and `Spliterator`

**1. Interview-ready answer**

`Iterator` is the basic one-way cursor with `hasNext`, `next`, `remove`, and `forEachRemaining`. `ListIterator` extends it for lists, adding backward traversal, `set`, `add`, and index methods. `Spliterator` is designed for splitting work, supports characteristics such as `ORDERED` and `SIZED`, and powers parallel streams.

**2. Detailed explanation**

| Cursor | Direction | Modify | Key methods | Used by |
|---|---|---|---|---|
| `Iterator` | Forward | `remove` | `hasNext`, `next` | All collections |
| `ListIterator` | Both | `add`, `set`, `remove` | `previous`, `nextIndex` | Lists |
| `Spliterator` | Split-aware | No direct remove | `tryAdvance`, `trySplit` | Streams |

**3. Internal working**

`ArrayList`'s iterator keeps an index and checks `modCount`. `LinkedList`'s list iterator holds a node reference. `Spliterator` can estimate remaining elements and split a source into independent chunks for parallel processing.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;
import java.util.Spliterator;

public class CursorDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>(List.of("ORD-1", "ORD-2"));

        ListIterator<String> iterator = orderIds.listIterator();
        while (iterator.hasNext()) {
            if ("ORD-2".equals(iterator.next())) {
                iterator.set("ORD-2-PAID");
            }
        }

        Spliterator<String> spliterator = orderIds.spliterator();
        System.out.println("size=" + spliterator.estimateSize());
        System.out.println(orderIds);
    }
}
```

Output:

```text
size=2
[ORD-1, ORD-2-PAID]
```

**5. Real-world use case**

A batch job uses `ListIterator.set` to update elements while traversing, and parallel streams internally use spliterators to split a large list across threads.

**6. Interview follow-up questions**

- **Can Iterator add elements?** No; ListIterator can.
- **Which cursor supports parallel splitting?** Spliterator.
- **What are characteristics?** Flags like ORDERED, DISTINCT, SIZED, SORTED, IMMUTABLE.
- **Is Iterator fail-fast?** For most core collections, yes.
- **Can you call remove twice after one next?** No; it throws `IllegalStateException`.

**7. Common mistakes**

- Calling `remove` before `next`.
- Using `ListIterator` on a `Set`.
- Assuming spliterators always split efficiently.
- Modifying a collection externally during iteration.

**8. Interview difficulty**

**Intermediate**

---

### Question 15: What is the difference between fail-fast and fail-safe iterators?

**1. Interview-ready answer**

Fail-fast iterators detect structural changes made after iteration starts and usually throw `ConcurrentModificationException`; examples include `ArrayList` and `HashMap` iterators. Fail-safe is a loose term for iterators that do not throw — either because they traverse a snapshot (`CopyOnWriteArrayList`) or are weakly consistent (`ConcurrentHashMap`). Fail-fast is best-effort, not a guaranteed correctness mechanism.

**2. Detailed explanation**

Fail-fast relies on `modCount`: the iterator records the expected count and checks it on each `next`. If it changed, it throws. It detects concurrent modification, not necessarily concurrent threads — modifying in the same thread during iteration also triggers it.

Weakly consistent iterators may reflect some updates, may not reflect others, and never throw CME. Snapshot iterators traverse the array as it existed at creation.

**3. Internal working**

`ConcurrentModificationException` is thrown by `checkForComodification`. The check happens during iteration, not at the exact moment of modification, so relying on it for synchronization is unsafe.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class IteratorBehaviorDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>(List.of("ORD-1", "ORD-2"));
        Iterator<String> iterator = orderIds.iterator();
        orderIds.add("ORD-3");

        try {
            iterator.next();
        } catch (Exception exception) {
            System.out.println(exception.getClass().getSimpleName());
        }

        List<String> snapshotList = new CopyOnWriteArrayList<>(List.of("ORD-1", "ORD-2"));
        Iterator<String> snapshotIterator = snapshotList.iterator();
        snapshotList.add("ORD-3");

        while (snapshotIterator.hasNext()) {
            System.out.println(snapshotIterator.next());
        }
    }
}
```

Output:

```text
ConcurrentModificationException
ORD-1
ORD-2
```

The snapshot iterator does not see ORD-3.

**5. Real-world use case**

A config listener list is a `CopyOnWriteArrayList` so one listener can be added while request threads iterate without CME.

**6. Interview follow-up questions**

- **Is fail-safe a formal term?** No; prefer “weakly consistent” or “snapshot.”
- **Does ConcurrentHashMap iterator throw CME?** No.
- **Why not rely on fail-fast?** It is best-effort and not thread-safety.
- **How do you safely remove during iteration?** Use `Iterator.remove`.
- **Why is CopyOnWrite costly for writes?** It copies the whole array.

**7. Common mistakes**

- Adding/removing directly in a for-each loop.
- Treating fail-fast as a locking mechanism.
- Assuming weakly consistent means no updates are visible.
- Using CopyOnWrite for high-write workloads.

**8. Interview difficulty**

**Intermediate**

---

---

## Part D — Contracts, utilities, and practical selection

### Question 16: Why does the `equals`/`hashCode` contract matter in hash collections?

**1. Interview-ready answer**

`HashMap` and `HashSet` first compare hashes, then call `equals`. Equal objects must have equal hashes; otherwise the same logical key can be stored in two buckets. Unequal objects may share a hash — that is allowed and handled by chaining. Keys must also be immutable while in the collection, because changing hash-changing fields makes the entry unreachable.

**2. Detailed explanation**

Contract:

- If `a.equals(b)`, then `a.hashCode() == b.hashCode()`.
- If `a.hashCode() == b.hashCode()`, `a.equals(b)` may be false.
- `equals` must be reflexive, symmetric, transitive, consistent, and return false for null.

**3. Internal working**

`put(k, v)` uses `k.hashCode()` to pick a bucket. If the key's hash changes after insertion, `get(k)` searches a different bucket and returns null. The entry still occupies memory until rehashed or removed by iteration.

**4. Practical Java/Spring Boot example**

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

public class MutableKeyDemo {

    static final class BadKey {

        String tenantId;

        BadKey(String tenantId) {
            this.tenantId = tenantId;
        }

        @Override
        public boolean equals(Object other) {
            return other instanceof BadKey key && tenantId.equals(key.tenantId);
        }

        @Override
        public int hashCode() {
            return Objects.hashCode(tenantId);
        }
    }

    public static void main(String[] args) {
        Map<BadKey, String> cache = new HashMap<>();
        BadKey key = new BadKey("T1");
        cache.put(key, "value");

        key.tenantId = "T2";

        System.out.println(cache.get(key));
        System.out.println(cache.size());
    }
}
```

Output:

```text
null
1
```

The entry exists but cannot be found by the mutated key.

**5. Real-world use case**

A cache keyed by a mutable JPA entity or DTO breaks after a field update. Use an immutable key such as a record of IDs instead.

**6. Interview follow-up questions**

- **Can unequal objects have the same hash?** Yes.
- **Can equal objects have different hashes?** No; that violates the contract.
- **Why are records good keys?** Components are final and equals/hashCode are generated.
- **What happens with a null key in HashMap?** It is stored in bucket 0.
- **Does ConcurrentHashMap allow null keys?** No.

**7. Common mistakes**

- Implementing only equals.
- Including mutable fields in hashCode.
- Using arrays as keys without Arrays.equals/hashCode.
- Using entities with lazy fields as map keys.

**8. Interview difficulty**

**Intermediate**

---

### Question 17: When should you use `Comparable` vs `Comparator`?

**1. Interview-ready answer**

Use `Comparable` for a class's single natural order, such as a version or ID. Use `Comparator` when you need alternative or external ordering, such as sorting orders by amount then ID, or when you cannot modify the class. Sorted collections like `TreeSet` and `TreeMap` use comparison, not `equals`, to decide uniqueness.

**2. Detailed explanation**

| Aspect | `Comparable` | `Comparator` |
|---|---|---|
| Method | `compareTo` | `compare` |
| Location | Inside the class | External class/lambda |
| Orders | One natural order | Many custom orders |
| Common use | `Collections.sort(list)` | `list.sort(comparator)` |
| Sorted collections | Natural ordering | Passed to constructor |

If comparison returns `0`, a `TreeSet` treats the elements as duplicates even if `equals` returns false.

**3. Internal working**

`TreeMap` and `TreeSet` call the comparator at each branch. If a mutable field used by the comparator changes after insertion, the tree's structure becomes invalid for that element.

**4. Practical Java/Spring Boot example**

```java
import java.util.Comparator;
import java.util.Set;
import java.util.TreeSet;

record Job(String id, int priority) {
}

public class ComparatorDemo {

    public static void main(String[] args) {
        Set<Job> jobs = new TreeSet<>(
                Comparator.comparingInt(Job::priority).thenComparing(Job::id));

        jobs.add(new Job("JOB-2", 2));
        jobs.add(new Job("JOB-1", 1));
        jobs.add(new Job("JOB-3", 1));

        System.out.println(jobs);
    }
}
```

Output:

```text
[Job[id=JOB-1, priority=1], Job[id=JOB-3, priority=1], Job[id=JOB-2, priority=2]]
```

Adding `thenComparing(Job::id)` prevents jobs with the same priority from being treated as duplicates.

**5. Real-world use case**

A queue view sorts jobs by priority, then by created time, then by ID for deterministic behavior.

**6. Interview follow-up questions**

- **Should compareTo be consistent with equals?** Strongly recommended for sorted sets/maps.
- **Can a Comparator be null-safe?** Yes with `Comparator.nullsLast`.
- **Can you sort descending?** `Comparator.reverseOrder()` or `.reversed()`.
- **Which is better for multiple orders?** Comparator.
- **Do HashMap keys need Comparable?** No.

**7. Common mistakes**

- Comparator returning `a.priority - b.priority` with integer overflow.
- Forgetting a tie-breaker in TreeSet.
- Changing sort fields after insertion.
- Using subtraction for double comparisons.

**8. Interview difficulty**

**Beginner**

---

### Question 18: What pitfalls exist in `Collections` utility methods?

**1. Interview-ready answer**

`Collections.unmodifiableList` returns a view, not an independent copy, so changes to the source are visible. `List.copyOf` creates an immutable snapshot. `Collections.synchronizedList` locks individual methods but still requires manual synchronization while iterating. `Collections.binarySearch` requires the list to be sorted by the same comparator. `emptyList` and singleton lists are immutable and shared.

**2. Detailed explanation**

Useful methods:

| Method | Purpose | Pitfall |
|---|---|---|
| `unmodifiableList` | Read-only view | Source changes still visible |
| `synchronizedList` | Coarse locking | Iterate inside `synchronized` |
| `emptyList` | Shared empty list | Immutable |
| `binarySearch` | O(log n) lookup | Requires sorted input |
| `frequency`, `min`, `max` | Simple scans | O(n) |
| `rotate`, `reverse`, `shuffle` | In-place algorithms | Mutates input |

**3. Internal working**

Unmodifiable wrappers delegate reads to the backing collection and throw `UnsupportedOperationException` on mutators. `binarySearch` compares against the requested comparator; a mismatch produces incorrect results rather than an exception.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CollectionsUtilityDemo {

    public static void main(String[] args) {
        List<String> source = new ArrayList<>(List.of("ORD-1", "ORD-2"));
        List<String> view = Collections.unmodifiableList(source);
        List<String> snapshot = List.copyOf(source);

        source.add("ORD-3");

        System.out.println(view);
        System.out.println(snapshot);

        int index = Collections.binarySearch(source, "ORD-2");
        System.out.println(index);
    }
}
```

Output:

```text
[ORD-1, ORD-2, ORD-3]
[ORD-1, ORD-2]
1
```

The unmodifiable view sees ORD-3; the snapshot does not.

**5. Real-world use case**

A service returns `List.copyOf(result)` from a cached mutable list so callers cannot mutate shared state.

**6. Interview follow-up questions**

- **What does binarySearch return if absent?** A negative insertion point.
- **Does `Collections.sort` mutate?** Yes.
- **Is `emptyList` safe to return?** Yes, it is immutable.
- **What exception do unmodifiable collections throw?** UnsupportedOperationException.
- **Why synchronize during iteration?** Compound iteration is not atomic.

**7. Common mistakes**

- Believing unmodifiable means immutable snapshot.
- Using binarySearch on unsorted lists.
- Returning internal mutable lists from services.
- Forgetting that sort changes the original list.

**8. Interview difficulty**

**Beginner**

---

### Question 19: Give a complexity cheat sheet and choose collections for a backend scenario

**1. Interview-ready answer**

`ArrayList` gives O(1) indexed access, `HashMap`/`HashSet` give O(1) average lookup, `LinkedHashMap`/`LinkedHashSet` add order at small cost, `TreeMap`/`TreeSet` give O(log n) ordered operations, and `PriorityQueue` gives O(log n) insert/poll with O(1) peek. Pick by required operation and ordering first, then by concurrency and memory.

**2. Detailed explanation**

| Operation | ArrayList | LinkedList | HashMap | TreeMap | PriorityQueue |
|---|---:|---:|---:|---:|---:|
| Get by index | O(1) | O(n) | n/a | n/a | n/a |
| Search by key/value | O(n) | O(n) | O(1) avg | O(log n) | O(n) |
| Insert end | O(1) amortized | O(1) | O(1) avg | O(log n) | O(log n) |
| Insert middle | O(n) | O(1) with node | n/a | n/a | n/a |
| Remove by key | O(n) | O(n) | O(1) avg | O(log n) | O(log n) poll |
| Ordered iteration | Insertion | Insertion | No | Sorted | Heap order only |
| Thread-safe | No | No | No | No | No |

**3. Internal working**

Average O(1) for hash collections assumes a good hash distribution and load factor. Worst case with many collisions is O(n) before treeification or O(log n) after treeification for `HashMap`. Big-O ignores constants; array-based structures often beat node-based ones despite similar asymptotic complexity.

**4. Practical Java/Spring Boot example**

```java
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import java.util.TreeSet;

public class CollectionSelectionDemo {

    public static void main(String[] args) {
        List<String> orderIds = new ArrayList<>(
                List.of("ORD-2", "ORD-1", "ORD-2", "ORD-3"));

        Set<String> uniqueInOrder = new LinkedHashSet<>(orderIds);

        Map<String, Integer> counts = new TreeMap<>();
        for (String id : orderIds) {
            counts.merge(id, 1, Integer::sum);
        }

        List<String> sortedUnique = new ArrayList<>(new TreeSet<>(uniqueInOrder));

        System.out.println(uniqueInOrder);
        System.out.println(counts);
        System.out.println(sortedUnique);
    }
}
```

Output:

```text
[ORD-2, ORD-1, ORD-3]
{ORD-1=1, ORD-2=2, ORD-3=1}
[ORD-1, ORD-2, ORD-3]
```

**5. Real-world use case**

An audit endpoint receives unordered events, deduplicates them in first-seen order, counts per event type, and returns sorted keys for stable JSON output.

**6. Interview follow-up questions**

- **Why is HashMap O(1) only average?** Collisions can degrade buckets.
- **Which collection gives deterministic output?** LinkedHashSet/LinkedHashMap/TreeSet/TreeMap.
- **Which is most memory-efficient list?** ArrayList.
- **When does LinkedList beat ArrayList?** Insert at a known iterator position.
- **Which concurrent collection for top-k?** `PriorityBlockingQueue`.

**7. Common mistakes**

- Using List for contains-heavy logic.
- Using TreeMap just because output should be stable; LinkedHashMap may be enough.
- Ignoring constants and cache locality.
- Assuming all implementations share the same complexity.

**8. Interview difficulty**

**Intermediate**

---

### Question 20: Coding challenge — find the top K frequent customers

**1. Interview-ready answer**

I count customer IDs with a `HashMap` and keep a min-heap of size k using `PriorityQueue`. Each entry enters the heap; when the heap is full, I replace the root only if the new count is better. Finally I sort the k results for presentation: frequency descending, then customer ID ascending.

**2. Detailed explanation**

**Problem statement:** Given a list of customer IDs from orders, return the k most frequent IDs with their frequencies. Treat null IDs as invalid and skip them. Break ties by lexicographically smaller customer ID.

**Approach:**
1. Count frequencies with `Map.merge`.
2. Maintain a `PriorityQueue` of size k ordered by lowest frequency at the root.
3. Use a tie-breaker so the lexicographically largest ID is evicted first on equal frequency.
4. Sort the final k entries for deterministic output.

**3. Internal working**

`HashMap` gives average O(1) counting. The heap root is the weakest current candidate, so comparing against it costs O(1). Heap updates are O(log k), which is much cheaper than sorting all n distinct customers.

**4. Practical Java solution**

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Queue;

public class TopKFrequentCustomers {

    record Count(String customerId, int frequency) {
    }

    public static List<Count> solve(List<String> customerIds, int k) {
        if (k <= 0) {
            return List.of();
        }

        Map<String, Integer> frequencies = new HashMap<>();
        for (String customerId : customerIds) {
            if (customerId != null) {
                frequencies.merge(customerId, 1, Integer::sum);
            }
        }

        Queue<Count> heap = new PriorityQueue<>(
                Comparator.comparingInt(Count::frequency)
                        .thenComparing(Count::customerId, Comparator.reverseOrder()));

        for (Map.Entry<String, Integer> entry : frequencies.entrySet()) {
            Count candidate = new Count(entry.getKey(), entry.getValue());
            Count weakest = heap.peek();

            if (heap.size() < k) {
                heap.offer(candidate);
            } else if (candidate.frequency() > weakest.frequency()
                    || (candidate.frequency() == weakest.frequency()
                        && candidate.customerId().compareTo(weakest.customerId()) < 0)) {
                heap.poll();
                heap.offer(candidate);
            }
        }

        List<Count> result = new ArrayList<>(heap);
        result.sort(Comparator.comparingInt(Count::frequency).reversed()
                .thenComparing(Count::customerId));
        return result;
    }

    public static void main(String[] args) {
        List<String> customerIds = List.of(
                "CUST-1", "CUST-2", "CUST-1", "CUST-3", "CUST-2",
                "CUST-1", "CUST-4", "CUST-2", "CUST-3", "CUST-1");

        System.out.println(solve(customerIds, 2));
    }
}
```

Output:

```text
[Count[customerId=CUST-1, frequency=4], Count[customerId=CUST-2, frequency=3]]
```

**5. Dry run**

| Customer | Count |
|---|---:|
| CUST-1 | 4 |
| CUST-2 | 3 |
| CUST-3 | 2 |
| CUST-4 | 1 |

Heap keeps CUST-1 and CUST-2; CUST-3 and CUST-4 never replace the root.

**6. Complexity**

- Time: O(n + m log k + k log k), where m is the number of distinct customers.
- Space: O(m + k).

**7. Edge cases**

- Empty input → empty result.
- `k <= 0` → empty result.
- Null IDs → skipped.
- Fewer distinct customers than k → all returned.
- Equal frequencies → lexicographically smaller ID wins.

**8. Real-world use case**

An analytics dashboard shows the top customers by order volume without sorting the entire customer base on every request.

**9. Interview follow-up questions**

- **Why not sort all entries?** That is O(m log m); heap is O(m log k).
- **Why skip nulls?** Null is not a valid customer identifier.
- **How would you do this on streaming data?** Count in Redis or a sliding-window structure and maintain a heap per window.
- **How do you make it thread-safe?** Feed IDs from a concurrent source, but keep the collector local to one task or use a mergeable frequency map.
- **How do you return IDs only?** `result.stream().map(Count::customerId).toList()`.

**10. Common mistakes**

- Using `TreeMap` when only top-k is needed.
- Keeping a max-heap and evicting the best candidate.
- Forgetting deterministic tie-breaking.
- Counting null IDs as valid.

**11. Interview difficulty**

**Intermediate**

---

## Chapter 7 Verification

| Syllabus topic | Question(s) |
|---|---|
| Collections hierarchy | Q1 |
| Choosing List/Set/Queue/Map | Q2, Q19 |
| ArrayList internals and growth | Q3, Q4 |
| LinkedList vs ArrayList | Q4 |
| HashSet, LinkedHashSet, TreeSet | Q5 |
| HashMap hashing and collisions | Q6 |
| HashMap resizing, load factor, treeification | Q7 |
| Hashtable, synchronizedMap, ConcurrentHashMap comparison | Q8 |
| ConcurrentHashMap internals | Q9 |
| LinkedHashMap and LRU | Q10 |
| TreeMap and red-black trees | Q11 |
| PriorityQueue | Q12, Q20 |
| ArrayDeque vs Stack/LinkedList | Q13 |
| Iterator, ListIterator, Spliterator | Q14 |
| Fail-fast vs weakly consistent iteration | Q15 |
| equals/hashCode contract | Q6, Q16 |
| Comparable vs Comparator | Q11, Q17 |
| Collections utility methods | Q18 |
| Complexity analysis and collection selection | Q19, Q20 |

**Total interview questions: 20** — all with the required 8-part format; coding questions include problem statement, approach, Java solution, dry run, complexity, and edge cases.

**Next:** [Chapter 8 — Multithreading and Concurrency](../03-Concurrency/08-Multithreading-and-Concurrency.md). Say `NEXT CHAPTER` to continue.
