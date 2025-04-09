# Java Stream API Interview Questions and Answers

This document contains a collection of interview questions based on real-world scenarios where the Java Stream API is used. Each question includes a problem description, a sample code solution, an explanation, and the expected output when the code is executed.

---

## Basic and Intermediate Stream API Questions

### Question 1: Filtering a List of Integers

**Scenario:**  
Given a list of integers, filter out only the even numbers.

**Answer (Code):**
```java
import java.util.List;
import java.util.stream.Collectors;

public class FilterEvenNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        List<Integer> evenNumbers = numbers.stream()
                                           .filter(n -> n % 2 == 0)
                                           .collect(Collectors.toList());
        System.out.println("Even numbers: " + evenNumbers);
    }
}
```

**Explanation:**  
This example uses the `filter()` method to retain only numbers divisible by 2 and then collects the result into a list.

**Expected Output:**
```
Even numbers: [2, 4, 6, 8, 10]
```

---

### Question 2: Transforming a List with Map

**Scenario:**  
Convert a list of strings to uppercase using the Stream API.

**Answer (Code):**
```java
import java.util.List;
import java.util.stream.Collectors;

public class UpperCaseTransformation {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry");
        List<String> upperCaseWords = words.stream()
                                           .map(String::toUpperCase)
                                           .collect(Collectors.toList());
        System.out.println("Uppercase words: " + upperCaseWords);
    }
}
```

**Explanation:**  
The `map()` method transforms each string to uppercase using a method reference (`String::toUpperCase`).

**Expected Output:**
```
Uppercase words: [APPLE, BANANA, CHERRY]
```

---

### Question 3: Flattening a List of Lists

**Scenario:**  
Given a list of lists of integers, flatten it into a single list.

**Answer (Code):**
```java
import java.util.List;
import java.util.stream.Collectors;

public class FlattenNestedLists {
    public static void main(String[] args) {
        List<List<Integer>> nestedNumbers = List.of(
            List.of(1, 2, 3),
            List.of(4, 5),
            List.of(6, 7, 8, 9)
        );
        List<Integer> flattened = nestedNumbers.stream()
                                                 .flatMap(List::stream)
                                                 .collect(Collectors.toList());
        System.out.println("Flattened list: " + flattened);
    }
}
```

**Explanation:**  
Each sub-list is converted into a stream by `flatMap()`, which then flattens them into a single stream of integers.

**Expected Output:**
```
Flattened list: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

### Question 4: Aggregating Values with Reduce

**Scenario:**  
Compute the sum of a list of integers using the Stream API.

**Answer (Code):**
```java
import java.util.List;

public class SumUsingReduce {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);
        int sum = numbers.stream()
                         .reduce(0, Integer::sum);
        System.out.println("Sum: " + sum);
    }
}
```

**Explanation:**  
The `reduce()` method aggregates the elements by summing them up, starting with an initial value of 0.

**Expected Output:**
```
Sum: 15
```

---

### Question 5: Grouping Elements by a Property

**Scenario:**  
Given a list of people, group them by age using the Stream API.

**Answer (Code):**
```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public int getAge() {
        return age;
    }
    
    @Override
    public String toString() {
        return name;
    }
}

public class GroupByAge {
    public static void main(String[] args) {
        List<Person> people = List.of(
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 30),
            new Person("David", 25)
        );
        Map<Integer, List<Person>> groupedByAge = people.stream()
                                                        .collect(Collectors.groupingBy(Person::getAge));
        System.out.println("Grouped by age: " + groupedByAge);
    }
}
```

**Explanation:**  
The `Collectors.groupingBy()` collector groups persons by their age, mapping each age value to the list of people with that age.

**Expected Output:**  
*Note: The order of keys in a map is not guaranteed, but one common output is:*
```
Grouped by age: {25=[Bob, David], 30=[Alice, Charlie]}
```

---

### Question 6: Sorting Elements

**Scenario:**  
Sort a list of strings in alphabetical order using streams.

**Answer (Code):**
```java
import java.util.List;
import java.util.stream.Collectors;

public class SortStrings {
    public static void main(String[] args) {
        List<String> fruits = List.of("Banana", "Apple", "Cherry", "Mango");
        List<String> sortedFruits = fruits.stream()
                                           .sorted()
                                           .collect(Collectors.toList());
        System.out.println("Sorted fruits: " + sortedFruits);
    }
}
```

**Explanation:**  
The `sorted()` method sorts the elements in their natural order (alphabetically for strings).

**Expected Output:**
```
Sorted fruits: [Apple, Banana, Cherry, Mango]
```

---

### Question 7: Merging Two Streams

**Scenario:**  
Combine two streams of integers into one stream and collect the result.

**Answer (Code):**
```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class MergeStreams {
    public static void main(String[] args) {
        Stream<Integer> stream1 = Stream.of(1, 2, 3);
        Stream<Integer> stream2 = Stream.of(4, 5, 6);
        List<Integer> mergedList = Stream.concat(stream1, stream2)
                                         .collect(Collectors.toList());
        System.out.println("Merged List: " + mergedList);
    }
}
```

**Explanation:**  
`Stream.concat()` merges two streams into one, which is then collected into a list.

**Expected Output:**
```
Merged List: [1, 2, 3, 4, 5, 6]
```

---

### Question 8: Finding Maximum and Minimum Values

**Scenario:**  
Find the maximum and minimum values from a list of integers using streams.

**Answer (Code):**
```java
import java.util.List;
import java.util.Optional;

public class FindMaxMin {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(5, 3, 9, 1, 4);
        Optional<Integer> max = numbers.stream().max(Integer::compare);
        Optional<Integer> min = numbers.stream().min(Integer::compare);
        
        max.ifPresent(m -> System.out.println("Max: " + m));
        min.ifPresent(m -> System.out.println("Min: " + m));
    }
}
```

**Explanation:**  
The `max()` and `min()` methods use a comparator (here, `Integer::compare`) to determine the largest and smallest elements, respectively.

**Expected Output:**
```
Max: 9
Min: 1
```

---

### Question 9: Parallel Stream Processing

**Scenario:**  
Process a large list of numbers in parallel to determine how many are prime numbers.

**Answer (Code):**
```java
import java.util.List;

public class ParallelPrimeCount {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(101, 103, 107, 109, 113, 127, 131, 137);
        long primeCount = numbers.parallelStream()
                                 .filter(ParallelPrimeCount::isPrime)
                                 .count();
        System.out.println("Number of primes: " + primeCount);
    }
    
    private static boolean isPrime(int number) {
        if (number < 2) return false;
        for (int i = 2; i <= Math.sqrt(number); i++) {
            if (number % i == 0) return false;
        }
        return true;
    }
}
```

**Explanation:**  
This example uses a parallel stream to concurrently filter out prime numbers and then counts them. Parallel streams can improve performance on multi-core systems for compute-intensive tasks.

**Expected Output:**
```
Number of primes: 8
```

---

### Question 10: Using Optional with Streams

**Scenario:**  
Find the first string that starts with the letter "A" from a list and return it using an Optional.

**Answer (Code):**
```java
import java.util.List;
import java.util.Optional;

public class FindFirstA {
    public static void main(String[] args) {
        List<String> names = List.of("Bob", "Alice", "Andrew", "Charlie");
        Optional<String> firstA = names.stream()
                                       .filter(name -> name.startsWith("A"))
                                       .findFirst();
        firstA.ifPresentOrElse(
            name -> System.out.println("First name starting with A: " + name),
            () -> System.out.println("No name starting with A found")
        );
    }
}
```

**Explanation:**  
After filtering names that start with "A," `findFirst()` retrieves the first occurrence. The result is safely handled with an `Optional`.

**Expected Output:**
```
First name starting with A: Alice
```

---

## Advanced Stream API Interview Questions

### Question 11: Creating a Custom Collector for Summary Statistics

**Scenario:**  
Write a custom collector that calculates summary statistics (count, sum, and average) for a stream of integers.

**Answer (Code):**
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collector;

class IntSummary {
    int count;
    int sum;
    
    public void accept(int value) {
        count++;
        sum += value;
    }
    
    public IntSummary combine(IntSummary other) {
        this.count += other.count;
        this.sum += other.sum;
        return this;
    }
    
    public double getAverage() {
        return count == 0 ? 0 : (double) sum / count;
    }
    
    @Override
    public String toString() {
        return "IntSummary[count=" + count + ", sum=" + sum + ", average=" + getAverage() + "]";
    }
}

public class CustomCollectorExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(3, 5, 7, 9, 11);
        IntSummary summary = numbers.stream()
            .collect(Collector.of(
                IntSummary::new,             // supplier
                IntSummary::accept,          // accumulator
                IntSummary::combine          // combiner
            ));
        System.out.println("Summary Statistics: " + summary);
    }
}
```

**Explanation:**  
A custom collector is built using `Collector.of()`. The supplier creates a new `IntSummary` instance, the accumulator adds each number, and the combiner merges two summary objects. Finally, the `toString()` method shows count, sum, and average.

**Expected Output:**
```
Summary Statistics: IntSummary[count=5, sum=35, average=7.0]
```

---

### Question 12: Partitioning Data with Downstream Collectors

**Scenario:**  
Given a list of integers, partition them into even and odd numbers and count the elements in each partition.

**Answer (Code):**
```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class PartitionByEvenOdd {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        Map<Boolean, Long> partitioned = numbers.stream()
            .collect(Collectors.partitioningBy(
                n -> n % 2 == 0, 
                Collectors.counting()
            ));
        System.out.println("Partitioned Count (Even/Not Even): " + partitioned);
    }
}
```

**Explanation:**  
`Collectors.partitioningBy()` is used with a downstream collector (`Collectors.counting()`) to partition numbers into even (`true`) and odd (`false`), then count each group.

**Expected Output:**
```
Partitioned Count (Even/Not Even): {false=5, true=5}
```

---

### Question 13: Handling Exceptions within Stream Operations

**Scenario:**  
Given a list of file paths (as strings), map each to its file size. Some files might not exist, so handle exceptions gracefully within the stream.

**Answer (Code):**
```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;
import java.util.stream.Collectors;

public class FileSizeStream {
    public static void main(String[] args) {
        List<String> filePaths = List.of("file1.txt", "file2.txt", "file3.txt");
        List<Long> fileSizes = filePaths.stream()
            .map(FileSizeStream::getFileSizeSafe)
            .collect(Collectors.toList());
        System.out.println("File sizes: " + fileSizes);
    }

    private static long getFileSizeSafe(String path) {
        try {
            return Files.size(Path.of(path));
        } catch (IOException e) {
            // Log or handle the exception appropriately
            return 0L;
        }
    }
}
```

**Explanation:**  
A helper method `getFileSizeSafe` wraps the checked exception within a try-catch block, allowing the stream to continue processing and returning `0L` when an exception occurs.

**Expected Output:**  
*Assuming none of the files exist on the file system, the output will be:*
```
File sizes: [0, 0, 0]
```

---

### Question 14: Demonstrating Lazy Evaluation and Short-circuiting

**Scenario:**  
Show that stream operations are lazily evaluated by using `peek()` to log processing. Find the first name that starts with "C" from a list of names.

**Answer (Code):**
```java
import java.util.List;

public class LazyEvaluationDemo {
    public static void main(String[] args) {
        List<String> names = List.of("Alice", "Bob", "Charlie", "David", "Catherine");
        String result = names.stream()
            .peek(name -> System.out.println("Processing: " + name))
            .filter(name -> name.startsWith("C"))
            .findFirst()
            .orElse("None");
        System.out.println("First matching name: " + result);
    }
}
```

**Explanation:**  
The `peek()` method logs each element processed. With the short-circuiting operation `findFirst()`, only the necessary elements are processed until a name starting with "C" is found.

**Expected Output:**
```
Processing: Alice
Processing: Bob
Processing: Charlie
First matching name: Charlie
```

---
