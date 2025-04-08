# Builder Design Pattern in Java

The Builder design pattern is a creational pattern that allows you to create complex objects step by step. It separates the construction of an object from its representation so that the same construction process can create different representations. This pattern is particularly useful when an object needs to be immutable or when there are many optional parameters.

---

## 1. Overview

### 1.1 What is the Builder Pattern?

- **Definition:**  
  The Builder pattern uses a separate Builder object to encapsulate the construction of a complex object. The client code uses the Builder to set various properties before finally “building” the object.

- **Key Characteristics:**  
  - **Separation of Concerns:** Separates object creation from its representation.  
  - **Fluent Interface:** Often uses a fluent API, meaning method calls can be chained for better readability.  
  - **Immutability:** Supports the construction of immutable objects.  
  - **Step-by-Step Construction:** Enables incremental setup of various object properties.

### 1.2 Where is it Implemented in Java?

- **Java Standard Library Examples:**  
  - **`StringBuilder`:** Even though it’s mutable, it demonstrates the step-by-step approach in construction.  
  - **`ProcessBuilder`:** Used to create and configure operating system processes.  
  - **`Stream.Builder`:** Introduced in Java 8 for creating streams.

- **Frameworks and Libraries:**  
  - **Lombok:** Provides the `@Builder` annotation to automatically generate builder code.
  - Other libraries like Apache HttpClient often implement builders for constructing HTTP requests.

---

## 2. Real-World Use Case

Consider a **Pizza Ordering System** where there are many options: crust type, size, cheese, sauce, vegetables, and extras. Without a Builder, constructors can become complicated, error-prone, and hard to maintain. The Builder pattern helps simplify the creation of such complex objects.

### 2.1 Example: Building a Pizza Object

```java
public class Pizza {
    // Required parameters
    private final String size;
    private final String crust;
    
    // Optional parameters
    private final boolean cheese;
    private final boolean pepperoni;
    private final boolean bacon;
    private final boolean extraSauce;

    // Private constructor accessible only via the builder.
    private Pizza(Builder builder) {
        this.size = builder.size;
        this.crust = builder.crust;
        this.cheese = builder.cheese;
        this.pepperoni = builder.pepperoni;
        this.bacon = builder.bacon;
        this.extraSauce = builder.extraSauce;
    }

    public static class Builder {
        // Required parameters
        private final String size;
        private final String crust;
        
        // Optional parameters with default values
        private boolean cheese = false;
        private boolean pepperoni = false;
        private boolean bacon = false;
        private boolean extraSauce = false;
        
        public Builder(String size, String crust) {
            this.size = size;
            this.crust = crust;
        }
        
        public Builder cheese(boolean value) {
            this.cheese = value;
            return this;
        }
        
        public Builder pepperoni(boolean value) {
            this.pepperoni = value;
            return this;
        }
        
        public Builder bacon(boolean value) {
            this.bacon = value;
            return this;
        }
        
        public Builder extraSauce(boolean value) {
            this.extraSauce = value;
            return this;
        }
        
        public Pizza build() {
            return new Pizza(this);
        }
    }

    @Override
    public String toString() {
        return "Pizza{" +
                "size='" + size + '\'' +
                ", crust='" + crust + '\'' +
                ", cheese=" + cheese +
                ", pepperoni=" + pepperoni +
                ", bacon=" + bacon +
                ", extraSauce=" + extraSauce +
                '}';
    }
}
```

**Usage Example:**

```java
public class PizzaOrder {
    public static void main(String[] args) {
        Pizza myPizza = new Pizza.Builder("Large", "Thin")
                .cheese(true)
                .pepperoni(true)
                .extraSauce(true)
                .build();

        System.out.println(myPizza);
    }
}
```

**Explanation:**
- **Immutable Object:** Once built, the `Pizza` object remains unchanged.  
- **Readable Code:** The fluent API clearly expresses how the pizza is configured.  
- **Optional Parameters:** Only the required parameters are enforced; other parameters can be set as needed.

---

## 3. When to Use the Builder Pattern

### 3.1 Use Cases

- **Complex Object Construction:** Use when creating an object with many parameters, especially when some are optional or have default values.
- **Immutable Objects:** When you need the final object to be immutable.
- **Improving Readability:** Avoid telescoping constructors and enhance code maintainability.
- **Step-by-Step Configuration:** Situations where objects are constructed in multiple phases.

### 3.2 Advantages

- **Enhanced Clarity:** Constructs code that is clean, readable, and self-documenting.
- **Flexibility:** Easily add, remove, or modify optional parameters.
- **Separation:** Isolates the construction logic in a dedicated builder class.
- **Support for Immutability:** Promotes the creation of immutable objects that are thread-safe.

### 3.3 Limitations and Twisted Cases

- **Increased Boilerplate:** For simple objects, the Builder pattern may be overkill.
- **Inheritance Challenges:** Using builders with class hierarchies requires extra care (e.g., self-referential generics).
- **Concurrency:** Builder instances, being mutable, are not thread-safe unless guarded.
- **Debugging Difficulties:** Complex fluent method chains can make debugging tougher if a step in the chain fails.

---

## 4. Advanced Considerations

### 4.1 Handling Inheritance

When subclasses require their own builder, you can use generics with recursive type parameters (the self-type pattern):

```java
public abstract class Vehicle {
    protected String make;
    protected String model;

    protected Vehicle(Builder<?> builder) {
        this.make = builder.make;
        this.model = builder.model;
    }

    abstract static class Builder<T extends Builder<T>> {
        private String make;
        private String model;
        
        public T make(String make) {
            this.make = make;
            return self();
        }
        
        public T model(String model) {
            this.model = model;
            return self();
        }
        
        protected abstract T self();
        public abstract Vehicle build();
    }
}

public class Car extends Vehicle {
    private final int seats;
    
    private Car(Builder builder) {
        super(builder);
        this.seats = builder.seats;
    }
    
    public static class Builder extends Vehicle.Builder<Builder> {
        private int seats;
        
        public Builder seats(int seats) {
            this.seats = seats;
            return self();
        }
        
        @Override
        protected Builder self() {
            return this;
        }
        
        @Override
        public Car build() {
            return new Car(this);
        }
    }
}

public class BuilderInheritanceDemo {
    public static void main(String[] args) {
        Car car = new Car.Builder()
                .make("Toyota")
                .model("Corolla")
                .seats(5)
                .build();

        System.out.println("Car built: " + car);
    }
}
```

### 4.2 Handling Optional Dependencies and Defaults

The builder pattern handles default values well; however, ensure that you document which values are defaults and validate the state in the `build()` method.

### 4.3 Fluent API Considerations

While fluent APIs enhance readability, overusing method chaining can complicate debugging. Always perform appropriate validations in the `build()` method to catch errors early.

### 4.4 Concurrency Considerations

Typically, builder instances are not shared between threads. Once the object is built and immutable, it is safe for concurrent use. Consider thread confinement or synchronization if builder sharing is unavoidable.

---

## 5. Using Lombok to Implement the Builder Pattern

[Lombok](https://projectlombok.org/) is a popular library that reduces boilerplate in Java by generating code at compile time. Lombok’s `@Builder` annotation can be used to automatically create a builder for a class, dramatically simplifying the implementation.

### 5.1 Basic Example

With Lombok, you simply annotate your class with `@Builder`, and Lombok generates the corresponding builder code for you. For example:

```java
import lombok.Builder;
import lombok.ToString;

@Builder
@ToString
public class Employee {
    private final int id;
    private final String firstName;
    private final String lastName;
    private final String department;
    private final double salary;
}
```

**Usage:**

```java
public class EmployeeTest {
    public static void main(String[] args) {
        Employee employee = Employee.builder()
                .id(101)
                .firstName("John")
                .lastName("Doe")
                .department("Engineering")
                .salary(75000.00)
                .build();

        System.out.println(employee);
    }
}
```

### 5.2 Customizing the Lombok Builder

Lombok also allows for customization. For example, you can specify default values or create a builder for inner classes:

```java
import lombok.Builder;
import lombok.Getter;
import lombok.ToString;

@Getter
@ToString
@Builder
public class Order {
    private final int orderId;
    private final String product;
    private final int quantity;
    @Builder.Default
    private final String status = "NEW";
}
```

**Usage:**

```java
public class OrderTest {
    public static void main(String[] args) {
        Order order = Order.builder()
                .orderId(5001)
                .product("Laptop")
                .quantity(2)
                .build();

        System.out.println(order);  // status will print "NEW"
    }
}
```

Lombok’s `@Builder` annotation automatically takes care of creating a fluent API for object construction. This greatly reduces boilerplate code while ensuring that your design benefits—such as immutability and clarity—are maintained.

---

## 6. Conclusion

The Builder design pattern is an effective solution to the problem of constructing complex and immutable objects with many parameters. It enhances code readability, supports step-by-step construction, and reduces the likelihood of errors due to telescoping constructors.

**Key Takeaways:**
- **When to Use:**  
  Use the Builder pattern when object construction is complex or requires several optional parameters, or when you need to enforce immutability.
  
- **Real-World Examples:**  
  Widely employed in Java's standard libraries (e.g., `ProcessBuilder`, `Stream.Builder`) and in modern frameworks.

- **Lombok Integration:**  
  With Lombok’s `@Builder` annotation, you can implement the Builder pattern with minimal boilerplate, making your code cleaner and more maintainable.

By understanding and applying the Builder pattern—including the ease of integrating Lombok—you can create robust, readable, and maintainable Java applications that handle complex object construction scenarios effectively.
