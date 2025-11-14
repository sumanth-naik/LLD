# Top 10 Design Patterns for LLD Interviews

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

---

## 1. Singleton Pattern

**Problem:** Need exactly one instance of a class (e.g., database connection, configuration manager, logger).

### ❌ Bad Code

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

---

## 2. Factory Pattern

**Problem:** Need to create objects without specifying exact classes (decouple object creation from usage).

### ❌ Bad Code

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

```java
// Tight coupling - subject knows about all observers
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

```java
// ✅ Observer interface
public interface Observer {
    void update(String news);
}

// ✅ Subject interface
public interface Subject {
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

// ✅ Concrete subject
public class NewsAgency implements Subject {
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
- ✅ Loose coupling between subject and observers
- ✅ Dynamic subscription/unsubscription
- ✅ One-to-many dependency
- ✅ Open/Closed Principle (easy to add observers)

---

## 6. Decorator Pattern

**Problem:** Need to add behavior to objects dynamically without altering structure (coffee add-ons, stream processing).

### ❌ Bad Code

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

```java
// Incompatible interfaces
public class LegacyPaymentSystem {
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

```java
// ✅ Target interface (what client expects)
public interface PaymentProcessor {
    void processPayment(String userId, double amount);
}

// ✅ Adaptee (legacy system - incompatible)
public class LegacyPaymentSystem {
    public void makePayment(String account, double amount) {
        System.out.println("Legacy: Paying $" + amount + " from account " + account);
    }
}

// ✅ Adapter - adapts legacy to modern interface
public class LegacyPaymentAdapter implements PaymentProcessor {
    private LegacyPaymentSystem legacySystem;
    
    public LegacyPaymentAdapter(LegacyPaymentSystem legacySystem) {
        this.legacySystem = legacySystem;
    }
    
    @Override
    public void processPayment(String userId, double amount) {
        // Adapt the interface
        String account = "ACC-" + userId;  // Convert userId to account format
        legacySystem.makePayment(account, amount);
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
LegacyPaymentSystem legacy = new LegacyPaymentSystem();
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

## Summary: When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| **Singleton** | Need exactly one instance (config, database connection) |
| **Factory** | Don't know exact class at compile time |
| **Builder** | Complex object construction with many optional parameters |
| **Strategy** | Need to switch algorithms at runtime |
| **Observer** | Need to notify multiple objects of state changes |
| **Decorator** | Need to add behavior dynamically without subclassing |
| **Adapter** | Need to integrate incompatible interfaces |
| **Command** | Need undo/redo, queuing, or logging operations |
| **Template Method** | Have algorithm with steps that vary |
| **State** | Object behavior changes significantly based on state |

---

## Key Takeaways

- ✅ **Start simple, refactor when needed** - Don't over-engineer
- ✅ **Patterns solve specific problems** - Use them when they fit
- ✅ **Code smell first, pattern second** - Recognize the problem, then apply pattern
- ✅ **Combine patterns** - Real-world code often uses multiple patterns
- ✅ **Practice** - Understanding comes from implementing

---

*These patterns are essential for LLD interviews. Practice implementing them and recognizing when to use each one.*

