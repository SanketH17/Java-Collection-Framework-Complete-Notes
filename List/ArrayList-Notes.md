# Java ArrayList — Complete Beginner-Friendly Notes

> Part of the [Java Collection Framework Notes](README.md)
> **For:** Java learners preparing for interviews (beginner to intermediate).
> **Last Updated:** June 2026 | Java 21 LTS

---

## Table of Contents

1. [Introduction to ArrayList](#1-introduction-to-arraylist)
2. [Why Do We Need ArrayList?](#2-why-do-we-need-arraylist)
3. [Array vs ArrayList](#3-array-vs-arraylist)
4. [How ArrayList Works Internally](#4-how-arraylist-works-internally)
5. [Creating an ArrayList](#5-creating-an-arraylist)
6. [ArrayList with Generics](#6-arraylist-with-generics)
7. [Adding Elements](#7-adding-elements)
8. [Accessing Elements](#8-accessing-elements)
9. [Updating Elements](#9-updating-elements)
10. [Removing Elements](#10-removing-elements)
11. [Searching in ArrayList](#11-searching-in-arraylist)
12. [Iterating Over ArrayList](#12-iterating-over-arraylist)
13. [Sorting](#13-sorting)
14. [Commonly Used Methods](#14-commonly-used-methods)
15. [ArrayList of Custom Objects](#15-arraylist-of-custom-objects)
16. [Null Handling](#16-null-handling)
17. [Thread Safety](#17-thread-safety)
18. [Stream API with ArrayList](#18-stream-api-with-arraylist)
19. [Converting ArrayList](#19-converting-arraylist)
20. [Performance — Time Complexity](#20-performance--time-complexity)
21. [ArrayList vs LinkedList](#21-arraylist-vs-linkedlist)
22. [ArrayList vs Vector](#22-arraylist-vs-vector)
23. [Best Practices](#23-best-practices)
24. [Common Mistakes Beginners Make](#24-common-mistakes-beginners-make)
25. [Practical Examples](#25-practical-examples)
26. [Interview Questions and Answers](#26-interview-questions-and-answers)
27. [Quick Reference Cheat Sheet](#27-quick-reference-cheat-sheet)
28. [Summary](#28-summary)

---

## 1. Introduction to ArrayList

### What is ArrayList?

`ArrayList` is a **resizable list** that can grow or shrink automatically. It is part of the Java Collection Framework and lives in the `java.util` package.

Think of it as an **upgraded array** — it does everything an array does, plus much more.

```java
import java.util.ArrayList;

ArrayList<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Charlie");

System.out.println(names);
System.out.println(names.get(1));
System.out.println(names.size());
```

Output:

```
[Alice, Bob, Charlie]
Bob
3
```

### Simple Analogy

> **Array** = A fixed-size box with exactly 5 slots. Once built, you can never add a 6th slot.
>
> **ArrayList** = An expandable bag. You can keep putting items in, and the bag stretches to fit them. Take items out, and it adjusts.

### Key Properties at a Glance

| Property | ArrayList |
|---|---|
| Size | Dynamic — grows and shrinks automatically |
| Maintains insertion order | Yes |
| Allows duplicate elements | Yes |
| Allows null values | Yes |
| Fast access by index | Yes — O(1) |
| Thread-safe | No |
| Stores primitives directly | No — uses wrapper classes |

---

## 2. Why Do We Need ArrayList?

### The Problem with Regular Arrays

Regular arrays in Java have a **fixed size**. You must decide the size when you create them, and you can never change it.

```java
String[] names = new String[3]; // can only hold 3 items — forever

names[0] = "Alice";
names[1] = "Bob";
names[2] = "Charlie";

// names[3] = "Dave"; // ERROR! ArrayIndexOutOfBoundsException
```

What if you don't know in advance how many items you will store? What if you need to add or remove items freely?

### The Solution — ArrayList

ArrayList handles size management for you. Just add or remove items — it takes care of the rest.

```java
ArrayList<String> names = new ArrayList<>();

names.add("Alice");
names.add("Bob");
names.add("Charlie");
names.add("Dave");    // no problem — ArrayList grows automatically!

names.remove("Bob");  // ArrayList shrinks — no empty gaps

System.out.println(names);
```

Output:

```
[Alice, Charlie, Dave]
```

**Bottom line:** Use arrays when the size is known and fixed. Use ArrayList when the size can change.

---

## 3. Array vs ArrayList

| Feature | Array | ArrayList |
|---|---|---|
| Size | Fixed — set once, never changes | Dynamic — grows and shrinks |
| Stores primitives | Yes (`int`, `double`, etc.) | No — must use wrapper classes (`Integer`, `Double`) |
| Built-in methods | Almost none | Many — `add()`, `remove()`, `contains()`, `sort()`, etc. |
| Syntax | `String[] names = new String[5];` | `ArrayList<String> names = new ArrayList<>();` |
| Performance | Slightly faster (no overhead) | Slightly slower (but negligible for most uses) |
| Part of Collections Framework | No | Yes |

### Side-by-Side Code Comparison

**Array:**

```java
String[] fruits = new String[3];
fruits[0] = "Apple";
fruits[1] = "Banana";
fruits[2] = "Mango";

System.out.println(fruits[1]);
```

Output:

```
Banana
```

**ArrayList:**

```java
ArrayList<String> fruits = new ArrayList<>();
fruits.add("Apple");
fruits.add("Banana");
fruits.add("Mango");

System.out.println(fruits.get(1));
```

Output:

```
Banana
```

Both store data in order, both use zero-based indexing. But ArrayList gives you the flexibility to add and remove items freely.

---

## 4. How ArrayList Works Internally

### The Secret: It Uses an Array Inside

Internally, ArrayList stores your data in a regular `Object[]` array. The magic is that ArrayList manages this array for you — creating bigger arrays and copying data over when needed.

```
What you see:      ArrayList with 4 items
What's inside:     An Object[] array that might have 10 slots

+------+------+------+------+------+------+------+------+------+------+
| "Ali"| "Bob"| "Car"| "Dan"| null | null | null | null | null | null |
+------+------+------+------+------+------+------+------+------+------+
  [0]    [1]    [2]    [3]    [4]    [5]    [6]    [7]    [8]    [9]

  <-- your 4 items (size=4) -->  <-- empty reserved slots (capacity=10) -->
```

### Size vs Capacity

These two terms confuse many beginners. Here is the simplest way to understand them:

> **Capacity** = Total number of seats in a cinema hall (e.g., 200 seats).
> **Size** = Number of people actually sitting (e.g., 75 people).

- **Size** — How many elements you have actually stored. You can check this with `list.size()`.
- **Capacity** — How much room the internal array has. This is managed internally — there is **no public method** to check it.

```java
ArrayList<String> list = new ArrayList<>(); // capacity = 0, size = 0
list.add("Alice");                          // capacity = 10, size = 1
list.add("Bob");                            // capacity = 10, size = 2

System.out.println(list.size());
```

Output:

```
2
```

(This only shows actual elements, not internal capacity.)

### Dynamic Resizing — How the ArrayList Grows

When all slots in the internal array are full and you add another element, ArrayList:

1. Creates a **new, bigger array** (about 1.5x the old size).
2. **Copies** all existing elements to the new array.
3. **Discards** the old array.
4. Stores your new element.

```
Step 1 — Array is full (capacity = 4, size = 4):
+------+------+------+------+
|  "A" |  "B" |  "C" |  "D" |
+------+------+------+------+

Step 2 — You call add("E"). ArrayList creates a bigger array (capacity = 6):
+------+------+------+------+------+------+
|  "A" |  "B" |  "C" |  "D" |  "E" |      |
+------+------+------+------+------+------+
(Old array is thrown away by garbage collector)
```

**Growth formula:** `newCapacity = oldCapacity + (oldCapacity / 2)`

```
After first add:   capacity = 10  (default)
After 1st resize:  10 + 5  = 15
After 2nd resize:  15 + 7  = 22
After 3rd resize:  22 + 11 = 33
```

### Why is add() Still Fast?

Even though resizing copies all elements (which takes time), it happens very rarely. Most of the time, `add()` just puts the element in the next empty slot — instant.

On average, `add()` at the end takes **O(1) amortized** time.

> **Simple analogy:** Imagine you move to a bigger house every time your current one is full. Moving is expensive, but you only move a few times over many years. On a day-to-day basis, your cost is almost nothing.

---

## 5. Creating an ArrayList

You need to import it first:

```java
import java.util.ArrayList;
```

### Method 1 — Empty ArrayList (most common)

```java
ArrayList<String> names = new ArrayList<>();
// Starts empty. Capacity is 10 after the first add.
```

### Method 2 — With Initial Capacity

Use this when you know roughly how many items you will add. It avoids unnecessary resizing.

```java
ArrayList<String> names = new ArrayList<>(500);
// Pre-allocates space for 500 items. No resize until item 501.
```

### Method 3 — From Another Collection

```java
ArrayList<String> original = new ArrayList<>();
original.add("Alice");
original.add("Bob");

ArrayList<String> copy = new ArrayList<>(original);
// copy is now [Alice, Bob] — an independent list
```

### Best Practice — Declare as the Interface Type

```java
// PREFERRED — flexible
List<String> names = new ArrayList<>();

// AVOID — locked to ArrayList
ArrayList<String> names = new ArrayList<>();
```

**Why?** If you later decide to use `LinkedList` instead, you only change one word in the preferred version. In the avoided version, you would have to change every method that accepts or returns this type.

---

## 6. ArrayList with Generics

### What Are Generics?

Generics let you tell the ArrayList **what type** of items it should hold. You write the type inside angle brackets `<>`.

```java
ArrayList<String> names = new ArrayList<>();    // only Strings allowed
ArrayList<Integer> numbers = new ArrayList<>(); // only Integers allowed
```

Without generics, the list could hold anything, leading to runtime errors:

```java
// BAD — raw type, no type safety
ArrayList list = new ArrayList();
list.add("Hello");
list.add(42);
String s = (String) list.get(1); // CRASH! ClassCastException at runtime
```

With generics, mistakes are caught at **compile time** — before your code even runs:

```java
// GOOD — type-safe
ArrayList<String> list = new ArrayList<>();
list.add("Hello");
// list.add(42);  // COMPILE ERROR — caught immediately
```

### Primitives Need Wrapper Classes

ArrayList cannot store primitive types (`int`, `double`, `char`, etc.) directly. You must use their **wrapper classes**.

| Primitive | Wrapper Class |
|---|---|
| `int` | `Integer` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |
| `float` | `Float` |
| `long` | `Long` |

```java
// WRONG
ArrayList<int> numbers = new ArrayList<>(); // compile error

// CORRECT
ArrayList<Integer> numbers = new ArrayList<>();
numbers.add(10);    // Java auto-converts int 10 → Integer(10) — called autoboxing
int n = numbers.get(0); // Java auto-converts Integer → int — called unboxing
```

---

## 7. Adding Elements

### Add at the End — `add(element)` — O(1)

The most common way to add items. Places the element at the end of the list.

```java
List<String> cart = new ArrayList<>();
cart.add("Laptop");    // index 0
cart.add("Mouse");     // index 1
cart.add("Keyboard");  // index 2

System.out.println(cart);
```

Output:

```
[Laptop, Mouse, Keyboard]
```

### Add at a Specific Position — `add(index, element)` — O(n)

Inserts the element at the given index. All elements after that index shift one place to the right.

```java
List<String> names = new ArrayList<>();
names.add("Alice");   // [Alice]
names.add("Charlie"); // [Alice, Charlie]

names.add(1, "Bob");  // insert Bob at index 1

System.out.println(names);
```

Output:

```
[Alice, Bob, Charlie]
```

**What happens inside:**
```
Before:  [ Alice | Charlie ]
                    ↓ shift right
After:   [ Alice | Bob | Charlie ]
```

> Adding in the middle is slow (O(n)) because elements need to shift. If you do this often, consider other data structures.

### Add Multiple Elements — `addAll(collection)` — O(n)

Adds all elements from another collection to the end.

```java
List<String> team1 = new ArrayList<>(List.of("Alice", "Bob"));
List<String> team2 = List.of("Charlie", "Dave");

team1.addAll(team2);
System.out.println(team1);
```

Output:

```
[Alice, Bob, Charlie, Dave]
```

You can also insert at a specific position:

```java
team1.addAll(1, team2); // inserts team2 elements starting at index 1
```

---

## 8. Accessing Elements

### Get by Index — `get(index)` — O(1)

This is ArrayList's **superpower**. It can jump directly to any element by index — no scanning needed.

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Delhi", "Bangalore"));

System.out.println(cities.get(0));
System.out.println(cities.get(2));
System.out.println(cities.get(cities.size() - 1));
```

Output:

```
Mumbai
Bangalore
Bangalore
```

> **Analogy:** Think of indexes as locker numbers. Saying "open locker 5" takes you straight there — no need to check lockers 1 through 4 first.

### Watch Out for Invalid Indexes

```java
cities.get(10);  // IndexOutOfBoundsException — only 0, 1, 2 are valid
cities.get(-1);  // IndexOutOfBoundsException — no negative indexes
```

Always make sure your index is between `0` and `size() - 1`:

```java
int index = 5;
if (index >= 0 && index < cities.size()) {
    System.out.println(cities.get(index));
}
```

### Check Size and Emptiness

```java
System.out.println(cities.size());
System.out.println(cities.isEmpty());
```

Output:

```
3
false
```

> Prefer `isEmpty()` over `size() == 0` — it is more readable.

---

## 9. Updating Elements

### Replace at Index — `set(index, newElement)` — O(1)

Replaces the element at the given index and **returns the old element**.

```java
List<String> menu = new ArrayList<>(List.of("Burger", "Sandwitch", "Pizza"));

String old = menu.set(1, "Sandwich"); // fix the typo

System.out.println("Replaced: " + old);
System.out.println(menu);
```

Output:

```
Replaced: Sandwitch
[Burger, Sandwich, Pizza]
```

> `set()` **replaces** an existing element. It does NOT add a new one or change the list size.

---

## 10. Removing Elements

### Remove by Index — `remove(index)` — O(n)

Removes the element at the given position. Elements after it shift one place to the left.

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));
String removed = names.remove(1); // removes Bob

System.out.println("Removed: " + removed);
System.out.println(names);
```

Output:

```
Removed: Bob
[Alice, Charlie]
```

**What happens inside:**
```
Before:  [ Alice | Bob | Charlie ]
Remove index 1, shift left:
After:   [ Alice | Charlie ]
```

### Remove by Value — `remove(Object)` — O(n)

Removes the **first occurrence** of the value. Returns `true` if found.

```java
List<String> fruits = new ArrayList<>(List.of("Apple", "Banana", "Apple"));
fruits.remove("Apple"); // removes the FIRST "Apple" only

System.out.println(fruits);
```

Output:

```
[Banana, Apple]
```

### The Integer Trap — remove(int) vs remove(Object)

This is a **classic interview question** and a very common bug.

When your list holds `Integer` values, `remove(1)` removes the item **at index 1**, NOT the value `1`.

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30));

numbers.remove(1); // removes element at INDEX 1 → removes 20
System.out.println(numbers);
```

Output:

```
[10, 30]
```

To remove the **value** 20, wrap it:

```java
numbers.remove(Integer.valueOf(20));
```

### Conditional Removal — `removeIf()` — O(n) *(Java 8+)*

Removes all elements that match a condition. Clean and safe.

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
numbers.removeIf(n -> n % 2 == 0); // remove all even numbers

System.out.println(numbers);
```

Output:

```
[1, 3, 5]
```

### Other Removal Methods

```java
List<String> items = new ArrayList<>(List.of("A", "B", "C", "D", "E"));

// Remove all elements that exist in another collection
items.removeAll(List.of("B", "D"));
System.out.println(items);
```

Output:

```
[A, C, E]
```

```java
// Keep ONLY elements that exist in another collection
items.retainAll(List.of("A", "E"));
System.out.println(items);
```

Output:

```
[A, E]
```

```java
// Remove everything
items.clear();
System.out.println(items);
System.out.println(items.size());
```

Output:

```
[]
0
```

---

## 11. Searching in ArrayList

### Check if Element Exists — `contains()` — O(n)

```java
List<String> fruits = new ArrayList<>(List.of("Apple", "Banana", "Mango"));

System.out.println(fruits.contains("Banana"));
System.out.println(fruits.contains("Grapes"));
```

Output:

```
true
false
```

> `contains()` uses the `equals()` method to compare. For custom objects, always override `equals()`.

### Find the Index — `indexOf()` and `lastIndexOf()` — O(n)

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Alice", "Charlie"));

System.out.println(names.indexOf("Alice"));
System.out.println(names.lastIndexOf("Alice"));
System.out.println(names.indexOf("Dave"));
```

Output:

```
0
2
-1
```

(`indexOf` returns the first occurrence, `lastIndexOf` returns the last. `-1` means not found.)

### Binary Search — O(log n) *(list must be sorted)*

For large sorted lists, binary search is much faster than `contains()`.

```java
List<Integer> sorted = new ArrayList<>(List.of(10, 20, 30, 40, 50));
int index = Collections.binarySearch(sorted, 30);
System.out.println(index);
```

Output:

```
2
```

> **Warning:** The list MUST be sorted before calling `binarySearch()`. Results are undefined on unsorted lists.

---

## 12. Iterating Over ArrayList

There are **6 ways** to loop through an ArrayList. Here they are, from simplest to most powerful.

### Method 1 — Enhanced For-Each Loop *(most common)*

Best for simple reading — when you just need each element.

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

for (String name : names) {
    System.out.println(name);
}
```

Output:

```
Alice
Bob
Charlie
```

### Method 2 — Classic For Loop

Best when you need the **index** or want to loop in reverse.

```java
for (int i = 0; i < names.size(); i++) {
    System.out.println(i + ": " + names.get(i));
}
```

Output:

```
0: Alice
1: Bob
2: Charlie
```

### Method 3 — forEach with Lambda *(Java 8+)*

A shorter, cleaner way to apply an action to each element.

```java
names.forEach(name -> System.out.println(name));

// Even shorter with method reference:
names.forEach(System.out::println);
```

### Method 4 — Iterator

Best when you need to **safely remove** elements while looping.

```java
List<String> tasks = new ArrayList<>(List.of("Done", "Pending", "Done", "In Progress"));

Iterator<String> it = tasks.iterator();
while (it.hasNext()) {
    String task = it.next();
    if (task.equals("Done")) {
        it.remove(); // safe removal — no exception
    }
}

System.out.println(tasks);
```

Output:

```
[Pending, In Progress]
```

### Method 5 — ListIterator *(bidirectional)*

Can go both forward and backward. Can also replace or insert elements during iteration.

```java
List<String> names = new ArrayList<>(List.of("alice", "bob"));
ListIterator<String> lit = names.listIterator();

// Go forward and capitalize
while (lit.hasNext()) {
    String name = lit.next();
    lit.set(name.substring(0, 1).toUpperCase() + name.substring(1));
}
System.out.println(names);
```

Output:

```
[Alice, Bob]
```

```java
// Go backward
while (lit.hasPrevious()) {
    System.out.println(lit.previous());
}
```

Output:

```
Bob
Alice
```

### Method 6 — Stream API *(Java 8+)*

Best for filtering, transforming, and collecting data — functional style.

```java
List<String> names = new ArrayList<>(List.of("Alice", "Adam", "Bob", "Anna"));

names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

Output:

```
ALICE
ADAM
ANNA
```

### Quick Comparison

| Method | Need Index? | Safe Remove? | Backward? | Best For |
|---|---|---|---|---|
| Enhanced for-each | No | No | No | Simple read-only loops |
| Classic for loop | Yes | Yes (by index) | Yes | Index-based work |
| forEach lambda | No | No | No | Clean one-liners |
| Iterator | No | Yes | No | Removing during loop |
| ListIterator | Yes | Yes | Yes | Replace/insert/reverse |
| Stream API | No | No | No | Filter/transform/collect |

---

## 13. Sorting

### Sort in Natural Order (A-Z, small to large)

```java
List<String> cities = new ArrayList<>(List.of("Mumbai", "Ahmedabad", "Delhi"));
Collections.sort(cities);
System.out.println(cities);
```

Output:

```
[Ahmedabad, Delhi, Mumbai]
```

```java
List<Integer> scores = new ArrayList<>(List.of(89, 45, 92, 71));
Collections.sort(scores);
System.out.println(scores);
```

Output:

```
[45, 71, 89, 92]
```

### Sort in Reverse Order

```java
Collections.sort(scores, Collections.reverseOrder());
System.out.println(scores);
```

Output:

```
[92, 89, 71, 45]
```

### Sort by Custom Rule — `list.sort(Comparator)` *(Java 8+)*

```java
List<String> words = new ArrayList<>(List.of("Banana", "Kiwi", "Watermelon", "Fig"));

// Sort by length — shortest first
words.sort(Comparator.comparingInt(String::length));
System.out.println(words);
```

Output:

```
[Fig, Kiwi, Banana, Watermelon]
```

### Sort Custom Objects

```java
class Employee {
    String name;
    int salary;

    Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    public String toString() { return name + "($" + salary + ")"; }
}

List<Employee> team = new ArrayList<>();
team.add(new Employee("Alice", 80000));
team.add(new Employee("Bob", 60000));
team.add(new Employee("Carol", 70000));

// Sort by salary (ascending)
team.sort(Comparator.comparingInt(e -> e.salary));
System.out.println(team);
```

Output:

```
[Bob($60000), Carol($70000), Alice($80000)]
```

```java
// Sort by salary descending, then by name ascending
team.sort(Comparator.comparingInt((Employee e) -> e.salary)
                     .reversed()
                     .thenComparing(e -> e.name));
```

> **Interview Tip:** Java uses **TimSort** internally — a hybrid of merge sort and insertion sort. It is **stable**, meaning equal elements keep their original order. Time complexity: O(n log n).

---

## 14. Commonly Used Methods

### All Methods at a Glance

| Method | What It Does | Returns | Time |
|---|---|---|---|
| `add(element)` | Adds at end | `true` | O(1) |
| `add(index, element)` | Inserts at position | void | O(n) |
| `get(index)` | Gets element at position | Element | O(1) |
| `set(index, element)` | Replaces element | Old element | O(1) |
| `remove(index)` | Removes by position | Removed element | O(n) |
| `remove(object)` | Removes first match by value | `true`/`false` | O(n) |
| `size()` | Number of elements | int | O(1) |
| `isEmpty()` | Is the list empty? | boolean | O(1) |
| `contains(object)` | Does element exist? | boolean | O(n) |
| `indexOf(object)` | First index of element | int (-1 if not found) | O(n) |
| `lastIndexOf(object)` | Last index of element | int (-1 if not found) | O(n) |
| `clear()` | Removes all elements | void | O(n) |
| `addAll(collection)` | Adds all from another collection | `true` | O(n) |
| `removeAll(collection)` | Removes all matching elements | `true`/`false` | O(n²) |
| `retainAll(collection)` | Keeps only matching elements | `true`/`false` | O(n²) |
| `removeIf(predicate)` | Removes elements matching condition | `true`/`false` | O(n) |
| `sort(comparator)` | Sorts the list | void | O(n log n) |
| `subList(from, to)` | Returns a view of a range | List (view) | O(1) |
| `toArray()` | Converts to array | Object[] | O(n) |
| `ensureCapacity(n)` | Pre-allocates internal space | void | O(n) |
| `trimToSize()` | Releases unused internal space | void | O(n) |

### subList — Getting a Portion of the List

`subList(fromIndex, toIndex)` returns a **view** — not a separate copy. Changes affect the original.

```java
List<String> week = new ArrayList<>(List.of("Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"));

List<String> workdays = week.subList(0, 5); // from index 0 to 4 (5 is excluded)
System.out.println(workdays);
```

Output:

```
[Mon, Tue, Wed, Thu, Fri]
```

```java
// Modify the subList — original list changes too!
workdays.set(0, "MONDAY");
System.out.println(week.get(0));
```

Output:

```
MONDAY
```

```java
// Delete a range efficiently
week.subList(5, 7).clear(); // removes Sat and Sun
System.out.println(week);
```

Output:

```
[MONDAY, Tue, Wed, Thu, Fri]
```

To make an **independent copy** of a subList:

```java
List<String> copy = new ArrayList<>(week.subList(0, 3));
```

### Useful Collections Utility Methods

```java
List<Integer> nums = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

Collections.sort(nums);            // sorts in ascending order
Collections.reverse(nums);         // reverses in place
Collections.shuffle(nums);         // random order
Collections.max(nums);             // largest element
Collections.min(nums);             // smallest element
Collections.frequency(nums, 1);    // how many times 1 appears
Collections.swap(nums, 0, nums.size() - 1); // swap first and last
```

---

## 15. ArrayList of Custom Objects

In real projects, you rarely store just strings or numbers. You store **objects** — students, employees, products, etc.

### Basic Example

```java
class Student {
    int id;
    String name;
    int marks;

    Student(int id, String name, int marks) {
        this.id = id;
        this.name = name;
        this.marks = marks;
    }

    public String toString() {
        return id + " " + name + " (" + marks + ")";
    }
}

List<Student> students = new ArrayList<>();
students.add(new Student(1, "Alice", 90));
students.add(new Student(2, "Bob", 75));
students.add(new Student(3, "Charlie", 85));

for (Student s : students) {
    System.out.println(s);
}
```

Output:

```
1 Alice (90)
2 Bob (75)
3 Charlie (85)
```

### Search by Field

```java
int searchId = 2;
for (Student s : students) {
    if (s.id == searchId) {
        System.out.println("Found: " + s.name);
    }
}
```

Output:

```
Found: Bob
```

### Sort by Field

```java
// Sort by marks — ascending
students.sort(Comparator.comparingInt(s -> s.marks));
System.out.println(students);
```

Output:

```
[2 Bob (75), 3 Charlie (85), 1 Alice (90)]
```

```java
// Sort by marks — descending
students.sort(Comparator.comparingInt((Student s) -> s.marks).reversed());
```

### Why Override equals() and hashCode()?

For `contains()`, `remove(Object)`, and `indexOf()` to work correctly with custom objects, you must override `equals()` (and `hashCode()`):

```java
class Student {
    int id;
    String name;

    // constructor, toString...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student s)) return false;
        return id == s.id && Objects.equals(name, s.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

Now this works correctly:

```java
students.contains(new Student(2, "Bob")); // returns true
```

Without overriding `equals()`, Java compares **memory addresses** — two different objects with the same data would be treated as different.

---

## 16. Null Handling

ArrayList **allows null values** (unlike `List.of()` or `HashSet`).

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add(null);
names.add("Bob");

System.out.println(names);
System.out.println(names.contains(null));
System.out.println(names.indexOf(null));

names.remove(null); // removes the first null
System.out.println(names);
```

Output:

```
[Alice, null, Bob]
true
1
[Alice, Bob]
```

### Watch Out for NullPointerException

```java
List<String> data = new ArrayList<>(List.of("Alice", "Bob"));
data.add(null);

for (String s : data) {
    System.out.println(s.toUpperCase()); // CRASH on null! NullPointerException
}
```

Safe version:

```java
for (String s : data) {
    if (s != null) {
        System.out.println(s.toUpperCase());
    }
}
```

Output:

```
ALICE
BOB
```

### Sorting with Nulls

```java
List<String> data = new ArrayList<>(Arrays.asList("Bob", null, "Alice"));

// Collections.sort(data); // CRASH — NullPointerException

// Safe ways:
data.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(data);
```

Output:

```
[null, Alice, Bob]
```

```java
data.sort(Comparator.nullsLast(Comparator.naturalOrder()));
System.out.println(data);
```

Output:

```
[Alice, Bob, null]
```

---

## 17. Thread Safety

**ArrayList is NOT thread-safe.** If multiple threads read and write to the same ArrayList at the same time, data can become corrupted.

### For Single-Threaded Programs

Just use `ArrayList` normally. No issues.

### For Multi-Threaded Programs

**Option 1: `Collections.synchronizedList()`**

Wraps the ArrayList to make individual operations thread-safe.

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
syncList.add("Alice"); // thread-safe

// IMPORTANT: You must still synchronize iteration manually!
synchronized (syncList) {
    for (String name : syncList) {
        System.out.println(name);
    }
}
```

**Option 2: `CopyOnWriteArrayList`**

Creates a copy of the internal array on every write. Reads are fast and lock-free. Best when reads are far more frequent than writes.

```java
List<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("Alice");
cowList.add("Bob");

// Safe to iterate even if another thread modifies it
for (String name : cowList) {
    System.out.println(name);
}
```

### Quick Guide

| Scenario | Use |
|---|---|
| Single-threaded code | `ArrayList` |
| Multi-threaded, many writes | `Collections.synchronizedList()` |
| Multi-threaded, mostly reads | `CopyOnWriteArrayList` |

---

## 18. Stream API with ArrayList

Streams (Java 8+) provide a powerful way to filter, transform, and aggregate data.

### Filter — Keep Matching Elements

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6, 7, 8));

List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evens);
```

Output:

```
[2, 4, 6, 8]
```

### Map — Transform Each Element

```java
List<String> names = new ArrayList<>(List.of("alice", "bob", "charlie"));

List<String> upper = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

System.out.println(upper);
```

Output:

```
[ALICE, BOB, CHARLIE]
```

### Reduce — Combine Into a Single Result

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30, 40));

int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

System.out.println(sum);
```

Output:

```
100
```

### Sort and Remove Duplicates

```java
List<Integer> nums = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6, 5));

List<Integer> result = nums.stream()
    .distinct()
    .sorted()
    .collect(Collectors.toList());

System.out.println(result);
```

Output:

```
[1, 2, 3, 4, 5, 6, 9]
```

---

## 19. Converting ArrayList

### ArrayList → Array

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

// Recommended (Java 11+)
String[] arr = names.toArray(String[]::new);

// Pre-Java 11
String[] arr = names.toArray(new String[0]);
```

### Array → ArrayList

```java
String[] array = {"Alice", "Bob", "Charlie"};

// This list has FIXED size — you cannot add or remove!
List<String> fixed = Arrays.asList(array);

// This list is fully flexible
List<String> flexible = new ArrayList<>(Arrays.asList(array));
flexible.add("Dave"); // works fine
```

### Copying an ArrayList

```java
List<String> original = new ArrayList<>(List.of("A", "B", "C"));

// Independent mutable copy
List<String> copy = new ArrayList<>(original);

// Immutable copy (Java 10+) — cannot be changed
List<String> snapshot = List.copyOf(original);
```

> **Important:** All of these are **shallow copies**. If the elements are mutable objects, both lists point to the same objects in memory.

### Immutable Lists

| Method | Mutable? | Allows Null? | Since |
|---|---|---|---|
| `new ArrayList<>()` | Yes | Yes | Java 1.2 |
| `Arrays.asList(...)` | Partial (set only) | Yes | Java 1.2 |
| `Collections.unmodifiableList()` | No (view) | Yes | Java 1.2 |
| `List.of(...)` | No (truly immutable) | No | Java 9 |
| `List.copyOf(...)` | No (truly immutable) | No | Java 10 |

---

## 20. Performance — Time Complexity

| Operation | Time | Why |
|---|---|---|
| `get(index)` | **O(1)** | Direct array access — instant |
| `set(index, element)` | **O(1)** | Direct array write — instant |
| `add(element)` at end | **O(1)** amortized | O(n) on rare resize |
| `add(index, element)` in middle | **O(n)** | Must shift elements right |
| `remove(index)` last element | **O(1)** | No shifting needed |
| `remove(index)` middle | **O(n)** | Must shift elements left |
| `remove(Object)` | **O(n)** | Linear scan + shift |
| `contains(object)` | **O(n)** | Checks one by one |
| `indexOf(object)` | **O(n)** | Checks one by one |
| `size()` / `isEmpty()` | **O(1)** | Stored as a field |
| `sort()` | **O(n log n)** | TimSort algorithm |

### What This Means in Practice

```java
// FAST operations — use freely
list.add("item");          // add at end
list.get(500);             // access by index
list.set(3, "newValue");   // replace by index

// SLOW operations — avoid in loops over large lists
list.add(0, "item");       // insert at beginning (everything shifts)
list.contains("target");   // scans entire list each time
list.remove(0);            // remove from beginning (everything shifts)
```

> **Tip:** If you need frequent `contains()` checks, put your data in a `HashSet` — it does `contains()` in O(1).

---

## 21. ArrayList vs LinkedList

| Feature | ArrayList | LinkedList |
|---|---|---|
| Internal structure | Dynamic array | Chain of linked nodes |
| Access by index `get(i)` | **O(1)** — fast | O(n) — slow (must walk the chain) |
| Add at end | **O(1)** | O(1) |
| Add at beginning | O(n) — slow (shifts all) | **O(1)** — fast |
| Memory per element | ~4 bytes (reference) | ~32 bytes (node + pointers) |
| Iteration speed | **Faster** (data is contiguous in memory) | Slower (nodes scattered in memory) |

> **Analogy:**
> - **ArrayList** = Row of numbered lockers. Want locker 47? Walk straight to it.
> - **LinkedList** = Treasure hunt. Clue 1 leads to Clue 2, which leads to Clue 3... To reach Clue 47, you follow all 46 clues before it.

**Bottom line:** Use `ArrayList` as your default choice. Use `LinkedList` only when you frequently add/remove at the beginning and don't need random access.

---

## 22. ArrayList vs Vector

| Feature | ArrayList | Vector |
|---|---|---|
| Thread safety | Not synchronized | Every method synchronized |
| Performance | **Faster** | Slower (lock overhead) |
| Growth rate | 1.5x | 2x |
| Since | Java 1.2 | Java 1.0 (legacy) |
| Use in new code? | **Yes** | **No** |

> **Vector is outdated.** It has been replaced by `ArrayList` since Java 1.2. For thread safety, use `Collections.synchronizedList()` or `CopyOnWriteArrayList` instead.

---

## 23. Best Practices

### Do

1. **Declare as the interface type** — `List<String>` instead of `ArrayList<String>`.
2. **Always use generics** — `ArrayList<String>`, never raw `ArrayList`.
3. **Pre-allocate capacity** when you know the approximate size — `new ArrayList<>(1000)`.
4. **Use `isEmpty()`** instead of `size() == 0`.
5. **Use `removeIf()`** for conditional removal — clean and safe.
6. **Use `Iterator.remove()`** when removing during iteration.
7. **Return unmodifiable views** from getters to protect internal data.

```java
// Example: protecting internal state
public List<String> getEmployees() {
    return Collections.unmodifiableList(employees);
}
```

### Don't

1. **Don't use raw types** — `ArrayList list = new ArrayList()` has no type safety.
2. **Don't modify a list inside a for-each loop** — causes `ConcurrentModificationException`.
3. **Don't confuse `remove(int)` and `remove(Integer)`** — index vs value.
4. **Don't assume `Arrays.asList()` is fully mutable** — `add()` and `remove()` will throw exceptions.
5. **Don't use `Vector`** in new code.

---

## 24. Common Mistakes Beginners Make

### Mistake 1 — Removing During Enhanced For-Each Loop

```java
// WRONG — throws ConcurrentModificationException
for (String name : names) {
    if (name.equals("Bob")) {
        names.remove(name); // CRASH!
    }
}

// CORRECT — use Iterator
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    if (it.next().equals("Bob")) {
        it.remove(); // safe
    }
}

// EVEN BETTER — use removeIf (Java 8+)
names.removeIf(name -> name.equals("Bob"));
```

### Mistake 2 — Accessing an Invalid Index

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.get(5); // CRASH — IndexOutOfBoundsException

// Always check bounds first:
if (index >= 0 && index < names.size()) {
    System.out.println(names.get(index));
}
```

### Mistake 3 — Integer Remove Confusion

```java
List<Integer> nums = new ArrayList<>(List.of(10, 20, 30));

nums.remove(1);                   // removes at INDEX 1 → removes 20
nums.remove(Integer.valueOf(20)); // removes VALUE 20
```

### Mistake 4 — Thinking Arrays.asList() Returns a Normal ArrayList

```java
List<String> list = Arrays.asList("A", "B", "C");
list.add("D"); // CRASH — UnsupportedOperationException

// Fix: wrap it in a real ArrayList
List<String> mutable = new ArrayList<>(Arrays.asList("A", "B", "C"));
mutable.add("D"); // works
```

### Mistake 5 — Forgetting That ArrayList Is Not Thread-Safe

```java
// WRONG in multi-threaded code — data can become corrupted
ArrayList<String> shared = new ArrayList<>();

// CORRECT
List<String> shared = Collections.synchronizedList(new ArrayList<>());
```

### Mistake 6 — Using == Instead of equals() for String Comparison

```java
String name = names.get(0);

// WRONG — compares memory addresses
if (name == "Alice") { }

// CORRECT — compares actual content
if (name.equals("Alice")) { }

// SAFEST — handles null gracefully
if ("Alice".equals(name)) { }
```

---

## 25. Practical Examples

### Example 1 — Shopping Cart

```java
List<String> cart = new ArrayList<>();
cart.add("Laptop");
cart.add("Mouse");
cart.add("Keyboard");

System.out.println("Cart: " + cart);
System.out.println("Total items: " + cart.size());

if (cart.contains("Mouse")) {
    System.out.println("Mouse is in the cart.");
}

cart.remove("Mouse");
System.out.println("After removal: " + cart);
```

Output:

```
Cart: [Laptop, Mouse, Keyboard]
Total items: 3
Mouse is in the cart.
After removal: [Laptop, Keyboard]
```

### Example 2 — Finding Max and Min

```java
List<Integer> scores = new ArrayList<>(List.of(45, 92, 10, 88, 67));

System.out.println("Max: " + Collections.max(scores));
System.out.println("Min: " + Collections.min(scores));
```

Output:

```
Max: 92
Min: 10
```

### Example 3 — Removing Duplicates

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 10, 30, 20));

// Using LinkedHashSet — preserves order, removes duplicates
List<Integer> unique = new ArrayList<>(new LinkedHashSet<>(numbers));

System.out.println(unique);
```

Output:

```
[10, 20, 30]
```

### Example 4 — Filtering with Streams

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 15, 20, 25, 30));

List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evens);
```

Output:

```
[10, 20, 30]
```

### Example 5 — Sum of Numbers

```java
List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30, 40));

// Using loop
int sum = 0;
for (int n : numbers) {
    sum += n;
}
System.out.println("Sum: " + sum);
```

Output:

```
Sum: 100
```

```java
// Using streams
int streamSum = numbers.stream().mapToInt(Integer::intValue).sum();
System.out.println("Sum: " + streamSum);
```

Output:

```
Sum: 100
```

### Example 6 — Student Management with Custom Objects

```java
class Student {
    int id;
    String name;
    int marks;

    Student(int id, String name, int marks) {
        this.id = id;
        this.name = name;
        this.marks = marks;
    }

    public String toString() {
        return name + " (" + marks + ")";
    }
}

List<Student> students = new ArrayList<>();
students.add(new Student(1, "Alice", 90));
students.add(new Student(2, "Bob", 75));
students.add(new Student(3, "Charlie", 85));

// Sort by marks
students.sort(Comparator.comparingInt(s -> s.marks));
System.out.println("Sorted: " + students);
```

Output:

```
Sorted: [Bob (75), Charlie (85), Alice (90)]
```

```java
// Find topper
Student topper = Collections.max(students, Comparator.comparingInt(s -> s.marks));
System.out.println("Topper: " + topper);
```

Output:

```
Topper: Alice (90)
```

```java
// Remove students with marks below 80
students.removeIf(s -> s.marks < 80);
System.out.println("After filter: " + students);
```

Output:

```
After filter: [Alice (90), Charlie (85)]
```

---

## 26. Interview Questions and Answers

**Q1. What is ArrayList in Java?**

> ArrayList is a resizable array implementation of the `List` interface from `java.util`. It stores elements in an `Object[]` array internally and grows automatically when full.

---

**Q2. How is ArrayList different from a regular Array?**

> Arrays have a fixed size set at creation. ArrayList grows and shrinks dynamically. Arrays can store primitives directly; ArrayList stores objects only (primitives are autoboxed). ArrayList provides many built-in methods that arrays lack.

---

**Q3. What is the default capacity of ArrayList?**

> The internal array starts **empty** (zero-length). On the first `add()`, capacity is set to **10** (the default). After that, it grows by approximately 1.5x each time it runs out of space.

---

**Q4. How does ArrayList grow internally?**

> When the internal array is full, ArrayList creates a new array with `newCapacity = oldCapacity + (oldCapacity >> 1)` (roughly 1.5x). It copies all elements to the new array using `Arrays.copyOf()` and discards the old array.

---

**Q5. What is the difference between size() and capacity?**

> `size()` is a public method returning the number of elements actually stored. Capacity is the length of the internal array — how much space exists before the next resize. There is no public method to check capacity.

---

**Q6. What is the time complexity of ArrayList operations?**

> - `get(index)` / `set(index)` → O(1)
> - `add()` at end → O(1) amortized
> - `add(index)` / `remove(index)` in middle → O(n)
> - `contains()` / `indexOf()` → O(n)
> - `sort()` → O(n log n)

---

**Q7. Can ArrayList store duplicate values?**

> Yes. ArrayList allows duplicate elements. `[1, 2, 2, 3, 3, 3]` is perfectly valid.

---

**Q8. Can ArrayList store null?**

> Yes. Unlike `List.of()` or `HashSet`, ArrayList allows null values, including multiple nulls.

---

**Q9. Is ArrayList synchronized (thread-safe)?**

> No. For thread safety, use `Collections.synchronizedList(new ArrayList<>())` or `CopyOnWriteArrayList`.

---

**Q10. What is the difference between ArrayList and LinkedList?**

> ArrayList uses a dynamic array — O(1) random access, O(n) insert/delete in the middle. LinkedList uses doubly-linked nodes — O(n) random access, O(1) add/remove at head/tail. ArrayList is better for most use cases due to CPU cache locality.

---

**Q11. What is the difference between ArrayList and Vector?**

> Both are resizable arrays, but Vector synchronizes every method (slower). ArrayList is not synchronized (faster). Vector grows by 2x, ArrayList by 1.5x. Vector is legacy — do not use it in new code.

---

**Q12. What happens when you modify a list inside a for-each loop?**

> `ConcurrentModificationException` is thrown. The iterator checks an internal `modCount` counter on every `next()` call. If the list was modified outside the iterator, the counter mismatch triggers the exception. Fix: use `iterator.remove()` or `removeIf()`.

---

**Q13. What is the difference between `remove(int)` and `remove(Object)` for `List<Integer>`?**

> `remove(1)` removes the element at **index 1**. `remove(Integer.valueOf(1))` removes the first occurrence of **value 1**. Always be explicit with `Integer` lists.

---

**Q14. What is the difference between `List.of()` and `Arrays.asList()`?**

> `List.of()` (Java 9+) creates a truly **immutable** list — no add, remove, set, or null allowed. `Arrays.asList()` returns a fixed-size list backed by the array — `set()` works, but `add()` and `remove()` throw exceptions.

---

**Q15. What sorting algorithm does ArrayList use?**

> **TimSort** — a hybrid of merge sort and insertion sort. O(n log n) worst case, O(n) best case. It is stable — equal elements preserve their original order.

---

**Q16. What is the difference between shallow copy and deep copy?**

> A shallow copy (like `new ArrayList<>(original)`) copies the references — both lists point to the same objects. A deep copy creates new instances of every element. ArrayList only provides shallow copies by default.

---

**Q17. How do you make an ArrayList thread-safe?**

> Two common options:
> 1. `Collections.synchronizedList(new ArrayList<>())` — synchronizes individual operations.
> 2. `CopyOnWriteArrayList` — creates a copy of the array on every write; best for read-heavy scenarios.

---

**Q18. Can you store primitive types in ArrayList?**

> Not directly. Java auto-converts primitives to their wrapper classes (autoboxing): `int` → `Integer`, `double` → `Double`, etc. For large numeric datasets, use arrays for better performance.

---

**Q19. What is `ensureCapacity()` and when should you use it?**

> `ensureCapacity(n)` pre-allocates internal space for at least `n` elements, avoiding multiple resize operations. Use it when you know you will add a large number of elements — e.g., `ensureCapacity(10000)` before a loop that adds 10,000 items.

---

**Q20. What is `trimToSize()` and when should you use it?**

> `trimToSize()` reduces the internal array's capacity to match the current size, releasing unused memory. Use it after a list is fully built and will only be read.

---

## 27. Quick Reference Cheat Sheet

```java
import java.util.*;
import java.util.stream.*;

// ── CREATE ──────────────────────────────────────────────────────
List<String> list = new ArrayList<>();                           // empty
List<String> list = new ArrayList<>(100);                       // with capacity
List<String> list = new ArrayList<>(List.of("A", "B", "C"));   // from collection
List<String> list = new ArrayList<>(Arrays.asList("A", "B"));  // from array

// ── ADD ─────────────────────────────────────────────────────────
list.add("X");                      // add at end            O(1)
list.add(0, "X");                   // insert at index       O(n)
list.addAll(otherList);             // add all from another  O(n)

// ── READ ────────────────────────────────────────────────────────
String x  = list.get(2);            // by index              O(1)
int sz    = list.size();            // count                 O(1)
boolean e = list.isEmpty();         // empty check           O(1)

// ── UPDATE ──────────────────────────────────────────────────────
String old = list.set(1, "Y");      // replace at index      O(1)

// ── REMOVE ──────────────────────────────────────────────────────
list.remove(1);                     // by index              O(n)
list.remove("X");                   // first match by value  O(n)
list.remove(Integer.valueOf(5));    // Integer by value      O(n)
list.removeIf(s -> s.isEmpty());    // by condition          O(n)
list.clear();                       // remove all            O(n)

// ── SEARCH ──────────────────────────────────────────────────────
boolean found = list.contains("X"); // exists?               O(n)
int idx  = list.indexOf("X");       // first index           O(n)
int lidx = list.lastIndexOf("X");   // last index            O(n)

// ── SORT ────────────────────────────────────────────────────────
Collections.sort(list);                                // natural order
list.sort(Comparator.reverseOrder());                  // reverse
list.sort(Comparator.comparing(Obj::getField));        // by field

// ── ITERATE ─────────────────────────────────────────────────────
for (String s : list) { }                              // for-each
for (int i = 0; i < list.size(); i++) { list.get(i); } // index loop
list.forEach(System.out::println);                     // lambda

// ── CONVERT ─────────────────────────────────────────────────────
String[] arr = list.toArray(String[]::new);            // list → array
List<String> l = new ArrayList<>(Arrays.asList(arr));  // array → list
List<String> snap = List.copyOf(list);                 // immutable copy

// ── UTILITY ─────────────────────────────────────────────────────
Collections.reverse(list);
Collections.shuffle(list);
Collections.max(list);
Collections.min(list);
Collections.frequency(list, "X");
```

---

## 28. Summary

```
ArrayList in a Nutshell:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1. It is a dynamic, resizable array from java.util.
  2. It maintains insertion order.
  3. It allows duplicates and null values.
  4. It provides O(1) index-based access (its superpower).
  5. It grows by ~1.5x when full (automatic resizing).
  6. It is NOT thread-safe.
  7. It stores objects only — primitives use wrapper classes.
  8. It is the best default choice for most list needs.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**When to use ArrayList:**
- You need a dynamic-size list.
- You frequently access elements by index.
- You mostly add/remove at the end.
- You need to store duplicates or maintain insertion order.

**When NOT to use ArrayList:**
- You frequently insert/remove at the beginning or middle → consider `ArrayDeque` or `LinkedList`.
- You need unique elements only → use `HashSet` or `LinkedHashSet`.
- You need thread safety → use `Collections.synchronizedList()` or `CopyOnWriteArrayList`.
- You need fast `contains()` checks → use `HashSet` (O(1) vs O(n)).

---

*Part of [Java Collection Framework Notes](README.md).*
