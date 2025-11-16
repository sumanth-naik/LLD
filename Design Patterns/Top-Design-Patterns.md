# Top Design Patterns for LLD Interviews

Progressive refactoring examples showing bad code improved with each design pattern.

---

## Table of Contents

1. [Singleton Pattern](#1-singleton-pattern)
2. [Factory Pattern](#2-factory-pattern)
3. [Builder Pattern](#3-builder-pattern)
4. [Strategy Pattern](#4-strategy-pattern)
5. [Observer Pattern](#5-observer-pattern)
6. [Decorator Pattern](#6-decorator-pattern)
7. [Adapter Pattern](#7-adapter-pattern)
8. [Command Pattern](#8-command-pattern)
9. [Template Method Pattern](#9-template-method-pattern)
10. [State Pattern](#10-state-pattern)
11. [Composite Pattern](#11-composite-pattern)
12. [Chain of Responsibility Pattern](#12-chain-of-responsibility-pattern)
13. [Abstract Factory Pattern](#13-abstract-factory-pattern)
14. [Facade Pattern](#14-facade-pattern)
15. [Proxy Pattern](#15-proxy-pattern)
16. [Iterator Pattern](#16-iterator-pattern)
17. [Mediator Pattern](#17-mediator-pattern)

**Creational Patterns:**
- **Singleton**: Configuration, Logger, Cache
- **Factory**: Object creation based on type
- **Builder**: Complex object construction
- **Abstract Factory**: Families of related objects

**Structural Patterns:**
- **Adapter**: Integrate incompatible interfaces
- **Decorator**: Add behavior dynamically
- **Facade**: Simplify complex subsystem
- **Proxy**: Control access, lazy loading
- **Composite**: Part-whole hierarchies

**Behavioral Patterns:**
- **Strategy**: Interchangeable algorithms
- **Observer**: One-to-many dependency
- **State**: Behavior changes with state
- **Command**: Encapsulate requests
- **Chain of Responsibility**: Request handling chain
- **Template Method**: Algorithm skeleton
- **Mediator**: Reduce coupling between objects
- **Iterator**: Traverse collections


---

## 1. Singleton Pattern

**Problem:** Need exactly one instance of a class (e.g., database connection, configuration manager, logger).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         DatabaseConnection                  │
├─────────────────────────────────────────────┤
│ - String connectionString                   │
├─────────────────────────────────────────────┤
│ + DatabaseConnection()                      │
│ + connect(): void                           │
└─────────────────────────────────────────────┘

❌ Problems:
- Public constructor allows multiple instances
- No instance control
- Resource waste
- Inconsistent state possible

Client Code:
  DatabaseConnection db1 = new DatabaseConnection();
  DatabaseConnection db2 = new DatabaseConnection();
  // ❌ Two different instances created!
```

**Code:**

```java
// Multiple instances can be created - wasteful and inconsistent
public class DatabaseConnection {
    private String connectionString;
    
    public DatabaseConnection() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Creating new database connection...");
        // Expensive operation
    }
    
    public void connect() {
        System.out.println("Connected to: " + connectionString);
    }
}

// Usage - creates multiple instances
DatabaseConnection db1 = new DatabaseConnection();
DatabaseConnection db2 = new DatabaseConnection(); // Another connection!
// Problem: Multiple connections, resource waste, inconsistent state
```

### ✅ Singleton Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         DatabaseConnection                  │
├─────────────────────────────────────────────┤
│ - static final DatabaseConnection instance  │
│ - String connectionString                   │
├─────────────────────────────────────────────┤
│ - DatabaseConnection()                      │
│   (private constructor)                     │
│ + static getInstance(): DatabaseConnection  │
│ + connect(): void                           │
└─────────────────────────────────────────────┘

✅ Benefits:
- Private constructor prevents external instantiation
- Static instance ensures single object
- getInstance() provides global access point
- Thread-safe (eager initialization)

Client Code:
  DatabaseConnection db1 = DatabaseConnection.getInstance();
  DatabaseConnection db2 = DatabaseConnection.getInstance();
  // ✅ db1 == db2 (same instance)
```

**Code:**

```java
// ✅ Eager initialization (thread-safe)
public class DatabaseConnection {
    private static final DatabaseConnection instance = new DatabaseConnection();
    private String connectionString;
    
    // Private constructor - prevents external instantiation
    private DatabaseConnection() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Creating database connection (singleton)...");
    }
    
    // Public method to get instance
    public static DatabaseConnection getInstance() {
        return instance;
    }
    
    public void connect() {
        System.out.println("Connected to: " + connectionString);
    }
}

// ⚠️ Lazy initialization - NON-THREAD-SAFE (common mistake!)
public class DatabaseConnectionLazyUnsafe {
    private static DatabaseConnectionLazyUnsafe instance;
    private String connectionString;
    
    private DatabaseConnectionLazyUnsafe() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Creating database connection (lazy singleton)...");
    }
    
    // ❌ NOT THREAD-SAFE - multiple threads can create multiple instances!
    public static DatabaseConnectionLazyUnsafe getInstance() {
        if (instance == null) {
            // Problem: Two threads can both pass this check and create instances
            instance = new DatabaseConnectionLazyUnsafe();
        }
        return instance;
    }
    
    public void connect() {
        System.out.println("Connected to: " + connectionString);
    }
}

**UML Class Diagram - Lazy Initialization (Double-Checked Locking):**

```
┌─────────────────────────────────────────────┐
│      DatabaseConnectionLazy                 │
├─────────────────────────────────────────────┤
│ - static volatile DatabaseConnectionLazy    │
│   instance                                  │
│ - String connectionString                   │
├─────────────────────────────────────────────┤
│ - DatabaseConnectionLazy()                  │
│   (private constructor)                     │
│ + static getInstance():                     │
│   DatabaseConnectionLazy                    │
│   (double-checked locking)                  │
│ + connect(): void                           │
└─────────────────────────────────────────────┘

✅ Benefits:
- Lazy initialization (created only when needed)
- Thread-safe (double-checked locking)
- volatile keyword ensures visibility
- More efficient than synchronized method
- Single instance guaranteed

Key Features:
- First null check (outside synchronized block)
- Synchronized block for thread safety
- Second null check (inside synchronized block)
- volatile prevents instruction reordering
```

**Code:**

```java
// ✅ Lazy initialization with double-checked locking (thread-safe)
public class DatabaseConnectionLazy {
    private static volatile DatabaseConnectionLazy instance;
    private String connectionString;
    
    private DatabaseConnectionLazy() {
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Creating database connection (lazy singleton)...");
    }
    
    public static DatabaseConnectionLazy getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionLazy.class) {
                if (instance == null) {
                    instance = new DatabaseConnectionLazy();
                }
            }
        }
        return instance;
    }
    
    public void connect() {
        System.out.println("Connected to: " + connectionString);
    }
}

// Usage - always same instance
DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
// db1 == db2 (same instance)
```

**Benefits:**
- ✅ Single instance guaranteed
- ✅ Global access point
- ✅ Lazy initialization possible
- ✅ Resource efficient

### ⚠️ Common Mistakes

#### 1. **Non-Thread-Safe Lazy Initialization**
```java
// ❌ WRONG - Not thread-safe!
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            // Problem: Multiple threads can enter here simultaneously
            instance = new Singleton();
        }
        return instance;
    }
}

// Scenario: Two threads calling getInstance() simultaneously
// Thread 1: checks instance == null → true, enters if block
// Thread 2: checks instance == null → true (before Thread 1 creates), enters if block
// Result: Two instances created! ❌
```

**Fix:** Use synchronization or eager initialization
```java
// ✅ Option 1: Synchronized method (simple but slower)
public static synchronized Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
}

// ✅ Option 2: Double-checked locking (faster)
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

#### 2. **Missing `volatile` Keyword in Double-Checked Locking**
```java
// ❌ WRONG - Without volatile, can have visibility issues
public class Singleton {
    private static Singleton instance;  // Missing volatile!
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                    // Problem: Without volatile, another thread might see
                    // partially constructed object due to instruction reordering
                }
            }
        }
        return instance;
    }
}
```

**Why `volatile` is Critical: Understanding Visibility Issues**

Without `volatile`, you can encounter two major problems:

##### **Problem 1: Instruction Reordering**

When you write `instance = new Singleton()`, it's actually **three separate operations**:

```java
// What you write:
instance = new Singleton();

// What actually happens (conceptually):
1. memory = allocate();           // Allocate memory for object
2. constructor(memory);            // Call constructor to initialize
3. instance = memory;              // Assign reference to instance variable
```

**The Problem:** The JVM and CPU can **reorder** these instructions for optimization:

```java
// Possible reordering:
1. memory = allocate();           // Allocate memory
2. instance = memory;              // ⚠️ Assign reference BEFORE constructor completes!
3. constructor(memory);            // Initialize object (happens after assignment)
```

**What happens without `volatile`:**

```java
// Thread 1 (inside synchronized block):
instance = new Singleton();
// Due to reordering, instance might be assigned BEFORE object is fully constructed

// Thread 2 (outside synchronized block, first null check):
if (instance == null) {  // ❌ Might see non-null but partially constructed object!
    // Thread 2 sees instance != null, so skips synchronized block
    return instance;  // ❌ Returns partially constructed object!
}
```

**Detailed Example:**

```java
public class Singleton {
    private static Singleton instance;  // Missing volatile!
    private int value = 42;  // Instance field
    
    private Singleton() {
        // Simulate some initialization work
        this.value = 42;
        // Without volatile, Thread 2 might see instance before this completes
    }
    
    public static Singleton getInstance() {
        if (instance == null) {  // First check (NOT synchronized!)
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                    // Thread 1: Due to reordering, instance reference might be
                    // assigned before constructor finishes initializing value
                }
            }
        }
        return instance;  // Thread 2 might get partially constructed object
    }
    
    public int getValue() {
        return value;  // Might return 0 (default) instead of 42!
    }
}

// Scenario:
// Thread 1: Creates instance (inside synchronized block)
//   - Due to reordering: instance = memory happens BEFORE constructor completes
// Thread 2: Checks instance == null (outside synchronized block)
//   - Sees instance != null (because reference was assigned)
//   - Returns instance immediately
//   - But object might not be fully initialized!
//   - Thread 2 calls getValue() → might get 0 instead of 42 ❌
```

##### **Problem 2: Memory Visibility**

Even without reordering, there's a **visibility problem**:

```java
// Thread 1 (inside synchronized block):
instance = new Singleton();  // Writes to instance
// Exits synchronized block

// Thread 2 (outside synchronized block):
if (instance == null) {  // Reads instance
    // Problem: Thread 2 might not see Thread 1's write!
    // Without volatile, writes in one thread might not be visible to other threads
    // immediately (due to CPU cache, not main memory)
}
```

**The Java Memory Model (JMM) Issue:**

- Each thread has its own **CPU cache**
- Without `volatile`, writes might stay in **local cache** and not flush to **main memory**
- Other threads reading from **main memory** won't see the updated value
- `synchronized` provides visibility, but only **within** the synchronized block
- The **first null check** is **outside** the synchronized block, so it doesn't have visibility guarantees!

**Timeline Example:**

```
Time    Thread 1                          Thread 2
----    --------                          --------
T1      Enter synchronized block
T2      instance = new Singleton()
        (writes to Thread 1's cache)
T3      Exit synchronized block
        (write might not be flushed to main memory yet)
T4                                      if (instance == null)
                                        (reads from main memory)
                                        Sees null! ❌
T5                                      Enters synchronized block
T6                                      Creates ANOTHER instance! ❌
```

**How `volatile` Fixes Both Issues:**

```java
private static volatile Singleton instance;
```

**`volatile` provides:**
1. **Prevents reordering:** Creates a "happens-before" relationship
   - Write to `volatile` variable happens-before any subsequent read
   - Prevents compiler/CPU from reordering instructions around volatile access

2. **Ensures visibility:** Writes to `volatile` are immediately visible to all threads
   - Forces writes to flush to main memory
   - Forces reads to read from main memory (not cache)

**With `volatile`, the correct order is guaranteed:**

```java
// Thread 1:
instance = new Singleton();
// volatile ensures:
// 1. Constructor completes BEFORE assignment
// 2. Write is immediately visible to all threads

// Thread 2:
if (instance == null) {  // ✅ Sees correct value (either null or fully constructed)
    // ...
}
```

**Fix:** Always use `volatile` with double-checked locking
```java
// ✅ CORRECT
private static volatile Singleton instance;

// This ensures:
// 1. No instruction reordering
// 2. Immediate visibility across all threads
// 3. Thread-safe singleton creation
```

#### 3. **Not Making Constructor Private**
```java
// ❌ WRONG - Constructor is public, anyone can create instances
public class Singleton {
    private static Singleton instance;
    
    public Singleton() {  // Should be private!
        // ...
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// Usage - breaks singleton!
Singleton s1 = Singleton.getInstance();
Singleton s2 = new Singleton();  // ❌ Can create new instance!
```

**Fix:** Always make constructor private
```java
// ✅ CORRECT
private Singleton() {
    // ...
}
```

#### 4. **Serialization Breaking Singleton**
```java
// ❌ WRONG - Serialization can create new instances
public class Singleton implements Serializable {
    private static Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
}

// Problem: When deserialized, creates new instance
ObjectOutputStream out = new ObjectOutputStream(...);
out.writeObject(Singleton.getInstance());

ObjectInputStream in = new ObjectInputStream(...);
Singleton s1 = (Singleton) in.readObject();
Singleton s2 = Singleton.getInstance();
// s1 != s2 ❌
```

**Fix:** Implement `readResolve()` method
```java
// ✅ CORRECT
public class Singleton implements Serializable {
    private static Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
    
    // Prevents new instance during deserialization
    protected Object readResolve() {
        return getInstance();
    }
}
```

#### 5. **Reflection Breaking Singleton**
```java
// ❌ Problem: Reflection can access private constructor
Singleton s1 = Singleton.getInstance();

Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton s2 = constructor.newInstance();  // ❌ New instance created!
```

**Fix:** Throw exception if instance already exists
```java
// ✅ CORRECT
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // Prevent reflection from creating new instances
        if (instance != null) {
            throw new IllegalStateException("Singleton instance already exists!");
        }
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 6. **Clone Breaking Singleton**
```java
// ❌ WRONG - Clone can create new instance
public class Singleton implements Cloneable {
    private static Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
    
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();  // ❌ Creates new instance!
    }
}

// Usage
Singleton s1 = Singleton.getInstance();
Singleton s2 = (Singleton) s1.clone();  // ❌ New instance!
```

**Fix:** Override clone to return same instance or throw exception
```java
// ✅ CORRECT
@Override
public Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException("Singleton cannot be cloned");
    // OR return getInstance(); // Return same instance
}
```

#### 7. **Using Enum Singleton (Best Practice for Java)**
```java
// ✅ BEST PRACTICE - Thread-safe, serialization-safe, reflection-safe
public enum Singleton {
    INSTANCE;
    
    private String connectionString;
    
    public void initialize(String connectionString) {
        this.connectionString = connectionString;
    }
    
    public void connect() {
        System.out.println("Connected: " + connectionString);
    }
}

// Usage
Singleton.INSTANCE.initialize("jdbc:mysql://localhost:3306/mydb");
Singleton.INSTANCE.connect();
```

**Why Enum is Best:**
- ✅ Thread-safe by default
- ✅ Serialization handled automatically
- ✅ Reflection cannot break it
- ✅ Clone not applicable
- ✅ Simple and concise

---

## 2. Factory Pattern

**Problem:** Need to create objects without specifying exact classes (decouple object creation from usage).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         NotificationClient                  │
├─────────────────────────────────────────────┤
│ + sendNotification(type, message): void     │
└─────────────────────────────────────────────┘
         │
         │ directly creates
         │
    ┌────┴────────────┬──────────────┬─────────────┐
    │                 │              │             │
┌───┴─────────┐ ┌─────┴──────────┐ ┌─┴──────────────┐
│ Email       │ │ SMS            │ │ Push           │
│Notification │ │Notification    │ │Notification    │
├─────────────┤ ├────────────────┤ ├────────────────┤
│ +send()     │ │ +send()        │ │ +send()        │
└─────────────┘ └────────────────┘ └────────────────┘

❌ Problems:
- Client tightly coupled to concrete classes
- Adding new type requires modifying client
- Hard to test (can't mock easily)
- Violates Open/Closed Principle
```

**Code:**

```java
// Tight coupling - client knows about all concrete classes
public class NotificationClient {
    public void sendNotification(String type, String message) {
        if (type.equals("email")) {
            EmailNotification email = new EmailNotification();
            email.send(message);
        } else if (type.equals("sms")) {
            SMSNotification sms = new SMSNotification();
            sms.send(message);
        } else if (type.equals("push")) {
            PushNotification push = new PushNotification();
            push.send(message);
        }
        // Problem: Adding new type requires modifying this code
    }
}

class EmailNotification {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SMSNotification {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

class PushNotification {
    public void send(String message) {
        System.out.println("Sending push: " + message);
    }
}
```

### ✅ Factory Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│            Notification                     │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + send(message): void                       │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────────────┬──────────────┬─────────────┐
    │                 │              │             │
┌───┴─────────┐ ┌─────┴──────────┐ ┌─┴──────────────┐
│ Email       │ │ SMS            │ │ Push           │
│Notification │ │Notification    │ │Notification    │
├─────────────┤ ├────────────────┤ ├────────────────┤
│ +send()     │ │ +send()        │ │ +send()        │
└─────────────┘ └────────────────┘ └────────────────┘
    │             │              │
    └─────────────┴──────────────┘
                   │
                   │ creates
                   ▼
┌─────────────────────────────────────────────┐
│         NotificationFactory                 │
├─────────────────────────────────────────────┤
│ + static createNotification(type):          │
│   Notification                              │
└─────────────────────────────────────────────┘
              │
              │ uses
              ▼
┌─────────────────────────────────────────────┐
│         NotificationClient                  │
├─────────────────────────────────────────────┤
│ + sendNotification(type, message): void     │
└─────────────────────────────────────────────┘

✅ Benefits:
- Client depends on abstraction (interface)
- Factory encapsulates object creation
- Easy to add new types without modifying client
- Decouples object creation from usage
```

**Code:**

```java
// ✅ Common interface
public interface Notification {
    void send(String message);
}

// ✅ Concrete implementations
public class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class SMSNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

public class PushNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending push: " + message);
    }
}

// ✅ Factory class - encapsulates object creation
public class NotificationFactory {
    public static Notification createNotification(String type) {
        switch (type.toLowerCase()) {
            case "email":
                return new EmailNotification();
            case "sms":
                return new SMSNotification();
            case "push":
                return new PushNotification();
            default:
                throw new IllegalArgumentException("Unknown notification type: " + type);
        }
    }
}

// ✅ Client code - decoupled from concrete classes
public class NotificationClient {
    public void sendNotification(String type, String message) {
        Notification notification = NotificationFactory.createNotification(type);
        notification.send(message);
        // Easy to add new types - just extend factory, no client change needed
    }
}

// Usage
NotificationClient client = new NotificationClient();
client.sendNotification("email", "Hello");
client.sendNotification("sms", "Hello");
```

**Benefits:**
- ✅ Decouples object creation from usage
- ✅ Easy to add new types
- ✅ Single responsibility (factory handles creation)
- ✅ Client code is cleaner

---

## 3. Builder Pattern

**Problem:** Complex object construction with many optional parameters (avoid telescoping constructors).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────────────────────┐
│                         User                                │
├─────────────────────────────────────────────────────────────┤
│ - String name                                               │
│ - String email                                              │
│ - int age                                                   │
│ - String phone                                              │
│ - String address                                            │
│ - boolean isActive                                          │
├─────────────────────────────────────────────────────────────┤
│ + User(name: String)                                        │
│ + User(name: String, email: String)                         │
│ + User(name: String, email: String, age: int)               │
│ + User(name: String, email: String, age: int, phone: String)│
│ + User(name: String, email: String, age: int,               │
│       phone: String, address: String)                       │
│ + User(name: String, email: String, age: int,               │
│       phone: String, address: String, isActive: boolean)    │
│ + setName(name: String): void                               │
│ + setEmail(email: String): void                             │
│ + setAge(age: int): void                                    │
│ + setPhone(phone: String): void                             │
│ + setAddress(address: String): void                         │
│ + setActive(isActive: boolean): void                        │
│ + getName(): String                                         │
│ + getEmail(): String                                        │
│ + getAge(): int                                             │
│ + getPhone(): String                                        │
│ + getAddress(): String                                      │
│ + isActive(): boolean                                       │
└─────────────────────────────────────────────────────────────┘

❌ Problems:
- Multiple constructors (telescoping)
- Mutable fields (not final)
- Inconsistent state possible
- Hard to remember parameter order
- No validation
```

**Code:**

```java
// Telescoping constructors - hard to use and error-prone
public class User {
    private String name;
    private String email;
    private int age;
    private String phone;
    private String address;
    private boolean isActive;
    
    // Too many constructors!
    public User(String name) {
        this.name = name;
    }
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    // ... more constructors ...
    
    // Or setters - but object can be in inconsistent state
    public void setName(String name) { this.name = name; }
    public void setEmail(String email) { this.email = email; }
    // Problem: Object can be used before all required fields are set
}

// Usage - confusing and error-prone
User user1 = new User("John", "john@email.com", 30, "123-456", "123 St", true);
// Which parameter is which? Hard to remember order!
```

### ✅ Builder Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────────────────────┐
│                         User                                │
├─────────────────────────────────────────────────────────────┤
│ - final String name                                         │
│ - final String email                                        │
│ - final int age                                             │
│ - final String phone                                        │
│ - final String address                                      │
│ - final boolean isActive                                    │
├─────────────────────────────────────────────────────────────┤
│ - User(builder: UserBuilder)                                │
│ + getName(): String                                         │
│ + getEmail(): String                                        │
│ + getAge(): int                                             │
│ + getPhone(): String                                        │
│ + getAddress(): String                                      │
│ + isActive(): boolean                                       │
└─────────────────────────────────────────────────────────────┘
         ▲
         │ creates
         │
┌─────────────────────────────────────────────────────────────┐
│                    UserBuilder                              │
├─────────────────────────────────────────────────────────────┤
│ - final String name                                         │
│ - final String email                                        │
│ - int age                                                   │
│ - String phone                                              │
│ - String address                                            │
│ - boolean isActive                                          │
├─────────────────────────────────────────────────────────────┤
│ + UserBuilder(name: String, email: String)                  │
│ + age(age: int): UserBuilder                                │
│ + phone(phone: String): UserBuilder                         │
│ + address(address: String): UserBuilder                     │
│ + isActive(isActive: boolean): UserBuilder                  │
│ + build(): User                                             │
└─────────────────────────────────────────────────────────────┘

✅ User Benefits:
- Immutable (final fields)
- Private constructor
- Only builder can create

✅ UserBuilder Benefits:
- Fluent interface
- Validation in build()
- Required params in constructor
- Optional params via methods
```

**Code:**

```java
// ✅ Product class
public class User {
    private final String name;        // Required
    private final String email;       // Required
    private final int age;            // Optional
    private final String phone;       // Optional
    private final String address;     // Optional
    private final boolean isActive;   // Optional
    
    // Private constructor - only builder can create
    private User(UserBuilder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
        this.isActive = builder.isActive;
    }
    
    // Getters
    public String getName() { return name; }
    public String getEmail() { return email; }
    public int getAge() { return age; }
    public String getPhone() { return phone; }
    public String getAddress() { return address; }
    public boolean isActive() { return isActive; }
    
    // ✅ Builder class
    public static class UserBuilder {
        private final String name;    // Required
        private final String email;   // Required
        private int age = 0;
        private String phone = "";
        private String address = "";
        private boolean isActive = true;
        
        // Required parameters in constructor
        public UserBuilder(String name, String email) {
            this.name = name;
            this.email = email;
        }
        
        // Optional parameters with fluent interface
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }
        
        public UserBuilder isActive(boolean isActive) {
            this.isActive = isActive;
            return this;
        }
        
        // Build method
        public User build() {
            // Validation
            if (name == null || email == null) {
                throw new IllegalArgumentException("Name and email are required");
            }
            return new User(this);
        }
    }
}

// ✅ Usage - clear, readable, flexible
User user1 = new User.UserBuilder("John", "john@email.com")
    .age(30)
    .phone("123-456-7890")
    .address("123 Main St")
    .isActive(true)
    .build();

User user2 = new User.UserBuilder("Jane", "jane@email.com")
    .age(25)
    .build();  // Only required fields

// Fluent interface - easy to read and understand
```

**Sequence Diagram:**

```
Client                UserBuilder              User
  │                       │                      │
  │──new UserBuilder─────>│                      │
  │  ("John",             │                      │
  │   "john@email.com")   │                      │
  │                       │ Required params      │
  │                       │ set in constructor   │
  │                       │                      │
  │<──return builder──────│                      │
  │                       │                      │
  │──.age(30)────────────>│                      │
  │<──return this─────────│                      │
  │                       │                      │
  │──.phone("123-456")───>│                      │
  │<──return this─────────│                      │
  │                       │                      │
  │──.address("123 St")──>│                      │
  │<──return this─────────│                      │
  │                       │                      │
  │──.isActive(true)─────>│                      │
  │<──return this─────────│                      │
  │                       │                      │
  │──.build()────────────>│                      │
  │                       │ Validates required   │
  │                       │ fields               │
  │                       │──new User(this)─────>│
  │                       │                      │ Private constructor
  │                       │                      │ Creates immutable
  │                       │<──User instance──────│ object
  │<──User instance───────│                      │
  │                       │                      │
  │ ✅ Object is fully constructed and validated │
```

**Comparison Diagram:**

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ Bad Approach: Telescoping Constructors                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Constructor 1: name                                       │
│         │                                                       │
│         ▼                                                       │
│  User Constructor 2: name, email                                │
│         │                                                       │
│         ▼                                                       │
│  User Constructor 3: name, email, age                           │
│         │                                                       │
│         ▼                                                       │
│  User Constructor 4: name, email, age, phone                    │
│         │                                                       │
│         ▼                                                       │
│  User Constructor 5: name, email, age, phone, address           │
│         │                                                       │
│         ▼                                                       │
│  User Constructor 6: name, email, age, phone, address, isActive │
│         │                                                       │
│         │                                                       │
│  Client Code                                                    │
│    │                                                            │
│    └──> new User("John", "email", 30, "phone", "addr", true)    │
│         │                                                       │
│         ▼                                                       │
│  ❌ Problems:                                                   │
│  - Hard to remember order                                       │
│  - Error-prone                                                  │
│  - Not readable                                                 │
│  - Mutable objects                                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ ✅ Good Approach: Builder Pattern                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Client Code                                                    │
│    │                                                            │
│    ├──> 1. new UserBuilder(name, email)                         │
│    ├──> 2. .age(30)                                             │
│    ├──> 3. .phone("123")                                        │
│    ├──> 4. .address("addr")                                     │
│    ├──> 5. .isActive(true)                                      │
│    └──> 6. .build()                                             │
│         │                                                       │
│         ▼                                                       │
│  UserBuilder                                                    │
│    │                                                            │
│    │ Creates & Validates                                        │
│    ▼                                                            │
│  User (Immutable Object)                                        │
│    │                                                            │
│    ▼                                                            │
│  ✅ Benefits:                                                   │
│  - Clear & readable                                             │
│  - Flexible                                                     │
│  - Immutable                                                    │
│  - Validated                                                    │
│  - Fluent interface                                             │
└─────────────────────────────────────────────────────────────────┘
```

**Benefits:**
- ✅ Clear, readable object construction
- ✅ Flexible - can set any combination of optional parameters
- ✅ Immutable objects (if fields are final)
- ✅ Validation in build() method
- ✅ No telescoping constructors

---

## 4. Strategy Pattern

**Problem:** Need to switch between different algorithms at runtime (payment methods, sorting, compression).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         PaymentProcessor                    │
├─────────────────────────────────────────────┤
│ + processPayment(method, amount): void      │
└─────────────────────────────────────────────┘

❌ Problems:
- Hard-coded if-else chains
- Adding new payment method requires modifying this class
- Violates Open/Closed Principle
- Hard to test individual payment methods
```

**Code:**

```java
// Hard-coded algorithms - hard to extend
public class PaymentProcessor {
    public void processPayment(String method, double amount) {
        if (method.equals("creditcard")) {
            System.out.println("Processing credit card payment: $" + amount);
            // Complex credit card logic
        } else if (method.equals("paypal")) {
            System.out.println("Processing PayPal payment: $" + amount);
            // Complex PayPal logic
        } else if (method.equals("crypto")) {
            System.out.println("Processing crypto payment: $" + amount);
            // Complex crypto logic
        }
        // Problem: Adding new payment method requires modifying this class
    }
}

// Usage
PaymentProcessor processor = new PaymentProcessor();
processor.processPayment("creditcard", 100.0);
```

### ✅ Strategy Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         PaymentStrategy                     │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + pay(amount): void                         │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────────────┬──────────────┬─────────────┐
    │                 │              │             │
┌───┴─────────┐ ┌─────┴──────────┐ ┌─┴──────────────┐
│ CreditCard  │ │ PayPal         │ │ Crypto         │
│ Payment     │ │ Payment        │ │ Payment        │
├─────────────┤ ├────────────────┤ ├────────────────┤
│ +pay()      │ │ +pay()         │ │ +pay()         │
└─────────────┘ └────────────────┘ └────────────────┘
    │             │              │
    └─────────────┴──────────────┘
                   │
                   │ uses/has-a
                   ▼
┌─────────────────────────────────────────────┐
│         PaymentProcessor                    │
├─────────────────────────────────────────────┤
│ - PaymentStrategy strategy                  │
├─────────────────────────────────────────────┤
│ + PaymentProcessor(strategy)                │
│ + setStrategy(strategy): void               │
│ + processPayment(amount): void              │
└─────────────────────────────────────────────┘

✅ Benefits:
- Algorithms are interchangeable
- Easy to add new strategies
- Eliminates conditional statements
- Runtime strategy selection
```

**Code:**

```java
// ✅ Strategy interface
public interface PaymentStrategy {
    void pay(double amount);
}

// ✅ Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String name;
    
    public CreditCardPayment(String cardNumber, String name) {
        this.cardNumber = cardNumber;
        this.name = name;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
        System.out.println("Card: " + cardNumber + ", Name: " + name);
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
        System.out.println("Email: " + email);
    }
}

public class CryptoPayment implements PaymentStrategy {
    private String walletAddress;
    
    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Processing crypto payment of $" + amount);
        System.out.println("Wallet: " + walletAddress);
    }
}

// ✅ Context class - uses strategy
public class PaymentProcessor {
    private PaymentStrategy strategy;
    
    public PaymentProcessor(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    // Can change strategy at runtime
    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void processPayment(double amount) {
        strategy.pay(amount);
    }
}

// ✅ Usage - flexible and extensible
PaymentStrategy creditCard = new CreditCardPayment("1234-5678", "John Doe");
PaymentProcessor processor = new PaymentProcessor(creditCard);
processor.processPayment(100.0);

// Can switch strategies at runtime
processor.setStrategy(new PayPalPayment("john@email.com"));
processor.processPayment(50.0);

// Easy to add new payment methods - just implement PaymentStrategy
```

**Benefits:**
- ✅ Algorithms are interchangeable
- ✅ Easy to add new strategies
- ✅ Eliminates conditional statements
- ✅ Runtime strategy selection

---

## 5. Observer Pattern

**Problem:** Need to notify multiple objects when state changes (event system, model-view updates).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         NewsAgency                          │
├─────────────────────────────────────────────┤
│ - String news                               │
│ - NewsChannel channel1                      │
│ - NewsChannel channel2                      │
│ - NewsChannel channel3                      │
├─────────────────────────────────────────────┤
│ + setNews(news): void                       │
└─────────────────────────────────────────────┘
         │
         │ hard-coded references
         │
┌────────┴────────┐
│  NewsChannel    │
├─────────────────┤
│ + update(news)  │
└─────────────────┘

❌ Problems:
- Observable tightly coupled to concrete observers
- Adding new observer requires modifying observable
- Hard-coded observer references
- Not flexible
```

**Code:**

```java
// Tight coupling - observable knows about all observers
public class NewsAgency {
    private String news;
    private NewsChannel channel1;
    private NewsChannel channel2;
    private NewsChannel channel3;
    
    public void setNews(String news) {
        this.news = news;
        // Hard-coded - must modify this class to add/remove observers
        if (channel1 != null) channel1.update(news);
        if (channel2 != null) channel2.update(news);
        if (channel3 != null) channel3.update(news);
    }
    
    // Problem: Adding new observer requires modifying this class
}

class NewsChannel {
    public void update(String news) {
        System.out.println("Channel received: " + news);
    }
}
```

### ✅ Observer Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│            Observable                       │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + attach(observer): void                    │
│ + detach(observer): void                    │
│ + notifyObservers(): void                   │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
┌─────────────────────────────────────────────┐
│         NewsAgency                          │
├─────────────────────────────────────────────┤
│ - List<Observer> observers                  │
│ - String news                               │
├─────────────────────────────────────────────┤
│ + attach(observer): void                    │
│ + detach(observer): void                    │
│ + notifyObservers(): void                   │
│ + setNews(news): void                       │
└─────────────────────────────────────────────┘
         │
         │ notifies
         │
┌─────────────────────────────────────────────┐
│            Observer                         │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + update(news): void                        │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────────────┬─────────────┐
    │                 │             │
┌───┴─────────┐ ┌─────┴──────────┐
│ NewsChannel │ │ MobileApp      │
├─────────────┤ ├────────────────┤
│ +update()   │ │ +update()      │
└─────────────┘ └────────────────┘

✅ Benefits:
- Loose coupling between observable and observers
- Dynamic subscription/unsubscription
- One-to-many dependency
- Easy to add new observers
```

**Code:**

```java
// ✅ Observer interface
public interface Observer {
    void update(String news);
}

// ✅ Observable interface
public interface Observable {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// ✅ Concrete observers
public class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

public class MobileApp implements Observer {
    private String appName;
    
    public MobileApp(String appName) {
        this.appName = appName;
    }
    
    @Override
    public void update(String news) {
        System.out.println(appName + " notification: " + news);
    }
}

// ✅ Concrete observable
public class NewsAgency implements Observable {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();  // Notify all observers
    }
}

// ✅ Usage - loosely coupled
NewsAgency agency = new NewsAgency();

Observer channel1 = new NewsChannel("CNN");
Observer channel2 = new NewsChannel("BBC");
Observer app = new MobileApp("NewsApp");

agency.attach(channel1);
agency.attach(channel2);
agency.attach(app);

agency.setNews("Breaking: Important news!");
// All observers are notified automatically

agency.detach(channel2);
agency.setNews("Update: More news!");
// Only channel1 and app are notified
```

**Benefits:**
- ✅ Loose coupling between observable and observers
- ✅ Dynamic subscription/unsubscription
- ✅ One-to-many dependency
- ✅ Open/Closed Principle (easy to add observers)

---

## 6. Decorator Pattern

**Problem:** Need to add behavior to objects dynamically without altering structure (coffee add-ons, stream processing).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────┐
│         Coffee              │
├─────────────────────────────┤
│ + getDescription(): String  │
│ + getCost(): double         │
└─────────────────────────────┘
         ▲
         │ extends
         │
    ┌────┴──────────────────────────┬─────────────┐
    │                               │             │
┌───┴──────────────┐ ┌──────────────┴────┐ ┌──────┴─────────────┐
│ CoffeeWithMilk   │ │ CoffeeWithSugar    │ │ CoffeeWith         │
│                  │ │                    │ │ MilkAndSugar       │
├──────────────────┤ ├────────────────────┤ ├────────────────────┤
│ +getDescription()│ │ +getDescription()  │ │ ...                │
│ +getCost()       │ │ +getCost()         │ │                    │
└──────────────────┘ └────────────────────┘ └────────────────────┘
      │
      │ ... more combinations
      │
❌ Problems:
- Class explosion (exponential growth)
- Need class for every combination
- Hard to maintain
- Not flexible
```

**Code:**

```java
// Class explosion - need class for every combination
public class Coffee {
    public String getDescription() {
        return "Coffee";
    }
    
    public double getCost() {
        return 2.0;
    }
}

class CoffeeWithMilk extends Coffee {
    @Override
    public String getDescription() {
        return "Coffee with Milk";
    }
    
    @Override
    public double getCost() {
        return 2.5;
    }
}

class CoffeeWithSugar extends Coffee {
    // ...
}

class CoffeeWithMilkAndSugar extends Coffee {
    // ...
}

class CoffeeWithMilkAndSugarAndCaramel extends Coffee {
    // ...
}
// Problem: Exponential class explosion!
```

### ✅ Decorator Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│            Coffee                           │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + getDescription(): String                  │
│ + getCost(): double                         │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴───────────────────────────┐
    │                                │
┌──────────────────┐  ┌──────────────┴──────────────────┐
│  SimpleCoffee    │  │    CoffeeDecorator              │
│                  │  │    <<abstract>>                 │
├──────────────────┤  ├─────────────────────────────────┤
│ +getDescription()│  │ - Coffee coffee                 │
│ +getCost()       │  │ + CoffeeDecorator(coffee)       │
└──────────────────┘  │ + getDescription(): String      │
                      │ + getCost(): double             │
                      └─────────────────────────────────┘
                           ▲
                           │ extends
                           │
              ┌────────────┼───────────┬─────────────┐
              │            │           │             │
        ┌─────┴─────────┐ ┌────────────┴────┐ ┌──────┴─────────┐
        │ Milk          │ │ Sugar           │ │ Caramel        │
        │ Decorator     │ │ Decorator       │ │ Decorator      │
        ├───────────────┤ ├─────────────────┤ ├────────────────┤
        │ +getDesc()    │ │ +getDesc()      │ │ +getDesc()     │
        │ +getCost()    │ │ +getCost()      │ │ +getCost()     │
        └───────────────┘ └─────────────────┘ └────────────────┘
              │             │              │
              └─────────────┴──────────────┘

✅ Benefits:
- Add behavior dynamically
- Avoids class explosion
- Flexible composition
- Single Responsibility (each decorator adds one feature)
```

**Code:**

```java
// ✅ Component interface
public interface Coffee {
    String getDescription();
    double getCost();
}

// ✅ Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.0;
    }
}

// ✅ Base decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// ✅ Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.2;
    }
}

public class CaramelDecorator extends CoffeeDecorator {
    public CaramelDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Caramel";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.7;
    }
}

// ✅ Usage - compose decorators dynamically
Coffee coffee = new SimpleCoffee();
System.out.println(coffee.getDescription() + " - $" + coffee.getCost());

coffee = new MilkDecorator(coffee);
System.out.println(coffee.getDescription() + " - $" + coffee.getCost());

coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " - $" + coffee.getCost());

coffee = new CaramelDecorator(new MilkDecorator(new SimpleCoffee()));
System.out.println(coffee.getDescription() + " - $" + coffee.getCost());
// Output: Simple Coffee, Milk, Caramel - $3.2
```

**Benefits:**
- ✅ Add behavior dynamically
- ✅ Avoids class explosion
- ✅ Flexible composition
- ✅ Single Responsibility (each decorator adds one feature)

---

## 7. Adapter Pattern

**Problem:** Need to integrate incompatible interfaces (legacy systems, third-party APIs).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│      LegacyPaymentProcessor                 │
├─────────────────────────────────────────────┤
│ + makePayment(account, amount): void        │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│    ModernPaymentProcessor                   │
├─────────────────────────────────────────────┤
│ + processPayment(request): void             │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│         PaymentClient                       │
├─────────────────────────────────────────────┤
│ + pay(processor, userId, amount): void      │
└─────────────────────────────────────────────┘

❌ Problems:
- Incompatible interfaces
- Can't use legacy system with new code
- Need to modify client code
```

**Code:**

```java
// Incompatible interfaces
public class LegacyPaymentProcessor {
    public void makePayment(String account, double amount) {
        System.out.println("Legacy: Paying $" + amount + " from " + account);
    }
}

// New system expects different interface
public class ModernPaymentProcessor {
    public void processPayment(PaymentRequest request) {
        // Expects PaymentRequest object
        System.out.println("Modern: Processing payment");
    }
}

class PaymentRequest {
    private String userId;
    private double amount;
    // ...
}

// Problem: Can't use legacy system with new code directly
```

### ✅ Adapter Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         PaymentProcessor                    │
│          <<interface>>                      │
│         (Target)                            │
├─────────────────────────────────────────────┤
│ + processPayment(userId, amount): void      │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────────────────────────────────────┐
    │                                         │
┌───┴──────────────────┐ ┌────────────────────┴───────┐
│ LegacyPayment        │ │ ModernPayment              │
│ Adapter              │ │ Processor                  │
├──────────────────────┤ ├────────────────────────────┤
│ - LegacyPayment      │ │ + processPayment()         │
│   Processor          │ │                            │
│   legacyProcessor    │ │                            │
│ + processPayment()   │ │                            │
│   (adapts)           │ │                            │
└──────────────────────┘ └────────────────────────────┘
    │
    │ uses/wraps
    │ (only LegacyPaymentAdapter)
    ▼
┌─────────────────────────────────────────────┐
│      LegacyPaymentProcessor                 │
│         (Adaptee)                           │
├─────────────────────────────────────────────┤
│ + makePayment(account, amount): void        │
└─────────────────────────────────────────────┘

✅ Benefits:
- Makes incompatible interfaces work together
- Reuses existing code
- Client code doesn't need to change
- Single Responsibility (adapter handles conversion)
```

**Code:**

```java
// ✅ Target interface (what client expects)
public interface PaymentProcessor {
    void processPayment(String userId, double amount);
}

// ✅ Adaptee (legacy system - incompatible)
public class LegacyPaymentProcessor {
    public void makePayment(String account, double amount) {
        System.out.println("Legacy: Paying $" + amount + " from account " + account);
    }
}

// ✅ Adapter - adapts legacy to modern interface
public class LegacyPaymentAdapter implements PaymentProcessor {
    private LegacyPaymentProcessor legacyProcessor;
    
    public LegacyPaymentAdapter(LegacyPaymentProcessor legacyProcessor) {
        this.legacyProcessor = legacyProcessor;
    }
    
    @Override
    public void processPayment(String userId, double amount) {
        // Adapt the interface
        String account = "ACC-" + userId;  // Convert userId to account format
        legacyProcessor.makePayment(account, amount);
    }
}

// ✅ Modern implementation (for comparison)
public class ModernPaymentProcessor implements PaymentProcessor {
    @Override
    public void processPayment(String userId, double amount) {
        System.out.println("Modern: Processing payment of $" + amount + " for user " + userId);
    }
}

// ✅ Client code - works with both
public class PaymentClient {
    public void pay(PaymentProcessor processor, String userId, double amount) {
        processor.processPayment(userId, amount);
    }
}

// ✅ Usage - adapter makes legacy system compatible
LegacyPaymentProcessor legacy = new LegacyPaymentProcessor();
PaymentProcessor adapter = new LegacyPaymentAdapter(legacy);

PaymentClient client = new PaymentClient();
client.pay(adapter, "user123", 100.0);  // Works with legacy system!

PaymentProcessor modern = new ModernPaymentProcessor();
client.pay(modern, "user123", 100.0);  // Works with modern system too!
```

**Benefits:**
- ✅ Makes incompatible interfaces work together
- ✅ Reuses existing code
- ✅ Client code doesn't need to change
- ✅ Single Responsibility (adapter handles conversion)

---

## 8. Command Pattern

**Problem:** Need to encapsulate requests as objects (undo/redo, queuing, remote control, logging).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         TextEditor                          │
├─────────────────────────────────────────────┤
│ - StringBuilder text                        │
├─────────────────────────────────────────────┤
│ + addText(text): void                       │
│ + deleteText(length): void                  │
│ + clear(): void                             │
└─────────────────────────────────────────────┘

Client
  │
  └──> editor.addText("Hello")
  └──> editor.deleteText(2)
  // ❌ Can't undo operations
  // ❌ Can't queue operations
  // ❌ Can't log operations

❌ Problems:
- Direct method calls
- Can't undo operations
- Can't queue operations
- Can't log operations
```

**Code:**

```java
// Direct method calls - can't undo, queue, or log
public class TextEditor {
    private StringBuilder text = new StringBuilder();
    
    public void addText(String text) {
        this.text.append(text);
    }
    
    public void deleteText(int length) {
        if (text.length() >= length) {
            text.delete(text.length() - length, text.length());
        }
    }
    
    public void clear() {
        text = new StringBuilder();
    }
    
    // Problem: Can't undo operations, can't queue them, can't log them
}

// Usage
TextEditor editor = new TextEditor();
editor.addText("Hello");
editor.deleteText(2);
// Can't undo!
```

### ✅ Command Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│            Command                          │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + execute(): void                           │
│ + undo(): void                              │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴──────────────────────────┐
    │                               │
┌───┴──────────────────┐ ┌──────────┴──────────────────┐
│ AddText              │ │ Delete                      │
│ Command              │ │ Command                     │
├──────────────────────┤ ├─────────────────────────────┤
│ - TextEditor editor  │ │ - TextEditor editor         │
│ - String text        │ │ - int length                │
│                      │ │ - String deletedText        │
├──────────────────────┤ ├─────────────────────────────┤
│ +execute(): void     │ │ +execute(): void            │
│ +undo(): void        │ │ +undo(): void               │
└──────────────────────┘ └─────────────────────────────┘
    │             │
    │ uses        │ uses
    ▼             ▼
┌─────────────────────────────────────────────┐
│         TextEditor                          │
│         (Receiver)                          │
├─────────────────────────────────────────────┤
│ + addText(text): void                       │
│ + deleteText(length): void                  │
│ + getText(): String                         │
└─────────────────────────────────────────────┘
         ▲
         │ uses
         │
┌─────────────────────────────────────────────┐
│         CommandInvoker                      │
│         (Invoker)                           │
├─────────────────────────────────────────────┤
│ - List<Command> history                     │
│ - int currentIndex                          │
├─────────────────────────────────────────────┤
│ + executeCommand(command): void             │
│ + undo(): void                              │
│ + redo(): void                              │
└─────────────────────────────────────────────┘

✅ Benefits:
- Encapsulates requests as objects
- Supports undo/redo
- Can queue and log commands
- Decouples invoker from receiver
```

**Code:**

```java
// ✅ Command interface
public interface Command {
    void execute();
    void undo();
}

// ✅ Receiver
public class TextEditor {
    private StringBuilder text = new StringBuilder();
    
    public void addText(String text) {
        this.text.append(text);
    }
    
    public void deleteText(int length) {
        if (text.length() >= length) {
            text.delete(text.length() - length, text.length());
        }
    }
    
    public String getText() {
        return text.toString();
    }
}

// ✅ Concrete commands
public class AddTextCommand implements Command {
    private TextEditor editor;
    private String text;
    
    public AddTextCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.text = text;
    }
    
    @Override
    public void execute() {
        editor.addText(text);
    }
    
    @Override
    public void undo() {
        editor.deleteText(text.length());
    }
}

public class DeleteTextCommand implements Command {
    private TextEditor editor;
    private int length;
    private String deletedText;
    
    public DeleteTextCommand(TextEditor editor, int length) {
        this.editor = editor;
        this.length = length;
    }
    
    @Override
    public void execute() {
        String currentText = editor.getText();
        if (currentText.length() >= length) {
            deletedText = currentText.substring(currentText.length() - length);
            editor.deleteText(length);
        }
    }
    
    @Override
    public void undo() {
        if (deletedText != null) {
            editor.addText(deletedText);
        }
    }
}

// ✅ Invoker
public class CommandInvoker {
    private List<Command> history = new ArrayList<>();
    private int currentIndex = -1;
    
    public void executeCommand(Command command) {
        // Remove any commands after current index (for redo)
        while (history.size() > currentIndex + 1) {
            history.remove(history.size() - 1);
        }
        
        command.execute();
        history.add(command);
        currentIndex++;
    }
    
    public void undo() {
        if (currentIndex >= 0) {
            Command command = history.get(currentIndex);
            command.undo();
            currentIndex--;
        }
    }
    
    public void redo() {
        if (currentIndex < history.size() - 1) {
            currentIndex++;
            Command command = history.get(currentIndex);
            command.execute();
        }
    }
}

// ✅ Usage - supports undo/redo, queuing, logging
TextEditor editor = new TextEditor();
CommandInvoker invoker = new CommandInvoker();

invoker.executeCommand(new AddTextCommand(editor, "Hello"));
System.out.println(editor.getText());  // "Hello"

invoker.executeCommand(new AddTextCommand(editor, " World"));
System.out.println(editor.getText());  // "Hello World"

invoker.undo();
System.out.println(editor.getText());  // "Hello"

invoker.redo();
System.out.println(editor.getText());  // "Hello World"
```

**Benefits:**
- ✅ Encapsulates requests as objects
- ✅ Supports undo/redo
- ✅ Can queue and log commands
- ✅ Decouples invoker from receiver

---

## 9. Template Method Pattern

**Problem:** Define algorithm skeleton, let subclasses implement specific steps (data processing, report generation).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│      PDFReportGenerator                     │
├─────────────────────────────────────────────┤
│ + generateReport(data): void                │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│     HTMLReportGenerator                     │
├─────────────────────────────────────────────┤
│ + generateReport(data): void                │
└─────────────────────────────────────────────┘

❌ Problems:
- Algorithm structure duplicated
- Code duplication
- Hard to maintain
- Adding new report type requires duplicating algorithm
```

**Code:**

```java
// Code duplication - same algorithm structure repeated
public class PDFReportGenerator {
    public void generateReport(String data) {
        // Step 1: Read data
        System.out.println("Reading data: " + data);
        
        // Step 2: Process data
        String processed = data.toUpperCase();
        
        // Step 3: Format for PDF
        System.out.println("Formatting for PDF: " + processed);
        
        // Step 4: Generate PDF
        System.out.println("Generating PDF report");
    }
}

public class HTMLReportGenerator {
    public void generateReport(String data) {
        // Step 1: Read data (duplicated!)
        System.out.println("Reading data: " + data);
        
        // Step 2: Process data (duplicated!)
        String processed = data.toUpperCase();
        
        // Step 3: Format for HTML (different)
        System.out.println("Formatting for HTML: " + processed);
        
        // Step 4: Generate HTML (different)
        System.out.println("Generating HTML report");
    }
}
// Problem: Algorithm structure duplicated, hard to maintain
```

### ✅ Template Method Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│      ReportGenerator                        │
│      <<abstract>>                           │
├─────────────────────────────────────────────┤
│ + generateReport(data): void                │
│   (template method - final)                 │
│ # readData(data): String                    │
│   (hook method)                             │
│ # processData(data): String                 │
│   (abstract - must implement)               │
│ # formatOutput(data): String                │
│   (abstract - must implement)               │
│ # generateOutput(output): void              │
│   (abstract - must implement)               │
└─────────────────────────────────────────────┘
         ▲
         │ extends
         │
    ┌────┴──────────────────────────┐
    │                               │
┌───┴──────────────────┐ ┌──────────┴──────────────────┐
│ PDF                  │ │ HTML                        │
│ Report               │ │ Report                      │
│ Generator            │ │ Generator                   │
├──────────────────────┤ ├─────────────────────────────┤
│ #processData():      │ │ #processData():             │
│   String             │ │   String                    │
│ #formatOutput():     │ │ #formatOutput():            │
│   String             │ │   String                    │
│ #generateOutput():   │ │ #generateOutput():          │
│   void               │ │   void                      │
└──────────────────────┘ └─────────────────────────────┘

✅ Benefits:
- Code reuse (algorithm structure defined once)
- Consistent algorithm structure
- Easy to add new report types
- Control over algorithm flow
```

**Code:**

```java
// ✅ Abstract class with template method
public abstract class ReportGenerator {
    // ✅ Template method - defines algorithm skeleton
    public final void generateReport(String data) {
        String processedData = readData(data);
        String formattedData = processData(processedData);
        String output = formatOutput(formattedData);
        generateOutput(output);
    }
    
    // ✅ Common steps - can be concrete or hook methods
    protected String readData(String data) {
        System.out.println("Reading data: " + data);
        return data;
    }
    
    // ✅ Abstract method - must be implemented by subclasses
    protected abstract String processData(String data);
    
    // ✅ Abstract method - must be implemented by subclasses
    protected abstract String formatOutput(String data);
    
    // ✅ Abstract method - must be implemented by subclasses
    protected abstract void generateOutput(String output);
}

// ✅ Concrete implementations
public class PDFReportGenerator extends ReportGenerator {
    @Override
    protected String processData(String data) {
        return data.toUpperCase();
    }
    
    @Override
    protected String formatOutput(String data) {
        return "PDF Format: " + data;
    }
    
    @Override
    protected void generateOutput(String output) {
        System.out.println("Generating PDF: " + output);
    }
}

public class HTMLReportGenerator extends ReportGenerator {
    @Override
    protected String processData(String data) {
        return data.toLowerCase();
    }
    
    @Override
    protected String formatOutput(String data) {
        return "<html><body>" + data + "</body></html>";
    }
    
    @Override
    protected void generateOutput(String output) {
        System.out.println("Generating HTML: " + output);
    }
}

// ✅ Usage - algorithm structure is consistent
ReportGenerator pdfGen = new PDFReportGenerator();
pdfGen.generateReport("Sales Data");

ReportGenerator htmlGen = new HTMLReportGenerator();
htmlGen.generateReport("Sales Data");
```

**Benefits:**
- ✅ Code reuse (algorithm structure defined once)
- ✅ Consistent algorithm structure
- ✅ Easy to add new report types
- ✅ Control over algorithm flow

---

## 10. State Pattern

**Problem:** Object behavior changes based on state (vending machine, order status, game character).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         VendingMachine                      │
├─────────────────────────────────────────────┤
│ - String state                              │
│   ("idle", "hasMoney", "dispensing")        │
│ - int money                                 │
├─────────────────────────────────────────────┤
│ + insertMoney(amount): void                 │
│   (if-else based on state)                  │
│ + selectProduct(product): void              │
│   (if-else based on state)                  │
│ + dispenseProduct(product): void            │
│   (if-else based on state)                  │
└─────────────────────────────────────────────┘

❌ Problems:
- State managed with if-else chains
- Adding new state requires modifying all methods
- Hard to maintain
- Not extensible
```

**Code:**

```java
// State managed with if-else - hard to maintain and extend
public class VendingMachine {
    private String state;  // "idle", "hasMoney", "dispensing"
    private int money;
    
    public void insertMoney(int amount) {
        if (state.equals("idle")) {
            money += amount;
            state = "hasMoney";
            System.out.println("Money inserted: $" + amount);
        } else if (state.equals("hasMoney")) {
            money += amount;
            System.out.println("More money inserted: $" + amount);
        } else {
            System.out.println("Cannot insert money in " + state + " state");
        }
    }
    
    public void selectProduct(String product) {
        if (state.equals("hasMoney")) {
            System.out.println("Product selected: " + product);
            state = "dispensing";
            dispenseProduct(product);
        } else {
            System.out.println("Please insert money first");
        }
    }
    
    public void dispenseProduct(String product) {
        if (state.equals("dispensing")) {
            System.out.println("Dispensing: " + product);
            money = 0;
            state = "idle";
        }
    }
    
    // Problem: Adding new state requires modifying all methods
}
```

### ✅ State Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│      VendingMachineState                    │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + insertMoney(machine, amount): void        │
│ + selectProduct(machine, product): void     │
│ + dispenseProduct(machine, product): void   │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴──────────────────────────┬──────────────┐
    │                               │              │
┌───┴──────────────────┐ ┌──────────┴──────────┐ ┌─┴───────────────────┐
│ Idle                 │ │ HasMoney            │ │ Dispensing          │
│ State                │ │ State               │ │ State               │
├──────────────────────┤ ├─────────────────────┤ ├─────────────────────┤
│ +insertMoney(machine,│ │+insertMoney(machine,│ │+insertMoney(machine,│
│   amount): void      │ │   amount): void     │ │   amount): void     │
│ +selectProduct(      │ │ +selectProduct(     │ │ +selectProduct(     │
│   machine, product): │ │   machine, product):│ │   machine, product):│
│   void               │ │   void              │ │   void              │
│ +dispenseProduct(    │ │ +dispenseProduct(   │ │ +dispenseProduct(   │
│   machine, product): │ │   machine, product):│ │   machine, product):│
│   void               │ │   void              │ │   void              │
└──────────────────────┘ └─────────────────────┘ └─────────────────────┘
    │                     │                      │
    └─────────────────────┴──────────────────────┘
                   │
                   │ uses/has-a
                   ▼
┌─────────────────────────────────────────────┐
│         VendingMachine                      │
│         (Context)                           │
├─────────────────────────────────────────────┤
│ - VendingMachineState state                 │
│ - int money                                 │
│ - String selectedProduct                    │
├─────────────────────────────────────────────┤
│ + setState(state): void                     │
│ + addMoney(amount): void                    │
│ + insertMoney(amount): void                 │
│ + selectProduct(product): void              │
│ + dispenseProduct(product): void            │
└─────────────────────────────────────────────┘

✅ Benefits:
- State-specific behavior encapsulated
- Easy to add new states
- No complex if-else chains
- State transitions are clear
```

**Code:**

```java
// ✅ State interface
public interface VendingMachineState {
    void insertMoney(VendingMachine machine, int amount);
    void selectProduct(VendingMachine machine, String product);
    void dispenseProduct(VendingMachine machine, String product);
}

// ✅ Concrete states
public class IdleState implements VendingMachineState {
    @Override
    public void insertMoney(VendingMachine machine, int amount) {
        machine.addMoney(amount);
        machine.setState(new HasMoneyState());
        System.out.println("Money inserted: $" + amount);
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("Please insert money first");
    }
    
    @Override
    public void dispenseProduct(VendingMachine machine, String product) {
        System.out.println("Please insert money and select product first");
    }
}

public class HasMoneyState implements VendingMachineState {
    @Override
    public void insertMoney(VendingMachine machine, int amount) {
        machine.addMoney(amount);
        System.out.println("More money inserted: $" + amount);
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("Product selected: " + product);
        machine.setSelectedProduct(product);
        machine.setState(new DispensingState());
    }
    
    @Override
    public void dispenseProduct(VendingMachine machine, String product) {
        System.out.println("Please select a product first");
    }
}

public class DispensingState implements VendingMachineState {
    @Override
    public void insertMoney(VendingMachine machine, int amount) {
        System.out.println("Please wait, product is being dispensed");
    }
    
    @Override
    public void selectProduct(VendingMachine machine, String product) {
        System.out.println("Please wait, product is being dispensed");
    }
    
    @Override
    public void dispenseProduct(VendingMachine machine, String product) {
        System.out.println("Dispensing: " + product);
        machine.resetMoney();
        machine.setState(new IdleState());
    }
}

// ✅ Context class
public class VendingMachine {
    private VendingMachineState state;
    private int money;
    private String selectedProduct;
    
    public VendingMachine() {
        this.state = new IdleState();
        this.money = 0;
    }
    
    public void setState(VendingMachineState state) {
        this.state = state;
    }
    
    public void addMoney(int amount) {
        this.money += amount;
    }
    
    public void resetMoney() {
        this.money = 0;
    }
    
    public void setSelectedProduct(String product) {
        this.selectedProduct = product;
    }
    
    // Delegate to current state
    public void insertMoney(int amount) {
        state.insertMoney(this, amount);
    }
    
    public void selectProduct(String product) {
        state.selectProduct(this, product);
    }
    
    public void dispenseProduct(String product) {
        state.dispenseProduct(this, product);
    }
}

// ✅ Usage - state transitions are clean
VendingMachine machine = new VendingMachine();

machine.insertMoney(5);        // Idle -> HasMoney
machine.selectProduct("Coke"); // HasMoney -> Dispensing
machine.dispenseProduct("Coke"); // Dispensing -> Idle
```

**Benefits:**
- ✅ State-specific behavior encapsulated
- ✅ Easy to add new states
- ✅ No complex if-else chains
- ✅ State transitions are clear

---

## 11. Composite Pattern

**Problem:** Need to represent part-whole hierarchies (treat individual objects and compositions uniformly).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         File                                │
├─────────────────────────────────────────────┤
│ - String name                               │
│ - int size                                  │
├─────────────────────────────────────────────┤
│ + getName(): String                         │
│ + getSize(): int                            │
│ + print(): void                             │
└─────────────────────────────────────────────┘
      ▲
      │ extends
   ┌──┴────────────────────┐
┌──┴─────────────────┐ ┌───┴────────────────┐
│                    │ │                    │
│ FileItem           │ │  Directory         │
│                    │ │                    │
├────────────────────┤ ├────────────────────┤
│ +print()           │ │ - List<File> files │
│                    │ │ +addFile()         │
│                    │ │ +removeFile()      │
│                    │ │ +print()           │
└────────────────────┘ └────────────────────┘

❌ Problems:
- Different interfaces for files and directories
- Client must know if it's a file or directory
- Can't treat files and directories uniformly
- Complex code for traversing directory structure which client/caller has to write. Composite pattern makes the callee write it(sort of in a recursive way).
```

**Code:**

```java
// Different classes with different interfaces
public class FileItem {
    private String name;
    private int size;
    
    public FileItem(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    public String getName() {
        return name;
    }
    
    public int getSize() {
        return size;
    }
    
    public void print() {
        System.out.println("File: " + name + " (" + size + " bytes)");
    }
}

public class Directory {
    private String name;
    private List<FileItem> files = new ArrayList<>();
    
    public Directory(String name) {
        this.name = name;
    }
    
    public void addFile(FileItem file) {
        files.add(file);
    }
    
    public void print() {
        System.out.println("Directory: " + name);
        for (FileItem file : files) {
            file.print();
        }
    }
    
    // Problem: Can't add directories to directories easily
    // Problem: Client code must check type before calling methods
}

// Usage - client must know the type
FileItem file = new FileItem("document.txt", 1024);
Directory dir = new Directory("documents");
dir.addFile(file);

// Can't treat them uniformly
if (file instanceof FileItem) {
    file.print();
} else if (file instanceof Directory) {
    // Different method or handling needed
}
```

### ✅ Composite Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         FileSystemComponent                 │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + getName(): String                         │
│ + getSize(): int                            │
│ + print(): void                             │
│ + add(component): void                      │
│ + remove(component): void                   │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────────────┐
    │                 │             
┌───┴─────────┐ ┌─────┴──────────┐
│ File        │ │ Directory      │
│ (Leaf)      │ │ (Composite)    │
├─────────────┤ ├────────────────┤
│ - String    │ │ - String name  │
│   name      │ │ - List<        │
│ - int size  │ │   FileSystem   │
│             │ │   Component>   │
│             │ │   children     │
├─────────────┤ ├────────────────┤
│ +getName()  │ │ +getName()     │
│ +getSize()  │ │ +getSize()     │
│ +print()    │ │ +print()       │
│ +add()      │ │ +add()         │
│   (throws)  │ │ +remove()      │
│ +remove()   │ │                │
│   (throws)  │ │                │
└─────────────┘ └────────────────┘
```

**Code:**

```java
// ✅ Common interface for both files and directories
public interface FileSystemComponent {
    String getName();
    int getSize();
    void print();
    void add(FileSystemComponent component);
    void remove(FileSystemComponent component);
}

// ✅ Leaf - represents individual files
public class File implements FileSystemComponent {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public int getSize() {
        return size;
    }
    
    @Override
    public void print() {
        System.out.println("File: " + name + " (" + size + " bytes)");
    }
    
    @Override
    public void add(FileSystemComponent component) {
        throw new UnsupportedOperationException("Cannot add to a file");
    }
    
    @Override
    public void remove(FileSystemComponent component) {
        throw new UnsupportedOperationException("Cannot remove from a file");
    }
}

// ✅ Composite - represents directories
public class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> children = new ArrayList<>();
    
    public Directory(String name) {
        this.name = name;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public int getSize() {
        int totalSize = 0;
        for (FileSystemComponent component : children) {
            totalSize += component.getSize();
        }
        return totalSize;
    }
    
    @Override
    public void print() {
        System.out.println("Directory: " + name);
        for (FileSystemComponent component : children) {
            component.print(); // Recursive call
        }
    }
    
    @Override
    public void add(FileSystemComponent component) {
        children.add(component);
    }
    
    @Override
    public void remove(FileSystemComponent component) {
        children.remove(component);
    }
}

// ✅ Usage - treat files and directories uniformly
FileSystemComponent file1 = new File("document.txt", 1024);
FileSystemComponent file2 = new File("image.jpg", 2048);
FileSystemComponent dir1 = new Directory("documents");
FileSystemComponent dir2 = new Directory("images");

dir1.add(file1);
dir2.add(file2);

FileSystemComponent root = new Directory("root");
root.add(dir1);
root.add(dir2);

// Can treat all components uniformly
root.print();        // Prints entire tree structure
System.out.println("Total size: " + root.getSize());
```

**Benefits:**
- ✅ Treat individual objects and compositions uniformly
- ✅ Simplifies client code (no type checking needed)
- ✅ Easy to add new component types
- ✅ Recursive structure support

---

## 12. Chain of Responsibility Pattern

**Problem:** Need to pass requests along a chain of handlers (each handler decides to process or pass to next).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Logger                              │
├─────────────────────────────────────────────┤
│ + log(level, message): void                 │
└─────────────────────────────────────────────┘

❌ Problems:
- All log levels handled in one place
- Hard to add new log handlers
- Violates Single Responsibility Principle
- Can't easily change log processing order
```

**Code:**

```java
// All log handling in one class
public class Logger {
    public void log(String level, String message) {
        if (level.equals("DEBUG")) {
            System.out.println("[DEBUG] " + message);
        } else if (level.equals("INFO")) {
            System.out.println("[INFO] " + message);
            // Also write to file
            writeToFile("[INFO] " + message);
        } else if (level.equals("ERROR")) {
            System.err.println("[ERROR] " + message);
            writeToFile("[ERROR] " + message);
            sendEmail("[ERROR] " + message);
        }
        // Problem: Adding new level requires modifying this class
        // Problem: Can't easily change processing order
    }
    
    private void writeToFile(String message) {
        // File writing logic
    }
    
    private void sendEmail(String message) {
        // Email sending logic
    }
}
```

### ✅ Chain of Responsibility Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         LogHandler                          │
│          <<abstract>>                       │
├─────────────────────────────────────────────┤
│ # LogHandler nextHandler                    │
├─────────────────────────────────────────────┤
│ + setNext(handler): void                    │
│ + handle(level, message): void              │
│ # canHandle(level): boolean                 │
│   (abstract)                                │
│ # doHandle(message): void                   │
│   (abstract)                                │
└─────────────────────────────────────────────┘
         ▲
         │ extends
         │
    ┌────┴──────────────────────────┬─────────────┐
    │                               │             │
┌───┴──────────────┐ ┌──────────────┴────┐ ┌──────┴─────────────┐
│ Debug            │ │ Info              │ │ Error              │
│ Handler          │ │ Handler           │ │ Handler            │
├──────────────────┤ ├───────────────────┤ ├────────────────────┤
│#canHandle(level):│ │ #canHandle(level):│ │ #canHandle(level): │
│  boolean         │ │   boolean         │ │   boolean          │
│#doHandle(        │ │ #doHandle(        │ │ #doHandle(         │
│  message): void  │ │   message): void  │ │   message): void   │
└──────────────────┘ └───────────────────┘ └────────────────────┘
```

**Code:**

```java
// ✅ Abstract handler
public abstract class LogHandler {
    protected LogHandler nextHandler;
    
    public void setNext(LogHandler handler) {
        this.nextHandler = handler;
    }
    
    public void handle(String level, String message) {
        if (canHandle(level)) {
            doHandle(message);
        }
        if (nextHandler != null) {
            nextHandler.handle(level, message);
        }
    }
    
    protected abstract boolean canHandle(String level);
    protected abstract void doHandle(String message);
}

// ✅ Concrete handlers
public class DebugHandler extends LogHandler {
    @Override
    protected boolean canHandle(String level) {
        return "DEBUG".equals(level);
    }
    
    @Override
    protected void doHandle(String message) {
        System.out.println("[DEBUG] " + message);
    }
}

public class InfoHandler extends LogHandler {
    @Override
    protected boolean canHandle(String level) {
        return "INFO".equals(level);
    }
    
    @Override
    protected void doHandle(String message) {
        System.out.println("[INFO] " + message);
        writeToFile("[INFO] " + message);
    }
    
    private void writeToFile(String message) {
        // File writing logic
    }
}

public class ErrorHandler extends LogHandler {
    @Override
    protected boolean canHandle(String level) {
        return "ERROR".equals(level);
    }
    
    @Override
    protected void doHandle(String message) {
        System.err.println("[ERROR] " + message);
        writeToFile("[ERROR] " + message);
        sendEmail("[ERROR] " + message);
    }
    
    private void writeToFile(String message) {
        // File writing logic
    }
    
    private void sendEmail(String message) {
        // Email sending logic
    }
}

// ✅ Usage - chain handlers together
LogHandler debugHandler = new DebugHandler();
LogHandler infoHandler = new InfoHandler();
LogHandler errorHandler = new ErrorHandler();

debugHandler.setNext(infoHandler);
infoHandler.setNext(errorHandler);

// All handlers process the request
debugHandler.handle("ERROR", "Something went wrong");
// Output:
// [ERROR] Something went wrong
// [ERROR] Something went wrong (to file)
// [ERROR] Something went wrong (email sent)
```

**Benefits:**
- ✅ Decouples sender from receiver
- ✅ Easy to add/remove handlers
- ✅ Dynamic chain composition
- ✅ Single Responsibility Principle

**Why Chain of Responsibility over Strategy Pattern for Logger?**

**Key Difference:**
- **Strategy Pattern (DebugStrategy, InfoStrategy, ErrorStrategy)**: Client must SELECT which strategy to use based on log level
- **Chain of Responsibility**: Client sends message once, chain automatically routes to appropriate handlers

---

## 13. Abstract Factory Pattern

**Problem:** Need to create families of related objects without specifying concrete classes.

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         UIFactory                           │
├─────────────────────────────────────────────┤
│ + createButton(): Button                    │
│ + createDialog(): Dialog                    │
└─────────────────────────────────────────────┘
         │
         │ creates
         │
    ┌────┴────┐
    │         │            
┌───┴────┐ ┌──┴──────┐
│Button  │ │ Dialog  │
└────────┘ └─────────┘

❌ Problems:
- Can't ensure UI components match (Windows button with Mac dialog)
- Hard to add new UI themes
- Client must know about all concrete classes
```

**Code:**

```java
// Problem: Can create mismatched UI components
public class UIFactory {
    public Button createButton(String type) {
        if (type.equals("Windows")) {
            return new WindowsButton();
        } else if (type.equals("Mac")) {
            return new MacButton();
        }
        return null;
    }
    
    public Dialog createDialog(String type) {
        if (type.equals("Windows")) {
            return new WindowsDialog();
        } else if (type.equals("Mac")) {
            return new MacDialog();
        }
        return null;
    }
}

// Problem: Can accidentally create mismatched components
UIFactory factory = new UIFactory();
Button button = factory.createButton("Windows");
Dialog dialog = factory.createDialog("Mac"); // Mismatch!
```

### ✅ Abstract Factory Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         UIFactory                           │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + createButton(): Button                    │
│ + createDialog(): Dialog                    │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴───────────────────┐
    │                        │
┌───┴───────────┐ ┌──────────┴────┐
│ Windows       │ │ Mac           │
│ UIFactory     │ │ UIFactory     │
├───────────────┤ ├───────────────┤
│+createButton()│ │+createButton()│
│+createDialog()│ │+createDialog()│
└───────────────┘ └───────────────┘
    │                  │
  creates            creates
  Windows            Mac
  Button &           Button & 
  Dialog             Dialog
                     
┌────────────────────────────┐
│         Button             │
│      <<interface>>         │
├────────────────────────────┤
│ + render(): void           │
└────────────────────────────┘
    ▲                  ▲
    │ implements       │ implements
    │                  │
┌───┴────┐         ┌───┴────┐
│Windows │         │ Mac    │
│Button  │         │Button  │
└────────┘         └────────┘

┌─────────────────────────────────────────────┐
│         Dialog                              │
│      <<interface>>                          │
├─────────────────────────────────────────────┤
│ + show(): void                              │
└─────────────────────────────────────────────┘
    ▲                  ▲
    │ implements       │ implements
    │                  │
┌───┴────┐         ┌───┴────┐
│Windows │         │ Mac    │
│Dialog  │         │Dialog  │
└────────┘         └────────┘
```

**Code:**

```java
// ✅ Abstract products
public interface Button {
    void render();
}

public interface Dialog {
    void show();
}

// ✅ Concrete products - Windows family
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

public class WindowsDialog implements Dialog {
    @Override
    public void show() {
        System.out.println("Showing Windows dialog");
    }
}

// ✅ Concrete products - Mac family
public class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

public class MacDialog implements Dialog {
    @Override
    public void show() {
        System.out.println("Showing Mac dialog");
    }
}

// ✅ Abstract factory
public interface UIFactory {
    Button createButton();
    Dialog createDialog();
}

// ✅ Concrete factories
public class WindowsUIFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Dialog createDialog() {
        return new WindowsDialog();
    }
}

public class MacUIFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Dialog createDialog() {
        return new MacDialog();
    }
}

// ✅ Usage - ensures matching components
UIFactory factory = new WindowsUIFactory();
Button button = factory.createButton();
Dialog dialog = factory.createDialog();

button.render();  // Windows button
dialog.show();    // Windows dialog - guaranteed to match!
```

**Benefits:**
- ✅ Ensures products from same family are used together
- ✅ Isolates concrete classes from client
- ✅ Easy to add new product families
- ✅ Promotes consistency

---

## 14. Facade Pattern

**Problem:** Need to provide a simplified interface to a complex subsystem.

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Client                              │
├─────────────────────────────────────────────┤
│ + watchMovie(): void                        │
└─────────────────────────────────────────────┘
         │
         │ directly uses
         │
    ┌────┴──────────────────────────┬─────────────┐
    │                               │             │
┌───┴──────────────┐ ┌──────────────┴────┐ ┌──────┴─────────────┐
│ DVD              │ │ Projector         │ │ Sound              │
│ Player           │ │                   │ │ System             │
├──────────────────┤ ├───────────────────┤ ├────────────────────┤
│ +on(): void      │ │ +on(): void       │ │ +on(): void        │
│ +play(): void    │ │ +setInput(input): │ │ +setVolume(volume):│
│ +off(): void     │ │   void            │ │   void             │
│                  │ │ +off(): void      │ │ +off(): void       │
└──────────────────┘ └───────────────────┘ └────────────────────┘

❌ Problems:
- Client must know about all subsystems
- Complex setup code in client
- Tight coupling
- Hard to change subsystem
```

**Code:**

```java
// Client must interact with all subsystems directly
public class HomeTheaterClient {
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private SoundSystem soundSystem;
    
    public void watchMovie() {
        // Complex setup - client must know all steps
        dvdPlayer.on();
        projector.on();
        projector.setInput("DVD");
        soundSystem.on();
        soundSystem.setVolume(10);
        dvdPlayer.play();
        // Problem: Too many steps, easy to forget one
    }
}
```

### ✅ Facade Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Client                              │
├─────────────────────────────────────────────┤
│ + watchMovie(): void                        │
└─────────────────────────────────────────────┘
         │
         │ uses simplified interface
         │
┌─────────────────────────────────────────────┐
│         HomeTheaterFacade                   │
├─────────────────────────────────────────────┤
│ - DVDPlayer dvdPlayer                       │
│ - Projector projector                       │
│ - SoundSystem soundSystem                   │
├─────────────────────────────────────────────┤
│ + watchMovie(): void                        │
│ + endMovie(): void                          │
│ + listenToMusic(): void                     │
└─────────────────────────────────────────────┘
         │
         │ uses
         │
    ┌────┴────┬────────────┐
    │         │            │             
┌───┴────┐ ┌──┴──────┐ ┌───┴──────┐
│DVD     │ │Projector│ │Sound     │
│Player  │ │         │ │System    │
└────────┘ └─────────┘ └──────────┘
```

**Code:**

```java
// ✅ Subsystem classes
public class DVDPlayer {
    public void on() {
        System.out.println("DVD Player on");
    }
    
    public void play() {
        System.out.println("Playing movie");
    }
    
    public void off() {
        System.out.println("DVD Player off");
    }
}

public class Projector {
    public void on() {
        System.out.println("Projector on");
    }
    
    public void setInput(String input) {
        System.out.println("Projector input set to " + input);
    }
    
    public void off() {
        System.out.println("Projector off");
    }
}

public class SoundSystem {
    public void on() {
        System.out.println("Sound system on");
    }
    
    public void setVolume(int volume) {
        System.out.println("Volume set to " + volume);
    }
    
    public void off() {
        System.out.println("Sound system off");
    }
}

// ✅ Facade - simplified interface
public class HomeTheaterFacade {
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private SoundSystem soundSystem;
    
    public HomeTheaterFacade(DVDPlayer dvd, Projector proj, SoundSystem sound) {
        this.dvdPlayer = dvd;
        this.projector = proj;
        this.soundSystem = sound;
    }
    
    public void watchMovie() {
        System.out.println("Get ready to watch a movie...");
        dvdPlayer.on();
        projector.on();
        projector.setInput("DVD");
        soundSystem.on();
        soundSystem.setVolume(10);
        dvdPlayer.play();
    }
    
    public void endMovie() {
        System.out.println("Shutting movie theater down...");
        dvdPlayer.off();
        projector.off();
        soundSystem.off();
    }
    
    public void listenToMusic() {
        System.out.println("Getting ready for music...");
        soundSystem.on();
        soundSystem.setVolume(5);
    }
}

// ✅ Usage - simple interface
HomeTheaterFacade theater = new HomeTheaterFacade(
    new DVDPlayer(), new Projector(), new SoundSystem()
);

theater.watchMovie();  // One simple call!
theater.endMovie();
```

**Benefits:**
- ✅ Simplifies complex subsystem
- ✅ Reduces coupling between client and subsystem
- ✅ Provides single entry point
- ✅ Easy to change subsystem without affecting client

---

## 15. Proxy Pattern

**Problem:** Need to control access to an object (lazy loading, access control, logging).

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Client                              │
├─────────────────────────────────────────────┤
│ + loadImage(): void                         │
└─────────────────────────────────────────────┘
         │
         │ directly uses
         │
┌─────────────────────────────────────────────┐
│         Image                               │
├─────────────────────────────────────────────┤
│ - String filename                           │
├─────────────────────────────────────────────┤
│ + Image(filename)                           │
│   (loads immediately)                       │
│ + display(): void                           │
└─────────────────────────────────────────────┘

❌ Problems:
- Image loaded even if not displayed
- No access control
- No caching
- Wastes memory
```

**Code:**

```java
// Image loads immediately, even if not needed
public class Image {
    private String filename;
    
    public Image(String filename) {
        this.filename = filename;
        loadFromDisk(); // Expensive operation - loads immediately
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + filename + " from disk...");
        // Expensive I/O operation
    }
    
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

// Problem: Image loaded even if never displayed
Image image = new Image("large-photo.jpg"); // Loads immediately!
// ... later, maybe we display it, maybe we don't
```

### ✅ Proxy Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Image                               │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + display(): void                           │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴─────────────────┐
    │                      │
┌───┴─────────┐ ┌──────────┴────┐
│ RealImage   │ │ ImageProxy    │
├─────────────┤ ├───────────────┤
│ - String    │ │ - String      │
│   filename  │ │   filename    │
│ - RealImage │ │ - RealImage   │
│   realImage │ │   realImage   │
├─────────────┤ ├───────────────┤
│ +display()  │ │ +display()    │
│             │ │   (lazy load) │
└─────────────┘ └───────────────┘
```

**Code:**

```java
// ✅ Subject interface
public interface Image {
    void display();
}

// ✅ Real subject - expensive to create
public class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + filename + " from disk...");
        // Expensive I/O operation
    }
    
    @Override
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

// ✅ Proxy - controls access to real image
public class ImageProxy implements Image {
    private String filename;
    private RealImage realImage;
    
    public ImageProxy(String filename) {
        this.filename = filename;
        // Doesn't load image yet!
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename); // Lazy loading
        }
        realImage.display();
    }
}

// ✅ Usage - image only loaded when needed
Image image = new ImageProxy("large-photo.jpg"); // No loading yet!
// ... do other things ...
image.display(); // Now it loads!
image.display(); // Uses cached instance
```

**Benefits:**
- ✅ Lazy loading (load only when needed)
- ✅ Access control (can add permission checks)
- ✅ Caching (reuse expensive objects)

---

## 16. Iterator Pattern

**Problem:** Need to traverse elements of a collection without exposing its internal structure.

### ❌ Bad Code

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Playlist                            │
├─────────────────────────────────────────────┤
│ - List<Song> songs                          │
├─────────────────────────────────────────────┤
│ + getSongs(): List<Song>                    │
└─────────────────────────────────────────────┘

❌ Problems:
- Exposes internal data structure
- Client must know implementation details
- Can't change implementation without breaking client
- No way to iterate in different ways
```

**Code:**

```java
// Exposes internal structure
public class Playlist {
    private List<Song> songs = new ArrayList<>();
    
    public void addSong(Song song) {
        songs.add(song);
    }
    
    public List<Song> getSongs() {
        return songs; // Exposes internal structure!
    }
}

// Client must know it's a List
Playlist playlist = new Playlist();
playlist.addSong(new Song("Song 1"));
playlist.addSong(new Song("Song 2"));

List<Song> songs = playlist.getSongs();
for (Song song : songs) { // Client knows it's a List
    System.out.println(song.getName());
}
// Problem: If we change to Set or array, client breaks
```

### ✅ Iterator Pattern

**UML Class Diagram:**

```
┌─────────────────────────────────────────────┐
│         Iterator<T>                         │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + hasNext(): boolean                        │
│ + next(): T                                 │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
┌─────────────────────────────────────────────┐
│         PlaylistIterator                    │
├─────────────────────────────────────────────┤
│ - List<Song> songs                          │
│ - int position                              │
├─────────────────────────────────────────────┤
│ + hasNext(): boolean                        │
│ + next(): Song                              │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│         Iterable<T>                         │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + iterator(): Iterator<T>                   │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
┌─────────────────────────────────────────────┐
│         Playlist                            │
├─────────────────────────────────────────────┤
│ - List<Song> songs                          │
├─────────────────────────────────────────────┤
│ + addSong(song): void                       │
│ + iterator(): Iterator<Song>                │
└─────────────────────────────────────────────┘
```

**Code:**

```java
// ✅ Iterator interface
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

// ✅ Iterable interface
public interface Iterable<T> {
    Iterator<T> iterator();
}

// ✅ Song class
public class Song {
    private String name;
    
    public Song(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}

// ✅ Concrete iterator
public class PlaylistIterator implements Iterator<Song> {
    private List<Song> songs;
    private int position = 0;
    
    public PlaylistIterator(List<Song> songs) {
        this.songs = songs;
    }
    
    @Override
    public boolean hasNext() {
        return position < songs.size();
    }
    
    @Override
    public Song next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return songs.get(position++);
    }
}

// ✅ Collection with iterator
public class Playlist implements Iterable<Song> {
    private List<Song> songs = new ArrayList<>();
    
    public void addSong(Song song) {
        songs.add(song);
    }
    
    @Override
    public Iterator<Song> iterator() {
        return new PlaylistIterator(songs);
    }
}

// ✅ Usage - client doesn't know internal structure
Playlist playlist = new Playlist();
playlist.addSong(new Song("Song 1"));
playlist.addSong(new Song("Song 2"));

Iterator<Song> iterator = playlist.iterator();
while (iterator.hasNext()) {
    Song song = iterator.next();
    System.out.println(song.getName());
}

// Or using enhanced for loop (if Iterable is implemented)
for (Song song : playlist) {
    System.out.println(song.getName());
}
```

**Benefits:**
- ✅ Hides internal structure
- ✅ Multiple iteration strategies
- ✅ Uniform interface for different collections
- ✅ Easy to change collection implementation

---

## 17. Mediator Pattern

**Problem:** Need to reduce coupling between multiple objects that communicate with each other.

### ❌ Bad Code

**UML Class Diagram:**

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  User1   │──────│  User2   │──────│  User3   │
└──────────┘      └──────────┘      └──────────┘
     │                 │                 │
     └─────────────────┴─────────────────┘
              (direct communication)

❌ Problems:
- Tight coupling between users
- Each user must know about all other users
- Hard to add/remove users
- Complex communication logic
```

**Code:**

```java
// Users directly communicate with each other
public class User {
    private String name;
    private List<User> users = new ArrayList<>();
    
    public User(String name) {
        this.name = name;
    }
    
    public void addUser(User user) {
        users.add(user);
    }
    
    public void sendMessage(String message) {
        System.out.println(name + " sends: " + message);
        // Must send to all other users directly
        for (User user : users) {
            if (user != this) {
                user.receiveMessage(this.name, message);
            }
        }
    }
    
    public void receiveMessage(String sender, String message) {
        System.out.println(name + " receives from " + sender + ": " + message);
    }
}

// Problem: Tight coupling, complex management
User user1 = new User("Alice");
User user2 = new User("Bob");
User user3 = new User("Charlie");

user1.addUser(user2);
user1.addUser(user3);
user2.addUser(user1);
user2.addUser(user3);
// ... complex setup
```

### ✅ Mediator Pattern

**UML Class Diagram:**

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  User1   │      │  User2   │      │  User3   │
└──────────┘      └──────────┘      └──────────┘
     │                 │                 │
     └─────────────────┴─────────────────┘
              (chat mediator)


┌─────────────────────────────────────────────┐
│         ChatMediator                        │
│          <<interface>>                      │
├─────────────────────────────────────────────┤
│ + sendMessage(user, message): void          │
│ + addUser(user): void                       │
└─────────────────────────────────────────────┘
         ▲
         │ implements
         │
┌─────────────────────────────────────────────┐
│         ChatRoom                            │
├─────────────────────────────────────────────┤
│ - List<User> users                          │
├─────────────────────────────────────────────┤
│ + sendMessage(user, message): void          │
│ + addUser(user): void                       │
└─────────────────────────────────────────────┘
         ▲
         │ uses
         │
┌─────────────────────────────────────────────┐
│         User                                │
├─────────────────────────────────────────────┤
│ - String name                               │
│ - ChatMediator mediator                     │
├─────────────────────────────────────────────┤
│ + sendMessage(message): void                │
│ + receiveMessage(sender, message): void     │
└─────────────────────────────────────────────┘
```

**Code:**

```java
// ✅ Mediator interface
public interface ChatMediator {
    void sendMessage(String sender, String message);
    void addUser(User user);
}

// ✅ Concrete mediator
public class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();
    
    @Override
    public void addUser(User user) {
        users.add(user);
    }
    
    @Override
    public void sendMessage(String sender, String message) {
        for (User user : users) {
            if (!user.getName().equals(sender)) {
                user.receiveMessage(sender, message);
            }
        }
    }
}

// ✅ Colleague - users don't know about each other
public class User {
    private String name;
    private ChatMediator mediator;
    
    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
        mediator.addUser(this);
    }
    
    public String getName() {
        return name;
    }
    
    public void sendMessage(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(name, message);
    }
    
    public void receiveMessage(String sender, String message) {
        System.out.println(name + " receives from " + sender + ": " + message);
    }
}

// ✅ Usage - loose coupling
ChatMediator chatRoom = new ChatRoom();

User alice = new User("Alice", chatRoom);
User bob = new User("Bob", chatRoom);
User charlie = new User("Charlie", chatRoom);

alice.sendMessage("Hello everyone!");
// Output:
// Alice sends: Hello everyone!
// Bob receives from Alice: Hello everyone!
// Charlie receives from Alice: Hello everyone!
```

**Benefits:**
- ✅ Reduces coupling between objects
- ✅ Centralizes communication logic
- ✅ Easy to add/remove colleagues
- ✅ Single Responsibility Principle

---

## Summary: When to Use Each Pattern

| Pattern | Use When | Famous LLD Examples |
|---------|----------|---------------------|
| **Singleton** | Need exactly one instance | Database connection, Logger, Configuration manager, Cache manager, Thread pool |
| **Factory** | Don't know exact class at compile time | Notification system (Email/SMS/Push), Payment gateway (CreditCard/PayPal), Document parser (PDF/Word/Excel), UI components (Button/Dialog) |
| **Builder** | Complex object construction with many optional parameters | User object, HTTP request builder, SQL query builder, Pizza builder, Car configuration |
| **Strategy** | Need to switch algorithms at runtime | Payment methods (CreditCard/PayPal/Crypto), Sorting algorithms (Quick/Merge/Heap), Compression (ZIP/RAR/7Z), Discount calculation |
| **Observer** | Need to notify multiple objects of state changes | News agency, Stock market updates, Model-View-Controller (MVC), Event system, Weather station |
| **Decorator** | Need to add behavior dynamically without subclassing | Coffee add-ons (Milk/Sugar/Caramel), Stream processing, I/O streams, Text formatting, Pizza toppings |
| **Adapter** | Need to integrate incompatible interfaces | Legacy system integration, Third-party API wrapper, Media players (MP3/MP4), Payment gateway adapter |
| **Command** | Need undo/redo, queuing, or logging operations | Text editor undo/redo, Remote control, Job scheduler, Macro recording, Transaction system |
| **Template Method** | Have algorithm with steps that vary | Report generation (PDF/HTML/CSV), Data processing pipeline, Game loop, Build process, Test framework |
| **State** | Object behavior changes significantly based on state | Vending machine, Order status (Pending/Shipped/Delivered), Game character states, TCP connection states, ATM states |
| **Composite** | Need to represent part-whole hierarchies | File system (files and directories), UI components (containers and widgets), Organization structure, XML/JSON parsing |
| **Chain of Responsibility** | Need to pass requests along a chain of handlers | Logger system (DEBUG/INFO/ERROR handlers), Request processing pipeline, Exception handling, Middleware in web frameworks |
| **Abstract Factory** | Need to create families of related objects | UI toolkit (Windows/Mac/Linux components), Database drivers, Cross-platform libraries, Theme systems |
| **Facade** | Need to simplify complex subsystem | Home theater system, API wrappers, Library interfaces, System integration layers |
| **Proxy** | Need to control access to an object | Lazy loading, Access control, Caching, Remote objects, Virtual proxy for expensive resources |
| **Iterator** | Need to traverse collections without exposing structure | Playlist traversal, Tree traversal, Custom collection iteration, Database result sets |
| **Mediator** | Need to reduce coupling between communicating objects | Chat applications, Air traffic control, GUI components communication, Event coordination systems |

---

## Key Takeaways

- ✅ **Start simple, refactor when needed** - Don't over-engineer
- ✅ **Patterns solve specific problems** - Use them when they fit
- ✅ **Code smell first, pattern second** - Recognize the problem, then apply pattern
- ✅ **Combine patterns** - Real-world code often uses multiple patterns
- ✅ **Practice** - Understanding comes from implementing

---


## Salesforce Core Design Patterns
- Functional Tests(Ftests) - An integration test framework where the test framework create Core App and its DB and runs automated tests. 
    - Template Design Pattern.
        - ftestSetup() 
            - { setups us the feature permissions and all}
        - run() 
            - { runs the actual test }
        - ftestTearDown 
            - { typically test writer cleans the DB records and resets permissions} 
- Message Queue Framework (MQ framework)
    - Strategy design pattern along with Spring Boot
- SOQL Builder(Salesforce Object Query Language - like SQL)
    - Builder pattern
- Connect API (Wrapper on Rest API)
    - Builder pattern for Input params and output params    
---


