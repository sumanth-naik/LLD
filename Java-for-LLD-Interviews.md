# Java for LLD Interviews

A focused guide covering only the Java syntax and features essential for Low Level Design interviews. For developers with coding experience who are new to Java.

---

## Table of Contents

1. [OOP Fundamentals](#1-oop-fundamentals)
2. [Inheritance & Polymorphism](#2-inheritance--polymorphism)
3. [Collections Framework](#3-collections-framework)
4. [Generics](#4-generics)
5. [equals() and hashCode()](#5-equals-and-hashcode)
6. [Enums](#6-enums)
7. [Exceptions](#7-exceptions)
8. [Quick Reference](#8-quick-reference)

---

## 1. OOP Fundamentals

### Classes and Objects

```java
// Class definition
public class Car {
    // Instance variables (fields)
    private String brand;
    private int year;
    
    // Constructor
    public Car(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }
    
    // Getter methods
    public String getBrand() {
        return brand;
    }
    
    // Setter methods
    public void setBrand(String brand) {
        this.brand = brand;
    }
    
    // Instance method
    public void start() {
        System.out.println(brand + " started");
    }
}

// Creating objects
Car myCar = new Car("Toyota", 2023);
myCar.start();
```

### Access Modifiers

```java
public class Example {
    public int publicVar;        // Accessible everywhere
    protected int protectedVar;  // Accessible in same package + subclasses
    int packageVar;              // Package-private (default) - same package only
    private int privateVar;      // Only within this class
    
    // Same rules apply to methods
    public void publicMethod() {}
    private void privateMethod() {}
}
```

### Static vs Instance

```java
public class Counter {
    // Instance variable - each object has its own copy
    private int count;
    
    // Static variable - shared across all instances
    private static int totalCount;
    
    // Instance method - operates on instance data
    public void increment() {
        count++;
        totalCount++;
    }
    
    // Static method - can't access instance variables directly
    public static int getTotalCount() {
        return totalCount;
    }
    
    // Static method can be called without creating object
    // Counter.getTotalCount();
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
c1.increment();
c2.increment();
System.out.println(Counter.getTotalCount()); // 2
```

### Method Overloading

```java
public class Calculator {
    // Same method name, different parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
}
```

### this Keyword

```java
public class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;  // 'this' refers to current object
    }
    
    public void setName(String name) {
        this.name = name;  // Distinguish parameter from field
    }
}
```

---

## 2. Inheritance & Polymorphism

### extends (Inheritance)

```java
// Parent class
public class Animal {
    protected String name;  // protected allows subclass access
    
    public Animal(String name) {
        this.name = name;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

// Child class
public class Dog extends Animal {
    public Dog(String name) {
        super(name);  // Call parent constructor
    }
    
    // Method overriding
    @Override
    public void eat() {
        super.eat();  // Call parent method
        System.out.println("Dog is eating dog food");
    }
    
    // New method specific to Dog
    public void bark() {
        System.out.println(name + " is barking");
    }
}
```

### Abstract Classes

```java
// Cannot instantiate abstract class
public abstract class Shape {
    protected String color;
    
    // Abstract method - must be implemented by subclasses
    public abstract double area();
    
    // Concrete method - can have implementation
    public void setColor(String color) {
        this.color = color;
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    // Must implement abstract method
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

### Interfaces

```java
// Interface - defines contract
public interface Drawable {
    // All methods are public abstract by default
    void draw();
    
    // Default method (Java 8+) - has implementation
    default void printInfo() {
        System.out.println("This is drawable");
    }
    
    // Static method (Java 8+)
    static void utilityMethod() {
        System.out.println("Utility method");
    }
}

// Class implements interface
public class Rectangle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing rectangle");
    }
}

// Multiple interfaces allowed
public class Square implements Drawable, Resizable {
    // Must implement all methods from both interfaces
}
```

### Abstract Class vs Interface

**Use Abstract Class when:**
- You have shared code/state between related classes
- You want to provide default implementations
- You need constructors

**Use Interface when:**
- You want to define a contract
- Multiple unrelated classes need to implement same behavior
- You want multiple inheritance of type (Java supports multiple interfaces)

### Polymorphism

```java
// Parent reference, child object
Animal animal = new Dog("Buddy");
animal.eat();  // Calls Dog's eat() method (runtime polymorphism)

// Works with interfaces too
Drawable shape = new Rectangle();
shape.draw();  // Calls Rectangle's draw() method

// List of different types
List<Animal> animals = new ArrayList<>();
animals.add(new Dog("Buddy"));
animals.add(new Cat("Whiskers"));
for (Animal a : animals) {
    a.eat();  // Calls appropriate eat() method
}
```

---

## 3. Collections Framework

### List

```java
import java.util.*;

// ArrayList - dynamic array, fast random access
List<String> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
list.add(0, "Cherry");  // Insert at index
String first = list.get(0);
int size = list.size();
boolean hasApple = list.contains("Apple");
list.remove("Banana");
list.remove(0);

// LinkedList - fast insertion/deletion, slower access
List<Integer> linkedList = new LinkedList<>();
linkedList.add(10);
linkedList.add(20);

// Iteration
for (String item : list) {
    System.out.println(item);
}

// Or with index
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

### Set

```java
// HashSet - no duplicates, no order, O(1) operations
Set<String> hashSet = new HashSet<>();
hashSet.add("Apple");
hashSet.add("Banana");
hashSet.add("Apple");  // Duplicate, ignored
boolean contains = hashSet.contains("Apple");
hashSet.remove("Banana");

// LinkedHashSet - maintains insertion order
Set<String> linkedHashSet = new LinkedHashSet<>();

// TreeSet - sorted order, O(log n) operations
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(5);
treeSet.add(2);
treeSet.add(8);
// Elements: 2, 5, 8 (sorted)
```

### Map

```java
// HashMap - key-value pairs, no duplicates keys, no order
Map<String, Integer> map = new HashMap<>();
map.put("Apple", 10);
map.put("Banana", 20);
map.put("Apple", 15);  // Updates existing key
int count = map.get("Apple");  // 15
boolean hasKey = map.containsKey("Apple");
boolean hasValue = map.containsValue(20);
map.remove("Banana");

// Iterate over map
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Or iterate keys/values separately
for (String key : map.keySet()) {
    System.out.println(key);
}
for (Integer value : map.values()) {
    System.out.println(value);
}

// LinkedHashMap - maintains insertion order
Map<String, Integer> linkedMap = new LinkedHashMap<>();

// TreeMap - sorted by keys
Map<String, Integer> treeMap = new TreeMap<>();
```

### Queue

```java
// PriorityQueue - min-heap by default
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(5);  // Add element
pq.offer(2);
pq.offer(8);
int min = pq.poll();  // Removes and returns 2 (smallest)
int next = pq.peek();  // Returns 5 without removing

// Max-heap
Queue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// Deque - double-ended queue
Deque<Integer> deque = new ArrayDeque<>();
deque.addFirst(1);
deque.addLast(2);
int first = deque.removeFirst();
int last = deque.removeLast();
```

### When to Use Which Collection

- **ArrayList**: Need indexed access, frequent get/set operations
- **LinkedList**: Frequent insertions/deletions in middle, don't need random access
- **HashSet**: Need uniqueness, don't care about order
- **LinkedHashSet**: Need uniqueness + insertion order
- **TreeSet**: Need uniqueness + sorted order
- **HashMap**: Key-value pairs, don't care about order
- **LinkedHashMap**: Key-value pairs + insertion order
- **TreeMap**: Key-value pairs + sorted keys
- **PriorityQueue**: Need min/max element quickly

---

## 4. Generics

**Essential for LLD:** You'll use generics constantly with Collections. Understanding how to use them is critical.

### Using Generics with Collections

```java

// List with type parameter
List<String> list = new ArrayList<>();
list.add("Apple");
String item = list.get(0);  // Type-safe - no casting needed

// Map with type parameters
Map<String, Integer> map = new HashMap<>();
map.put("count", 10);
Integer value = map.get("count");  // Type-safe

// Set with type parameter
Set<Person> people = new HashSet<>();
people.add(new Person("John"));
// Type-safe iteration
for (Person person : people) {
    // person is already Person type, no casting
}

// Nested generics
Map<String, List<Integer>> scores = new HashMap<>();
List<Integer> studentScores = new ArrayList<>();
scores.put("John", studentScores);
```

**Key Points:**
- Always specify type parameters: `List<String>`, not `List`
- Type safety: No need to cast when retrieving elements
- Compile-time error checking: Prevents ClassCastException
- Diamond operator `<>`: `new ArrayList<>()` (Java 7+)

---

### ⚠️ Extra Info - Not Needed in LLD Interviews

The following topics are rarely needed in LLD interviews. You mainly need to **use** generics with Collections, not create your own generic classes.

#### Creating Generic Classes

```java
public class Box<T> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.setItem("Hello");
String value = stringBox.getItem();
```

#### Generic Methods

```java
// Generic method - occasionally useful
public class Utils {
    public static <T> void swap(List<T> list, int i, int j) {
        T temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }
    
    // Bounded type parameter
    public static <T extends Comparable<T>> T max(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}
```

#### Wildcards

```java
// Wildcards - almost never needed in LLD interviews
// ? extends - upper bound (read-only)
List<? extends Number> numbers = new ArrayList<Integer>();
Number num = numbers.get(0);  // OK

// ? super - lower bound (write-only)
List<? super Integer> integers = new ArrayList<Number>();
integers.add(5);  // OK

// Unbounded wildcard
List<?> list = new ArrayList<String>();
Object obj = list.get(0);  // OK
```

**Bottom Line:** For LLD interviews, focus on using `List<T>`, `Map<K,V>`, `Set<T>` correctly. You don't need to create generic classes or understand wildcards.

---

## 5. equals() and hashCode()

### The Contract

1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true
2. If `a.hashCode() == b.hashCode()`, `a.equals(b)` may or may not be true
3. `equals()` must be: reflexive, symmetric, transitive, consistent

### Implementation

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        // 1. Check if same reference
        if (this == obj) return true;
        
        // 2. Check if null or different class
        if (obj == null || getClass() != obj.getClass()) return false;
        
        // 3. Cast and compare fields
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        // Use Objects.hash() for multiple fields
        return Objects.hash(name, age);
    }
}

// Usage
Person p1 = new Person("John", 30);
Person p2 = new Person("John", 30);
System.out.println(p1.equals(p2));  // true
System.out.println(p1.hashCode() == p2.hashCode());  // true
```

### Why It Matters for Collections

```java
// HashMap uses hashCode() to find bucket, equals() to find exact match
Map<Person, String> map = new HashMap<>();
Person p1 = new Person("John", 30);
Person p2 = new Person("John", 30);

map.put(p1, "Engineer");
String value = map.get(p2);  // Returns "Engineer" only if equals() and hashCode() are correct

// HashSet also uses both
Set<Person> set = new HashSet<>();
set.add(p1);
boolean contains = set.contains(p2);  // true only if equals() and hashCode() are correct
```

### Common Mistakes

```java
// WRONG - doesn't override hashCode()
@Override
public boolean equals(Object obj) {
    // implementation
}
// This breaks the contract!

// WRONG - uses == instead of equals() for objects
if (name == person.name)  // Should use Objects.equals(name, person.name)

// CORRECT - use Objects.equals() for null-safe comparison
Objects.equals(name, person.name)
```

---

## 6. Enums

### Basic Enum

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Usage
Day today = Day.MONDAY;
if (today == Day.MONDAY) {
    System.out.println("Start of week");
}

// Switch statement
switch (today) {
    case MONDAY:
        System.out.println("Monday");
        break;
    case FRIDAY:
        System.out.println("Friday");
        break;
    default:
        System.out.println("Other day");
}
```

### Enum with Values and Methods [Optional for LLD]

```java
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6);
    
    private final double mass;   // in kilograms
    private final double radius; // in meters
    
    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
    
    // Methods
    public double getMass() {
        return mass;
    }
    
    public double getRadius() {
        return radius;
    }
    
    public double surfaceGravity() {
        double G = 6.67300E-11;
        return G * mass / (radius * radius);
    }
}

// Usage
Planet earth = Planet.EARTH;
double gravity = earth.surfaceGravity();
```

### Enum with State Machine Pattern [Optional for LLD]

```java
public enum OrderStatus {
    PENDING {
        @Override
        public OrderStatus next() {
            return CONFIRMED;
        }
    },
    CONFIRMED {
        @Override
        public OrderStatus next() {
            return SHIPPED;
        }
    },
    SHIPPED {
        @Override
        public OrderStatus next() {
            return DELIVERED;
        }
    },
    DELIVERED {
        @Override
        public OrderStatus next() {
            return this;  // Final state
        }
    };
    
    public abstract OrderStatus next();
}

// Usage
OrderStatus status = OrderStatus.PENDING;
status = status.next();  // CONFIRMED
```

---

## 7. Exceptions

### Checked vs Unchecked

```java
// Checked Exception - must be handled or declared
// [Optional for LLD] File I/O example - file operations are rarely needed in LLD
public void readFile() throws IOException {
    // Code that might throw IOException
    throw new IOException("File not found");
}

// Unchecked Exception (RuntimeException) - optional to handle
public void divide(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("Cannot divide by zero");
    }
    // No throws declaration needed
}
```

### Try-Catch-Finally

```java
// Basic try-catch
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Error: " + e.getMessage());
}

// Multiple catch blocks
// [Optional for LLD] File I/O exceptions - file operations are rarely needed in LLD
try {
    // code
} catch (FileNotFoundException e) {
    // handle file not found
} catch (IOException e) {
    // handle other IO errors
}

// Finally block - always executes
try {
    // code
} catch (Exception e) {
    // handle exception
} finally {
    // cleanup code - always runs
    // Close resources, etc.
}

// Try-with-resources (Java 7+) - auto-closes resources
// [Optional for LLD] File I/O example - file operations are rarely needed in LLD
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} catch (IOException e) {
    // handle exception
} // br automatically closed here
```

### Throwing Exceptions

```java
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }
    if (age > 150) {
        throw new IllegalArgumentException("Age cannot exceed 150");
    }
}

// Custom exception [Optional for LLD]
// You can usually use built-in exceptions like IllegalArgumentException, IllegalStateException
public class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message);
    }
}

public void withdraw(double amount) throws InsufficientFundsException {
    if (amount > balance) {
        throw new InsufficientFundsException("Insufficient funds");
    }
    balance -= amount;
}
```

### Exception Hierarchy

```
                          Throwable
                         /          \
                        /            \
                 Exception            Error (unchecked)
                /          \              |
               /            \             |
    RuntimeException      Checked         |
    (unchecked)        Exceptions         |
         |                  |             |
         |                  |             |
    ┌────┴─────┐      ┌─────┴─────┐      |
    |          |      |           |      |
NullPointer  Illegal  IOException  SQL   OutOfMemoryError
Exception    Argument              Exception  StackOverflowError
             Exception
IndexOutOf
BoundsException
```

**Visual Representation:**
- **Throwable** (root)
  - **Exception** (checked by default)
    - **RuntimeException** (unchecked) ← Most common in LLD
      - `NullPointerException`
      - `IllegalArgumentException`
      - `IndexOutOfBoundsException`
    - **Checked Exceptions** (must handle. Else, Compile time errors)
      - `IOException`
      - `FileNotFoundException`
      - `SQLException`
  - **Error** (unchecked, serious)
    - `OutOfMemoryError`
    - `StackOverflowError`

**Key Points:**
- **Throwable**: Base class for all errors and exceptions
- **Error**: Serious problems (unchecked) - usually not recoverable
- **Exception**: Problems that can be caught and handled
  - **RuntimeException** (unchecked): Programming errors - don't need to declare `throws`
  - **Checked Exceptions**: Must be handled or declared with `throws`

### Common Exception Types

- **RuntimeException** (unchecked): `NullPointerException`, `IllegalArgumentException`, `IndexOutOfBoundsException` ✅ **Essential for LLD**
- **Checked**: `IOException`, `FileNotFoundException`, `SQLException` [Optional for LLD - file/database operations rarely needed]
- **Error** (unchecked): `OutOfMemoryError`, `StackOverflowError` [Optional for LLD - rarely thrown in interview code]

---

## 8. Quick Reference

### Syntax Cheat Sheet

```java
// Variable declaration
int number = 10;
String text = "Hello";
boolean flag = true;
List<String> list = new ArrayList<>();

// Arrays
int[] arr = new int[10];
int[] arr2 = {1, 2, 3, 4, 5};
String[] words = {"apple", "banana"};

// Conditional
if (condition) {
    // code
} else if (condition2) {
    // code
} else {
    // code
}

// Ternary operator
int max = (a > b) ? a : b;

// Switch (Java 14+)
switch (value) {
    case 1 -> System.out.println("One");
    case 2 -> System.out.println("Two");
    default -> System.out.println("Other");
}

// Loops
for (int i = 0; i < 10; i++) {
    // code
}

for (String item : list) {
    // code
}

while (condition) {
    // code
}

// String operations
String str = "Hello";
int len = str.length();
String sub = str.substring(0, 3);  // "Hel"
boolean contains = str.contains("ell");
String upper = str.toUpperCase();
String[] parts = str.split(" ");

// StringBuilder (mutable string)
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" World");
String result = sb.toString();
```

### Common Operations

```java
// List operations
List<String> list = new ArrayList<>();
list.add("item");
list.add(0, "first");
list.get(0);
list.set(0, "new");
list.remove(0);
list.remove("item");
list.size();
list.isEmpty();
list.contains("item");
list.clear();
Collections.sort(list);

// Map operations
Map<String, Integer> map = new HashMap<>();
map.put("key", 10);
map.get("key");
map.getOrDefault("key", 0);  // Returns 0 if key not found
map.containsKey("key");
map.containsValue(10);
map.remove("key");
map.size();
map.isEmpty();
map.clear();

// Set operations
Set<String> set = new HashSet<>();
set.add("item");
set.remove("item");
set.contains("item");
set.size();
set.isEmpty();

// String comparison
String s1 = "hello";
String s2 = "hello";
s1.equals(s2);  // true - compares content
s1 == s2;  // might be true (string pool) but don't rely on it
s1.equalsIgnoreCase("HELLO");  // true

// Object comparison
Objects.equals(obj1, obj2);  // null-safe equals
Objects.isNull(obj);
Objects.nonNull(obj);

// Math operations
Math.max(a, b);
Math.min(a, b);
Math.abs(x);
Math.pow(base, exponent);
Math.sqrt(x);
Math.round(x);

// Array operations
int[] arr = {1, 2, 3};
int len = arr.length;
Arrays.sort(arr);
Arrays.asList(arr);  // Convert array to list
```

### Type Conversions

```java
// String to int
int num = Integer.parseInt("123");
int num2 = Integer.valueOf("123");

// int to String
String str = String.valueOf(123);
String str2 = Integer.toString(123);
String str3 = "" + 123;

// String to double
double d = Double.parseDouble("3.14");

// Object casting
Object obj = "Hello";
String str = (String) obj;  // Explicit cast

// instanceof check
if (obj instanceof String) {
    String str = (String) obj;
}
```

### Useful Utility Classes

```java
import java.util.*;

// Collections utility
Collections.sort(list);
Collections.reverse(list);
Collections.max(list);
Collections.min(list);
Collections.shuffle(list);

// Arrays utility
Arrays.sort(arr);
Arrays.binarySearch(arr, key);
Arrays.fill(arr, value);
Arrays.toString(arr);

// Objects utility
Objects.equals(a, b);
Objects.hash(a, b, c);
Objects.requireNonNull(obj);
```

---

## Key Takeaways for LLD Interviews

1. **Use appropriate collections**: Choose based on access patterns (List vs Set vs Map)
2. **Implement equals() and hashCode()**: Critical when using objects as keys in HashMap or elements in HashSet
3. **Understand polymorphism**: Use interfaces/abstract classes for flexibility
4. **Use generics with Collections**: Always use `List<T>`, `Map<K,V>`, `Set<T>` for type safety (you don't need to create generic classes)
5. **Handle exceptions properly**: Know when to use checked vs unchecked
6. **Enums for constants**: Better than magic strings/numbers
7. **Static vs instance**: Know when to use each

---

*This guide covers the essential Java syntax needed for LLD interviews. For design patterns and best practices, refer to other resources in this repository.*

