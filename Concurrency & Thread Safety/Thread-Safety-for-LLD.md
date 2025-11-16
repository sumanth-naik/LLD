# Thread Safety & Concurrency for LLD Interviews

Essential concurrency concepts and thread safety patterns for Low-Level Design interviews.

---

## Table of Contents

1. [Race Conditions](#1-race-conditions)
2. [Synchronization](#2-synchronization)
3. [Thread-Safe Collections](#3-thread-safe-collections)
4. [Atomic Operations](#4-atomic-operations)
5. [Deadlocks](#5-deadlocks)
6. [Common Patterns](#6-common-patterns)

---

## 1. Race Conditions

**Problem:** Multiple threads accessing shared data simultaneously can lead to inconsistent state.

### ❌ Bad Code

```java
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // ❌ Not thread-safe!
    }
    
    public int getCount() {
        return count;  // ❌ Can read stale value!
    }
}

// Problem: Multiple threads can call increment() simultaneously
// Thread 1: reads count (0), increments to 1, writes 1
// Thread 2: reads count (0), increments to 1, writes 1
// Result: count = 1 (should be 2!)
```

**What happens:**
- `count++` is NOT atomic (read-modify-write operation)
- Two threads can read the same value before either writes
- Lost updates occur

### ✅ Solution 1: Synchronized Method

```java
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // ✅ Thread-safe
    }
    
    public synchronized int getCount() {
        return count;  // ✅ Thread-safe read
    }
}
```

**How it works:**
- `synchronized` ensures only one thread can execute the method at a time
- Other threads wait until the lock is released
- Prevents race conditions

### ✅ Solution 2: Synchronized Block

```java
public class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized(lock) {
            count++;  // ✅ Thread-safe
        }
    }
    
    public int getCount() {
        synchronized(lock) {
            return count;  // ✅ Thread-safe read
        }
    }
}
```

**Advantages:**
- More granular control
- Can synchronize only critical sections
- Better performance than synchronizing entire method

### ✅ Solution 3: AtomicInteger (Best for counters)

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // ✅ Atomic operation
    }
    
    public int getCount() {
        return count.get();  // ✅ Thread-safe read
    }
}
```

**Advantages:**
- Lock-free implementation (uses CAS - Compare And Swap)
- Better performance than synchronized
- Designed specifically for atomic operations

---

## 2. Synchronization

### Synchronized Methods

```java
public class BankAccount {
    private double balance = 0;
    
    // Synchronized method - locks on 'this'
    public synchronized void deposit(double amount) {
        balance += amount;
    }
    
    public synchronized void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
    
    public synchronized double getBalance() {
        return balance;
    }
}
```

### Synchronized Blocks

```java
public class BankAccount {
    private double balance = 0;
    private final Object lock = new Object();
    
    public void deposit(double amount) {
        synchronized(lock) {
            balance += amount;
        }
    }
    
    public void withdraw(double amount) {
        synchronized(lock) {
            if (balance >= amount) {
                balance -= amount;
            }
        }
    }
}
```

### ReentrantLock (More Control)

```java
import java.util.concurrent.locks.ReentrantLock;

public class BankAccount {
    private double balance = 0;
    private final ReentrantLock lock = new ReentrantLock();
    
    public void deposit(double amount) {
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock();  // Always unlock in finally block
        }
    }
    
    public void withdraw(double amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                balance -= amount;
            }
        } finally {
            lock.unlock();
        }
    }
}
```

**Advantages of ReentrantLock:**
- Can try to acquire lock with timeout
- Can check if lock is held
- More flexible than synchronized

---

## 3. Thread-Safe Collections

### ❌ Bad Code

```java
// ❌ NOT thread-safe
List<String> list = new ArrayList<>();
Map<String, Integer> map = new HashMap<>();

// Multiple threads modifying these will cause issues
list.add("item");  // Can throw ConcurrentModificationException
map.put("key", 1);  // Can lose updates
```

### ✅ Thread-Safe Collections

```java
import java.util.concurrent.*;

// ✅ Thread-safe list
List<String> list = new CopyOnWriteArrayList<>();
list.add("item");  // Thread-safe

// ✅ Thread-safe map
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("key", 1);  // Thread-safe

// ✅ Thread-safe queue
Queue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("item");  // Thread-safe

// ✅ Blocking queue (for producer-consumer)
BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>();
blockingQueue.put("item");  // Blocks if queue is full
String item = blockingQueue.take();  // Blocks if queue is empty
```

**When to use:**
- **CopyOnWriteArrayList**: Read-heavy, write-rarely scenarios
- **ConcurrentHashMap**: General-purpose thread-safe map
- **ConcurrentLinkedQueue**: Lock-free queue
- **BlockingQueue**: Producer-consumer patterns

---

## 4. Atomic Operations

**Problem:** Need atomic operations without full synchronization overhead.

### Atomic Classes

```java
import java.util.concurrent.atomic.*;

public class AtomicExample {
    private AtomicInteger count = new AtomicInteger(0);
    private AtomicLong total = new AtomicLong(0);
    private AtomicReference<String> value = new AtomicReference<>("initial");
    
    public void increment() {
        count.incrementAndGet();  // Atomic increment
    }
    
    public void add(long amount) {
        total.addAndGet(amount);  // Atomic add
    }
    
    public void updateValue(String newValue) {
        value.set(newValue);  // Atomic set
    }
    
    // Compare and swap
    public boolean compareAndSet(String expected, String update) {
        return value.compareAndSet(expected, update);
    }
}
```

### Real-World Example: Inventory Counter

```java
public class Inventory {
    private AtomicInteger stock = new AtomicInteger(100);
    
    public boolean purchase(int quantity) {
        int current;
        int next;
        do {
            current = stock.get();
            if (current < quantity) {
                return false;  // Not enough stock
            }
            next = current - quantity;
        } while (!stock.compareAndSet(current, next));
        
        return true;  // Purchase successful
    }
    
    public void restock(int quantity) {
        stock.addAndGet(quantity);
    }
    
    public int getStock() {
        return stock.get();
    }
}
```

---

## 5. Deadlocks

**Problem:** Two or more threads waiting for each other to release locks.

### ❌ Bad Code (Deadlock Example)

```java
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {
            synchronized(lock2) {
                // Do something
            }
        }
    }
    
    public void method2() {
        synchronized(lock2) {  // ❌ Different order!
            synchronized(lock1) {
                // Do something
            }
        }
    }
}

// Thread 1: acquires lock1, waits for lock2
// Thread 2: acquires lock2, waits for lock1
// DEADLOCK!
```

### ✅ Solution: Consistent Lock Ordering

```java
public class NoDeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {
            synchronized(lock2) {
                // Do something
            }
        }
    }
    
    public void method2() {
        synchronized(lock1) {  // ✅ Same order!
            synchronized(lock2) {
                // Do something
            }
        }
    }
}
```

### ✅ Solution: TryLock with Timeout

```java
import java.util.concurrent.locks.ReentrantLock;

public class SafeLockExample {
    private final ReentrantLock lock1 = new ReentrantLock();
    private final ReentrantLock lock2 = new ReentrantLock();
    
    public void method1() {
        boolean lock1Acquired = false;
        boolean lock2Acquired = false;
        
        try {
            lock1Acquired = lock1.tryLock(100, TimeUnit.MILLISECONDS);
            if (lock1Acquired) {
                lock2Acquired = lock2.tryLock(100, TimeUnit.MILLISECONDS);
                if (lock2Acquired) {
                    // Do something
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock2Acquired) lock2.unlock();
            if (lock1Acquired) lock1.unlock();
        }
    }
}
```

---

## 6. Common Patterns

### Pattern 1: Thread-Safe Singleton

```java
public class ThreadSafeSingleton {
    private static volatile ThreadSafeSingleton instance;
    
    private ThreadSafeSingleton() {}
    
    public static ThreadSafeSingleton getInstance() {
        if (instance == null) {
            synchronized(ThreadSafeSingleton.class) {
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
}
```

**Key points:**
- Double-checked locking
- `volatile` ensures visibility across threads
- Synchronized block prevents multiple instantiation

### Pattern 2: Producer-Consumer

```java
import java.util.concurrent.BlockingQueue;

public class ProducerConsumer {
    private BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
    
    // Producer
    public void produce(String item) throws InterruptedException {
        queue.put(item);  // Blocks if queue is full
        System.out.println("Produced: " + item);
    }
    
    // Consumer
    public String consume() throws InterruptedException {
        String item = queue.take();  // Blocks if queue is empty
        System.out.println("Consumed: " + item);
        return item;
    }
}
```

### Pattern 3: Thread-Safe Cache

```java
import java.util.concurrent.ConcurrentHashMap;

public class ThreadSafeCache<K, V> {
    private final ConcurrentHashMap<K, V> cache = new ConcurrentHashMap<>();
    
    public V get(K key) {
        return cache.get(key);
    }
    
    public void put(K key, V value) {
        cache.put(key, value);
    }
    
    // Thread-safe compute-if-absent
    public V getOrCompute(K key, Function<K, V> computeFunction) {
        return cache.computeIfAbsent(key, computeFunction);
    }
}
```

### Pattern 4: Read-Write Lock

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockExample {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private String data = "initial";
    
    // Multiple readers can read simultaneously
    public String read() {
        lock.readLock().lock();
        try {
            return data;
        } finally {
            lock.readLock().unlock();
        }
    }
    
    // Only one writer at a time
    public void write(String newData) {
        lock.writeLock().lock();
        try {
            data = newData;
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

**When to use:**
- Read-heavy, write-rarely scenarios
- Better performance than full synchronization
- Allows concurrent reads

---

## Common Interview Scenarios

### Scenario 1: Thread-Safe Counter

```java
public class ThreadSafeCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public void decrement() {
        count.decrementAndGet();
    }
    
    public int get() {
        return count.get();
    }
}
```

### Scenario 2: Thread-Safe Stack

```java
import java.util.concurrent.ConcurrentLinkedStack;

public class ThreadSafeStack<T> {
    private final ConcurrentLinkedStack<T> stack = new ConcurrentLinkedStack<>();
    
    public void push(T item) {
        stack.push(item);
    }
    
    public T pop() {
        return stack.pop();
    }
    
    public boolean isEmpty() {
        return stack.isEmpty();
    }
}
```

### Scenario 3: Thread-Safe LRU Cache

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ThreadSafeLRUCache<K, V> {
    private final int capacity;
    private final ConcurrentHashMap<K, Node<K, V>> map;
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private Node<K, V> head, tail;
    
    public ThreadSafeLRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new ConcurrentHashMap<>();
    }
    
    public V get(K key) {
        lock.readLock().lock();
        try {
            Node<K, V> node = map.get(key);
            if (node == null) return null;
            moveToHead(node);
            return node.value;
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void put(K key, V value) {
        lock.writeLock().lock();
        try {
            // Implementation details...
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    private void moveToHead(Node<K, V> node) {
        // Move node to head of doubly linked list
    }
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev, next;
    }
}
```

---

## Key Takeaways

1. **Use synchronized** for simple thread safety
2. **Use Atomic classes** for counters and simple operations
3. **Use ConcurrentHashMap** for thread-safe maps
4. **Use BlockingQueue** for producer-consumer patterns
5. **Always acquire locks in same order** to prevent deadlocks
6. **Use ReadWriteLock** for read-heavy scenarios
7. **Always unlock in finally block** when using ReentrantLock

---

## Quick Reference

| Scenario | Solution |
|----------|----------|
| Counter | `AtomicInteger` |
| Map | `ConcurrentHashMap` |
| List | `CopyOnWriteArrayList` (read-heavy) |
| Queue | `ConcurrentLinkedQueue` or `BlockingQueue` |
| Simple synchronization | `synchronized` keyword |
| Fine-grained control | `ReentrantLock` |
| Read-heavy, write-rarely | `ReadWriteLock` |
| Multiple locks | Consistent ordering or `tryLock` |

---

## LLD Interview Questions Using These Concepts

### Questions Requiring Thread Safety:

**1. Design a Parking Lot System**
- **Concepts:** Synchronized blocks, ConcurrentHashMap
- **Why:** Multiple vehicles entering/exiting simultaneously, concurrent spot allocation
- **Use:** Thread-safe spot allocation, atomic operations for spot booking

**2. Design an Elevator System**
- **Concepts:** Synchronized, BlockingQueue, Concurrent access
- **Why:** Multiple floor requests simultaneously, elevator car button presses
- **Use:** Thread-safe request queue, synchronized elevator state

**3. Design a Vending Machine**
- **Concepts:** Synchronized, AtomicInteger, State management
- **Why:** Multiple users trying to purchase simultaneously
- **Use:** Thread-safe inventory counter, synchronized state transitions

**4. Design an Online Shopping System**
- **Concepts:** Synchronized, AtomicInteger, ConcurrentHashMap
- **Why:** Concurrent orders, inventory management, cart operations
- **Use:** Thread-safe inventory updates, atomic order creation

**5. Design a Movie Ticket Booking System**
- **Concepts:** Synchronized, Atomic operations, Lock ordering
- **Why:** Multiple users booking same seats simultaneously
- **Use:** Thread-safe seat allocation, prevent double booking

**6. Design a Library Management System**
- **Concepts:** Synchronized, ConcurrentHashMap
- **Why:** Multiple members trying to issue same book
- **Use:** Thread-safe book availability, atomic issue/return operations

**7. Design a File System**
- **Concepts:** ReadWriteLock, Synchronized
- **Why:** Concurrent file operations (read/write)
- **Use:** ReadWriteLock for multiple readers, synchronized for writes

**8. Design a Cache System (LRU Cache)**
- **Concepts:** ConcurrentHashMap, Synchronized, ReentrantReadWriteLock
- **Why:** Concurrent cache access, eviction operations
- **Use:** Thread-safe cache operations, synchronized eviction

**9. Design a Logger System**
- **Concepts:** Concurrent collections, Synchronized appenders
- **Why:** Multiple threads logging simultaneously
- **Use:** Thread-safe log appenders, concurrent log queue

**10. Design a Task Scheduler**
- **Concepts:** BlockingQueue, Synchronized, Atomic operations
- **Why:** Concurrent task submission and execution
- **Use:** Thread-safe task queue, atomic task state updates

**11. Design a Rate Limiter**
- **Concepts:** AtomicInteger, Synchronized, ConcurrentHashMap
- **Why:** Concurrent requests from multiple users
- **Use:** Atomic counters for rate tracking, thread-safe token bucket

**12. Design a Distributed Lock Manager**
- **Concepts:** ReentrantLock, Synchronized, Deadlock prevention
- **Why:** Managing locks across threads, preventing deadlocks
- **Use:** Lock acquisition/release, consistent lock ordering

**13. Design a Configuration Management System**
- **Concepts:** ReadWriteLock, Volatile, Synchronized
- **Why:** Concurrent config reads, occasional config updates
- **Use:** ReadWriteLock for read-heavy access, volatile for config changes

**14. Design a Restaurant Management System**
- **Concepts:** Synchronized, BlockingQueue
- **Why:** Concurrent order placement, kitchen queue management
- **Use:** Thread-safe order queue, synchronized table allocation

**15. Design a Food Delivery System**
- **Concepts:** Synchronized, Atomic operations, ConcurrentHashMap
- **Why:** Concurrent orders, delivery partner assignment
- **Use:** Thread-safe order management, atomic assignment operations

**16. Design a Hotel Management System**
- **Concepts:** Synchronized, Atomic operations
- **Why:** Concurrent room bookings, check-in/check-out
- **Use:** Thread-safe room allocation, prevent double booking

**17. Design a Cab Booking System**
- **Concepts:** Synchronized, Atomic operations, ConcurrentHashMap
- **Why:** Concurrent ride requests, driver matching
- **Use:** Thread-safe ride booking, atomic driver assignment

**18. Design a Stock Trading System**
- **Concepts:** Synchronized, Atomic operations, BlockingQueue
- **Why:** Concurrent order placement, order matching
- **Use:** Thread-safe order queue, atomic trade execution

**19. Design a Banking System**
- **Concepts:** Synchronized, Atomic operations, Deadlock prevention
- **Why:** Concurrent transactions, account balance updates
- **Use:** Thread-safe balance updates, prevent deadlocks in transfers

**20. Design an ATM System**
- **Concepts:** Synchronized, Atomic operations
- **Why:** Concurrent ATM access (though typically one user per ATM)
- **Use:** Thread-safe transaction processing, atomic balance updates

**21. Design a Distributed Queue System**
- **Concepts:** BlockingQueue, Synchronized, Producer-Consumer pattern
- **Why:** Concurrent message enqueue/dequeue
- **Use:** Thread-safe queue operations, blocking operations

**22. Design a Multi-threaded Web Crawler**
- **Concepts:** ConcurrentHashMap, BlockingQueue, Synchronized
- **Why:** Multiple threads crawling URLs simultaneously
- **Use:** Thread-safe URL queue, concurrent visited URL tracking

---

## Concept Mapping

| Concept | Questions |
|---------|-----------|
| **Synchronized** | Parking Lot, Elevator, Vending Machine, Shopping, Movie Booking, Library, Cache, Logger, Task Scheduler, Restaurant, Food Delivery, Hotel, Cab Booking, Stock Trading, Banking, ATM |
| **Atomic Operations** | Parking Lot, Shopping, Movie Booking, Library, Task Scheduler, Rate Limiter, Food Delivery, Hotel, Cab Booking, Stock Trading, Banking, ATM |
| **ConcurrentHashMap** | Parking Lot, Shopping, Library, Cache, Rate Limiter, Food Delivery, Cab Booking, Web Crawler |
| **BlockingQueue** | Elevator, Task Scheduler, Restaurant, Stock Trading, Queue System, Web Crawler |
| **ReadWriteLock** | File System, Cache, Configuration Management |
| **Deadlock Prevention** | Banking System, Distributed Lock Manager |
| **Producer-Consumer** | Elevator, Restaurant, Queue System, Web Crawler |

---

*Master these patterns for LLD interviews. Focus on understanding when to use each approach.*

