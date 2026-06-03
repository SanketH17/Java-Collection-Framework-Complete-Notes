# Java Collection Framework — Complete Beginner-Friendly Notes

> **For:** Java learners preparing for interviews (beginner to intermediate).
> **Last Updated:** June 2026 | Java 21 LTS

---

## Table of Contents

1. [What is the Java Collection Framework?](#1-what-is-the-java-collection-framework)
2. [Why Does JCF Exist?](#2-why-does-jcf-exist)
3. [JCF Hierarchy](#3-jcf-hierarchy)
4. [Core Interfaces](#4-core-interfaces)
5. [Core Concepts Before You Code](#5-core-concepts-before-you-code)
   - [5.1 Generics](#51-generics)
   - [5.2 Autoboxing and Unboxing](#52-autoboxing-and-unboxing)
   - [5.3 equals() and hashCode()](#53-equals-and-hashcode)
   - [5.4 Comparable vs Comparator](#54-comparable-vs-comparator)
   - [5.5 Iterator Pattern](#55-iterator-pattern)
   - [5.6 Fail-Fast vs Fail-Safe Iterators](#56-fail-fast-vs-fail-safe-iterators)
   - [5.7 Immutability](#57-immutability)
6. [Which Implementation Should I Use?](#6-which-implementation-should-i-use)
7. [Interview Questions and Answers](#7-interview-questions-and-answers)

---

## 1. What is the Java Collection Framework?

### In Simple Words

The **Java Collection Framework (JCF)** is a set of ready-made classes and interfaces that Java gives you for storing and working with groups of objects.

It lives in the `java.util` package and has been part of Java since **Java 1.2 (1998)**.

### Why Do We Need It?

Without JCF, you would have to build your own data structures from scratch — your own list, your own set, your own sorting method. That is slow, error-prone, and every developer would do it differently.

JCF gives everyone the **same high-quality tools** so you can focus on solving problems, not reinventing lists and maps.

### Simple Analogy

> Think of JCF as a **professional toolbox**.
>
> Before JCF, every carpenter had to forge their own hammer, saw, and screwdriver from raw metal. It worked, but it was slow and no two carpenters' tools were compatible.
>
> JCF gave every Java developer the same battle-tested tools. You just pick the right one and start building.

```
JCF = Toolbox

  [ ArrayList  ]  →  Resizable list with numbered slots
  [ LinkedList ]  →  Chain of connected items
  [ HashMap    ]  →  Labeled file cabinet (key → value)
  [ HashSet    ]  →  Bag that rejects duplicates
  [ TreeMap    ]  →  Alphabetically sorted file cabinet
  [ Queue      ]  →  Ticket counter — first come, first served
```

### What JCF Gives You

| Component | What It Is | Example |
|---|---|---|
| **Interfaces** | Contracts — define *what* can be done | `List`, `Set`, `Map`, `Queue` |
| **Implementations** | Classes — define *how* it is done | `ArrayList`, `HashMap`, `TreeSet` |
| **Algorithms** | Utility methods — ready-made operations | `Collections.sort()`, `Collections.shuffle()` |

---

## 2. Why Does JCF Exist?

### Life Before JCF (Pre-Java 1.2)

Java only had a few built-in options, and they all had problems:

| Old Tool | Problem |
|---|---|
| `Array` | Fixed size — decide the size upfront, cannot change it later |
| `Vector` | Resizable, but every method was `synchronized` — painfully slow |
| `Hashtable` | Key-value store, but synchronized and no null keys/values allowed |
| `Stack` | Extends `Vector` — a design mistake Java itself acknowledged |

```java
// Life before JCF — painful and error-prone
String[] names = new String[10]; // What if we need 11 names?
names[0] = "Alice";
// Need more space? Write your own resize logic.
// Need to search? Write your own loop.
// Need to sort? Write your own algorithm.
```

### What Problems Did This Cause?

| Problem | What It Meant |
|---|---|
| No common interface | Code written for `Vector` could not work with arrays |
| Reinventing the wheel | Every team wrote their own LinkedList, Stack, Queue |
| No standard algorithms | Every developer wrote sorting/searching from scratch |
| Poor reusability | Changing data structure meant rewriting all code that used it |

### What JCF Fixed

JCF solved all of this by giving you:

- **One common interface** (`Collection`) that all collections follow.
- **Flexibility**: write a method that accepts `List` and pass `ArrayList`, `LinkedList`, or anything — same code works.
- **Ready-made algorithms**: sorting, searching, shuffling — fast and correct, out of the box.
- **Type safety** with generics (Java 5+) — mistakes caught at compile time, not runtime.

```java
// Life after JCF — clean, flexible, powerful
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
Collections.sort(names);
System.out.println(names);
```

Output:

```
[Alice, Bob]
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

### Simplified View

```
Iterable<E>
  └── Collection<E>
        ├── List<E>         → ordered, indexed, duplicates OK
        ├── Set<E>          → no duplicates
        │     └── SortedSet<E>    → sorted, no duplicates
        └── Queue<E>        → first in, first out (FIFO)
              └── Deque<E>  → add/remove from both ends

Map<K,V>  (separate — NOT a Collection)
  └── SortedMap<K,V>        → sorted by key
```

### How to Read This

Think of it as a **family tree**:

- `Iterable` is the great-grandparent — it just says "you can loop through me."
- `Collection` is the grandparent — it adds basic operations like add, remove, contains.
- `List`, `Set`, `Queue` are parents — each adds their own special rules.
- `ArrayList`, `HashSet`, etc. are the actual classes you use in your code.

---

## 4. Core Interfaces

These are the contracts that define what each type of collection can do. Think of them as **job descriptions** — the interface says what the job requires, the implementation class does the actual work.

| Interface | What It Does | Think of It As | Main Classes |
|---|---|---|---|
| `Iterable<E>` | Can be looped with for-each | "I am walkable" | All collections |
| `Collection<E>` | Base — add, remove, contains, size | A generic container | (abstract base) |
| `List<E>` | Ordered, indexed, duplicates OK | Numbered notebook pages | `ArrayList`, `LinkedList` |
| `Set<E>` | No duplicates allowed | Guest list — unique names only | `HashSet`, `TreeSet` |
| `SortedSet<E>` | Sorted, no duplicates | Alphabetically sorted guest list | `TreeSet` |
| `Queue<E>` | First in, first out (FIFO) | Cinema ticket line | `PriorityQueue`, `ArrayDeque` |
| `Deque<E>` | Add/remove from both ends | Deck of cards | `ArrayDeque`, `LinkedList` |
| `Map<K,V>` | Key → Value, unique keys | Dictionary / phone book | `HashMap`, `TreeMap` |
| `SortedMap<K,V>` | Sorted by key | Sorted dictionary | `TreeMap` |

### What Methods Does Each Interface Have?

```
Collection<E>              List<E> (adds to Collection)     Set<E>
─────────────              ──────────────────────────       ──────────────────
add(e)                     get(index)        → O(1)*        (same as Collection
remove(o)                  set(index, e)     → O(1)*         — no extra methods)
contains(o)                add(index, e)     → O(n)
size()                     remove(index)     → O(n)
isEmpty()                  indexOf(o)        → O(n)
iterator()                 subList(from, to)
clear()                    listIterator()
```

*\* O(1) for ArrayList. Slower for LinkedList.*

---

## 5. Core Concepts Before You Code

These are the **building blocks** you need to understand before working with any collection.

---

### 5.1 Generics

#### What Is It?

Generics let you tell a collection **what type** of items it should hold. You write the type inside angle brackets `<>`.

#### Why Do We Need It?

Without generics, a collection can hold anything — strings, numbers, objects, all mixed together. Mistakes only show up when the program **crashes at runtime**. With generics, the compiler catches type mistakes **before the code even runs**.

#### Simple Analogy

> A generic is like a **label on a storage box**.
>
> Without a label: you might accidentally put kitchen utensils in a box meant for books.
>
> With a label (`Box<Books>`): only books go in. Try putting a screwdriver in — stopped immediately.

```java
// WITHOUT generics — dangerous
List rawList = new ArrayList();
rawList.add("Alice");
rawList.add(42);                        // no error — types are mixed!
String name = (String) rawList.get(1);  // CRASH! ClassCastException at runtime
```

```java
// WITH generics — safe
List<String> names = new ArrayList<>();
names.add("Alice");
// names.add(42);                       // COMPILE ERROR — caught immediately
String name = names.get(0);            // no cast needed, works perfectly
```

> **Interview Tip:** Generics use **type erasure** — they only exist at compile time. At runtime, `ArrayList<String>` is just `ArrayList`. This is why you cannot write `instanceof ArrayList<String>`.

---

### 5.2 Autoboxing and Unboxing

#### What Is It?

Collections can only hold **objects** — they cannot store primitives like `int`, `double`, or `boolean` directly. Java automatically converts between primitives and their wrapper classes.

| Direction | Name | What Happens |
|---|---|---|
| `int` → `Integer` | **Autoboxing** | Java wraps the primitive in an object |
| `Integer` → `int` | **Unboxing** | Java unwraps the object to a primitive |

#### Simple Analogy

> A collection is like a **parcel delivery service** — it only accepts boxed packages (objects).
>
> **Autoboxing** = the service puts your loose item (primitive) into a box for you.
> **Unboxing** = you open the box on delivery to get the item out.

```java
List<Integer> scores = new ArrayList<>();
scores.add(95);                // autoboxing: int 95 → Integer(95)
int topScore = scores.get(0);  // unboxing:   Integer(95) → int 95
```

> **Performance Tip:** In tight loops, constant autoboxing creates many short-lived objects. For large numeric datasets, prefer plain `int[]` arrays.

---

### 5.3 equals() and hashCode()

#### What Is It?

- `equals()` tells Java when two objects should be considered **"the same"**.
- `hashCode()` gives each object a **numeric fingerprint** used for fast lookups in hash-based collections.

#### Why Do We Need It?

Methods like `contains()`, `remove(Object)`, and `indexOf()` use `equals()` to compare objects. Without overriding it, Java compares **memory addresses** — two objects with identical data are treated as different.

#### Example

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

System.out.println(students.contains(new Student("Alice", 101)));
```

Output:

```
true
```

Without the `equals()` override, this would print `false` because Java would compare memory addresses instead of actual data.

> **Rule:** Always override `equals()` and `hashCode()` **together**. Java's contract says: if two objects are equal, they MUST have the same hash code.

---

### 5.4 Comparable vs Comparator

#### What Is It?

Two ways to tell Java **how to sort** your objects.

| | `Comparable<T>` | `Comparator<T>` |
|---|---|---|
| Where defined | **Inside** the class itself | **Outside** — as a separate object |
| Method | `compareTo(T other)` | `compare(T o1, T o2)` |
| Use case | Default/natural ordering | Custom or multiple orderings |
| How many per class | One | As many as you want |

#### Simple Analogy

> **Comparable** = A person who knows their own place in line (e.g., by age). They carry their ordering rule with them.
>
> **Comparator** = An event organizer who decides the ordering for each event — by height for one, by ticket number for another.

#### Comparable — Default Ordering (Inside the Class)

```java
class Employee implements Comparable<Employee> {
    String name;
    int salary;

    Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary); // sort by salary
    }

    public String toString() { return name + "($" + salary + ")"; }
}

List<Employee> team = new ArrayList<>();
team.add(new Employee("Alice", 80000));
team.add(new Employee("Bob", 60000));
team.add(new Employee("Carol", 70000));

Collections.sort(team); // uses compareTo() — sorts by salary
System.out.println(team);
```

Output:

```
[Bob($60000), Carol($70000), Alice($80000)]
```

#### Comparator — Custom Ordering (Outside the Class)

```java
// Sort by name
team.sort(Comparator.comparing(e -> e.name));
System.out.println(team);
```

Output:

```
[Alice($80000), Bob($60000), Carol($70000)]
```

```java
// Sort by salary descending
team.sort(Comparator.comparingInt((Employee e) -> e.salary).reversed());
System.out.println(team);
```

Output:

```
[Alice($80000), Carol($70000), Bob($60000)]
```

---

### 5.5 Iterator Pattern

#### What Is It?

A standard way to walk through **any** collection one element at a time, without knowing what is going on inside.

```
Collection<E>
     |
     +-- iterator()
              |
              v
         Iterator<E>
              +-- hasNext()  → is there another element?
              +-- next()     → give me the next element
              +-- remove()   → safely remove the current element
```

#### Why Do We Need It?

Without it, you would need to know the internals of each collection to loop through it. With `Iterator`, the **same code** works for `ArrayList`, `LinkedList`, `HashSet`, and any other collection.

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Delhi", "Bangalore"));

// Using Iterator directly
Iterator<String> it = cities.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

Output:

```
Mumbai
Delhi
Bangalore
```

The enhanced for-each loop is just a shortcut for the above:

```java
// This is exactly the same thing, just shorter
for (String city : cities) {
    System.out.println(city);
}
```

Output:

```
Mumbai
Delhi
Bangalore
```

---

### 5.6 Fail-Fast vs Fail-Safe Iterators

#### What Is It?

This decides what happens when someone **modifies a collection while you are looping through it**.

#### Simple Analogy

> **Fail-fast** = A photocopier that **stops immediately** if someone changes the original document mid-copy. It refuses to continue with an inconsistent state.
>
> **Fail-safe** = A photocopier that **takes a snapshot first**, then copies from the snapshot. Changes to the original do not affect the copy job.

| | Fail-Fast | Fail-Safe |
|---|---|---|
| Examples | `ArrayList`, `HashMap`, `HashSet` | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| When modified during loop | Throws `ConcurrentModificationException` | No error — works on a snapshot |
| Extra memory | None | Yes — needs space for the snapshot |
| Best for | Single-threaded programs | Multi-threaded programs with many reads |

```java
// FAIL-FAST — modifying during for-each loop CRASHES
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
for (String s : list) {
    list.remove(s);  // CRASH! ConcurrentModificationException
}
```

```java
// FAIL-SAFE — works because it iterates over a snapshot
List<String> cowList = new CopyOnWriteArrayList<>(List.of("A", "B", "C"));
for (String s : cowList) {
    cowList.remove(s);  // No error
}
System.out.println(cowList);
```

Output:

```
[]
```

---

### 5.7 Immutability

#### What Is It?

Creating collections that **cannot be changed** after they are created — no adding, removing, or replacing elements.

#### Why Do We Need It?

Immutable collections are safe to share across threads, safe to return from methods, and prevent accidental changes.

| Method | Can Modify? | Allows Null? | Since |
|---|---|---|---|
| `new ArrayList<>()` | Yes — fully mutable | Yes | Java 1.2 |
| `Arrays.asList(...)` | Partial — `set()` works, `add()`/`remove()` don't | Yes | Java 1.2 |
| `Collections.unmodifiableList()` | No — but original can still change | Yes | Java 1.2 |
| `List.of(...)` | No — truly locked down | No | Java 9 |
| `List.copyOf(...)` | No — truly locked down | No | Java 10 |

```java
// Truly immutable — nobody can change it
List<String> menu = List.of("Pizza", "Pasta", "Salad");
// menu.add("Soup");     // CRASH — UnsupportedOperationException
```

```java
// Unmodifiable VIEW — the original CAN still be changed!
List<String> mutable = new ArrayList<>(List.of("A", "B"));
List<String> readOnly = Collections.unmodifiableList(mutable);

mutable.add("C");             // this works — changes through original reference
System.out.println(readOnly); // the view reflects the change!
```

Output:

```
[A, B, C]
```

```java
// Truly independent snapshot — Java 10+
List<String> snapshot = List.copyOf(mutable);
mutable.add("D");
System.out.println(snapshot); // snapshot is unaffected
```

Output:

```
[A, B, C]
```

> **Key Difference:** `List.of()` is truly immutable — nobody can change it. `Collections.unmodifiableList()` is just a read-only *view* — the original list can still be changed through its own reference.

---

## 6. Which Implementation Should I Use?

This section helps you pick the right class for the job.

### List — Ordered, Indexed, Duplicates OK

| Class | Best For | Avoid When |
|---|---|---|
| `ArrayList` | General use — your default choice | Frequent insert/delete in the middle |
| `LinkedList` | Frequent add/remove at the start or end | You need fast access by index |
| `Vector` | Legacy code only — **do not use in new code** | Always |
| `CopyOnWriteArrayList` | Multi-threaded, mostly reading | Lots of writes |

### Set — Unique Elements Only

| Class | Best For | Avoid When |
|---|---|---|
| `HashSet` | Fast O(1) lookups, no duplicates | You need elements in a specific order |
| `LinkedHashSet` | Unique elements + keeps insertion order | Memory is tight |
| `TreeSet` | Sorted elements, range queries | Null elements or performance-critical code |

### Map — Key-Value Pairs

| Class | Best For | Avoid When |
|---|---|---|
| `HashMap` | Fast O(1) lookups — your default choice | You need keys in order |
| `LinkedHashMap` | Keeps insertion order of keys | Memory is tight |
| `TreeMap` | Sorted keys, range queries | You need O(1) speed |
| `Hashtable` | Legacy code only — **do not use in new code** | Always |

### Queue and Deque — Processing in Order

| Class | Best For | Notes |
|---|---|---|
| `PriorityQueue` | Process by priority (smallest/largest first) | Not FIFO — heap ordered |
| `ArrayDeque` | Stack or Queue — best all-around Deque | Faster than `LinkedList` for most uses |
| `LinkedList` | When you already use it as a List and need Deque | Uses more memory per item |

### Quick Decision Guide

```
What do you need?

Ordered list with duplicates?         → ArrayList  (default)
Unique elements only?                 → HashSet    (default)
Key-value pairs?                      → HashMap    (default)
First-in-first-out queue?             → ArrayDeque
Sorted elements?                      → TreeSet / TreeMap
Thread safety + heavy reads?          → CopyOnWriteArrayList / ConcurrentHashMap
```

> For a complete deep dive into **ArrayList**, see [ArrayList-Notes.md](ArrayList-Notes.md).

---

## 7. Interview Questions and Answers

**Q1. What is the Java Collection Framework?**

> A set of ready-made interfaces, classes, and algorithms in `java.util` for storing and working with groups of objects. It was introduced in Java 1.2 to replace the old `Vector`/`Hashtable`/`Stack` tools.

---

**Q2. Does `Map` extend `Collection`?**

> No. `Map<K,V>` is a completely separate hierarchy. It does NOT extend `Collection`. This is one of the most common trick questions in interviews.

---

**Q3. What is the difference between `Collection` and `Collections`?**

> - `Collection` (singular) is an **interface** — the root of the collection hierarchy.
> - `Collections` (plural) is a **utility class** with static helper methods like `sort()`, `shuffle()`, `reverse()`, and `binarySearch()`.

---

**Q4. What is the difference between `List`, `Set`, and `Map`?**

> - **List**: ordered, indexed, allows duplicates. Example: `[Alice, Bob, Alice]`
> - **Set**: no duplicates, no index access. Example: `{Alice, Bob}`
> - **Map**: key-value pairs, keys must be unique. Example: `{name=Alice, age=25}`

---

**Q5. What is type erasure in generics?**

> At compile time, generics check types. At runtime, the type info is erased. `ArrayList<String>` and `ArrayList<Integer>` both become plain `ArrayList` at runtime. This is why `instanceof ArrayList<String>` does not work.

---

**Q6. What is the difference between `Comparable` and `Comparator`?**

> - **Comparable** is written **inside** the class — defines one default ordering.
> - **Comparator** is written **outside** — you can create as many as you want for different orderings.

---

**Q7. What is a fail-fast iterator?**

> An iterator that throws `ConcurrentModificationException` if the collection is modified while you are looping through it (unless you use the iterator's own `remove()` method). Iterators of `ArrayList`, `HashMap`, and `HashSet` are all fail-fast.

---

**Q8. What is the difference between `List.of()` and `Collections.unmodifiableList()`?**

> - `List.of()` creates a truly **immutable** list — nobody can change it, and it rejects null.
> - `Collections.unmodifiableList()` creates a read-only **view** — the original list can still be changed through its own reference, and the view will reflect those changes.

---

**Q9. When would you use `Set` over `List`?**

> When you need **unique elements** and do not need index-based access. `HashSet.contains()` is O(1), while `ArrayList.contains()` is O(n).

---

**Q10. What is autoboxing and why does it matter?**

> Autoboxing is Java automatically converting `int` → `Integer`, `double` → `Double`, etc. Collections only store objects, so primitives are silently wrapped. In tight loops, this creates many short-lived objects and slows things down. For large numeric data, use plain arrays instead.

---

**Q11. Why is `Iterable` at the top of the hierarchy?**

> Because it provides the `iterator()` method, which the for-each loop (`for (E e : collection)`) relies on. By having `Collection` extend `Iterable`, every collection automatically works with for-each — no extra code needed.

---

**Q12. What is the difference between `Iterator` and `ListIterator`?**

> - **Iterator** works on all collections — forward only (`hasNext()`, `next()`, `remove()`).
> - **ListIterator** is for Lists only — adds backward movement (`hasPrevious()`, `previous()`), plus `set()` (replace) and `add()` (insert). Also gives you the current index.

---
