# ArrayList — Complete Deep Dive

> Part of the [Java Collection Framework Notes](README.md)
> **Audience:** Java developers preparing for interviews (fresher to senior).
> **Coverage:** Internals → CRUD → Iteration → Sorting → Thread Safety → Interview Q&A.
> **Last Updated:** June 2026 | Java 21 LTS

---

## Table of Contents

1. [What is ArrayList?](#1-what-is-arraylist)
2. [Internal Working — How it Works Under the Hood](#2-internal-working--how-it-works-under-the-hood)
3. [Capacity vs Size — The Most Important Distinction](#3-capacity-vs-size--the-most-important-distinction)
4. [Dynamic Resizing](#4-dynamic-resizing)
5. [Creating an ArrayList](#5-creating-an-arraylist)
6. [CRUD Operations](#6-crud-operations)
   - 6.1 [Adding Elements (Create)](#61-adding-elements-create)
   - 6.2 [Accessing Elements (Read)](#62-accessing-elements-read)
   - 6.3 [Modifying Elements (Update)](#63-modifying-elements-update)
   - 6.4 [Removing Elements (Delete)](#64-removing-elements-delete)
7. [Searching](#7-searching)
8. [Iterating Over ArrayList](#8-iterating-over-arraylist)
9. [Sorting](#9-sorting)
10. [Useful Methods](#10-useful-methods)
    - 10.1 [subList — Working with Ranges](#101-sublist--working-with-ranges)
    - 10.2 [Bulk Operations](#102-bulk-operations)
    - 10.3 [Converting ArrayList](#103-converting-arraylist)
11. [Null Handling](#11-null-handling)
12. [Fail-Fast Iterator and ConcurrentModificationException](#12-fail-fast-iterator-and-concurrentmodificationexception)
13. [Thread Safety](#13-thread-safety)
14. [Memory Management](#14-memory-management)
15. [Stream API with ArrayList](#15-stream-api-with-arraylist)
16. [Java 9+ Factory Methods](#16-java-9-factory-methods)
17. [Performance Analysis — Big-O](#17-performance-analysis--big-o)
18. [ArrayList vs Array](#18-arraylist-vs-array)
19. [ArrayList vs LinkedList](#19-arraylist-vs-linkedlist)
20. [ArrayList vs Vector](#20-arraylist-vs-vector)
21. [Best Practices](#21-best-practices)
22. [Common Pitfalls](#22-common-pitfalls)
23. [Interview Questions and Answers](#23-interview-questions-and-answers)
24. [Quick Reference Cheat Sheet](#24-quick-reference-cheat-sheet)

---

## 1. What is ArrayList?

### What is it?

`ArrayList<E>` is a **resizable array** implementation of the `List` interface from `java.util`.

It solves one of the biggest limitations of regular arrays — the **fixed size** problem.

### Why do we need it?

With a regular array, you must decide the size upfront. Once set, you cannot grow or shrink it:

```java
String[] names = new String[10]; // Can NEVER hold more than 10 items
```

With ArrayList, the size adjusts automatically as you add or remove elements.

### Real-World Analogy

> Think of ArrayList as a **dynamic notebook**.
>
> A regular array is a notebook bought with exactly 50 pages — bound, fixed, done. You can never add a page.
>
> ArrayList is a notebook that gets more pages as you write. Behind the scenes, when you run out of space, Java quietly swaps it for a bigger notebook and copies everything over — but you never notice this happening.

### Key Characteristics at a Glance

```
ArrayList<E>
──────────────────────────────────────────────────
  Maintains insertion order           YES
  Allows duplicate elements           YES
  Allows null values                  YES
  Random access by index              YES — O(1)
  Grows automatically                 YES
  Thread-safe                         NO
  Fast insert/delete in the middle    NO  — O(n)
──────────────────────────────────────────────────
```

### First Example

```java
import java.util.ArrayList;
import java.util.List;

// Always declare as List<E>, not ArrayList<E>
List<String> employees = new ArrayList<>();

employees.add("Alice");    // index 0
employees.add("Bob");      // index 1
employees.add("Charlie");  // index 2
employees.add("Alice");    // duplicates allowed

System.out.println(employees);          // [Alice, Bob, Charlie, Alice]
System.out.println(employees.get(1));   // Bob — direct O(1) access
System.out.println(employees.size());   // 4
```

> **Interview Tip:** Always declare the variable as `List<String>`, not `ArrayList<String>`. This programs to the interface and lets you swap implementations later without changing every method signature.

---

## 2. Internal Working — How it Works Under the Hood

Understanding internals is what separates good candidates from great ones.

### The Backing Array

Internally, ArrayList stores your elements in a plain Java `Object[]` array named `elementData`.

```java
// Simplified from OpenJDK source
public class ArrayList<E> extends AbstractList<E> {

    private static final int DEFAULT_CAPACITY = 10;

    transient Object[] elementData;  // the actual storage array
    private int size;                // number of elements stored (NOT the array length)
}
```

### Internal Memory Representation

```
ArrayList object in memory:
+------------------------------+
|  elementData  ->  Object[]   |
|  size         =  4           |
+------------------------------+
              |
              v
+------+------+------+------+------+------+------+------+------+------+
| [0]  | [1]  | [2]  | [3]  | [4]  | [5]  | [6]  | [7]  | [8]  | [9]  |
|"Ali" |"Bob" |"Car" |"Dan" | null | null | null | null | null | null |
+------+------+------+------+------+------+------+------+------+------+
  <--- stored elements (size=4) --->  <--- empty slots (capacity=10) --->
```

### Why an Object Array?

Due to **type erasure**, generics only exist at compile time. At runtime, `ArrayList<String>` is just an `ArrayList` storing `Object` references. The `<String>` part is purely a compile-time safety net.

### How add() Works Step by Step

```
add("NewItem") is called:
        |
        v
Is size == elementData.length?  (is the array full?)
        |
        +-- NO  -->  elementData[size] = "NewItem"
        |             size++
        |             Done. O(1).
        |
        +-- YES -->  RESIZE (see Section 4)
                      then store element, size++
```

> **Interview Tip:** This is why `add()` at the end is O(1) *amortized* — it is almost always instant, but occasionally triggers an O(n) copy when the internal array needs to grow.

---

## 3. Capacity vs Size — The Most Important Distinction

This is the most commonly asked internal concept in ArrayList interviews.

### Real-World Analogy

> Think of ArrayList like a **cinema hall**.
>
> **Capacity** = the total number of seats built into the hall (e.g., 200 seats).
> **Size** = the number of people currently seated (e.g., 75 people).
>
> You can keep adding people until you hit 200. After that, you need a bigger hall.

```
Cinema Hall Analogy:
+----------------------------------------------------+
|  CAPACITY: 200 total seats                         |
|                                                    |
|  [P][P][P]... 75 people ...[ ][ ][ ][ ][ ][ ]...  |
|  <---- SIZE: 75 ---------->  <-- empty seats -->   |
+----------------------------------------------------+
```

### In Code

```java
ArrayList<String> list = new ArrayList<>(); // capacity=0 internally, size=0
list.add("Alice");  // capacity jumps to 10, size=1
list.add("Bob");    // capacity=10, size=2
list.add("Carol");  // capacity=10, size=3

System.out.println(list.size()); // 3 — how many elements stored
// Note: there is no list.capacity() — capacity is internal with no public getter
```

### Summary Table

| Term | What It Means | Public? |
|---|---|---|
| **size** | Number of elements currently stored | YES — `list.size()` |
| **capacity** | Length of the internal `Object[]` array | NO — internal only |

---

## 4. Dynamic Resizing

### What is it?

When the internal array is full and you add another element, ArrayList automatically creates a larger array and copies everything over.

### When Does Resize Happen?

```
add() is called
      |
      v
size == elementData.length? (array is full?)
      |
      +-- NO  -->  Store element, increment size. Done. O(1).
      |
      +-- YES -->  RESIZE:
                    1. newCapacity = old + (old >> 1)  [approx 1.5x]
                    2. Allocate new, larger array
                    3. Copy all elements with Arrays.copyOf()
                    4. Old array becomes garbage collectible
                    5. Store new element, increment size
```

### Growth Formula

```java
// OpenJDK source (simplified)
newCapacity = oldCapacity + (oldCapacity >> 1)
// = oldCapacity + oldCapacity/2 = approximately 1.5x growth
```

### Growth in Action

```
Initial capacity:  10  (after first add)
After 1st resize:  10 + 5  = 15
After 2nd resize:  15 + 7  = 22
After 3rd resize:  22 + 11 = 33
After 4th resize:  33 + 16 = 49
After 5th resize:  49 + 24 = 73
```

### Visualising a Resize

```
Before resize — array is FULL (capacity=4, size=4):
+------+------+------+------+
|  "A" |  "B" |  "C" |  "D" |
+------+------+------+------+

add("E") triggered — resize to capacity=6, copy elements:
+------+------+------+------+------+------+
|  "A" |  "B" |  "C" |  "D" |  "E" |      |
+------+------+------+------+------+------+
Old array is discarded (garbage collected)
```

### Why is add() O(1) Amortized?

Even though a resize costs O(n), it happens so infrequently that the average cost per add is still O(1).

```
Insertions 1-10:    no resize    -> 10 x O(1) = 10 ops
Insertion 11:       resize       -> copy 10 + store 1 = 11 ops
Insertions 12-15:   no resize    -> 4 x O(1) = 4 ops
Insertion 16:       resize       -> copy 15 + store 1 = 16 ops

Total for 16 adds: 10 + 11 + 4 + 16 = 41 ~= 2.5 x 16
Average per add: ~2.5 -> O(1) amortized
```

> **Real-World Analogy:** Imagine moving to a 1.5x bigger house each time you run out of space. Each individual move is expensive, but you move very rarely. Averaged over all the days you live there, the moving cost per day is essentially constant.

> **Interview Tip:** The correct answer to "What is the time complexity of ArrayList.add()?" is:
> **O(1) amortized for appending to the end. O(n) worst case during a resize.**

---

## 5. Creating an ArrayList

There are three constructors. Knowing when to use each demonstrates practical understanding.

### Constructor 1 — Default (empty list)

```java
List<String> employees = new ArrayList<>();
// Internally starts with an EMPTY array (zero-length — lazy initialization).
// On the first add(), capacity is set to DEFAULT_CAPACITY = 10.
```

### Constructor 2 — With Initial Capacity

```java
List<String> products = new ArrayList<>(500);
// Starts with capacity=500.
// No resize will occur until you add the 501st element.
// Use when you know approximately how many elements you will store.
```

> **Best Practice:** If you know you will add ~1000 elements, use `new ArrayList<>(1000)`. This avoids ~6 resize operations and their copying overhead.

### Constructor 3 — From Another Collection

```java
List<String> source = List.of("Alice", "Bob", "Charlie");
List<String> copy = new ArrayList<>(source);
// Creates a new, fully independent ArrayList with the same elements.
// Order follows the source collection's iterator order.
```

### Interface vs Implementation Declaration

```java
// PREFERRED — declare as the interface
List<String> employees = new ArrayList<>();

// AVOID — declares as the concrete implementation
ArrayList<String> employees = new ArrayList<>();

// Why it matters:
// With the preferred style, changing to LinkedList or CopyOnWriteArrayList
// requires changing only one word — the right side of the assignment.
// With the avoided style, every method signature using this type is locked to ArrayList.
```

---

## 6. CRUD Operations

### 6.1 Adding Elements (Create)

#### Append to End — `add(E e)` — O(1) amortized

```java
List<String> cart = new ArrayList<>();

cart.add("Laptop");    // index 0
cart.add("Keyboard");  // index 1
cart.add("Mouse");     // index 2

System.out.println(cart); // [Laptop, Keyboard, Mouse]
```

**Internal steps:**
```
add("Mouse"), size=2, capacity=10:
  elementData[2] = "Mouse"
  size = 3
  Done in one step — O(1).
```

---

#### Insert at Index — `add(int index, E element)` — O(n)

Inserts at a specific position. Every element from that index onwards is shifted one place to the right.

```java
List<String> agenda = new ArrayList<>(List.of("9AM Stand-up", "11AM Code Review"));
agenda.add(1, "10AM Interview");

System.out.println(agenda);
// [9AM Stand-up, 10AM Interview, 11AM Code Review]
```

**What happens internally:**
```
Before: [ "9AM Stand-up" | "11AM Code Review" ]
              index 0           index 1

Shift right from index 1:
        [ "9AM Stand-up" |         ???         | "11AM Code Review" ]

Insert: [ "9AM Stand-up" | "10AM Interview" | "11AM Code Review" ]
              index 0           index 1            index 2
```

> **Interview Tip:** Every middle insert shifts all elements to the right — that is why it is O(n). If your code frequently inserts at the beginning, consider `ArrayDeque` or `LinkedList` instead.

---

#### Add All Elements — `addAll(Collection<? extends E> c)` — O(n)

```java
List<String> teamA = new ArrayList<>(List.of("Alice", "Bob"));
List<String> teamB = List.of("Charlie", "Diana");

teamA.addAll(teamB);
System.out.println(teamA); // [Alice, Bob, Charlie, Diana]
```

#### Insert All at Index — `addAll(int index, Collection<? extends E> c)` — O(n)

```java
List<String> playlist = new ArrayList<>(List.of("Track 1", "Track 4"));
playlist.addAll(1, List.of("Track 2", "Track 3"));
System.out.println(playlist); // [Track 1, Track 2, Track 3, Track 4]
```

---

### 6.2 Accessing Elements (Read)

#### Get by Index — `get(int index)` — O(1)

This is ArrayList's **superpower**. Direct memory address calculation — no scanning needed.

```java
List<String> countries = new ArrayList<>(List.of("India", "USA", "Japan", "Germany"));

String first = countries.get(0);                     // "India"
String last  = countries.get(countries.size() - 1);  // "Germany"
String third = countries.get(2);                     // "Japan"
```

**What happens internally:**
```
get(2):
  return (E) elementData[2]
  Direct array index lookup — computed in one step. O(1).
```

> **Interview Tip:** `get(index)` is O(1) because the array is contiguous in memory. The JVM computes the address directly: `baseAddress + (index x referenceSize)`. No scanning, no traversal.

**Invalid indices throw immediately:**
```java
countries.get(-1);  // IndexOutOfBoundsException
countries.get(10);  // IndexOutOfBoundsException (valid: 0 to 3)
```

---

### 6.3 Modifying Elements (Update)

#### Replace at Index — `set(int index, E element)` — O(1)

Replaces the element at a given index. Returns the **old** element.

```java
List<String> menu = new ArrayList<>(List.of("Burger", "Sandwitch", "Pizza")); // typo!
String old = menu.set(1, "Sandwich");  // fix the typo

System.out.println("Was: " + old);   // "Sandwitch"
System.out.println("Menu: " + menu); // [Burger, Sandwich, Pizza]
```

> `set()` **replaces** an existing element. It does NOT increase the list size.

---

### 6.4 Removing Elements (Delete)

#### Remove by Index — `remove(int index)` — O(n)

Removes the element at the given position. All subsequent elements shift one place left.

```java
List<String> queue = new ArrayList<>(List.of("Alice", "Bob", "Charlie", "Diana"));
String served = queue.remove(0); // serve the first person

System.out.println("Served: " + served); // "Alice"
System.out.println("Queue:  " + queue);  // [Bob, Charlie, Diana]
```

**Internal shift operation:**
```
Before: [ "Alice" | "Bob" | "Charlie" | "Diana" ]
           idx 0     idx 1    idx 2       idx 3

Remove idx 0 and shift left:
After:  [ "Bob" | "Charlie" | "Diana" | null ]
           idx 0    idx 1      idx 2
          size = 3 (last slot nulled for GC)
```

---

#### Remove by Value — `remove(Object o)` — O(n)

Removes the **first** occurrence. Returns `true` if found, `false` if not.

```java
List<String> wishlist = new ArrayList<>(List.of("Laptop", "Phone", "Laptop", "Tablet"));
boolean removed = wishlist.remove("Laptop"); // removes FIRST "Laptop"

System.out.println("Removed: " + removed);  // true
System.out.println("Wishlist: " + wishlist); // [Phone, Laptop, Tablet]
```

---

#### Critical Trap — `remove(int)` vs `remove(Object)` for Integer Lists

```java
List<Integer> scores = new ArrayList<>(List.of(100, 200, 300, 200));

// remove(int) — removes by INDEX (position)
scores.remove(1);
// Removes element AT index 1 -> the value 200 is removed
// scores = [100, 300, 200]

// remove(Object) — removes by VALUE
scores.remove(Integer.valueOf(200));
// Removes the FIRST occurrence of the value 200
// scores = [100, 300]
```

> **Interview Tip:** For `List<Integer>`, `list.remove(1)` removes by *index*. To remove by *value*, use `list.remove(Integer.valueOf(1))`. Always be explicit.

---

#### Conditional Removal — `removeIf(Predicate)` — O(n)  (Java 8+)

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
numbers.removeIf(n -> n % 2 == 0); // remove all even numbers

System.out.println(numbers); // [1, 3, 5, 7, 9]
```

> Prefer `removeIf` over manual loop removal — it handles internal state safely and is more readable.

---

#### Remove All Matching — `removeAll(Collection<?> c)` — O(n^2)

```java
List<String> stock = new ArrayList<>(List.of("Apple", "Banana", "Cherry", "Banana", "Date"));
stock.removeAll(List.of("Banana", "Date"));

System.out.println(stock); // [Apple, Cherry]
```

---

#### Keep Only Matching — `retainAll(Collection<?> c)` — O(n^2)

```java
List<String> available = new ArrayList<>(List.of("Apple", "Banana", "Cherry", "Mango"));
available.retainAll(List.of("Banana", "Mango")); // keep only these

System.out.println(available); // [Banana, Mango]
```

---

#### Remove All Elements — `clear()` — O(n)

```java
List<String> cart = new ArrayList<>(List.of("Item1", "Item2", "Item3"));
cart.clear();

System.out.println(cart);          // []
System.out.println(cart.size());   // 0
// clear() nulls out all references (for GC) but does NOT reset capacity
```

---

## 7. Searching

### `contains(Object o)` — O(n)

```java
List<String> fruits = new ArrayList<>(List.of("Apple", "Banana", "Cherry"));

System.out.println(fruits.contains("Banana")); // true
System.out.println(fruits.contains("Mango"));  // false
System.out.println(fruits.contains(null));      // false (no null in this list)
```

> Uses `equals()` for comparison. Always override `equals()` in custom objects you plan to search for.

---

### `indexOf(Object o)` — O(n)

Returns the index of the **first** occurrence. Returns `-1` if not found.

```java
List<String> log = new ArrayList<>(List.of("INFO", "WARN", "ERROR", "INFO", "WARN"));

System.out.println(log.indexOf("WARN"));  // 1  (first occurrence)
System.out.println(log.indexOf("DEBUG")); // -1 (not found)
```

---

### `lastIndexOf(Object o)` — O(n)

```java
System.out.println(log.lastIndexOf("WARN")); // 4 (last occurrence)
```

---

### Binary Search — O(log n)  (sorted lists only)

```java
import java.util.Collections;

List<Integer> sorted = new ArrayList<>(List.of(10, 20, 30, 40, 50, 60));
int index = Collections.binarySearch(sorted, 40);
System.out.println(index); // 3

// WARNING: The list MUST be sorted before calling binarySearch.
// On an unsorted list, results are undefined.
```

---

### Size and Empty Checks

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C"));

System.out.println(list.size());    // 3
System.out.println(list.isEmpty()); // false

System.out.println(new ArrayList<>().isEmpty()); // true
```

> Prefer `isEmpty()` over `size() == 0` — more readable and intention-revealing.

---

## 8. Iterating Over ArrayList

There are **6 ways** to iterate an ArrayList. Knowing all of them is a key interview topic.

### Method 1 — Classic for-loop (index-based)

**Use when:** you need the index, or need to iterate in reverse.

```java
List<String> products = new ArrayList<>(List.of("Laptop", "Phone", "Tablet"));

for (int i = 0; i < products.size(); i++) {
    System.out.println(i + ". " + products.get(i));
}
// 0. Laptop
// 1. Phone
// 2. Tablet
```

---

### Method 2 — Enhanced for-each loop

**Use when:** you just need each element, no index required.

```java
for (String product : products) {
    System.out.println(product);
}
// Internally converted by the compiler to an Iterator loop.
// Cannot structurally modify the list during iteration.
```

---

### Method 3 — Iterator

**Use when:** you need to **safely remove** elements while iterating.

```java
List<String> tasks = new ArrayList<>(List.of("Done", "Pending", "Done", "In Progress"));

Iterator<String> it = tasks.iterator();
while (it.hasNext()) {
    String task = it.next();
    if (task.equals("Done")) {
        it.remove(); // safe — updates internal state properly
    }
}
System.out.println(tasks); // [Pending, In Progress]
```

---

### Method 4 — ListIterator (bidirectional)

**Use when:** you need to traverse **backwards**, or **replace/insert** elements during iteration.

```java
List<String> names = new ArrayList<>(List.of("alice", "bob", "charlie"));
ListIterator<String> lit = names.listIterator();

// Forward pass — capitalize each name
while (lit.hasNext()) {
    String name = lit.next();
    lit.set(name.substring(0, 1).toUpperCase() + name.substring(1));
}
System.out.println(names); // [Alice, Bob, Charlie]

// Backward pass
while (lit.hasPrevious()) {
    System.out.print(lit.previous() + " ");
}
// Charlie Bob Alice
```

**ListIterator methods:**

| Method | What it Does |
|---|---|
| `hasNext()` / `next()` | Forward traversal |
| `hasPrevious()` / `previous()` | Backward traversal |
| `nextIndex()` / `previousIndex()` | Current position |
| `set(E e)` | Replace current element |
| `add(E e)` | Insert before the next element |
| `remove()` | Remove current element |

---

### Method 5 — forEach with Lambda  (Java 8+)

**Use when:** applying an action to each element cleanly, no removal needed.

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Delhi", "Bangalore"));

cities.forEach(city -> System.out.println("Visiting: " + city));
cities.forEach(System.out::println); // method reference — even cleaner
```

---

### Method 6 — Stream API  (Java 8+)

**Use when:** filtering, transforming, or aggregating — functional style.

```java
List<String> employees = new ArrayList<>(List.of("Alice", "Adam", "Bob", "Anna", "Carol"));

employees.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .forEach(System.out::println);
// ALICE
// ADAM
// ANNA
```

---

### Comparison Table

| Method | Has Index? | Safe Remove? | Bidirectional? | Best For |
|---|---|---|---|---|
| Classic for-loop | YES | YES (by index) | YES (reverse) | Index-based work |
| Enhanced for-each | NO | NO | NO | Simple read-only loop |
| `Iterator` | NO | YES | NO | Safe removal during loop |
| `ListIterator` | YES | YES | YES | Add/set/remove/reverse |
| `forEach` lambda | NO | NO | NO | Clean one-liners |
| Stream | NO | NO | NO | Filter/map/collect |

---

## 9. Sorting

### Natural Order — `Collections.sort()`

Works for types that implement `Comparable` (like `String`, `Integer`, `LocalDate`).

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Ahmedabad", "Delhi", "Bangalore"));
Collections.sort(cities);
System.out.println(cities); // [Ahmedabad, Bangalore, Delhi, Mumbai]

List<Integer> scores = new ArrayList<>(List.of(89, 45, 92, 71, 55));
Collections.sort(scores);
System.out.println(scores); // [45, 55, 71, 89, 92]
```

---

### Custom Order — `list.sort(Comparator)`  (Java 8+)

```java
List<String> words = new ArrayList<>(List.of("Banana", "Kiwi", "Watermelon", "Fig"));

// Sort by length (shortest first)
words.sort(Comparator.comparingInt(String::length));
System.out.println(words); // [Fig, Kiwi, Banana, Watermelon]

// Reverse alphabetical order
words.sort(Comparator.reverseOrder());
System.out.println(words); // [Watermelon, Kiwi, Fig, Banana]
```

---

### Sorting Custom Objects

```java
class Employee {
    String name;
    int salary;

    Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    @Override
    public String toString() { return name + "($" + salary + ")"; }
}

List<Employee> team = new ArrayList<>();
team.add(new Employee("Alice", 80000));
team.add(new Employee("Bob",   60000));
team.add(new Employee("Carol", 80000));
team.add(new Employee("Dave",  70000));

// Sort by salary ascending
team.sort(Comparator.comparingInt(e -> e.salary));
System.out.println(team);
// [Bob($60000), Dave($70000), Alice($80000), Carol($80000)]

// Sort by salary descending, then name ascending (multi-field sort)
team.sort(Comparator.comparingInt((Employee e) -> e.salary)
                    .reversed()
                    .thenComparing(e -> e.name));
System.out.println(team);
// [Alice($80000), Carol($80000), Dave($70000), Bob($60000)]
```

---

### Sorting Algorithm — TimSort

```
TimSort:
  Algorithm:        Hybrid of merge sort and insertion sort
  Time complexity:  O(n log n) average and worst case
                    O(n) best case (nearly-sorted data)
  Stable:           YES — equal elements preserve their relative order
```

> **Interview Tip:** TimSort is *stable* — when two elements compare equal, their original relative order is preserved. This is critical for multi-field sorts.

---

## 10. Useful Methods

### 10.1 subList — Working with Ranges

`subList(fromIndex, toIndex)` returns a **view** (not a copy) of part of the list.
Range: `[from, to)` — from is inclusive, to is exclusive.

```java
List<String> week = new ArrayList<>(List.of("Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"));
List<String> workdays = week.subList(0, 5);  // Mon to Fri

System.out.println(workdays); // [Mon, Tue, Wed, Thu, Fri]
```

**It is a VIEW — changes affect the original:**

```java
workdays.set(0, "MONDAY");
System.out.println(week); // [MONDAY, Tue, Wed, Thu, Fri, Sat, Sun] — original changed!
```

**Common use — bulk delete by range:**

```java
week.subList(5, 7).clear(); // remove Sat and Sun
System.out.println(week);   // [MONDAY, Tue, Wed, Thu, Fri]
```

**Make an independent copy:**

```java
List<String> copy = new ArrayList<>(week.subList(0, 3)); // independent
```

> **Interview Tip:** Calling `add()` or `remove()` on the **parent** list after creating a subList invalidates the subList. Any subsequent operation on it throws `ConcurrentModificationException`.

---

### 10.2 Bulk Operations

```java
List<String> setA = new ArrayList<>(List.of("A", "B", "C", "D"));
List<String> setB = new ArrayList<>(List.of("C", "D", "E", "F"));

// Union (combine both)
List<String> union = new ArrayList<>(setA);
union.addAll(setB);
System.out.println(union); // [A, B, C, D, C, D, E, F]

// Intersection (common elements only)
List<String> intersection = new ArrayList<>(setA);
intersection.retainAll(setB);
System.out.println(intersection); // [C, D]

// Difference (in A but not in B)
List<String> difference = new ArrayList<>(setA);
difference.removeAll(setB);
System.out.println(difference); // [A, B]
```

**Useful Collections utility methods:**

```java
List<Integer> nums = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

Collections.sort(nums);                             // ascending sort
Collections.sort(nums, Collections.reverseOrder()); // descending sort
Collections.shuffle(nums);                          // random shuffle
Collections.reverse(nums);                         // reverse in place
Collections.max(nums);                              // maximum element
Collections.min(nums);                              // minimum element
Collections.frequency(nums, 1);                    // count occurrences of 1
Collections.swap(nums, 0, nums.size() - 1);        // swap first and last
Collections.fill(nums, 0);                          // set all to 0
```

---

### 10.3 Converting ArrayList

#### ArrayList to Array

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

// PREFERRED — typed array (Java 11+)
String[] arr = names.toArray(String[]::new);

// Alternative — typed array (pre-Java 11)
String[] arr = names.toArray(new String[0]);

// AVOID — returns Object[], requires unsafe cast
Object[] objArr = names.toArray();
```

#### Array to ArrayList

```java
String[] array = {"Alice", "Bob", "Charlie"};

// FIXED-SIZE list — add/remove not allowed!
List<String> fixedList = Arrays.asList(array);
// fixedList.add("Dave"); // UnsupportedOperationException

// MUTABLE ArrayList — fully flexible
List<String> mutableList = new ArrayList<>(Arrays.asList(array));
mutableList.add("Dave"); // works fine
```

#### Copying an ArrayList

```java
List<String> original = new ArrayList<>(List.of("A", "B", "C"));

List<String> copy = new ArrayList<>(original);   // shallow copy — recommended
List<String> immutable = List.copyOf(original);  // immutable copy (Java 10+)

// NOTE: All of the above are SHALLOW copies.
// If the elements are mutable objects, both lists point to the same objects.
```

---

## 11. Null Handling

ArrayList allows `null` values — unlike `List.of()`, `HashSet`, or `TreeSet`.

```java
List<String> list = new ArrayList<>();
list.add("Alice");
list.add(null);
list.add("Bob");
list.add(null);

System.out.println(list);                  // [Alice, null, Bob, null]
System.out.println(list.contains(null));   // true
System.out.println(list.indexOf(null));    // 1
list.remove(null);                          // removes FIRST null
System.out.println(list);                  // [Alice, Bob, null]
```

**Null pitfalls:**

```java
List<String> data = new ArrayList<>(Arrays.asList("Alice", null, "Bob"));

// NullPointerException — calling methods on null elements
for (String s : data) {
    System.out.println(s.toUpperCase()); // NPE when s is null
}

// Null-safe version
for (String s : data) {
    if (s != null) {
        System.out.println(s.toUpperCase());
    }
}

// Sorting with nulls throws NPE
Collections.sort(data); // NPE

// Null-safe sort
data.sort(Comparator.nullsFirst(Comparator.naturalOrder()));  // [null, Alice, Bob]
data.sort(Comparator.nullsLast(Comparator.naturalOrder()));   // [Alice, Bob, null]
```

---

## 12. Fail-Fast Iterator and ConcurrentModificationException

### How it Works

ArrayList maintains an internal `modCount` counter. Every structural change (add, remove, clear) increments it.

When an iterator is created, it saves the current `modCount` as `expectedModCount`. On every `next()` call:

```
if (modCount != expectedModCount)
    throw new ConcurrentModificationException()
```

### Wrong Approaches — Trigger CME

```java
List<String> tasks = new ArrayList<>(List.of("Task A", "Task B", "Task C"));

// Wrong 1: modifying list inside enhanced for-each
for (String task : tasks) {
    if (task.equals("Task B")) {
        tasks.remove(task);  // ConcurrentModificationException
    }
}

// Wrong 2: calling list.remove() directly inside Iterator loop
Iterator<String> it = tasks.iterator();
while (it.hasNext()) {
    if (it.next().equals("Task B")) {
        tasks.remove("Task B"); // ConcurrentModificationException
    }
}
```

### Correct Approaches

```java
// Option 1: Use the iterator's own remove()
Iterator<String> it = tasks.iterator();
while (it.hasNext()) {
    if (it.next().equals("Task B")) {
        it.remove(); // updates expectedModCount internally — safe
    }
}

// Option 2: removeIf() — cleanest, preferred
tasks.removeIf(task -> task.equals("Task B"));

// Option 3: collect candidates, then removeAll
List<String> toRemove = new ArrayList<>();
for (String task : tasks) {
    if (task.equals("Task B")) toRemove.add(task);
}
tasks.removeAll(toRemove);

// Option 4: iterate backwards by index
for (int i = tasks.size() - 1; i >= 0; i--) {
    if (tasks.get(i).equals("Task B")) {
        tasks.remove(i); // safe going from end to start
    }
}
```

> **Interview Tip:** CME is detected on a *best-effort* basis. It is a debugging tool, not a concurrency safety mechanism.

---

## 13. Thread Safety

**ArrayList is NOT thread-safe.** Concurrent modifications without synchronization can corrupt data.

### Option 1 — `Collections.synchronizedList()`

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

syncList.add("Alice"); // thread-safe individual operation

// IMPORTANT: Iteration MUST be manually synchronized!
synchronized (syncList) {
    for (String name : syncList) {
        System.out.println(name);
    }
}
```

---

### Option 2 — `CopyOnWriteArrayList`

On every write, creates a **complete copy** of the internal array. Reads are fast and lock-free; writes are expensive.

```java
import java.util.concurrent.CopyOnWriteArrayList;

List<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("Alice");
cowList.add("Bob");

// Iterators operate on a snapshot — no ConcurrentModificationException
for (String name : cowList) {
    cowList.add("Charlie"); // safe — modifies the next version
    System.out.println(name); // reads the snapshot
}
// Prints: Alice  Bob
```

---

### When to Use Which?

```
Single-threaded code                       --> ArrayList
Multi-threaded, write-heavy                --> Collections.synchronizedList()
Multi-threaded, read-heavy (rare writes)   --> CopyOnWriteArrayList
Event listeners / observer patterns        --> CopyOnWriteArrayList
```

---

## 14. Memory Management

### Pre-allocate with `ensureCapacity()`

```java
List<String> logs = new ArrayList<>();
// Pre-allocate for 10,000 elements — avoids multiple resize operations
((ArrayList<String>) logs).ensureCapacity(10000);

for (int i = 0; i < 10000; i++) {
    logs.add("Log entry " + i); // all O(1), no resizes
}
```

---

### Release Excess Memory with `trimToSize()`

```java
ArrayList<String> data = new ArrayList<>(1000); // pre-allocated for 1000
data.add("Alpha");
data.add("Beta");
data.add("Gamma");
// State: size=3, capacity=1000 — wasting 997 slots!

data.trimToSize();
// State: size=3, capacity=3 — excess memory reclaimed
```

> Use `trimToSize()` after a list is fully built and will only be read, to reduce memory footprint.

---

## 15. Stream API with ArrayList

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

// Filter
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(evens); // [2, 4, 6, 8, 10]

// Map — transform each element
List<String> labels = numbers.stream()
    .map(n -> "Item-" + n)
    .collect(Collectors.toList());

// Reduce — aggregate to a single result
int sum = numbers.stream().reduce(0, Integer::sum);
System.out.println(sum); // 55

// Sort and deduplicate
List<Integer> mixed = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6, 5, 3));
List<Integer> sortedUnique = mixed.stream()
    .distinct()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sortedUnique); // [1, 2, 3, 4, 5, 6, 9]

// Group by first character
List<String> words = new ArrayList<>(List.of("apple", "avocado", "banana", "cherry"));
Map<Character, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(w -> w.charAt(0)));
// {a=[apple, avocado], b=[banana], c=[cherry]}
```

### Collecting Back to ArrayList

```java
// Guaranteed ArrayList
ArrayList<Integer> result = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toCollection(ArrayList::new));

// Java 16+ — immutable result, cleanest
List<Integer> immutableResult = numbers.stream().filter(n -> n > 5).toList();
```

---

## 16. Java 9+ Factory Methods

### `List.of()` — Truly Immutable  (Java 9+)

```java
List<String> colours = List.of("Red", "Green", "Blue");

// colours.add("Yellow"); // UnsupportedOperationException
// colours.set(0, "Pink"); // UnsupportedOperationException
// colours.add(null);      // NullPointerException — no nulls!
```

### `List.copyOf()` — Immutable Independent Copy  (Java 10+)

```java
List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
List<String> snapshot = List.copyOf(mutable);

mutable.add("D");
System.out.println(snapshot); // [A, B, C] — unaffected
```

### Comparison Table

| Method | Mutable? | Null Allowed? | Since |
|---|---|---|---|
| `new ArrayList<>()` | YES | YES | Java 1.2 |
| `Arrays.asList(...)` | Partial (set only) | YES | Java 1.2 |
| `Collections.unmodifiableList()` | NO (view) | YES | Java 1.2 |
| `List.of(...)` | NO | NO | Java 9 |
| `List.copyOf(...)` | NO | NO | Java 10 |

---

## 17. Performance Analysis — Big-O

| Operation | Complexity | Notes |
|---|---|---|
| `get(index)` | O(1) | Direct array access |
| `set(index, e)` | O(1) | Direct array write |
| `add(e)` — append end | O(1) amortized | O(n) on resize |
| `add(index, e)` — middle | O(n) | Must shift elements right |
| `remove(index)` — last element | O(1) | No shift needed |
| `remove(index)` — middle | O(n) | Must shift elements left |
| `remove(Object)` | O(n) | Linear scan + shift |
| `contains(o)` | O(n) | Linear scan |
| `indexOf(o)` | O(n) | Linear scan |
| `size()` / `isEmpty()` | O(1) | Stored as a field |
| `clear()` | O(n) | Null out references for GC |
| `sort()` | O(n log n) | TimSort |
| Full iteration | O(n) | Visit all n elements |
| `binarySearch()` | O(log n) | List must be sorted |

### Performance Tips

```java
// FAST: append to end
list.add("item");              // O(1)

// SLOW: insert at beginning
list.add(0, "item");           // O(n) — use ArrayDeque for frequent head ops

// FAST: read by index
String x = list.get(500);     // O(1)

// SLOW: contains() on large list — linear scan every call
if (list.contains("target")) { } // O(n) — use HashSet for repeated lookups

// FAST: full iteration (cache-friendly)
for (String s : list) { }     // CPU prefetch works great on contiguous arrays
```

### Why ArrayList Beats LinkedList in Iteration Speed

```
ArrayList — contiguous memory:
+------+------+------+------+------+------+------+
|  A   |  B   |  C   |  D   |  E   |  F   |  G   |
+------+------+------+------+------+------+------+
  CPU loads an entire cache line at once -> very fast

LinkedList — scattered heap objects:
+------+     +------+     +------+     +------+
|  A   | --> |  B   | --> |  C   | --> |  D   |
+------+     +------+     +------+     +------+
  Each node is a separate heap object -> cache miss per node -> slow
```

---

## 18. ArrayList vs Array

| Feature | `ArrayList<E>` | `E[]` Array |
|---|---|---|
| Size | Dynamic | Fixed at creation |
| Element types | Objects only (primitives are boxed) | Objects + primitives |
| API richness | Rich (`sort`, `contains`, `stream`) | Limited (`Arrays.sort`, etc.) |
| Multi-dimensional | `List<List<E>>` | Native `int[][]` |
| Memory | Higher (object overhead) | Lean, especially for primitives |
| Use when | Unknown/variable size, objects | Known fixed size, primitives |

```java
// Arrays are better for: fixed-size primitive data
int[] scores = new int[1000]; // no boxing, minimal memory, very fast

// ArrayList is better for: dynamic size, object-rich data
List<String> names = new ArrayList<>(); // grows dynamically, rich API
```

---

## 19. ArrayList vs LinkedList

| Feature | `ArrayList` | `LinkedList` |
|---|---|---|
| Internal structure | Dynamic array (`Object[]`) | Doubly-linked `Node` objects |
| Random access `get(i)` | O(1) | O(n) |
| Add to end | O(1) amortized | O(1) |
| Add to beginning | O(n) | O(1) |
| Add/remove in middle | O(n) | O(n) (must traverse) |
| Memory per element | ~4 bytes (reference) | ~32 bytes (Node: data + prev + next) |
| Iteration speed | Faster (cache-friendly) | Slower (pointer chasing, cache misses) |
| Implements `Deque`? | NO | YES |
| Use when | Default choice, random access | Frequent add/remove at head or tail |

```
Analogy:

ArrayList  = Row of numbered lockers.
             "Give me locker 47" -> walk straight to it. O(1).

LinkedList = Treasure hunt.
             "Clue 1 leads to Clue 2, Clue 2 leads to Clue 3..."
             To reach Clue 47, you follow all 46 before it. O(n).
```

> **Interview Answer:** "In practice, ArrayList outperforms LinkedList in almost all scenarios due to CPU cache locality. LinkedList only wins at O(1) head/tail operations — and `ArrayDeque` handles that better anyway without the per-node memory overhead."

---

## 20. ArrayList vs Vector

| Feature | `ArrayList` | `Vector` |
|---|---|---|
| Thread safety | Not synchronized | Every method is `synchronized` |
| Performance | Faster | Slower (lock overhead) |
| Growth rate | ~1.5x (`oldCap + oldCap >> 1`) | 2x (doubles) |
| Since | Java 1.2 | Java 1.0 (legacy) |
| Use in new code? | YES | NO |

> `Vector` is a legacy class from Java 1.0 superseded by `ArrayList` in Java 1.2. **Never use `Vector` in new code.** For thread safety, use `Collections.synchronizedList()` or `CopyOnWriteArrayList`.

---

## 21. Best Practices

```java
// 1. Program to the interface, not the implementation
List<String> employees = new ArrayList<>();

// 2. Pre-size when you know the approximate count
List<Order> orders = new ArrayList<>(expectedCount);

// 3. Use removeIf() for conditional removal
employees.removeIf(name -> name.isEmpty());

// 4. Use isEmpty() not size() == 0
if (employees.isEmpty()) { ... }

// 5. Return unmodifiable view from getters to protect internal state
public List<String> getEmployees() {
    return Collections.unmodifiableList(employees);
}

// 6. Use Comparator chaining for clean multi-field sorting
employees.sort(Comparator.comparing(Employee::getDepartment)
                          .thenComparing(Employee::getName));

// 7. Use toArray(T[]::new) for typed array conversion (Java 11+)
String[] arr = employees.toArray(String[]::new);

// 8. Use List.copyOf() for defensive immutable snapshots
List<String> snapshot = List.copyOf(employees);
```

```java
// DON'T: use raw types
List raw = new ArrayList(); // no type safety

// DON'T: modify list inside for-each
for (String s : list) { list.remove(s); } // ConcurrentModificationException

// DON'T: confuse remove(int) and remove(Integer)
nums.remove(0);                   // removes by INDEX
nums.remove(Integer.valueOf(0));  // removes by VALUE — be explicit!

// DON'T: assume Arrays.asList() is fully mutable
List<String> fixed = Arrays.asList("A", "B");
fixed.add("C"); // UnsupportedOperationException

// DON'T: use Vector in new code
Vector<String> v = new Vector<>(); // outdated

// DON'T: iterate synchronizedList without a synchronized block
synchronized (sync) {
    for (String s : sync) { ... } // must do this
}
```

---

## 22. Common Pitfalls

### Pitfall 1 — IndexOutOfBoundsException

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C")); // valid: 0, 1, 2

list.get(5);       // IndexOutOfBoundsException
list.add(10, "Z"); // IndexOutOfBoundsException

// Fix: validate the index
if (index >= 0 && index < list.size()) {
    String value = list.get(index);
}
```

---

### Pitfall 2 — Sorting a List That Contains Null

```java
List<String> data = new ArrayList<>(Arrays.asList("Bob", null, "Alice"));

Collections.sort(data); // NullPointerException

// Fix: null-safe comparators
data.sort(Comparator.nullsFirst(Comparator.naturalOrder()));  // [null, Alice, Bob]
data.sort(Comparator.nullsLast(Comparator.naturalOrder()));   // [Alice, Bob, null]
```

---

### Pitfall 3 — Shallow Copy Trap with Mutable Objects

```java
List<StringBuilder> original = new ArrayList<>();
original.add(new StringBuilder("Hello"));

List<StringBuilder> copy = new ArrayList<>(original); // shallow copy!
copy.get(0).append(" World"); // modifies the SAME StringBuilder

System.out.println(original.get(0)); // "Hello World" — original changed!

// Fix: deep copy — create new instances
List<StringBuilder> deepCopy = original.stream()
    .map(sb -> new StringBuilder(sb.toString()))
    .collect(Collectors.toList());
```

---

### Pitfall 4 — Using ArrayList for Uniqueness Checking

```java
// O(n^2) — contains() is O(n), called n times
List<String> unique = new ArrayList<>();
for (String item : hugeDataset) {
    if (!unique.contains(item)) { // O(n) per call!
        unique.add(item);
    }
}

// O(n) — use HashSet as a seen-tracker
Set<String> seen = new HashSet<>();
List<String> unique = new ArrayList<>();
for (String item : hugeDataset) {
    if (seen.add(item)) { // O(1) per call
        unique.add(item);
    }
}
```

---

### Pitfall 5 — The Arrays.asList() Trap

```java
String[] array = {"A", "B", "C"};
List<String> list = Arrays.asList(array);

list.set(0, "Z"); // works — replacement is allowed
list.add("D");    // UnsupportedOperationException — fixed size!

// Fix: wrap in ArrayList
List<String> mutable = new ArrayList<>(Arrays.asList(array));
mutable.add("D"); // works fine
```

---

### Pitfall 6 — ClassCastException with Raw Types

```java
List raw = new ArrayList();
raw.add("hello");
raw.add(42);

for (Object o : raw) {
    String s = (String) o; // ClassCastException on the Integer
}

// Fix: always use generics
List<String> typed = new ArrayList<>();
```

---

## 23. Interview Questions and Answers

---

**Q1. What is the difference between `size()` and capacity in ArrayList?**

> `size()` is a public method returning how many elements are currently stored. Capacity is the length of the internal `Object[]` array — how much space exists before the next resize. Capacity has no public getter and is managed automatically.

---

**Q2. What is the default initial capacity and how does ArrayList grow?**

> Default capacity is 10 (allocated lazily — the internal array starts empty until the first `add()`). On resize, the new capacity is `oldCapacity + (oldCapacity >> 1)` — approximately **1.5x growth**.

---

**Q3. Why is `add()` to the end O(1) amortized and not simply O(1)?**

> Usually it is O(1) — just store at `elementData[size]`. But occasionally a resize copies all existing elements — O(n). Total copy work across n insertions is proportional to n (geometric series), so the *average* cost per add is O(1). Hence "amortized O(1)".

---

**Q4. What happens when you modify a list inside an enhanced for-each loop?**

> `ConcurrentModificationException` is thrown. ArrayList's iterator checks `modCount` against `expectedModCount` on every `next()` call. A structural modification increments `modCount`, causing the check to fail. Fix: use `iterator.remove()`, `removeIf()`, or collect-then-removeAll.

---

**Q5. Is ArrayList thread-safe? How do you make it thread-safe?**

> Not thread-safe. Options: (1) `Collections.synchronizedList(new ArrayList<>())` — synchronizes all methods, but iteration must also be manually synchronized; (2) `CopyOnWriteArrayList` — fail-safe iterator, best for read-heavy scenarios.

---

**Q6. What is the difference between `remove(int index)` and `remove(Object o)` for `List<Integer>`?**

> `remove(int)` removes by *position*. `remove(Object)` removes by *value*. For `List<Integer>`, `list.remove(1)` removes the element **at index 1**, while `list.remove(Integer.valueOf(1))` removes the **first occurrence of value 1**.

---

**Q7. What is the difference between ArrayList and LinkedList?**

> ArrayList: dynamic array — O(1) random access, O(n) insert/delete in middle. LinkedList: doubly-linked nodes — O(n) random access, O(1) head/tail insert/delete. In practice, ArrayList wins in almost all scenarios due to CPU cache locality.

---

**Q8. What is `Arrays.asList()` and how is it different from `new ArrayList<>()`?**

> `Arrays.asList()` returns a fixed-size `List` backed by the original array. `set()` works but `add()` and `remove()` throw `UnsupportedOperationException`. `new ArrayList<>()` is a fully independent, fully mutable list.

---

**Q9. How does `subList()` work? Is it a copy of the list?**

> `subList(from, to)` returns a **view** of the original list, not a copy. Changes to the subList reflect in the parent and vice versa. Structurally modifying the parent after creating the subList throws `ConcurrentModificationException` on the subList.

---

**Q10. What is `trimToSize()` and when would you use it?**

> Reduces the ArrayList's capacity to exactly match its current size, releasing excess array memory. Use it when a list is fully built and will only be read, to reduce memory footprint.

---

**Q11. What sorting algorithm does ArrayList use?**

> TimSort — a hybrid of merge sort and insertion sort. O(n log n) worst case, O(n) best case (nearly-sorted data). It is stable — equal elements preserve their original relative order.

---

**Q12. Can ArrayList store primitive types?**

> Not directly. Primitives are autoboxed to their wrapper types (`int` -> `Integer`). For large primitive datasets where performance matters, use arrays or libraries like Eclipse Collections.

---

**Q13. What is the difference between `List.of()` and `Collections.unmodifiableList()`?**

> `List.of()` creates a **truly immutable** list that nobody can change. It also rejects null values.
> `Collections.unmodifiableList()` creates an **unmodifiable view** — only prevents writes through the view. The original list can still be changed through its own reference, and the view reflects those changes.

---

**Q14. How do you sort a `List<Employee>` by salary descending, then name ascending?**

```java
employees.sort(
    Comparator.comparingInt(Employee::getSalary)
              .reversed()
              .thenComparing(Employee::getName)
);
```

---

**Q15. What is `modCount` in ArrayList?**

> An internal counter in `AbstractList` incremented on every structural modification (add, remove, clear). Iterators save the current `modCount` as `expectedModCount` at creation. On every `next()` call, they compare the two values. A mismatch throws `ConcurrentModificationException`.

---

## 24. Quick Reference Cheat Sheet

```java
import java.util.*;
import java.util.stream.*;

// -- CREATE -------------------------------------------------------------------
List<String> list = new ArrayList<>();                           // empty
List<String> list = new ArrayList<>(50);                        // with capacity
List<String> list = new ArrayList<>(List.of("A", "B", "C"));   // from collection
List<String> list = new ArrayList<>(Arrays.asList("A", "B"));  // from array

// -- ADD ----------------------------------------------------------------------
list.add("X");                      // append end            O(1) amortized
list.add(0, "X");                   // insert at index       O(n)
list.addAll(otherList);             // append collection     O(n)
list.addAll(1, otherList);          // insert collection     O(n)

// -- READ ---------------------------------------------------------------------
String x  = list.get(2);            // by index              O(1)
int sz    = list.size();            // element count         O(1)
boolean e = list.isEmpty();         // is empty?             O(1)

// -- UPDATE -------------------------------------------------------------------
String old = list.set(1, "Y");      // replace, return old   O(1)

// -- REMOVE -------------------------------------------------------------------
list.remove(1);                     // by index              O(n)
list.remove("X");                   // first by value        O(n)
list.remove(Integer.valueOf(5));    // Integer by value      O(n)
list.removeIf(s -> s.isEmpty());    // conditional           O(n)
list.removeAll(otherList);          // remove all matching   O(n^2)
list.retainAll(otherList);          // keep only matching    O(n^2)
list.clear();                       // remove all            O(n)

// -- SEARCH -------------------------------------------------------------------
boolean found = list.contains("X"); // linear search         O(n)
int idx       = list.indexOf("X");  // first, -1 if miss     O(n)
int lidx      = list.lastIndexOf("X"); // last               O(n)

// -- SORT ---------------------------------------------------------------------
Collections.sort(list);                                          // natural order
list.sort(Comparator.reverseOrder());                           // reverse
list.sort(Comparator.comparing(Obj::getField));                 // by field
list.sort(Comparator.comparing(Obj::getF1).thenComparing(Obj::getF2));

// -- ITERATE ------------------------------------------------------------------
for (int i = 0; i < list.size(); i++) { list.get(i); }  // index loop
for (String s : list) { }                                 // for-each
list.forEach(System.out::println);                        // lambda
Iterator<String> it = list.iterator();
while (it.hasNext()) { it.next(); it.remove(); }          // safe remove

// -- CONVERT ------------------------------------------------------------------
String[] arr  = list.toArray(String[]::new);              // to typed array
List<String> fixed   = Arrays.asList(arr);                // fixed-size from array
List<String> mutable = new ArrayList<>(Arrays.asList(arr)); // mutable from array
List<String> immutable = List.copyOf(list);               // immutable copy (Java 10+)

// -- SUBLIST ------------------------------------------------------------------
List<String> sub = list.subList(1, 4);  // view [from, to)
list.subList(1, 3).clear();             // bulk delete by range

// -- MEMORY -------------------------------------------------------------------
((ArrayList<String>) list).ensureCapacity(1000); // pre-allocate
((ArrayList<String>) list).trimToSize();         // release excess

// -- THREAD SAFETY ------------------------------------------------------------
List<String> sync = Collections.synchronizedList(new ArrayList<>());
List<String> cow  = new CopyOnWriteArrayList<>();

// -- STREAMS ------------------------------------------------------------------
list.stream().filter(...).map(...).collect(Collectors.toList()); // Java 8+
list.stream().filter(...).toList();                               // Java 16+ immutable
list.parallelStream().filter(...).count();                        // parallel (large data only)

// -- UTILITIES ----------------------------------------------------------------
Collections.shuffle(list);
Collections.reverse(list);
int max  = Collections.max(list);
int min  = Collections.min(list);
int freq = Collections.frequency(list, "X");
Collections.swap(list, i, j);
Collections.fill(list, "default");
int idx = Collections.binarySearch(sortedList, "X"); // list MUST be sorted!
```

---

*Part of [Java Collection Framework Notes](README.md) — open source and community-friendly. Feel free to fork, extend, and contribute.*
