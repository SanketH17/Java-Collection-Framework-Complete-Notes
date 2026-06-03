# Java Collection Framework — Complete Notes

> **Audience:** Java developers preparing for technical interviews (fresher to senior).
> **Coverage:** JCF background → hierarchy → core interfaces → core concepts → implementations overview.
> **Last Updated:** June 2026 | Java 21 LTS

---

## Table of Contents

- [Java Collection Framework — Complete Notes](#java-collection-framework--complete-notes)
  - [Table of Contents](#table-of-contents)
  - [1. What is the Java Collection Framework?](#1-what-is-the-java-collection-framework)
    - [What is it?](#what-is-it)
    - [Why do we need it?](#why-do-we-need-it)
    - [Real-World Analogy](#real-world-analogy)
    - [What JCF Provides](#what-jcf-provides)
  - [2. Why JCF Exists — The Problem It Solved](#2-why-jcf-exists--the-problem-it-solved)
    - [Before JCF (Pre-Java 1.2)](#before-jcf-pre-java-12)
    - [Problems This Caused](#problems-this-caused)
    - [What JCF Delivered](#what-jcf-delivered)
  - [3. JCF Hierarchy](#3-jcf-hierarchy)
    - [The Big Picture](#the-big-picture)
    - [Interface Hierarchy — Simplified](#interface-hierarchy--simplified)
    - [Interface Relationships Diagram](#interface-relationships-diagram)
  - [4. Core Interfaces](#4-core-interfaces)
    - [Key Method Groups](#key-method-groups)
  - [5. Core Concepts Before You Code](#5-core-concepts-before-you-code)
    - [5.1 Generics](#51-generics)
    - [5.2 Autoboxing and Unboxing](#52-autoboxing-and-unboxing)
    - [5.3 equals() and hashCode()](#53-equals-and-hashcode)
    - [5.4 Comparable vs Comparator](#54-comparable-vs-comparator)
    - [5.5 Iterator Pattern](#55-iterator-pattern)
    - [5.6 Fail-Fast vs Fail-Safe Iterators](#56-fail-fast-vs-fail-safe-iterators)
    - [5.7 Immutability](#57-immutability)
  - [6. Implementations at a Glance](#6-implementations-at-a-glance)
    - [List Implementations](#list-implementations)
    - [Set Implementations](#set-implementations)
    - [Map Implementations](#map-implementations)
    - [Queue and Deque Implementations](#queue-and-deque-implementations)
  - [7. Interview Questions — JCF Fundamentals](#7-interview-questions--jcf-fundamentals)

---

## 1. What is the Java Collection Framework?

### What is it?

The **Java Collection Framework (JCF)** is a set of ready-made classes and interfaces for storing and working with groups of objects.

It lives in the `java.util` package and has been the backbone of Java data-structure programming since **Java 1.2 (1998)**.

### Why do we need it?

Before JCF, every developer had to build data structures from scratch — no standard API, no shared code, no consistent behaviour. It was slow, inconsistent, and every team did it differently.

### Real-World Analogy

> Think of JCF as a **well-equipped professional toolbox**.
>
> Before JCF, every carpenter had to forge their own tools — hammers, saws, screwdrivers — from raw metal. It worked, but it was slow and every carpenter's tools were incompatible.
>
> JCF gave every Java developer the same set of high-quality, battle-tested tools. You just pick the right one for the job and focus on building, not on making tools.

```
JCF = Toolbox

  [ ArrayList  ]  →  Resizable tray with numbered slots
  [ LinkedList ]  →  Chain of connected boxes
  [ HashMap    ]  →  Labeled file cabinet (key → value)
  [ HashSet    ]  →  Bag that rejects duplicates
  [ TreeMap    ]  →  Alphabetically sorted file cabinet
  [ Queue      ]  →  Ticket counter — first in, first served
```

### What JCF Provides

| Component | What It Is | Example |
|---|---|---|
| **Interfaces** | Contracts — define *what* can be done | `List`, `Set`, `Map`, `Queue` |
| **Implementations** | Classes — define *how* it is done | `ArrayList`, `HashMap`, `TreeSet` |
| **Algorithms** | Utility static methods | `Collections.sort()`, `Collections.shuffle()` |

---

## 2. Why JCF Exists — The Problem It Solved

### Before JCF (Pre-Java 1.2)

Java had only a handful of built-in options, all with serious problems:

| Old Tool | Problem |
|---|---|
| `Array` | Fixed size — you must decide the size upfront and cannot change it |
| `Vector` | Resizable, but every single method was `synchronized` → painfully slow |
| `Hashtable` | Key-value store, but synchronized and no null keys/values allowed |
| `Stack` | Extends `Vector` — a design mistake Java itself has acknowledged |

```java
// Life before JCF — painful and error-prone
String[] names = new String[10]; // What if we need 11 names?
names[0] = "Alice";
// Need more space? Write your own resize logic.
// Need to search? Write your own loop.
// Need to sort? Write your own algorithm.
// Need to pass to a method? Hope it also uses arrays.
```

### Problems This Caused

| Problem | Real Impact |
|---|---|
| No unified interface | Code written for `Vector` could not work with arrays |
| Reinventing the wheel | Every team wrote its own LinkedList, Stack, Queue |
| No standard algorithms | Every developer wrote sorting/searching from scratch |
| Poor reusability | Changing data structure meant rewriting all methods that used it |

### What JCF Delivered

- **One interface** (`Collection<E>`) that all linear collections honour.
- **Polymorphism**: write a method accepting `List<E>` and pass `ArrayList`, `LinkedList`, or any future implementation — no code changes needed.
- **Battle-tested algorithms**: TimSort, binary search — fast and correct out of the box.
- **Generics** (Java 5+): compile-time type safety, no more accidental `ClassCastException`.

```java
// Life after JCF — clean, flexible, powerful
List<String> names = new ArrayList<>(); // or LinkedList — same API!
names.add("Alice");
names.add("Bob");
Collections.sort(names);               // sorting for free
System.out.println(names);             // [Alice, Bob]
```

---

## 3. JCF Hierarchy

### The Big Picture

```
                     java.lang.Iterable<E>
                             |
                     java.util.Collection<E>
                             |
            +----------------+----------------+
            |                |                |
         List<E>           Set<E>          Queue<E>
            |                |                |
    +-------+------+    +----+----+      +----+--------+
    |        |     |    |         |      |             |
ArrayList LinkedList  HashSet  TreeSet PriorityQueue ArrayDeque
  Vector  CopyOnWrite    |
            ArrayList LinkedHashSet

       --- separate root (NOT a Collection) ---

                     java.util.Map<K,V>
                             |
            +----------------+----------------+
            |                |                |
         HashMap          Hashtable        TreeMap
            |
       LinkedHashMap
```

> **Interview Tip:** `Map` does **NOT** extend `Collection`. It is a completely separate hierarchy. This is one of the most common trick questions in Java interviews.

### Interface Hierarchy — Simplified

```
Iterable<E>
  └── Collection<E>
        ├── List<E>         ordered, indexed, duplicates allowed
        ├── Set<E>          no duplicates
        |     └── SortedSet<E>    sorted, no duplicates
        └── Queue<E>        FIFO ordering
              └── Deque<E>  double-ended queue (both ends)

Map<K,V>  (separate hierarchy)
  └── SortedMap<K,V>        sorted by key
```

### Interface Relationships Diagram

```
                      Iterable
                          |
                     Collection
                    /     |     \
                 List    Set    Queue
                          |        \
                       SortedSet   Deque

              Map (completely separate)
                |
            SortedMap
```

---

## 4. Core Interfaces

These are the contracts every collection must honour. Learning the interfaces teaches you what is *possible*; the implementations tell you how it is *done*.

| Interface | Key Trait | Real-World Analogy | Primary Implementations |
|---|---|---|---|
| `Iterable<E>` | Can be walked with for-each | A queue you can walk along | All collections |
| `Collection<E>` | Base — add, remove, contains, size | A generic container | (abstract base) |
| `List<E>` | Ordered, indexed, allows duplicates | Numbered notebook pages | `ArrayList`, `LinkedList` |
| `Set<E>` | No duplicates allowed | A guest list — unique names only | `HashSet`, `TreeSet` |
| `SortedSet<E>` | No duplicates, sorted | Alphabetically sorted guest list | `TreeSet` |
| `Queue<E>` | FIFO — first in, first out | Cinema ticket line | `PriorityQueue`, `ArrayDeque` |
| `Deque<E>` | Double-ended — add/remove from both ends | Deck of cards | `ArrayDeque`, `LinkedList` |
| `Map<K,V>` | Key → Value, unique keys | Dictionary / phone book | `HashMap`, `TreeMap` |
| `SortedMap<K,V>` | Sorted by key | Sorted dictionary | `TreeMap` |

### Key Method Groups

```
Collection<E>              List<E>                    Set<E>
─────────────              ────────────────────────   ──────────────────
add(e)                     get(index)   → O(1)*       (inherits Collection)
remove(o)                  set(index, e) → O(1)*
contains(o)                add(index, e) → O(n)
size()                     remove(index) → O(n)
isEmpty()                  indexOf(o)    → O(n)
iterator()                 subList(from, to)
clear()                    listIterator()
```
*\* For ArrayList. Complexity differs for LinkedList.*

---

## 5. Core Concepts Before You Code

### 5.1 Generics

**What is it?**

Generics let you specify what *type* of objects a collection holds. Written as `<E>` (element), `<K, V>` (key-value), etc.

**Why do we need it?**

Without generics, any collection can hold anything — mistakes are only caught at runtime when something crashes. With generics, the compiler catches type mismatches before the code even runs.

**Real-World Analogy:**

> A generic is like a **labeled storage box**.
>
> Without a label, you have a box that holds "anything" — you might accidentally put kitchen utensils in a box meant for books.
>
> With a label (`Box<Books>`): only books go in. Try putting a screwdriver in — stopped immediately.

```java
// Without generics — dangerous raw types
List rawList = new ArrayList();
rawList.add("Alice");
rawList.add(42);                        // no error — mixing types!
String name = (String) rawList.get(1); // ClassCastException at runtime!

// With generics — type-safe
List<String> names = new ArrayList<>();
names.add("Alice");
// names.add(42);                      // COMPILE ERROR — caught before running
String name = names.get(0);           // no cast needed
```

> **Interview Tip:** Generics use **type erasure** — they only exist at compile time. At runtime, `ArrayList<String>` is just `ArrayList`. This is why `instanceof ArrayList<String>` is not valid and why you cannot create `new E[]` inside generic code.

---

### 5.2 Autoboxing and Unboxing

**What is it?**

Collections work with **objects only** — they cannot store primitives (`int`, `double`, `boolean`, etc.). Java automatically converts between primitives and their wrapper types.

| Direction | Name | Example |
|---|---|---|
| `int` → `Integer` | Autoboxing | `list.add(42)` |
| `Integer` → `int` | Unboxing | `int x = list.get(0)` |

**Real-World Analogy:**

> A collection is like a **parcel delivery service** — it only accepts boxed packages (objects). Autoboxing is the service automatically putting your loose item (primitive) into a box. Unboxing is opening the box on delivery.

```java
List<Integer> scores = new ArrayList<>();
scores.add(95);               // autoboxing: int 95 → Integer(95)
int topScore = scores.get(0); // unboxing: Integer(95) → int 95
```

> **Performance Note:** Frequent autoboxing in tight loops creates many short-lived objects and increases garbage collection pressure. For large numeric datasets, prefer `int[]` arrays or libraries like Eclipse Collections.

---

### 5.3 equals() and hashCode()

**What is it?**

`equals()` defines when two objects are considered "the same". `hashCode()` returns a numeric fingerprint used for fast bucket lookup in hash-based structures.

**Why do we need it?**

`contains()`, `remove(Object)`, and `indexOf()` compare objects using `equals()`. Without overriding it, Java compares memory addresses — which means two logically identical objects are treated as different.

```java
class Student {
    String name;
    int id;

    Student(String name, int id) {
        this.name = name;
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student s)) return false;
        return id == s.id && Objects.equals(name, s.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, id);
    }
}

List<Student> students = new ArrayList<>();
students.add(new Student("Alice", 101));

// With equals() override:
System.out.println(students.contains(new Student("Alice", 101))); // true  ✅
// Without equals() override: false — different objects in memory   ❌
```

> **Rule:** Always override `equals()` and `hashCode()` **together**. Java's contract: if two objects are equal, they MUST produce the same hash code.

---

### 5.4 Comparable vs Comparator

**What is it?**

Two ways to define how objects are ordered for sorting.

| | `Comparable<T>` | `Comparator<T>` |
|---|---|---|
| Where defined | **Inside** the class | **Outside** the class (separate) |
| Method | `compareTo(T other)` | `compare(T o1, T o2)` |
| Use case | Natural / default ordering | Custom or multiple orderings |
| Orderings per class | One | Many |

**Real-World Analogy:**

> `Comparable` is like a person who knows their default place in line — by age, for example. They carry their own ordering rule with them.
>
> `Comparator` is like an event organiser who decides the ordering rule for each event separately — by height for one event, by ticket number for another.

```java
// Comparable — class defines its own natural ordering
class Employee implements Comparable<Employee> {
    String name;
    int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary); // sort by salary
    }
}

List<Employee> team = new ArrayList<>();
Collections.sort(team); // uses compareTo() — natural order

// Comparator — external, flexible orderings
team.sort(Comparator.comparing(e -> e.name));                        // by name
team.sort(Comparator.comparingInt((Employee e) -> e.salary).reversed()); // salary desc
team.sort(Comparator.comparing((Employee e) -> e.name)
                     .thenComparingInt(e -> e.salary));               // name then salary
```

---

### 5.5 Iterator Pattern

**What is it?**

A standard way to walk through any collection one element at a time, without knowing the internal structure of the collection.

```
Collection<E>
     |
     +-- iterator()
              |
              v
         Iterator<E>
              |
              +-- hasNext() → boolean   (is there another element?)
              +-- next()    → E         (give me the next element)
              +-- remove()  → void      (remove current element safely)
```

**Why do we need it?**

Without a common iterator, you would need to know the internals of each data structure to loop through it. With `Iterator`, the same loop works for `ArrayList`, `LinkedList`, `HashSet`, and any other collection.

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Delhi", "Bangalore"));

Iterator<String> it = cities.iterator();
while (it.hasNext()) {
    String city = it.next();
    System.out.println(city);
}
```

The enhanced for-each loop is pure syntactic sugar for the above:

```java
for (String city : cities) {
    System.out.println(city);
}
// The compiler converts this to the iterator loop above
```

---

### 5.6 Fail-Fast vs Fail-Safe Iterators

**What is it?**

Defines what happens when a collection is modified while you are iterating over it.

**Real-World Analogy:**

> **Fail-fast** is like a photocopier that immediately stops and beeps if someone changes the original document mid-copy. It refuses to continue with a potentially inconsistent state.
>
> **Fail-safe** is like a photocopier that first takes a snapshot of the document, then copies from the snapshot. Changes to the original have no effect on the current copy job.

| | Fail-Fast | Fail-Safe |
|---|---|---|
| Examples | `ArrayList`, `HashMap`, `HashSet` | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| On modification | Throws `ConcurrentModificationException` | Works on snapshot, no exception |
| Memory overhead | None | Extra memory for snapshot |
| Consistency | Always current | May not see latest writes |
| Best for | Single-threaded, debugging | Concurrent multi-threaded reads |

```java
// Fail-fast example — WRONG
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
for (String s : list) {
    list.remove(s); // ❌ ConcurrentModificationException
}

// Fail-safe example — CORRECT
List<String> cowList = new CopyOnWriteArrayList<>(List.of("A", "B", "C"));
for (String s : cowList) {
    cowList.remove(s); // ✅ No exception (iterates over snapshot)
}
```

---

### 5.7 Immutability

**What is it?**

Creating collections that cannot be modified after they are created.

**Why do we need it?**

Immutable collections are safe to share across threads, safe to expose from APIs, and prevent accidental modification by calling code.

| Method | Mutable? | Allows Null? | Since |
|---|---|---|---|
| `new ArrayList<>()` | ✅ Yes | ✅ Yes | Java 1.2 |
| `Arrays.asList(...)` | Partial (set only) | ✅ Yes | Java 1.2 |
| `Collections.unmodifiableList(list)` | ❌ View only | ✅ Yes | Java 1.2 |
| `List.of(...)` | ❌ Truly immutable | ❌ No (NPE) | Java 9 |
| `List.copyOf(collection)` | ❌ Truly immutable | ❌ No (NPE) | Java 10 |

```java
// Truly immutable list — Java 9+
List<String> menu = List.of("Pizza", "Pasta", "Salad");
// menu.add("Soup");     // ❌ UnsupportedOperationException
// menu.add(null);       // ❌ NullPointerException

// Unmodifiable view — the original CAN still be changed!
List<String> mutable = new ArrayList<>(List.of("A", "B"));
List<String> readOnly = Collections.unmodifiableList(mutable);
mutable.add("C");            // works — modifies through original reference
System.out.println(readOnly); // [A, B, C] — the view reflects the change!

// Truly independent immutable snapshot — Java 10+
List<String> snapshot = List.copyOf(mutable); // independent copy
mutable.add("D");
System.out.println(snapshot); // [A, B, C] — snapshot is unaffected
```

> **Interview Tip:** `List.of()` is truly immutable — nobody can change it, not even the original reference. `Collections.unmodifiableList()` is a *view* — it only blocks writes through the view itself, but the underlying list can still be changed through its own reference.

---

## 6. Implementations at a Glance

### List Implementations

| Class | Best For | Avoid When |
|---|---|---|
| `ArrayList` | General use, random access, iteration | Frequent insert/delete in the middle |
| `LinkedList` | Frequent add/remove at head or tail | Random access needed |
| `Vector` | Legacy code only — do not use | All new code |
| `CopyOnWriteArrayList` | Read-heavy concurrent access | Write-heavy scenarios |

### Set Implementations

| Class | Best For | Avoid When |
|---|---|---|
| `HashSet` | Fast O(1) membership checks, no duplicates | Ordered iteration needed |
| `LinkedHashSet` | Insertion-order preserved, no duplicates | Memory is a concern |
| `TreeSet` | Sorted elements, range queries | Null elements, performance-critical paths |

### Map Implementations

| Class | Best For | Avoid When |
|---|---|---|
| `HashMap` | Fast O(1) key lookups, general use | Ordered keys needed |
| `LinkedHashMap` | Insertion-order or access-order iteration | Memory is a concern |
| `TreeMap` | Sorted keys, range queries (`subMap`, `headMap`) | Performance-critical O(1) lookups |
| `Hashtable` | Legacy code only — do not use | All new code |

### Queue and Deque Implementations

| Class | Best For | Notes |
|---|---|---|
| `PriorityQueue` | Priority-based processing (min/max heap) | Not FIFO — heap ordered |
| `ArrayDeque` | Stack or Queue, best general Deque | Faster than `LinkedList` for most uses |
| `LinkedList` | When already using it as a List and need Deque | High memory per node |

```
How to choose:

Need ordered, indexed, duplicates?    → List   → ArrayList (default)
Need unique elements only?            → Set    → HashSet (default)
Need key-value mapping?               → Map    → HashMap (default)
Need FIFO queue?                      → Queue  → ArrayDeque
Need sorted elements?                 → TreeSet / TreeMap
Need thread safety + heavy reads?     → CopyOnWriteArrayList / ConcurrentHashMap
```

> For a complete deep dive into **ArrayList**, see [ArrayList.md](ArrayList.md).

---

## 7. Interview Questions — JCF Fundamentals

---

**Q1. What is the Java Collection Framework?**

> A unified architecture in `java.util` for storing and manipulating groups of objects. It provides interfaces (contracts), implementations (classes), and algorithms (utility methods in `Collections` and `Arrays`). Introduced in Java 1.2 to replace the fragmented, legacy `Vector`/`Hashtable`/`Stack` tools.

---

**Q2. Does `Map` extend `Collection`?**

> No. `Map<K,V>` is a completely separate hierarchy with its own root interface. It does not extend `Collection`. This is one of the most common trick questions in Java interviews.

---

**Q3. What is the difference between `Collection` and `Collections`?**

> `Collection<E>` (with generic) is an **interface** — the root of the collection hierarchy.
> `Collections` (plural, no generic) is a **utility class** with static methods like `sort()`, `shuffle()`, `reverse()`, `binarySearch()`, `synchronizedList()`, etc.

---

**Q4. What is the difference between `List`, `Set`, and `Map`?**

> - `List`: ordered, indexed, allows duplicates, allows null.
> - `Set`: no duplicates, unordered (or sorted with TreeSet), no index-based access.
> - `Map`: key-value pairs, keys are unique, values can repeat.

---

**Q5. What is generics type erasure?**

> At compile time, generics are used for type checking. At runtime, the generic type information is erased. `ArrayList<String>` and `ArrayList<Integer>` are both just `ArrayList` at runtime. This is why you cannot use `instanceof ArrayList<String>` — the JVM has no knowledge of `String` at that point.

---

**Q6. What is the difference between `Comparable` and `Comparator`?**

> `Comparable` is implemented **inside** the class to define its natural ordering (one ordering per class). `Comparator` is implemented **externally** to define custom or alternative orderings, and is passed as a parameter to sort methods. A class can have one `Comparable` but many `Comparator`s.

---

**Q7. What is a fail-fast iterator?**

> An iterator that throws `ConcurrentModificationException` if the underlying collection is structurally modified during iteration (outside of the iterator's own `remove()` method). The check is done by comparing `modCount` (incremented on every structural change) with `expectedModCount` (saved at iterator creation). Examples: iterators of `ArrayList`, `HashMap`, `HashSet`.

---

**Q8. What is the difference between `List.of()` and `Collections.unmodifiableList()`?**

> `List.of()` creates a truly immutable list — the original cannot be changed by anyone. It also rejects null values. `Collections.unmodifiableList()` creates an unmodifiable *view* of a mutable list — the original list can still be mutated through its own reference, and the view will reflect those changes.

---

**Q9. When would you use `Set` over `List`?**

> Use `Set` when you need unique elements and do not require index-based access. `HashSet.contains()` is O(1) vs `ArrayList.contains()` at O(n). Use `LinkedHashSet` to preserve insertion order, `TreeSet` for sorted order.

---

**Q10. What is autoboxing and why does it matter for collections?**

> Autoboxing is Java's automatic conversion from primitive types (`int`, `double`) to their wrapper objects (`Integer`, `Double`). Collections only work with objects, so primitives are silently boxed. This matters for performance: in tight loops over large collections, constant boxing creates many short-lived objects and pressures the garbage collector. For large primitive datasets, use arrays or primitive-specialised libraries.

---

**Q11. Why is `Iterable` at the top of the collection hierarchy?**

> Implementing `Iterable<E>` means a class exposes an `iterator()` method, which the enhanced for-each loop (`for (E e : collection)`) relies on. By having `Collection` extend `Iterable`, every collection in JCF automatically works with for-each — without any extra code.

---

**Q12. What is the difference between `Iterator` and `ListIterator`?**

> `Iterator` is available on all collections — it supports forward traversal only (`hasNext()`, `next()`, `remove()`).
> `ListIterator` is specific to `List` — it adds bidirectional traversal (`hasPrevious()`, `previous()`), plus `set()` (replace current) and `add()` (insert before next). It also provides current index via `nextIndex()` and `previousIndex()`.

---
