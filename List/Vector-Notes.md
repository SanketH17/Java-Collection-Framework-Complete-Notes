# Vector in Java — Interview Notes (Short & Simple)

---

## What is Vector?

- A **resizable array** — just like ArrayList
- Part of Java since **version 1.0** (before the Collections Framework)
- It is **synchronized** — meaning only one thread can access it at a time
- Lives in `java.util.Vector`

> Think of it as the **older, slower brother of ArrayList** that carries a lock everywhere.

---

## Why is Vector Considered Legacy?

| Reason | Explanation |
|---|---|
| **Too much locking** | Every single method is synchronized, even when you don't need thread safety |
| **Slow** | Synchronization adds overhead on every operation |
| **Better alternatives exist** | `ArrayList` (no sync needed) or `CopyOnWriteArrayList` (smarter sync) |

> **One-liner:** Vector synchronizes everything by default, which makes it unnecessarily slow for most use cases.

---

## Vector vs ArrayList — The Interview Table

| Feature | ArrayList | Vector |
|---|---|---|
| Synchronized | ❌ No | ✅ Yes |
| Speed | ✅ Faster | ❌ Slower |
| Growth | Grows by **50%** | Grows by **100%** (doubles) |
| Introduced | Java 1.2 | Java 1.0 |
| Legacy iteration | ❌ No `Enumeration` | ✅ Has `Enumeration` |
| Modern iteration | ✅ `Iterator` | ✅ `Iterator` + `Enumeration` |
| When to use | Almost always | Almost never |

---

## Hierarchy

```
Iterable → Collection → List → Vector → Stack
```

> **Note:** Java's `Stack` class extends `Vector`. Both are legacy.

---

## Constructors

```java
Vector<String> v1 = new Vector<>();           // Default capacity: 10
Vector<String> v2 = new Vector<>(20);          // Initial capacity: 20
Vector<String> v3 = new Vector<>(20, 5);       // Capacity: 20, grows by 5 each time
Vector<String> v4 = new Vector<>(existingList); // From another collection
```

---

## Important Methods (Quick Reference)

All methods work the same as ArrayList. Just remember these Vector-specific ones:

| Method | What It Does |
|---|---|
| `addElement(E e)` | Legacy way to add (same as `add()`) |
| `removeElement(Object o)` | Legacy way to remove |
| `elementAt(int index)` | Legacy way to get (same as `get()`) |
| `elements()` | Returns `Enumeration` — legacy iterator |
| `capacity()` | Returns current internal array capacity |
| `trimToSize()` | Shrinks internal array to match actual size |

```java
Vector<String> v = new Vector<>();
v.add("A");
v.add("B");
v.addElement("C");  // Legacy method — same as add()

System.out.println(v.elementAt(0));  // A
System.out.println(v.capacity());    // 10 (default)
System.out.println(v.size());        // 3
```

---

## Enumeration vs Iterator

This is a common interview question:

| Feature | Enumeration (Legacy) | Iterator (Modern) |
|---|---|---|
| Used with | Vector, Hashtable | Any Collection |
| Can remove elements? | ❌ No | ✅ Yes (`remove()`) |
| Method names | `hasMoreElements()`, `nextElement()` | `hasNext()`, `next()` |
| Fail-fast? | ❌ No | ✅ Yes |

```java
// Enumeration (old way)
Enumeration<String> e = v.elements();
while (e.hasMoreElements()) {
    System.out.println(e.nextElement());
}

// Iterator (modern way — use this)
Iterator<String> it = v.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

---

## Growth Strategy — How Vector Grows

```
Default capacity = 10

When full:
  ArrayList → grows by 50%   (10 → 15 → 22 → 33...)
  Vector    → grows by 100%  (10 → 20 → 40 → 80...)
```

If you set a custom increment:
```java
Vector<Integer> v = new Vector<>(10, 5); // capacity=10, increment=5
// When full: 10 → 15 → 20 → 25...
```

---

## Top Interview Q&A

**Q1: When would you use Vector over ArrayList?**
> Almost never. If you need thread safety, use `Collections.synchronizedList(new ArrayList<>())` or `CopyOnWriteArrayList`. Vector's method-level synchronization is too coarse-grained.

**Q2: Is Vector thread-safe?**
> Yes, but only at the individual method level. A sequence of operations (like check-then-add) is still NOT atomic — you'd need external synchronization.

**Q3: Why does Stack extend Vector?**
> It was a design mistake in early Java. A Stack should only allow push/pop, but since it extends Vector, you can access any element by index — which breaks the Stack concept. Use `ArrayDeque` instead.

**Q4: What is the default capacity of Vector?**
> **10** — same as ArrayList.

**Q5: Difference between `capacity()` and `size()`?**
> `size()` = number of elements actually stored. `capacity()` = total slots available in the internal array. Example: a Vector with 3 elements might have a capacity of 10.

---

## Summary — What to Remember

```
✅ Vector = synchronized ArrayList (legacy, slow)
✅ Grows by 100% (ArrayList grows by 50%)
✅ Has Enumeration + Iterator
✅ Stack extends Vector (both legacy)
✅ Use ArrayList or CopyOnWriteArrayList instead
✅ Default capacity = 10
```

> [!TIP]
> **Interview strategy:** If asked about Vector, explain what it is in 2 sentences, then pivot to *why you wouldn't use it* and *what you'd use instead*. That shows modern Java knowledge.
