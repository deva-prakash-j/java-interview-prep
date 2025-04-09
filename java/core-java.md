# Core Java: A Comprehensive Guide

This document covers the following Core Java topics:
- JDK, JRE & JVM
- Java Memory Management and Garbage Collection
- Data Types
- Object-Oriented Programming (OOPS)
- Exception Handling

Each section contains explanations, examples, visual representations, tables, and tricky ("twisted") interview questions with answers and examples.

---

## Table of Contents

### Basic Concepts
1. [JDK, JRE & JVM](#jdk-jre--jvm)

### Java Memory Management and Garbage Collection
1. [Memory Model](#memory-model)
   - What is Heap and When It Is Used
   - What is Stack and When It Is Used
   - What is Method Area and When It Is Used
   - *Twisted interview questions with answers and examples*
2. [Garbage Collection](#garbage-collection)
   - GC Algorithms Table (Description, Usage, When/How to Use, Advantages/Disadvantages)
3. [Tuning GC](#tuning-gc)
4. [Memory Leaks](#memory-leaks)
   - What are Memory Leaks
   - How to Identify Memory Leaks
   - Ways to Fix Memory Leaks
5. [Performance Considerations](#performance-considerations)
   - Profiling and Optimizing Memory Usage
   - *Twisted interview questions with answers and examples*

### Data Types
1. [Primitive and Non-Primitive Data Types](#data-types)
   - Overview of Primitive Types, Wrapper Classes, and Why Wrappers?
   - Bits Used and Type Casting

### Object-Oriented Programming (OOPS)
1. [Class and Object](#class-and-object)
2. [Inheritance](#inheritance)
   - Types of Inheritance with Visual Representation and Examples
   - *Twisted interview questions with answers and examples*
3. [Polymorphism](#polymorphism)
   - Method Overloading and Method Overriding
   - Static vs Dynamic Binding
   - Covariant Return Types
   - Use of `super` Keyword
   - Use of `final` Keyword and Instance Initializer Block
   - *Twisted interview questions with answers and examples*
4. [Abstraction](#abstraction)
   - Abstract Classes vs Interfaces
   - Differences Between Abstract Class and Interface
   - *Twisted interview questions with answers and examples*
5. [Encapsulation](#encapsulation)
   - What is Encapsulation and Ways to Achieve It
   - Access Modifiers Comparison Table
   - *Twisted interview questions with answers and examples*

### Exception Handling
1. [Exception Handling Basics](#exception-handling)
   - Checked vs Unchecked Exceptions
   - Custom Exceptions
   - Try-With-Resources
   - Assertions
   - Exception Priority and Finally Block
   - *Twisted interview questions with answers and examples*

---

## JDK, JRE & JVM

- **JDK (Java Development Kit)**  
  A complete package to develop, compile, and run Java applications. It includes the JRE and development tools (like the compiler).

- **JRE (Java Runtime Environment)**  
  The runtime portion of the JDK. It provides libraries, Java Virtual Machine (JVM), and other components to run Java applications.

- **JVM (Java Virtual Machine)**  
  An abstract computing machine that enables a computer to run Java programs. It provides platform independence via bytecode execution and handles memory management and garbage collection.

---

## Java Memory Management and Garbage Collection

### Memory Model

#### What is Heap and When It Is Used
- **Heap**: The runtime data area in which objects are allocated.
- **Usage**: All objects and instance variables are stored on the heap. It is shared across all threads.

#### What is Stack and When It Is Used
- **Stack**: Memory for method execution, local variables, and call frames.
- **Usage**: Each thread has its own stack used to store primitive values, method frames, and references to objects in the heap.

#### What is Method Area and When It Is Used
- **Method Area**: Also known as the "PermGen" (in older versions) or "Metaspace" in Java 8+.
- **Usage**: Stores class structures like runtime constant pool, field and method data, and the code for methods and constructors.

#### Twisted Interview Questions: Memory Model
- **Q1**: *How does the stack differ from the heap in Java?*  
  **A**: The stack stores method calls and local variables with a LIFO structure, while the heap is a shared runtime area that holds all objects and instance data.
  
- **Q2**: *Can you provide an example to show when a variable is allocated on the heap vs the stack?*  
  **A (Example)**:
  ```java
  public class MemoryDemo {
      public void demoMethod() {
          int localVar = 5; // allocated on the stack
          Person p = new Person("John", 30); // 'p' reference on stack; object on heap
      }
      public static void main(String[] args) {
          new MemoryDemo().demoMethod();
      }
  }
  ```
- *Additional twisted questions may involve differences in scoping, lifetimes, and interactions between heap and stack memory.*

---

### Garbage Collection

Garbage Collection (GC) automatically frees memory by removing objects that are no longer referenced. Below is a table comparing different GC algorithms.

| GC Algorithm            | Description                                                     | Real-World Usage                        | When to Use                 | How to Use (Tuning Options)                     | Advantages                                           | Disadvantages                                 |
|-------------------------|-----------------------------------------------------------------|-----------------------------------------|-----------------------------|------------------------------------------------|-----------------------------------------------------|------------------------------------------------|
| **Serial GC**           | Single-threaded GC; suitable for small applications             | Desktop applications, single-threaded apps | Use in low-memory environments  | `-XX:+UseSerialGC`                              | Simple, low overhead for small heaps                | Not scalable for multi-threaded applications   |
| **Parallel GC**         | Multi-threaded GC; uses multiple threads for minor collections    | Multi-core systems, batch processing     | Use when throughput is priority   | `-XX:+UseParallelGC`, tune thread count            | High throughput; efficient for large heaps          | May cause longer pause times if not tuned       |
| **CMS (Concurrent Mark Sweep)** | Concurrent collection with minimal pause times         | Interactive applications, low-latency environments | Use when low pause times are critical | `-XX:+UseConcMarkSweepGC`                         | Minimal pauses; concurrent marking                  | Higher CPU usage; may cause fragmentation         |
| **G1 GC**               | Region-based GC; balances pause times and throughput               | Large-scale applications, server environments | Use when predictable pauses are desired  | `-XX:+UseG1GC`, tune regions and pause times        | Predictable pause times; good for large heaps       | Complex tuning; not always optimal for smaller heaps |

---

### Tuning GC

- **JVM Options**: Tune GC using JVM command-line options such as `-XX:MaxGCPauseMillis`, `-XX:ParallelGCThreads`, `-XX:ConcGCThreads`.
- **Profiling Tools**: Use tools like VisualVM, JConsole, or commercial profilers.
- **GC Logs**: Enable GC logging (`-Xlog:gc*`) to observe GC behavior and tweak parameters accordingly.

---

### Memory Leaks

#### What Are Memory Leaks?
- **Definition**: Occur when an application unintentionally holds references to unused objects, preventing GC from reclaiming that memory.

#### How to Identify Memory Leaks
- **Tools**: Use profilers (VisualVM, YourKit) to monitor heap usage.
- **Symptoms**: Increasing memory usage over time, frequent full GCs, and OutOfMemoryErrors.

#### Ways to Fix Memory Leaks
- **Remove Unused References**: Nullify references when not needed.
- **Use Weak References**: Employ `WeakReference`, `SoftReference`, or `WeakHashMap` where applicable.
- **Code Review and Profiling**: Regularly review code for unnecessary retention and use profiling tools to trace issues.

---

### Performance Considerations

- **Profiling**: Use profiling tools (e.g., VisualVM, JProfiler) to understand memory usage.
- **Optimization Techniques**:  
  - Minimize object allocation.
  - Reuse objects where possible.
  - Tune GC parameters.
- **Twisted Interview Q&A**:
  - **Q**: *How do you identify performance bottlenecks related to GC?*  
    **A**: By analyzing GC logs, profiling memory allocations, and monitoring pause times.
  - **Q**: *Can you provide an example of optimizing memory usage in Java?*  
    **A (Example)**:
    ```java
    public class OptimizationDemo {
        public static void main(String[] args) {
            // Instead of creating multiple small objects, use object pooling.
            ObjectPool<MyObject> pool = new ObjectPool<>(MyObject::new, 50);
            MyObject obj = pool.acquire();
            // Use the object...
            pool.release(obj);
        }
    }
    ```

---

## Data Types

### Overview: Primitive and Non-Primitive Types

- **Primitive Types**:  
  - Types: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`
  - **Bits Used**:
    - `byte`: 8 bits
    - `short`: 16 bits
    - `int`: 32 bits
    - `long`: 64 bits
    - `float`: 32 bits (IEEE 754)
    - `double`: 64 bits (IEEE 754)
    - `char`: 16 bits (Unicode)
    - `boolean`: Not precisely defined (typically one bit used logically)
- **Non-Primitive (Reference) Types**:  
  - Classes, Interfaces, Arrays, Enums, etc.
- **Wrapper Classes**:  
  - Provide object representations of primitives (e.g., `Integer` for `int`).
  - Enable usage in collections and offer utility methods.
- **Type Casting**:  
  - **Implicit Casting**: Automatic promotion (e.g., `int` to `long`).
  - **Explicit Casting**: Using cast operators when needed (e.g., `double` to `int`).

*Additional topics*: Boxing/unboxing, comparing wrapper objects using `.equals()` rather than `==`.

---

## OOPS

### Class and Object

- **Class**: A blueprint for objects. Defines state (fields) and behavior (methods).
- **Object**: An instance of a class with concrete state.

---

### Inheritance

Inheritance enables a class (child) to inherit properties and methods from another class (parent).

#### Types of Inheritance
- **Single Inheritance**: A class extends one superclass.
- **Multi-level Inheritance**: A derived class acts as a base class for further derivations.
- **Hierarchical Inheritance**: Multiple classes extend a single superclass.

#### Visual Representation

```
           [Animal]
              │
      ┌───────┴───────┐
      ▼               ▼
   [Dog]           [Cat]
```

#### Example
```java
public class Animal {
    public void eat() {
        System.out.println("Animal is eating");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barks");
    }
}
```

#### Twisted Interview Q&A for Inheritance
- **Q**: *Can Java support multiple inheritance?*  
  **A**: Java does not support multiple class inheritance (to avoid ambiguity) but supports multiple inheritance via interfaces.
- **Q**: *Give an example of multi-level inheritance in Java.*  
  **A (Example)**:
  ```java
  class A { }
  class B extends A { }
  class C extends B { }
  ```

---

### Polymorphism

Polymorphism means one interface can be used for a general class of actions.

#### Method Overloading
- Multiple methods in the same class with the same name but different parameters.

#### Method Overriding
- Subclasses provide a specific implementation for a method declared in its superclass.

#### Static vs Dynamic Binding
- **Static binding**: Resolved at compile time (e.g., static, private methods).
- **Dynamic binding**: Resolved at runtime (e.g., overridden methods).

#### Covariant Return Type
- Overridden methods can return a subtype of the declared return type.

#### Usage of `super` Keyword
- Refers to the parent class’s members.

#### Use of `final` Keyword
- Prevents method overriding or inheritance; ensures constants.

#### Instance Initializer Block
- Code block executed before the constructor during object creation.

#### Twisted Interview Q&A for Polymorphism
- **Q**: *How is method overriding different from method overloading?*  
  **A**: Overriding involves methods in different classes with an inheritance relationship, while overloading is within the same class.
- **Q**: *Can you provide an example of dynamic binding in Java?*  
  **A (Example)**:
  ```java
  class Parent {
      void display() { System.out.println("Parent Display"); }
  }
  class Child extends Parent {
      @Override
      void display() { System.out.println("Child Display"); }
  }
  public class Test {
      public static void main(String[] args) {
          Parent obj = new Child();
          obj.display(); // Output will be "Child Display" (dynamic binding)
      }
  }
  ```

---

### Abstraction

Abstraction hides complex implementation details and shows only the necessary features.

#### Abstract Classes
- Cannot be instantiated; may contain abstract methods that subclasses must implement.

#### Interfaces
- Completely abstract types (prior to Java 8); now can have default and static methods.

#### Differences
| Aspect                  | Abstract Class                      | Interface                          |
|-------------------------|-------------------------------------|------------------------------------|
| Inheritance             | Single inheritance allowed          | Multiple inheritance supported     |
| Implementation          | Can have implemented methods        | Methods are abstract by default    |
| Constructors            | Can have constructors               | No constructors                    |

#### Twisted Interview Q&A for Abstraction
- **Q**: *When would you choose an interface over an abstract class?*  
  **A**: Use interfaces to achieve multiple inheritance or when no common state is shared.
- **Q**: *Provide an example illustrating abstraction in Java.*  
  **A (Example)**:
  ```java
  abstract class Shape {
      abstract double area();
  }
  class Circle extends Shape {
      double radius;
      Circle(double radius) { this.radius = radius; }
      @Override
      double area() { return Math.PI * radius * radius; }
  }
  ```

---

### Encapsulation

Encapsulation wraps data (fields) and methods into a single unit, restricting direct access to some of an object's components.

#### Ways to Achieve Encapsulation
- Declare fields as `private`
- Provide public getter and setter methods

#### Access Modifiers Comparison Table

| Modifier   | Class | Package | Subclass (same package) | Subclass (different package) | World  |
|------------|-------|---------|-------------------------|------------------------------|--------|
| **public** | Yes   | Yes     | Yes                     | Yes                          | Yes    |
| **protected** | Yes   | Yes  | Yes                     | Yes                          | No     |
| **default** (no modifier) | Yes | Yes     | Yes                     | No                           | No     |
| **private** | Yes   | No      | No                      | No                           | No     |

#### Twisted Interview Q&A for Encapsulation
- **Q**: *Why is encapsulation important in Java?*  
  **A**: It protects object integrity by preventing unwanted access and modification of fields.
- **Q**: *Can you demonstrate encapsulation with an example?*  
  **A (Example)**:
  ```java
  public class Employee {
      private String name;
      private int id;

      public String getName() { return name; }
      public void setName(String name) { this.name = name; }
      public int getId() { return id; }
      public void setId(int id) { this.id = id; }
  }
  ```

---

## Exception Handling

### Checked vs Unchecked Exceptions
- **Checked Exceptions**: Must be either caught or declared (e.g., `IOException`).
- **Unchecked Exceptions**: Runtime exceptions (e.g., `NullPointerException`) that need not be declared.

### Custom Exceptions
- Create custom exceptions by extending `Exception` (for checked) or `RuntimeException` (for unchecked).

### Try-With-Resources
- Automatically closes resources that implement the `AutoCloseable` interface.
- **Example**:
  ```java
  try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
      System.out.println(br.readLine());
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```

### Assertions
- Used for testing assumptions in code.
- **Example**:
  ```java
  int value = 5;
  assert value > 0 : "Value must be positive";
  ```

### Exception Priority and Finally Block
- **Priority**: Catch most specific exceptions first.
- **Finally Block**: Always executes after try/catch, even if an exception is thrown.

### Twisted Interview Q&A for Exception Handling
- **Q**: *What is the difference between checked and unchecked exceptions?*  
  **A**: Checked exceptions require explicit handling, while unchecked exceptions do not.
- **Q**: *Can you provide an example where try-with-resources is preferable?*  
  **A (Example)**: See the try-with-resources example above.
- **Q**: *Explain the use of a finally block with an example.*  
  **A (Example)**:
  ```java
  try {
      // code that may throw an exception
  } catch(Exception e) {
      // exception handling
  } finally {
      System.out.println("Always executes regardless of an exception");
  }
  ```

---
