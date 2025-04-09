# Java Collections Framework: A Comprehensive Overview

Java Collections is a unified architecture that represents and manipulates groups of objects. It provides a set of interfaces and classes to store, retrieve, and manipulate data efficiently. Whether you need random access, sorted order, thread safety, or high-performance concurrent operations, the Collections Framework has a solution.

---

Below is a revised, properly formatted Table of Contents for the complete Markdown document. You can paste this at the top of your document for easy navigation.

---

# Table of Contents

## Basic Collections
1. [What Are Collections?](#what-are-collections)
2. [Collections Interface Hierarchy](#collections-interface-hierarchy)
3. [Core Interfaces and Implementations](#core-interfaces-and-implementations)
   - [List Implementations](#list-implementations)
   - [Set Implementations](#set-implementations)
   - [Queue Implementations](#queue-implementations)
   - [Map Implementations](#map-implementations)
4. [Summary](#summary)

## Iterators
1. [Iterator Overview](#iterator-overview)
2. [Ways to Iterate Collections](#ways-to-iterate-collections)
   - [Using Iterator](#using-iterator)
   - [Using Enhanced For-Each Loop](#using-enhanced-for-each-loop)
   - [Using ListIterator (for Lists)](#using-listiterator-for-lists)
   - [Using Enumeration (Legacy Collections)](#using-enumeration-legacy-collections)
3. [Fail-Fast vs Fail-Safe Iterators](#fail-fast-vs-fail-safe-iterators)
4. [Spliterator Overview](#spliterator-overview)
5. [Spliterator Usage and Examples](#spliterator-usage-and-examples)
6. [Summary of Iterators](#summary-of-iterators)

## Java Collections: Comparator, Immutable, and Concurrent Collections
1. [Comparator and Comparable](#comparator-and-comparable)
2. [Immutable and Unmodifiable Collections](#immutable-and-unmodifiable-collections)
3. [Concurrent Collections](#concurrent-collections)

---

This structured Table of Contents should help users quickly jump to any desired section in the document.

---

## What Are Collections?

- **Definition**: In Java, Collections are objects that group multiple elements into a single unit. They provide mechanisms to store, access, and manipulate data in a structured way.
- **Purpose**: They simplify programming tasks by providing reusable data structures such as lists, sets, queues, and maps.
- **Benefits**: Standardized operations (like sorting and searching), interoperability between different types of collections, and robust APIs for concurrent programming and safe data manipulation.

---

## Collections Interface Hierarchy

Below is an ASCII diagram that represents a simplified view of Java Collections’ interface hierarchy:

```
                             +-------------------+
                             |   Iterable<E>     |
                             +-------------------+
                                      │
                                      ▼
                              +----------------+
                              |  Collection<E> |
                              +----------------+
                                /      |     \
                               /       |      \
                              ▼        ▼       ▼
                          +-----+   +-----+   +-----+
                          | List|   | Set |   | Queue|
                          +-----+   +-----+   +-----+
                             │                   │
                             ▼                   ▼
                  +------------------+   +------------------+
                  |   ArrayList      |   |  PriorityQueue   |
                  |   LinkedList     |   |  Deque (LinkedList)|
                  |   Vector         |   +------------------+
                  |   Stack          |
                  |   CopyOnWriteArrayList |
                  +------------------+

           +-----------------+ 
           |     Map<K,V>    |   *(Not a child of Collection)*
           +-----------------+
                /      |      \
               /       |       \
           +------+ +---------+ +-------+
           |HashMap| |TreeMap | |LinkedHashMap|
           +------+ +---------+ +-------+
                   +---------+
                   | Hashtable|
                   +---------+
                   |ConcurrentHashMap|
                   +---------+
```

*Note*: The above diagram simplifies the full hierarchy and includes key interfaces and implementations.

---

## Core Interfaces and Implementations

Below are detailed tables for the primary collection types: **List**, **Set**, **Queue**, and **Map**. Each table lists common implementations with key details.

### List Implementations

Lists are ordered collections that allow duplicate elements. Use them when you need to access elements by index or maintain insertion order.

| Implementation         | Usage / When to Use                                                     | Internal Implementation & Growth Factor                                                      | Thread Safety           | Advantages                                              | Disadvantages                                                   | Important Methods                             |
|------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|-------------------------|---------------------------------------------------------|-----------------------------------------------------------------|-----------------------------------------------|
| **ArrayList**          | Fast random access, frequent iteration; ideal for read-heavy lists.     | Backed by an array; increases capacity (typically by 50% or 1.5× when needed).                | Not thread-safe         | Fast lookup and iteration; low overhead.               | Slow for mid-list inserts/deletes; resizing costs.              | `add()`, `get()`, `remove()`, `size()`, `iterator()`         |
| **LinkedList**         | When frequent insertions/deletions in the middle are expected.           | Doubly-linked list structure; no preset growth factor since nodes are added as needed.         | Not thread-safe         | Fast insertions/removals; supports Deque operations.     | Slower random access; uses extra memory for links.              | `add()`, `addFirst()`, `addLast()`, `remove()`, `getFirst()`, `iterator()` |
| **Vector**             | Legacy collections; when thread-safe operations on a list are necessary.  | Array-based; doubles capacity by default when expansion is required.                         | Thread-safe (synchronized)| Thread safety using synchronized methods; similar to ArrayList. | Overhead due to synchronization; considered outdated.           | `add()`, `get()`, `remove()`, `elements()`, `capacity()`      |
| **Stack**              | LIFO operations; legacy implementation (extends Vector).                | Inherits array-based structure from Vector.                                                   | Thread-safe (synchronized)| Provides push/pop/peek methods for stack behavior.       | Legacy design; better alternatives (e.g., Deque) exist.           | `push()`, `pop()`, `peek()`, `empty()`, `search()`             |
| **CopyOnWriteArrayList** | Concurrent read-heavy use cases with infrequent writes.               | Internally uses an array; on each mutation, a new copy of the array is created.                | Thread-safe             | Safe for concurrent iteration without locks; no ConcurrentModificationException. | High memory and CPU cost on writes; not suitable for write-intensive operations. | `add()`, `get()`, `remove()`, `iterator()` (snapshot-based)    |

---

### Set Implementations

Sets hold unique elements. They are best used when duplicate elements are not allowed.

| Implementation       | Usage / When to Use                                                   | Internal Implementation & Growth Factor                                                 | Thread Safety    | Advantages                                                | Disadvantages                                               | Important Methods                         |
|----------------------|------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|------------------|-----------------------------------------------------------|-------------------------------------------------------------|-------------------------------------------|
| **HashSet**          | When uniqueness is required without any ordering.                   | Backed by a HashMap; uses hash codes; expands (typically doubles) based on load factor (~0.75). | Not thread-safe  | Fast add, remove, search; efficient for large sets.       | No guaranteed order; performance depends on hashCode distribution.  | `add()`, `remove()`, `contains()`, `iterator()`, `size()`      |
| **LinkedHashSet**    | When insertion order needs to be preserved along with uniqueness.     | Similar to HashSet but with a doubly-linked list across entries; similar growth characteristics. | Not thread-safe  | Maintains insertion order; consistent iteration order.    | Slightly slower than HashSet due to additional link maintenance.  | `add()`, `remove()`, `contains()`, `iterator()`, `size()`      |
| **TreeSet**          | When a sorted set is required.                                         | Backed by a Red-Black tree; no growth factor concerns (tree-based).                       | Not thread-safe  | Automatically sorted; supports navigation methods (e.g., floor, ceiling). | Slower than hash-based sets; requires elements to be Comparable (or via Comparator). | `add()`, `remove()`, `contains()`, `iterator()`, `first()`, `last()`, `ceiling()`, `floor()` |

*For high-concurrency sorted sets, consider [ConcurrentSkipListSet](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentSkipListSet.html), which uses a skip list.*

---

### Queue Implementations

Queues are used for holding elements prior to processing. They are especially useful for scheduling tasks.

| Implementation         | Usage / When to Use                                               | Internal Implementation & Growth Factor                              | Thread Safety     | Advantages                                                     | Disadvantages                                                   | Important Methods                                         |
|------------------------|-------------------------------------------------------------------|----------------------------------------------------------------------|-------------------|----------------------------------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------------|
| **PriorityQueue**      | When elements need to be processed based on a natural ordering or comparator.  | Backed by an array-based binary heap; dynamic resizing is used.       | Not thread-safe   | Retrieves the smallest (or highest priority) element fast.      | Unordered iteration; not synchronized; must be reheapified on updates. | `add()`, `offer()`, `poll()`, `peek()`, `size()`           |
| **LinkedList (as Queue)** | When a FIFO (first-in, first-out) ordering is needed; also supports Deque operations. | Doubly-linked list; grows as nodes are added.                         | Not thread-safe   | Supports both queue and deque operations; flexible usage.        | Slower random access; overhead for linked nodes.               | `add()`, `offer()`, `poll()`, `peek()`, `addFirst()`, `addLast()` |

---

### Map Implementations

Maps store key-value pairs. Although not a child of the Collection interface, they are a core part of Java’s Collections Framework.

| Implementation         | Usage / When to Use                                                  | Internal Implementation & Growth Factor                                              | Thread Safety        | Advantages                                                       | Disadvantages                                                     | Important Methods                                           |
|------------------------|----------------------------------------------------------------------|--------------------------------------------------------------------------------------|----------------------|------------------------------------------------------------------|-------------------------------------------------------------------|-------------------------------------------------------------|
| **HashMap**            | General-purpose key-value storage; fast lookup and insertion.        | Backed by an array of buckets; load factor ~0.75 triggers doubling the bucket array. | Not thread-safe      | Fast operations; efficient memory usage; supports null keys/values. | Unordered; not synchronized (fail-fast iterators).                | `put()`, `get()`, `remove()`, `containsKey()`, `keySet()`, `values()`   |
| **LinkedHashMap**      | When insertion (or access) order must be maintained.                 | Combination of HashMap with a doubly-linked list; similar growth pattern to HashMap.  | Not thread-safe      | Preserves iteration order; predictable iteration over entries.    | Slightly slower than HashMap due to ordering overhead.             | `put()`, `get()`, `remove()`, `containsKey()`, `keySet()`, `values()`   |
| **TreeMap**            | When keys need to be stored in a sorted order.                       | Based on a Red-Black tree; no growth factor concerns.                                | Not thread-safe      | Sorted keys; supports range queries and navigation.              | Slower than HashMap; cannot store null keys.                       | `put()`, `get()`, `remove()`, `firstKey()`, `lastKey()`, `subMap()`      |
| **Hashtable**          | Legacy synchronized map; when thread-safe operations are required.   | Hash table implementation; capacity typically doubles; similar to HashMap.            | Thread-safe (synchronized)| Provides thread safety out-of-the-box.                             | Performance overhead due to synchronization; legacy design.         | `put()`, `get()`, `remove()`, `elements()`, `keys()`          |
| **ConcurrentHashMap**  | For high-concurrency scenarios; non-blocking reads and segmented locks. | Uses a segmented hash table (or bin-based locking); dynamically resizes.                | Thread-safe          | High throughput in concurrent environments; better scalability.    | Iterators are weakly consistent; slightly more complex semantics.    | `put()`, `get()`, `remove()`, `forEach()`, `compute()`         |

*For concurrent sorted maps, [ConcurrentSkipListMap](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentSkipListMap.html) is available.*

---

# Iterators and Spliterators in Java

## Iterator Overview

An **Iterator** is an object that enables you to traverse through a collection one element at a time.  
- **Key Methods**:
  - `hasNext()`: Checks if there is another element.
  - `next()`: Retrieves the next element.
  - `remove()`: (Optional) Removes the last element returned by the iterator from the underlying collection.

Iterators are available from most collection implementations via the `iterator()` method.

---

## Ways to Iterate Collections

### Using Iterator

Using the explicit **Iterator** is useful when you want fine-grained control over the iteration process, such as conditionally removing elements during traversal.

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorExample {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        
        Iterator<String> iterator = names.iterator();
        while (iterator.hasNext()) {
            String name = iterator.next();
            System.out.println("Name: " + name);
            // Remove an element if a condition is met
            if ("Bob".equals(name)) {
                iterator.remove(); // safe removal during iteration
            }
        }
        System.out.println("After removal: " + names);
    }
}
```

### Using Enhanced For-Each Loop

Java’s **for-each loop** provides a concise syntax that under the hood uses an iterator. This method is recommended when you only need to read elements without modifying the collection.

```java
import java.util.List;
import java.util.ArrayList;

public class ForEachExample {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>();
        cities.add("New York");
        cities.add("Paris");
        cities.add("Tokyo");

        for (String city : cities) {
            System.out.println("City: " + city);
        }
    }
}
```

### Using ListIterator (for Lists)

The **ListIterator** is available for List implementations. In addition to forward traversal, it allows backward iteration and element modification.

```java
import java.util.LinkedList;
import java.util.ListIterator;

public class ListIteratorExample {
    public static void main(String[] args) {
        LinkedList<String> fruits = new LinkedList<>();
        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Cherry");

        ListIterator<String> listIterator = fruits.listIterator();
        // Forward iteration:
        while (listIterator.hasNext()) {
            System.out.println("Fruit: " + listIterator.next());
        }
        
        // Backward iteration:
        while (listIterator.hasPrevious()) {
            System.out.println("Previous Fruit: " + listIterator.previous());
        }
    }
}
```

### Using Enumeration (Legacy Collections)

The **Enumeration** interface is primarily used with legacy collections like `Vector` or `Hashtable`. It provides methods `hasMoreElements()` and `nextElement()`.

```java
import java.util.Enumeration;
import java.util.Vector;

public class EnumerationExample {
    public static void main(String[] args) {
        Vector<Integer> numbers = new Vector<>();
        numbers.add(10);
        numbers.add(20);
        numbers.add(30);
        
        Enumeration<Integer> enumeration = numbers.elements();
        while (enumeration.hasMoreElements()) {
            System.out.println("Number: " + enumeration.nextElement());
        }
    }
}
```

---

## Fail-Fast vs Fail-Safe Iterators

When iterating over a collection, concurrent modifications (changes made by another thread or even by the same thread outside the iterator) can cause unpredictable results. Java collections implement two main strategies:

### Fail-Fast Iterators

- **Behavior**: They throw a `ConcurrentModificationException` if the collection is modified (other than by the iterator's own `remove()` method) while iterating.
- **Usage**: Almost all standard collection iterators (e.g., those from `ArrayList`, `HashMap`) are fail-fast.
- **Example**:

  ```java
  import java.util.ArrayList;
  import java.util.Iterator;
  import java.util.List;

  public class FailFastExample {
      public static void main(String[] args) {
          List<String> list = new ArrayList<>();
          list.add("A");
          list.add("B");
          list.add("C");

          try {
              for (String s : list) {
                  if ("B".equals(s)) {
                      list.remove(s); // Modifying collection outside the iterator
                  }
              }
          } catch (Exception e) {
              System.out.println("Exception caught: " + e);
          }
      }
  }
  ```
  
  *In the example above, a `ConcurrentModificationException` is thrown because the list is modified during iteration.*

### Fail-Safe Iterators

- **Behavior**: They operate on a clone of the underlying collection, meaning that they do not throw exceptions when the collection is modified during iteration. However, the iterator does not reflect those modifications.
- **Usage**: Collections like `CopyOnWriteArrayList` and `ConcurrentHashMap` use fail-safe iterators.
- **Example**:

  ```java
  import java.util.Iterator;
  import java.util.concurrent.CopyOnWriteArrayList;

  public class FailSafeExample {
      public static void main(String[] args) {
          CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
          list.add("X");
          list.add("Y");
          list.add("Z");

          Iterator<String> iterator = list.iterator();
          while (iterator.hasNext()) {
              String element = iterator.next();
              System.out.println("Element: " + element);
              // Modification during iteration does not throw an exception
              if ("Y".equals(element)) {
                  list.add("W");
              }
          }
          System.out.println("Final list: " + list);
      }
  }
  ```
  
  *In this example, even though the list is modified during iteration, the iterator works on a snapshot of the collection and does not throw an exception.*

---

## Spliterator Overview

Introduced in Java 8, a **Spliterator** is a special iterator designed for parallelism and bulk operations.

### Key Features:
- **Splitting**: A Spliterator can partition (or "split") its elements into several parts so that they can be processed concurrently.
- **Traversal Methods**:
  - `tryAdvance(Consumer<? super T> action)`: Processes a single element if available.
  - `forEachRemaining(Consumer<? super T> action)`: Processes all remaining elements.
  - `trySplit()`: Attempts to split the elements for parallel processing.
- **Characteristics**: Spliterators provide additional metadata such as size estimates and ordering. They also report characteristics (e.g., `ORDERED`, `SORTED`, `DISTINCT`) to guide parallel processing strategies.

---

## Spliterator Usage and Examples

### Basic Spliterator Example

```java
import java.util.ArrayList;
import java.util.Spliterator;
import java.util.function.Consumer;

public class SpliteratorExample {
    public static void main(String[] args) {
        ArrayList<Integer> numbers = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            numbers.add(i);
        }
        
        Spliterator<Integer> spliterator = numbers.spliterator();
        
        // Processing one element at a time using tryAdvance()
        System.out.println("Using tryAdvance():");
        while (spliterator.tryAdvance((Integer n) -> System.out.println("Number: " + n))) {
            // Each tryAdvance call processes one element
        }
        
        // Alternatively, use forEachRemaining() to process remaining elements
        spliterator = numbers.spliterator(); // get a new spliterator
        System.out.println("\nUsing forEachRemaining():");
        spliterator.forEachRemaining((Integer n) -> System.out.println("Number: " + n));
    }
}
```

### Splitting for Parallel Processing

The `trySplit()` method partitions the spliterator into two parts—one for immediate processing and one for later or concurrent processing.

```java
import java.util.ArrayList;
import java.util.Spliterator;

public class SpliteratorSplitExample {
    public static void main(String[] args) {
        ArrayList<String> items = new ArrayList<>();
        for (int i = 1; i <= 8; i++) {
            items.add("Item-" + i);
        }
        
        Spliterator<String> spliterator1 = items.spliterator();
        Spliterator<String> spliterator2 = spliterator1.trySplit();
        
        System.out.println("Spliterator 1:");
        spliterator1.forEachRemaining(item -> System.out.println("  " + item));
        
        System.out.println("\nSpliterator 2:");
        if (spliterator2 != null) {
            spliterator2.forEachRemaining(item -> System.out.println("  " + item));
        }
    }
}
```

*The above example shows how to split a collection into two parts, which can be processed in parallel.*

---

## Summary

- **Iterators** provide a standard way to traverse collections and include fail-fast behavior to prevent unpredictable modifications.
- **Enhanced For-Each Loops** simplify iteration when no modifications are needed.
- **ListIterator** offers additional features like bi-directional traversal for list collections.
- **Enumeration** is used with legacy collections.
- **Fail-Fast vs Fail-Safe Iterators**:  
  - *Fail-Fast* iterators (e.g., in `ArrayList`, `HashMap`) throw exceptions if the collection is modified during iteration.  
  - *Fail-Safe* iterators (e.g., in `CopyOnWriteArrayList`) operate on a snapshot and allow concurrent modifications.
- **Spliterators**—introduced in Java 8—support both sequential and parallel traversal. They can be split into multiple parts, making them ideal for parallel processing using streams or fork-join frameworks.

---

# Java Collections: Comparator, Immutable, and Concurrent Collections


## 1. Comparator and Comparable

In Java, these two interfaces are used to provide ordering semantics for objects.

### Comparable

- **Purpose**:  
  Implemented by a class to define its **natural ordering** via a single method—`compareTo()`.
  
- **Example**: Sorting persons by age.

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
       this.name = name;
       this.age = age;
    }
    
    @Override
    public int compareTo(Person other) {
       // Compare by age
       return Integer.compare(this.age, other.age);
    }
    
    @Override
    public String toString() {
       return name + " (" + age + ")";
    }
    
    // Getters and setters (if needed)
}
```

*Usage*: Now you can sort a list of `Person` objects naturally by age.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ComparableDemo {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Alice", 30));
        people.add(new Person("Bob", 25));
        people.add(new Person("Charlie", 35));

        // Sort using natural order (by age)
        Collections.sort(people);
        System.out.println(people);
    }
}
```

### Comparator

- **Purpose**:  
  The `Comparator<T>` interface is used to define **custom ordering**. Multiple comparators may be created for the same class, and they are particularly useful if the natural ordering (i.e. via Comparable) is not what you need.
  
- **Example**: Sorting persons by name with a lambda expression.

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class ComparatorDemo {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Charlie", 35));
        people.add(new Person("Alice", 30));
        people.add(new Person("Bob", 25));

        // Using a lambda to sort by name
        Comparator<Person> byName = (p1, p2) -> p1.toString().split(" ")[0].compareTo(p2.toString().split(" ")[0]);
        // Alternatively, using Comparator.comparing:
        // Comparator<Person> byName = Comparator.comparing(Person::toString); // if toString returns name

        people.sort(byName);
        System.out.println(people);
    }
}
```

*Usage with Lambdas*:  
Java 8 and later allow concise comparator definitions with lambda expressions or the static helper `Comparator.comparing()`:

```java
Comparator<Person> byName = Comparator.comparing(person -> person.toString().split(" ")[0]);
```

---

## 2. Immutable and Unmodifiable Collections

### Immutable Collections

Immutable collections are fixed after creation. Java 9+ provides factory methods that return truly immutable collections.

- **Characteristics**:  
  They cannot be modified after they are created. Any modification attempts will throw an exception.
  
- **Example**:  
  Creating an immutable list:

```java
import java.util.List;

public class ImmutableCollectionsDemo {
    public static void main(String[] args) {
        List<String> immutableList = List.of("Apple", "Banana", "Cherry");
        
        // immutableList.add("Date"); // This would throw UnsupportedOperationException
        
        System.out.println("Immutable List: " + immutableList);
    }
}
```

### Unmodifiable Collections

Unmodifiable collections are **read-only views** of an existing collection. They are created using methods from the `Collections` class. However, if the underlying collection is modified, the changes will be visible in the unmodifiable view.

- **Example**: Wrapping an existing list:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class UnmodifiableCollectionsDemo {
    public static void main(String[] args) {
        List<String> modifiableList = new ArrayList<>();
        modifiableList.add("Dog");
        modifiableList.add("Cat");
        
        List<String> unmodifiableList = Collections.unmodifiableList(modifiableList);
        
        System.out.println("Unmodifiable List: " + unmodifiableList);
        
        // modifiableList.add("Elephant");  // Changes the underlying list and will be visible in unmodifiableList
        // unmodifiableList.add("Giraffe");  // This will throw UnsupportedOperationException
    }
}
```

*Key Differences*:
- **Immutable collections** are completely fixed after creation.
- **Unmodifiable collections** are wrappers around mutable collections, offering a read-only view.

---

## 3. Concurrent Collections

Concurrent collections are specialized data structures designed for high-throughput and thread-safe operations in concurrent environments. The table below lists the key concurrent collections, their usage, internal implementations (including notes on growth factors when relevant), advantages, disadvantages, and important methods.

### Concurrent Collections Table

| Implementation               | Usage / When to Use                                                     | Internal Implementation & Growth Factor                                                                                                | Advantages                                                  | Disadvantages                                           | Important Methods                                                   |
|------------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------|----------------------------------------------------------------------|
| **ConcurrentHashMap**        | Thread-safe key-value storage for high concurrency; ideal for caching.  | Based on a hash table with non-blocking reads; uses CAS (compare-and-swap) and segmentation; capacity typically doubles on resize.     | High concurrency, non-blocking read operations; scales well. | Iterators are weakly consistent; does not allow null keys/values. | `put()`, `get()`, `remove()`, `computeIfAbsent()`, `forEach()`         |
| **CopyOnWriteArrayList**     | Read-heavy scenarios where writes are infrequent.                     | Backed by an array; on each mutative operation, it creates a new copy of the underlying array; similar growth behavior as ArrayList.      | Safe for concurrent iteration without locks; no CMEs.     | High cost for modifications (write-intensive scenarios).    | `add()`, `get()`, `remove()`, `iterator()` (snapshot-based)            |
| **CopyOnWriteArraySet**      | Thread-safe set with infrequent modifications; backed by CopyOnWriteArrayList. | Internally wraps a CopyOnWriteArrayList; uses array copying on write.                                                                    | Same benefits as CopyOnWriteArrayList; maintains uniqueness. | Same write cost as CopyOnWriteArrayList; more memory consumption. | `add()`, `remove()`, `contains()`, `iterator()`                        |
| **ConcurrentLinkedQueue**    | Lock-free FIFO queue for high-throughput data exchange.                 | A linked node-based structure using CAS for non-blocking operations; unbounded queue.                                                  | High concurrency, non-blocking algorithm; lightweight.    | Unbounded (risk of memory issues if not managed correctly). | `offer()`, `poll()`, `peek()`, `size()`                                  |
| **ConcurrentLinkedDeque**    | Lock-free double-ended queue for flexible FIFO and LIFO operations.     | Doubly-linked nodes supporting CAS operations; similar in design to ConcurrentLinkedQueue with additional endpoints.                      | Supports both ends; high throughput; non-blocking.          | Slightly more overhead compared to simple FIFO queues.    | `offerFirst()`, `offerLast()`, `pollFirst()`, `pollLast()`, `peekFirst()`, `peekLast()` |
| **ConcurrentSkipListMap**    | Sorted, thread-safe Map for concurrent access with natural ordering.    | Implements a skip list data structure; logarithmic time performance for most operations; does not use traditional growth factors.          | Provides sorted order; good performance under concurrency.  | Slower than unsorted maps for non-sorted access; no null keys.  | `put()`, `get()`, `remove()`, `firstEntry()`, `ceilingEntry()`, `subMap()`  |
| **ConcurrentSkipListSet**    | Thread-safe sorted Set based on ConcurrentSkipListMap.                  | Internally uses a ConcurrentSkipListMap; maintains sorted order; comparable to ConcurrentSkipListMap in behavior.                           | Retains order; supports high concurrency similar to SkipListMap. | Performance is tied to the underlying map; no null elements.   | `add()`, `remove()`, `contains()`, `iterator()`                          |

*Note*: Growth factor details are typically more relevant to array-based collections (like CopyOnWriteArrayList) rather than linked or skip list–based collections.

---

## Summary

- **Comparator and Comparable**:  
  - Implement **Comparable** within your class to provide its natural order (using `compareTo()`), and use **Comparator** for alternative or multiple orderings.  
  - Lambda expressions (e.g., via `Comparator.comparing()`) simplify the creation of comparators.

- **Immutable vs Unmodifiable Collections**:  
  - **Immutable Collections** (e.g., Java 9+ `List.of()`) are fixed and cannot be changed after creation.  
  - **Unmodifiable Collections** (e.g., `Collections.unmodifiableList()`) provide a read-only view of an existing collection, though the underlying collection might still be mutable.

- **Concurrent Collections**:  
  - The table above lists popular concurrent collection classes with details on their internals, when and why to use them, advantages/disadvantages, and their critical methods.
