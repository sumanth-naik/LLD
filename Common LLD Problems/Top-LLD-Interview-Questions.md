# Top Low-Level Design (LLD) Interview Questions

This document contains the most frequently asked Low-Level Design interview questions, organized by category. Each question tests your ability to design scalable, maintainable, and well-structured systems using object-oriented principles and design patterns.

---

## Table of Contents

1. [Gaming Systems](#gaming-systems)
2. [Parking & Transportation](#parking--transportation)
3. [E-commerce & Shopping](#e-commerce--shopping)
4. [Social & Communication](#social--communication)
5. [Financial Systems](#financial-systems)
6. [Entertainment & Media](#entertainment--media)
7. [Infrastructure & Utilities](#infrastructure--utilities)
8. [Food & Restaurant](#food--restaurant)
9. [Education & Learning](#education--learning)
10. [Miscellaneous](#miscellaneous)

---

## Gaming Systems

### 1. Design a Tic-Tac-Toe Game
Design a Tic-Tac-Toe game that can be played between two players. The game should support:
- 3x3 grid
- Two players (X and O)
- Win detection (horizontal, vertical, diagonal)
- Draw detection
- Turn-based gameplay
- Ability to reset the game

**Key Concepts:** State management, game logic, player turns

**Design Patterns:** State Pattern (game states: Playing, Won, Draw), Strategy Pattern (different win strategies)

**SOLID Principles:** Single Responsibility (separate game logic, board, player), Open/Closed (extensible for different game modes)

---

### 2. Design a Snake and Ladder Game
Design a Snake and Ladder game with:
- Multiple players
- Board with snakes and ladders
- Dice rolling mechanism
- Win condition (reaching the end)
- Game state tracking

**Key Concepts:** Board representation, random number generation, game state

**Design Patterns:** State Pattern (game states), Strategy Pattern (different dice strategies), Factory Pattern (player creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for different board sizes)

---

### 3. Design a Chess Game
Design a chess game with:
- 8x8 board
- All chess pieces (Pawn, Rook, Knight, Bishop, Queen, King)
- Move validation
- Check and checkmate detection
- Turn-based gameplay
- Castling and en passant rules

**Key Concepts:** Complex state management, move validation, inheritance hierarchy

**Design Patterns:** Strategy Pattern (move validation for each piece), State Pattern (game states), Template Method Pattern (common piece behavior), Factory Pattern (piece creation)

**SOLID Principles:** Single Responsibility (each piece handles its own moves), Open/Closed (easy to add new pieces), Liskov Substitution (all pieces are substitutable), Interface Segregation (separate interfaces for different piece behaviors)

---

### 4. Design a Blackjack Game
Design a Blackjack (21) card game with:
- Standard 52-card deck
- Multiple players and a dealer
- Card dealing and shuffling
- Hit, stand, double down, split actions
- Score calculation (Ace can be 1 or 11)
- Win/loss determination

**Key Concepts:** Card deck management, game rules, player actions

**Design Patterns:** Strategy Pattern (different player actions: hit, stand, double down), State Pattern (game states), Factory Pattern (deck creation), Command Pattern (player actions)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new actions), Dependency Inversion (depend on abstractions)

---

### 5. Design a Sudoku Solver
Design a Sudoku puzzle system with:
- 9x9 grid
- Puzzle generation
- Puzzle validation
- Solution finding (backtracking)
- Difficulty levels

**Key Concepts:** Backtracking algorithm, constraint satisfaction, puzzle generation

**Design Patterns:** Strategy Pattern (different solving algorithms), Template Method Pattern (puzzle generation steps), Factory Pattern (puzzle creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for different difficulty levels)

---

## Parking & Transportation

### 6. Design a Parking Lot System
Design a parking lot management system with:
- Multiple parking spots (compact, large, handicapped, motorcycle)
- Vehicle entry and exit
- Parking spot allocation
- Payment calculation (time-based)
- Multiple entry/exit gates
- Parking spot availability tracking

**Key Concepts:** Resource allocation, state management, payment processing

**Design Patterns:** Strategy Pattern (parking spot allocation strategies), State Pattern (spot states: Available, Occupied, Reserved), Factory Pattern (vehicle creation), Command Pattern (entry/exit operations)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new vehicle types), Dependency Inversion (depend on payment abstraction)

---

### 7. Design an Elevator System
Design an elevator control system with:
- Multiple elevators in a building
- Floor requests (up/down buttons)
- Elevator car buttons
- Elevator movement and direction
- Load capacity management
- Emergency stop functionality

**Key Concepts:** State machine, request queuing, scheduling algorithms

**Design Patterns:** State Pattern (elevator states: Moving, Stopped, Maintenance), Strategy Pattern (scheduling algorithms: SCAN, LOOK), Command Pattern (elevator requests), Observer Pattern (floor button notifications)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new scheduling algorithms), Dependency Inversion

---

### 8. Design a Car Rental System
Design a car rental system with:
- Vehicle inventory (cars, bikes, trucks)
- Customer management
- Reservation system
- Rental and return process
- Pricing based on vehicle type and duration
- Vehicle maintenance tracking

**Key Concepts:** Inventory management, reservation system, pricing strategy

**Design Patterns:** Factory Pattern (vehicle creation), Strategy Pattern (pricing strategies), State Pattern (vehicle states: Available, Rented, Maintenance), Builder Pattern (rental booking)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new vehicle types), Dependency Inversion

---

### 9. Design a Traffic Light System
Design a traffic light control system with:
- Multiple intersections
- Traffic light states (Red, Yellow, Green)
- Timing control
- Pedestrian crossing signals
- Emergency override
- Sensor-based timing adjustment

**Key Concepts:** State management, timing control, coordination

**Design Patterns:** State Pattern (traffic light states: Red, Yellow, Green), Observer Pattern (sensor notifications), Strategy Pattern (timing strategies)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new timing algorithms)

---

## E-commerce & Shopping

### 10. Design an Online Shopping System
Design an e-commerce platform with:
- Product catalog
- Shopping cart
- User accounts
- Order management
- Payment processing
- Inventory management
- Product search and filtering

**Key Concepts:** Shopping cart, order lifecycle, payment integration

**Design Patterns:** Factory Pattern (product creation), Strategy Pattern (payment methods), State Pattern (order states), Builder Pattern (order construction), Observer Pattern (inventory notifications)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new payment methods), Dependency Inversion (depend on payment abstraction), Interface Segregation

---

### 11. Design a Vending Machine
Design a vending machine with:
- Product slots and inventory
- Coin/bill acceptance
- Product selection
- Change dispensing
- Refund mechanism
- Admin functions (restocking, pricing)

**Key Concepts:** State machine, payment handling, inventory management

**Design Patterns:** State Pattern (vending machine states: Idle, HasMoney, Dispensing), Strategy Pattern (payment methods), Command Pattern (product selection)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new payment methods), State Pattern ensures clean state transitions

---

### 12. Design a Splitwise (Expense Sharing)
Design an expense sharing application with:
- User management
- Group creation
- Expense addition (equal, exact, percentage splits)
- Balance calculation
- Settlement suggestions
- Transaction history

**Key Concepts:** Graph algorithms, balance calculation, debt simplification

**Design Patterns:** Strategy Pattern (expense splitting strategies: equal, exact, percentage), Observer Pattern (balance updates), Command Pattern (expense operations)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new splitting strategies), Dependency Inversion

---

### 13. Design a Coupon/Voucher System
Design a coupon management system with:
- Coupon creation (percentage, fixed amount)
- Validity periods
- Usage limits
- Applicability rules
- Discount calculation
- Usage tracking

**Key Concepts:** Rule engine, discount calculation, validation

**Design Patterns:** Strategy Pattern (discount calculation strategies: percentage, fixed amount), Chain of Responsibility Pattern (coupon validation rules), Factory Pattern (coupon creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new discount types), Dependency Inversion

---

## Social & Communication

### 14. Design a Social Media Platform (like Facebook)
Design a social media platform with:
- User profiles
- Friend connections
- News feed generation
- Post creation (text, image, video)
- Like, comment, share functionality
- Privacy settings

**Key Concepts:** Graph data structure, feed generation, activity streams

**Design Patterns:** Observer Pattern (news feed updates), Strategy Pattern (feed ranking algorithms), Factory Pattern (post creation), Mediator Pattern (user interactions)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new post types), Dependency Inversion

---

### 15. Design a Chat Application (like WhatsApp)
Design a chat application with:
- User authentication
- One-on-one messaging
- Group chats
- Message delivery status (sent, delivered, read)
- Media sharing
- Message encryption

**Key Concepts:** Real-time messaging, message queuing, status tracking

**Design Patterns:** Mediator Pattern (message routing), Observer Pattern (message delivery status), Strategy Pattern (message delivery strategies), State Pattern (message states: Sent, Delivered, Read)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 16. Design a Notification System
Design a notification system with:
- Multiple notification types (email, SMS, push)
- User preferences
- Notification templates
- Delivery scheduling
- Retry mechanism
- Notification history

**Key Concepts:** Observer pattern, template system, delivery channels

**Design Patterns:** Observer Pattern (notification delivery), Strategy Pattern (delivery channels: Email, SMS, Push), Factory Pattern (notification creation), Template Method Pattern (notification templates)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new delivery channels), Dependency Inversion

---

### 17. Design a URL Shortener (like bit.ly)
Design a URL shortening service with:
- URL encoding/decoding
- Short URL generation
- Click tracking and analytics
- Expiration dates
- Custom short URLs
- Redirect mechanism

**Key Concepts:** Hash generation, base conversion, analytics tracking

**Design Patterns:** Strategy Pattern (hash generation algorithms), Factory Pattern (URL creation), Proxy Pattern (caching), Decorator Pattern (analytics tracking)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new hash algorithms)

---

## Financial Systems

### 18. Design an ATM System
Design an ATM machine system with:
- User authentication (PIN)
- Account balance inquiry
- Cash withdrawal
- Cash deposit
- Fund transfer
- Transaction history
- Card management

**Key Concepts:** Authentication, transaction processing, state management

**Design Patterns:** State Pattern (ATM states: Idle, Authenticated, Transaction), Strategy Pattern (transaction types), Command Pattern (transaction operations), Factory Pattern (transaction creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new transaction types), Dependency Inversion

---

### 19. Design a Banking System
Design a banking system with:
- Account management (savings, checking, credit)
- Transaction processing
- Interest calculation
- Loan management
- Account statements
- Multi-currency support

**Key Concepts:** Transaction management, interest calculation, account types

**Design Patterns:** Strategy Pattern (account types: Savings, Checking, Credit), Factory Pattern (account creation), Command Pattern (transactions), Template Method Pattern (interest calculation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new account types), Liskov Substitution, Dependency Inversion

---

### 20. Design a Stock Trading System
Design a stock trading platform with:
- Stock order placement (buy/sell)
- Order types (market, limit, stop-loss)
- Order matching engine
- Portfolio management
- Real-time price updates
- Trade history

**Key Concepts:** Order matching, priority queues, real-time updates

**Design Patterns:** Strategy Pattern (order types: Market, Limit, Stop-loss), Observer Pattern (price updates), Command Pattern (order operations), Factory Pattern (order creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new order types), Dependency Inversion

---

### 21. Design a Payment Gateway
Design a payment gateway with:
- Multiple payment methods (credit card, UPI, wallet)
- Payment processing
- Transaction status tracking
- Refund processing
- Fraud detection
- Payment reconciliation

**Key Concepts:** Payment processing, transaction states, fraud detection

**Design Patterns:** Strategy Pattern (payment methods: CreditCard, UPI, Wallet), State Pattern (transaction states), Adapter Pattern (third-party payment processors), Chain of Responsibility Pattern (fraud detection), Factory Pattern (payment creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new payment methods), Dependency Inversion, Interface Segregation

---

## Entertainment & Media

### 22. Design a Movie Ticket Booking System
Design a movie ticket booking system with:
- Movie catalog
- Theater and screen management
- Seat selection
- Booking and payment
- Show timing management
- Booking cancellation

**Key Concepts:** Seat allocation, booking management, concurrent access

**Design Patterns:** Strategy Pattern (seat allocation algorithms), State Pattern (booking states), Command Pattern (booking operations), Factory Pattern (show creation), Observer Pattern (seat availability)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 23. Design a Music Player
Design a music player application with:
- Playlist management
- Play, pause, next, previous controls
- Shuffle and repeat modes
- Queue management
- Song metadata
- Favorites and recently played

**Key Concepts:** State management, queue operations, media controls

**Design Patterns:** State Pattern (player states: Playing, Paused, Stopped), Strategy Pattern (play modes: Normal, Shuffle, Repeat), Iterator Pattern (playlist traversal), Command Pattern (media controls)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new play modes), Dependency Inversion

---

### 24. Design a Netflix-like System
Design a video streaming platform with:
- User accounts and profiles
- Video catalog
- Search and recommendations
- Playback controls
- Watch history
- Subscription management

**Key Concepts:** Content delivery, recommendation engine, user profiles

**Design Patterns:** Strategy Pattern (recommendation algorithms), Factory Pattern (content creation), Observer Pattern (watch history updates), Proxy Pattern (content caching), Decorator Pattern (content features)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new recommendation algorithms), Dependency Inversion

---

### 25. Design a Library Management System
Design a library management system with:
- Book catalog
- Member management
- Book issue and return
- Reservation system
- Fine calculation
- Book search

**Key Concepts:** Resource management, reservation system, fine calculation

**Design Patterns:** State Pattern (book states: Available, Issued, Reserved), Strategy Pattern (fine calculation), Factory Pattern (book creation), Observer Pattern (reservation notifications)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new fine calculation rules), Dependency Inversion

---

## Infrastructure & Utilities

### 26. Design a File System
Design a file system with:
- Directory and file structure
- Create, read, update, delete operations
- File permissions
- Directory navigation
- File search
- Disk space management

**Key Concepts:** Tree data structure, file operations, permissions

**Design Patterns:** Composite Pattern (files and directories), Strategy Pattern (permission strategies), Command Pattern (file operations), Proxy Pattern (lazy loading)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new file types), Liskov Substitution (files and directories are substitutable)

---

### 27. Design a Cache System (LRU Cache)
Design a caching system with:
- LRU (Least Recently Used) eviction policy
- Get and put operations
- Capacity management
- TTL (Time To Live) support
- Cache statistics

**Key Concepts:** LRU algorithm, hash map, doubly linked list

**Design Patterns:** Strategy Pattern (eviction policies: LRU, LFU, FIFO), Factory Pattern (cache creation), Decorator Pattern (cache features: TTL, statistics)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new eviction policies), Dependency Inversion

---

### 28. Design a Logger System
Design a logging system with:
- Multiple log levels (DEBUG, INFO, WARN, ERROR)
- Multiple appenders (file, console, database)
- Log formatting
- Log rotation
- Async logging
- Log filtering

**Key Concepts:** Chain of responsibility, observer pattern, async processing

**Design Patterns:** Chain of Responsibility Pattern (log level handlers), Strategy Pattern (appenders: File, Console, Database), Observer Pattern (log events), Factory Pattern (logger creation), Decorator Pattern (log formatting)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new appenders), Dependency Inversion

---

### 29. Design a Task Scheduler
Design a task scheduling system with:
- Task creation and scheduling
- One-time and recurring tasks
- Priority-based execution
- Task dependencies
- Task status tracking
- Failure handling and retries

**Key Concepts:** Priority queue, scheduling algorithms, dependency resolution

**Design Patterns:** Strategy Pattern (scheduling algorithms), Command Pattern (task execution), State Pattern (task states), Factory Pattern (task creation), Observer Pattern (task completion)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new scheduling algorithms), Dependency Inversion

---

### 30. Design a Rate Limiter
Design a rate limiting system with:
- Multiple algorithms (Token bucket, Sliding window, Fixed window)
- Per-user rate limiting
- API endpoint rate limiting
- Distributed rate limiting
- Configuration management

**Key Concepts:** Rate limiting algorithms, distributed systems, configuration

**Design Patterns:** Strategy Pattern (rate limiting algorithms: Token Bucket, Sliding Window, Fixed Window), Factory Pattern (rate limiter creation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new algorithms), Dependency Inversion

---

### 31. Design a Distributed Lock Manager
Design a distributed lock system with:
- Lock acquisition and release
- Lock expiration
- Deadlock detection
- Lock reentrancy
- Multiple lock types (read, write)

**Key Concepts:** Distributed systems, concurrency, lock management

**Design Patterns:** Strategy Pattern (lock types: Read, Write), State Pattern (lock states), Factory Pattern (lock creation), Proxy Pattern (lock management)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 32. Design a Configuration Management System
Design a configuration management system with:
- Configuration storage
- Environment-based configs
- Hot reloading
- Versioning
- Access control
- Configuration validation

**Key Concepts:** Configuration management, versioning, validation

**Design Patterns:** Singleton Pattern (configuration instance), Strategy Pattern (configuration sources), Observer Pattern (configuration changes), Factory Pattern (configuration creation), Builder Pattern (configuration setup)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new configuration sources), Dependency Inversion

---

## Food & Restaurant

### 33. Design a Restaurant Management System
Design a restaurant management system with:
- Menu management
- Table reservation
- Order management
- Kitchen order queue
- Bill generation
- Staff management

**Key Concepts:** Order management, queue system, reservation system

**Design Patterns:** State Pattern (order states), Strategy Pattern (table allocation), Command Pattern (order operations), Factory Pattern (order creation), Observer Pattern (order updates)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 34. Design a Food Delivery System (like Zomato/Swiggy)
Design a food delivery platform with:
- Restaurant management
- Menu and item catalog
- Order placement
- Delivery partner assignment
- Real-time order tracking
- Payment and rating system

**Key Concepts:** Order lifecycle, real-time tracking, assignment algorithms

**Design Patterns:** State Pattern (order states), Strategy Pattern (delivery partner assignment), Observer Pattern (order tracking), Factory Pattern (order creation), Command Pattern (order operations)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new assignment algorithms), Dependency Inversion

---

### 35. Design a Coffee Machine
Design a coffee vending machine with:
- Multiple beverage types
- Ingredient management
- Recipe system
- Payment processing
- Maintenance alerts
- Customization options

**Key Concepts:** Recipe system, ingredient management, state machine

**Design Patterns:** State Pattern (machine states), Strategy Pattern (beverage recipes), Factory Pattern (beverage creation), Command Pattern (beverage preparation)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new beverages), Dependency Inversion

---

## Education & Learning

### 36. Design a Learning Management System (LMS)
Design an LMS with:
- Course management
- Student enrollment
- Assignment submission
- Grading system
- Progress tracking
- Discussion forums

**Key Concepts:** Course management, enrollment system, progress tracking

**Design Patterns:** Strategy Pattern (enrollment strategies), State Pattern (course states), Observer Pattern (progress updates), Factory Pattern (course creation)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 37. Design an Online Examination System
Design an online exam platform with:
- Question bank
- Exam creation
- Student registration
- Timer management
- Answer submission
- Auto-grading
- Result generation

**Key Concepts:** Timer management, answer submission, grading system

**Design Patterns:** State Pattern (exam states), Strategy Pattern (grading strategies), Command Pattern (answer submission), Factory Pattern (exam creation), Observer Pattern (timer notifications)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new question types), Dependency Inversion

---

## Miscellaneous

### 38. Design a Meeting Scheduler (like Calendly)
Design a meeting scheduling system with:
- Calendar management
- Availability tracking
- Meeting booking
- Time zone handling
- Reminders and notifications
- Recurring meetings

**Key Concepts:** Calendar management, time zone handling, scheduling conflicts

**Design Patterns:** Strategy Pattern (scheduling algorithms), Factory Pattern (meeting creation), Observer Pattern (reminder notifications), Command Pattern (meeting operations)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 39. Design a Hotel Management System
Design a hotel management system with:
- Room inventory
- Reservation system
- Check-in and check-out
- Room service
- Billing
- Guest management

**Key Concepts:** Reservation system, inventory management, billing

**Design Patterns:** State Pattern (room states), Strategy Pattern (pricing strategies), Factory Pattern (reservation creation), Observer Pattern (availability updates), Command Pattern (booking operations)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 40. Design a Cab Booking System (like Uber/Ola)
Design a ride-sharing platform with:
- Driver and rider management
- Ride booking
- Driver matching
- Real-time tracking
- Pricing calculation
- Payment and rating

**Key Concepts:** Matching algorithms, real-time tracking, pricing strategy

**Design Patterns:** Strategy Pattern (matching algorithms, pricing strategies), State Pattern (ride states), Observer Pattern (real-time tracking), Factory Pattern (ride creation), Command Pattern (ride operations)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new matching algorithms), Dependency Inversion

---

### 41. Design a Search Engine (Basic)
Design a basic search engine with:
- Document indexing
- Inverted index
- Query processing
- Ranking algorithm
- Search results pagination

**Key Concepts:** Inverted index, ranking algorithms, query processing

**Design Patterns:** Strategy Pattern (ranking algorithms), Factory Pattern (index creation), Iterator Pattern (result traversal), Template Method Pattern (query processing)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new ranking algorithms), Dependency Inversion

---

### 42. Design a Distributed Key-Value Store
Design a distributed key-value store with:
- Key-value operations (get, put, delete)
- Data replication
- Consistency models
- Partitioning/sharding
- Load balancing

**Key Concepts:** Distributed systems, replication, consistency

**Design Patterns:** Strategy Pattern (replication strategies, consistency models), Factory Pattern (store creation), Proxy Pattern (distributed access)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 43. Design a Pub-Sub System
Design a publish-subscribe messaging system with:
- Topic management
- Publisher and subscriber registration
- Message publishing
- Message delivery
- Message persistence
- Topic filtering

**Key Concepts:** Observer pattern, message queuing, topic management

**Design Patterns:** Observer Pattern (publish-subscribe), Strategy Pattern (message delivery strategies), Factory Pattern (topic creation), Command Pattern (message operations)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 44. Design a Multi-threaded Web Crawler
Design a web crawler with:
- URL queue management
- Multi-threaded crawling
- Duplicate URL detection
- Robots.txt compliance
- Rate limiting
- Data extraction

**Key Concepts:** Multi-threading, queue management, web scraping

**Design Patterns:** Strategy Pattern (crawling strategies), Factory Pattern (crawler creation), Command Pattern (crawl operations), Iterator Pattern (URL traversal)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 45. Design a Distributed Counter
Design a distributed counter system with:
- Increment/decrement operations
- High throughput
- Eventual consistency
- Sharding
- Aggregation

**Key Concepts:** Distributed systems, sharding, consistency

**Design Patterns:** Strategy Pattern (sharding strategies, consistency models), Factory Pattern (counter creation), Proxy Pattern (distributed access)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 46. Design a Distributed Cache
Design a distributed caching system with:
- Cache operations (get, put, delete)
- Cache eviction policies
- Replication
- Consistency
- Load distribution

**Key Concepts:** Distributed caching, replication, consistency models

**Design Patterns:** Strategy Pattern (eviction policies, replication strategies), Factory Pattern (cache creation), Proxy Pattern (distributed access), Decorator Pattern (cache features)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 47. Design a Job Scheduler (like Cron)
Design a job scheduling system with:
- Job definition
- Cron-like scheduling
- Job execution
- Retry mechanism
- Job dependencies
- Monitoring and logging

**Key Concepts:** Scheduling algorithms, cron expressions, job execution

**Design Patterns:** Strategy Pattern (scheduling algorithms), Command Pattern (job execution), Factory Pattern (job creation), State Pattern (job states), Observer Pattern (job completion)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new scheduling types), Dependency Inversion

---

### 48. Design a Distributed Log Aggregator
Design a log aggregation system with:
- Log collection from multiple sources
- Log storage
- Log search and filtering
- Real-time log streaming
- Log retention policies
- Indexing

**Key Concepts:** Log aggregation, search, indexing, streaming

**Design Patterns:** Strategy Pattern (aggregation strategies), Factory Pattern (aggregator creation), Observer Pattern (log events), Iterator Pattern (log traversal), Template Method Pattern (log processing)

**SOLID Principles:** Single Responsibility, Open/Closed, Dependency Inversion

---

### 49. Design a Content Management System (CMS)
Design a CMS with:
- Content creation and editing
- Version control
- Publishing workflow
- User roles and permissions
- Content search
- Media management

**Key Concepts:** Version control, workflow management, permissions

**Design Patterns:** Strategy Pattern (workflow strategies), State Pattern (content states), Command Pattern (content operations), Factory Pattern (content creation), Observer Pattern (content updates), Chain of Responsibility Pattern (permission checks)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new content types), Dependency Inversion, Interface Segregation

---

### 50. Design a Distributed Queue System
Design a distributed message queue with:
- Queue creation and management
- Message enqueue and dequeue
- Message persistence
- Consumer groups
- Message acknowledgment
- Dead letter queue

**Key Concepts:** Message queuing, persistence, consumer groups

**Design Patterns:** Strategy Pattern (queue strategies, delivery strategies), Factory Pattern (queue creation), Observer Pattern (message events), Command Pattern (message operations), State Pattern (message states)

**SOLID Principles:** Single Responsibility, Open/Closed (extensible for new queue types), Dependency Inversion

---

## LLD Interview Answer Template

Use this structured approach for any LLD interview question:

### Step 1: Requirements Clarification (2-3 minutes)
**Ask clarifying questions:**
- Functional requirements (what the system should do)
- Main use cases and features
- Constraints (technology stack, specific requirements)
- Assumptions (clarify ambiguities)

**Example questions:**
- What are the main use cases?
- Any specific constraints or requirements?
- What features are in scope vs out of scope?
- Any specific technology preferences?

---

### Step 2: Identify Core Objects/Entities (5-7 minutes)
**List main classes (nouns become classes):**
- Core entities (User, Order, Product, etc.)
- Supporting classes (Payment, Notification, etc.)
- Manager/Service classes (OrderManager, PaymentService, etc.)

**For each class, define:**
- Attributes (properties/fields)
- Behaviors (methods/operations)
- Relationships with other classes

**Example:**
```
Core Entities:
- User (id, name, email, address)
- Order (id, user, items, status, total)
- Product (id, name, price, inventory)
- Payment (id, order, amount, method, status)

Service Classes:
- OrderService (createOrder, cancelOrder, updateOrder)
- PaymentService (processPayment, refundPayment)
- InventoryService (checkAvailability, updateInventory)
```

---

### Step 3: Design Patterns (5-7 minutes)
**Identify applicable patterns:**

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

**Example:**
```
Patterns to use:
- Factory Pattern: Create different payment processors
- Strategy Pattern: Different payment methods (CreditCard, UPI, Wallet)
- State Pattern: Order states (Pending, Confirmed, Shipped, Delivered)
- Observer Pattern: Notify users on order status changes
- Command Pattern: Order operations (place, cancel, update)
```

---

### Step 4: SOLID Principles (3-5 minutes)
**Apply SOLID principles:**

**S - Single Responsibility Principle:**
- Each class should have one reason to change
- Separate concerns (e.g., Order class handles order data, OrderService handles order operations)

**O - Open/Closed Principle:**
- Open for extension, closed for modification
- Use interfaces/abstract classes for extensibility
- Example: Add new payment method without modifying existing code

**L - Liskov Substitution Principle:**
- Subtypes must be substitutable for their base types
- Derived classes should not break base class contracts
- Example: All payment methods should be interchangeable

**I - Interface Segregation Principle:**
- Clients shouldn't depend on interfaces they don't use
- Create specific interfaces instead of one general interface
- Example: Separate interfaces for Readable, Writable, Deletable

**D - Dependency Inversion Principle:**
- Depend on abstractions, not concretions
- High-level modules shouldn't depend on low-level modules
- Example: OrderService depends on PaymentInterface, not CreditCardPayment

**Example:**
```
SOLID Application:
- Single Responsibility: Order class only manages order data
- Open/Closed: Payment interface allows adding new payment methods
- Liskov Substitution: All payment implementations are interchangeable
- Interface Segregation: Separate interfaces for different operations
- Dependency Inversion: Services depend on interfaces, not concrete classes
```

---

### Step 5: Define Relationships (3-5 minutes)
**Map relationships between classes:**

**Inheritance (is-a):**
- User is-a Customer
- CreditCardPayment is-a Payment

**Composition (has-a, strong ownership):**
- Order has-a List<OrderItem>
- Car has-an Engine

**Aggregation (has-a, weak ownership):**
- University has Students
- Library has Books

**Association (uses):**
- OrderService uses PaymentService
- User uses OrderService

**Dependency:**
- OrderService depends on PaymentInterface

---

### Step 6: Handle Edge Cases & Error Handling (3-5 minutes)
**Consider:**
- Null/empty inputs
- Invalid states/transitions
- Concurrent access (thread safety)
- Resource exhaustion
- Network failures
- Timeout scenarios
- Validation failures

**Example:**
```
Edge Cases:
- What if payment fails after order is created?
- What if inventory becomes 0 during checkout?
- What if multiple users try to book the same item?
- What if system crashes during transaction?
```

---

### Step 7: API Design (if applicable) (3-5 minutes)
**Design public interfaces:**
- Method signatures
- Input/output parameters
- Return types
- Exceptions

**Example:**
```java
public interface OrderService {
    Order createOrder(CreateOrderRequest request) throws InvalidRequestException;
    void cancelOrder(String orderId) throws OrderNotFoundException;
    OrderStatus getOrderStatus(String orderId) throws OrderNotFoundException;
}
```

---

### Step 8: Concurrency & Thread Safety (if applicable) (2-3 minutes)
**Consider:**
- Shared state access
- Locks/synchronization
- Race conditions
- Deadlocks
- Atomic operations

**Example:**
```
Concurrency:
- Use synchronized blocks for shared resources
- Use ConcurrentHashMap for thread-safe collections
- Use AtomicInteger for counters
- Consider thread-safe collections for multi-threaded access
```

---

### Step 9: Code Structure (Implementation) (10-15 minutes)
**Write clean code:**
- Follow naming conventions
- Add comments for complex logic
- Use proper access modifiers (private, protected, public)
- Implement equals(), hashCode(), toString() where needed
- Handle exceptions properly


---


**Good luck with your LLD interviews!** 🚀


