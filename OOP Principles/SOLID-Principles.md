# SOLID Principles - Progressive Refactoring

A single example that starts with horrible code and improves step-by-step using each SOLID principle.

**Example: Notification System** - A system that sends notifications to users via different channels.

---

## The Horrible Starting Code

### UML Diagram

```
┌─────────────────────────────────────────────┐
│         NotificationService                 │
├─────────────────────────────────────────────┤
│ - List<String> users                        │
├─────────────────────────────────────────────┤
│ + sendNotification(user, message, type)     │
│ + removeUser(user)                          │
│ + saveToDatabase(user, message)             │
└─────────────────────────────────────────────┘
         │
         │ ❌ Does everything:
         │    - User management
         │    - Logging
         │    - Notification sending
         │    - Database operations
```

```java
// ❌ HORRIBLE CODE - Violates all SOLID principles
public class NotificationService {
    private List<String> users = new ArrayList<>();
    
    // Does everything: user management, notification sending, logging, email, SMS
    public void sendNotification(String user, String message, String type) {
        // User management
        if (!users.contains(user)) {
            users.add(user);
            System.out.println("User added: " + user);
        }
        
        // Logging
        System.out.println("Log: Sending notification to " + user);
        
        // Notification sending logic
        if (type.equals("email")) {
            // Email sending
            System.out.println("Connecting to SMTP server...");
            System.out.println("Sending email to " + user + ": " + message);
            System.out.println("Email sent successfully");
        } else if (type.equals("sms")) {
            // SMS sending
            System.out.println("Connecting to SMS gateway...");
            System.out.println("Sending SMS to " + user + ": " + message);
            System.out.println("SMS sent successfully");
        } else if (type.equals("push")) {
            // Push notification
            System.out.println("Connecting to push service...");
            System.out.println("Sending push to " + user + ": " + message);
            System.out.println("Push sent successfully");
        }
        
        // More logging
        System.out.println("Log: Notification sent to " + user);
    }
    
    // User management mixed in
    public void removeUser(String user) {
        users.remove(user);
        System.out.println("User removed: " + user);
    }
    
    // Database operations mixed in
    public void saveToDatabase(String user, String message) {
        System.out.println("Saving to database: " + user + " - " + message);
    }
}
```

**Problems:**
- Does too many things (user management, notifications, logging, database)
- Hard to extend (adding new notification type requires modifying this class)
- Tightly coupled (all logic in one place)
- Hard to test
- Violates every SOLID principle

---

## Step 1: Single Responsibility Principle (SRP)

**Principle:** A class should have only one reason to change.

**Refactoring:** Separate concerns into different classes.

### UML Diagram

```
┌──────────────────────────┐
│      UserService         │
├──────────────────────────┤
│ - List<String> users     │
├──────────────────────────┤
│ + addUser(user)          │
│ + removeUser(user)       │
│ + userExists(user): bool │
└──────────────────────────┘
         ▲
         │ uses/has-a
         │
┌─────────────────────────────────────────────┐
│      NotificationService                    │
├─────────────────────────────────────────────┤
│ - UserService userService                   │
│ - Logger logger                             │
├─────────────────────────────────────────────┤
│ + sendNotification(user, message, type)     │
│ - sendEmail(user, message)                  │
│ - sendSMS(user, message)                    │
│ - sendPush(user, message)                   │
└─────────────────────────────────────────────┘
         │
         │ uses/has-a
         ▼
┌──────────────────────────┐
│        Logger            │
├──────────────────────────┤
│ + log(message)           │
└──────────────────────────┘

✅ UserService: Single responsibility - Only handles users
✅ Logger: Single responsibility - Only handles logging
✅ NotificationService: Single responsibility - Only handles notifications
```

```java
// ✅ Step 1: Separate User Management
public class UserService {
    private List<String> users = new ArrayList<>();
    
    public void addUser(String user) {
        if (!users.contains(user)) {
            users.add(user);
            System.out.println("User added: " + user);
        }
    }
    
    public void removeUser(String user) {
        users.remove(user);
        System.out.println("User removed: " + user);
    }
    
    public boolean userExists(String user) {
        return users.contains(user);
    }
}

// ✅ Step 1: Separate Logging
public class Logger {
    public void log(String message) {
        System.out.println("Log: " + message);
    }
}

// ✅ Step 1: Separate Notification Service (still has issues, but better)
public class NotificationService {
    private UserService userService;
    private Logger logger;
    
    public NotificationService() {
        this.userService = new UserService();
        this.logger = new Logger();
    }
    
    public void sendNotification(String user, String message, String type) {
        // Ensure user exists
        userService.addUser(user);
        
        // Log
        logger.log("Sending notification to " + user);
        
        // Send notification
        if (type.equals("email")) {
            sendEmail(user, message);
        } else if (type.equals("sms")) {
            sendSMS(user, message);
        } else if (type.equals("push")) {
            sendPush(user, message);
        }
        
        logger.log("Notification sent to " + user);
    }
    
    private void sendEmail(String user, String message) {
        System.out.println("Connecting to SMTP server...");
        System.out.println("Sending email to " + user + ": " + message);
        System.out.println("Email sent successfully");
    }
    
    private void sendSMS(String user, String message) {
        System.out.println("Connecting to SMS gateway...");
        System.out.println("Sending SMS to " + user + ": " + message);
        System.out.println("SMS sent successfully");
    }
    
    private void sendPush(String user, String message) {
        System.out.println("Connecting to push service...");
        System.out.println("Sending push to " + user + ": " + message);
        System.out.println("Push sent successfully");
    }
}
```

**Improvement:**
- ✅ Each class has a single responsibility
- ✅ UserService handles users
- ✅ Logger handles logging
- ✅ NotificationService handles notifications (but still has issues)

**Still Problems:**
- ❌ Adding new notification type requires modifying NotificationService
- ❌ Hard-coded if-else chains

---

## Step 2: Open/Closed Principle (OCP)

**Principle:** Software entities should be open for extension but closed for modification.

**Refactoring:** Use polymorphism to allow extension without modification.

### UML Diagram

```
┌──────────────────────────────┐
│  NotificationChannel         │
│      <<interface>>           │
├──────────────────────────────┤
│ + send(user, message)        │
└──────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴────┬────────────┬─────────────┬──────────────┐
    │         │            │             │              │
┌───┴────┐ ┌──┴──────┐ ┌───┴──────┐ ┌────┴─────────┐   any new channels 
│ Email  │ │   SMS   │ │   Push   │ │   WhatsApp   │   in future...
│Channel │ │ Channel │ │ Channel  │ │   Channel    │
├────────┤ ├─────────┤ ├──────────┤ ├──────────────┤
│+send() │ │+send()  │ │ +send()  │ │  +send()     │
└────────┘ └─────────┘ └──────────┘ └──────────────┘
    │         │            │             │
    └─────────┴────────────┴─────────────┘
                    │
                    │ uses/has-a
                    ▼
┌─────────────────────────────────────────────┐
│         NotificationService                 │
├─────────────────────────────────────────────┤
│ - UserService userService                   │
│ - Logger logger                             │
│ - Map<String,NotificationChannel> channels  │
├─────────────────────────────────────────────┤
│ + sendNotification(user, message, type)     │
│ + registerChannel(type, channel)            │
└─────────────────────────────────────────────┘

✅ Open for extension: Can add new channels without modification
✅ WhatsAppChannel: New channel added without modifying NotificationService
```

```java
// ✅ Step 2: Abstract notification channel
public interface NotificationChannel {
    void send(String user, String message);
}

// ✅ Step 2: Concrete implementations
public class EmailChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to SMTP server...");
        System.out.println("Sending email to " + user + ": " + message);
        System.out.println("Email sent successfully");
    }
}

public class SMSChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to SMS gateway...");
        System.out.println("Sending SMS to " + user + ": " + message);
        System.out.println("SMS sent successfully");
    }
}

public class PushChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to push service...");
        System.out.println("Sending push to " + user + ": " + message);
        System.out.println("Push sent successfully");
    }
}

// ✅ Step 2: NotificationService now uses abstraction
public class NotificationService {
    private UserService userService;
    private Logger logger;
    private Map<String, NotificationChannel> channels;
    
    public NotificationService() {
        this.userService = new UserService();
        this.logger = new Logger();
        this.channels = new HashMap<>();
        
        // Register default channels
        channels.put("email", new EmailChannel());
        channels.put("sms", new SMSChannel());
        channels.put("push", new PushChannel());
    }
    
    public void sendNotification(String user, String message, String type) {
        userService.addUser(user);
        logger.log("Sending notification to " + user);
        
        NotificationChannel channel = channels.get(type);
        if (channel != null) {
            channel.send(user, message);
        } else {
            throw new IllegalArgumentException("Unknown notification type: " + type);
        }
        
        logger.log("Notification sent to " + user);
    }
    
    // ✅ Can add new channels without modifying existing code
    public void registerChannel(String type, NotificationChannel channel) {
        channels.put(type, channel);
    }
}

// ✅ Step 2: Adding new notification type is easy - no modification needed!
public class WhatsAppChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to WhatsApp API...");
        System.out.println("Sending WhatsApp to " + user + ": " + message);
        System.out.println("WhatsApp sent successfully");
    }
}

// Usage: Can extend without modifying NotificationService
NotificationService service = new NotificationService();
service.registerChannel("whatsapp", new WhatsAppChannel());
service.sendNotification("user1", "Hello", "whatsapp");
```

**Improvement:**
- ✅ Can add new notification types without modifying NotificationService
- ✅ Uses polymorphism (interface-based design)
- ✅ Open for extension, closed for modification

**Still Problems:**
- ❌ What if we have a channel that can't send (like PhoneCallChannel that can only call)?
- ❌ Need to ensure substitutability - all implementations must fulfill the contract
- And a few other
---

## Step 3: Liskov Substitution Principle (LSP)

**Principle:** Objects of a superclass should be replaceable with objects of its subclasses without breaking the application. **A child should do at least what the parent does.**

### ❌ Common Misconception - What NOT to Do

**The Problem:** Forcing a child class to implement a method it shouldn't have.

### UML Diagram - Wrong Approach

```
┌──────────────────────────────┐
│  NotificationChannel         │
│      <<interface>>           │
├──────────────────────────────┤
│ + send(user, message)        │
└──────────────────────────────┘
         ▲
         │ implements
         │
    ┌────┴──────────────┐
    │                   │
┌───┴───┐      ┌────────┴──────────┐
│ Email │      │  PhoneCallChannel │
│Channel│      │                   │
├───────┤      ├───────────────────┤
│+send()│      │  +send() ❌       │
└───────┘      └───────────────────┘
✅              ❌ Violates LSP:
               Forced to implement send()
               but can only call, not send
```

```java
// ❌ WRONG APPROACH - Violates LSP
public interface NotificationChannel {
    void send(String user, String message);
}

// ❌ Problem: PhoneCallChannel is forced to implement send() but it can't send - it can only call
// This violates LSP - a child should do AT LEAST what parent does, not less!
public class PhoneCallChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        // VIOLATES LSP! This doesn't actually send - it makes a phone call
        // We're forcing this class to have a method it shouldn't have
        System.out.println("Making phone call to " + user + "...");
        // Actually makes a call, doesn't send - breaks the contract!
    }
}

// Usage - this breaks the contract
NotificationChannel channel = new PhoneCallChannel();
channel.send("user", "message");  // Doesn't send - makes a call instead - violates expectation!
// Code expecting sending will break!
```

**Why This is Wrong:**
- ❌ `PhoneCallChannel` implements `send()` but doesn't actually send - it makes phone calls instead
- ❌ We're **forcing** a child to implement a method it shouldn't have
- ❌ This violates LSP: "A child should do at least what the parent does" - but `PhoneCallChannel` does something DIFFERENT (calls vs sends)
- ❌ Any code expecting `NotificationChannel.send()` to send a message will break
- ❌ This is a **common misconception** - thinking "do something different" is acceptable when it breaks the contract

**The Core Issue:** If a class cannot fulfill the contract (sending messages), it shouldn't implement that interface. Use proper inheritance hierarchy to separate sending from calling. In most cases, you are not modelling the system properly!

---

### ✅ Correct Approach - Proper Inheritance Hierarchy

**Refactoring:** Ensure all implementations properly fulfill the contract. Use proper inheritance hierarchy to avoid "do nothing" implementations.

Not all Notification channels can `send`. So, `send` should not even be present in that class!

### UML Diagram - Correct Approach

```
┌──────────────────────────────┐
│  CommunicationChannel        │
│      <<interface>>           │
├──────────────────────────────┤
│ + getChannelType(): String   │
└──────────────────────────────┘
         ▲
         │ extends
    ┌────┴───────────────────────┐
    │                            │
┌───┴──────────┐  ┌──────────────┴──────────┐
│Notification  │  │      Call               │
│   Channel    │  │      Channel            │
│<<interface>> │  │    <<interface>>        │
├──────────────┤  ├─────────────────────────┤
│ + send(...)  │  │ + call(user, message)   │
└──────────────┘  │ + getCallDuration():int │
    ▲             └─────────────────────────┘
    │ implements           ▲
    │                      │ implements
    │                      │
┌───┴────┬──────────┐  ┌───┴────────┬──────────┐
│ Email  │   SMS    │  │  PhoneCall │ FaceTime │
│Channel │ Channel  │  │  Service   │ Service  │
├────────┤──────────│  ├────────────┤──────────│
│+send() │ +send()  │  │+call()     │ +call()  │
└────────┴──────────┘  │+getCall    │ +getCall │
                       │Duration()  │Duration()│
                       └────────────┴──────────┘
    │         │              │            │
    └─────────┴──────────────┴────────────┘
                    │
                    │ uses/has-a
                    ▼
┌─────────────────────────────────────────────┐
│         CommunicationService                │
├─────────────────────────────────────────────┤
│ - UserService userService                   │
│ - Logger logger                             │
│ - Map<String,CommunicationChannel> channels │
├─────────────────────────────────────────────┤
│ + sendCommunication(user, message, type)    │
└─────────────────────────────────────────────┘

✅ NotificationChannel: Contract - send()
   All implementations must send messages
✅ CallChannel: Contract - call()
   All implementations must make calls
✅ PhoneCallChannel: LSP compliant - Implements call(),
   not forced to send()
```

```java
import java.util.*;

// ✅ Step 3: Base interface - common contract for all channels
public interface CommunicationChannel {
    String getChannelType();
}

// ✅ Step 3: Interface for notification-based communication channels
// This ensures LSP - any NotificationChannel MUST send messages
public interface NotificationChannel extends CommunicationChannel {
    void send(String user, String message);
}

// ✅ Step 3: Interface for call-based communication channels
// This is a valid alternative - they're still CommunicationChannels but make calls instead of sending messages
public interface CallChannel extends CommunicationChannel {
    void call(String user, String message);
    int getCallDuration();  // Returns call duration in seconds
}

// ✅ Step 3: Proper implementations that can be substituted
// All NotificationChannel implementations MUST send messages
public class EmailChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to SMTP server...");
        System.out.println("Sending email to " + user + ": " + message);
        System.out.println("Email sent successfully");
    }
    
    @Override
    public String getChannelType() {
        return "Email";
    }
}

public class SMSChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to SMS gateway...");
        System.out.println("Sending SMS to " + user + ": " + message);
        System.out.println("SMS sent successfully");
    }
    
    @Override
    public String getChannelType() {
        return "SMS";
    }
}

public class PushChannel implements NotificationChannel {
    @Override
    public void send(String user, String message) {
        System.out.println("Connecting to push service...");
        System.out.println("Sending push to " + user + ": " + message);
        System.out.println("Push sent successfully");
    }
    
    @Override
    public String getChannelType() {
        return "Push";
    }
}

// ✅ Step 3: PhoneCallChannel - properly implements CallChannel
// This follows LSP - it's a valid CommunicationChannel that makes phone calls instead of sending messages
public class PhoneCallChannel implements CallChannel {
    private int lastCallDuration = 0;
    
    @Override
    public void call(String user, String message) {
        System.out.println("Dialing phone number for " + user + "...");
        System.out.println("Calling " + user + " with message: " + message);
        System.out.println("Playing automated message...");
        // Simulate call duration
        lastCallDuration = 30; // 30 seconds
        System.out.println("Phone call completed successfully");
    }
    
    @Override
    public int getCallDuration() {
        return lastCallDuration;
    }
    
    @Override
    public String getChannelType() {
        return "PhoneCall";
    }
}

// ✅ Step 3: FaceTimeChannel - properly implements CallChannel
// This follows LSP - it's a valid CommunicationChannel that makes video calls instead of sending messages
public class FaceTimeChannel implements CallChannel {
    private int lastCallDuration = 0;
    
    @Override
    public void call(String user, String message) {
        System.out.println("Initiating FaceTime call to " + user + "...");
        System.out.println("Connecting video call to " + user + " with message: " + message);
        System.out.println("Displaying message on video call...");
        // Simulate call duration
        lastCallDuration = 45; // 45 seconds
        System.out.println("FaceTime call completed successfully");
    }
    
    @Override
    public int getCallDuration() {
        return lastCallDuration;
    }
    
    @Override
    public String getChannelType() {
        return "FaceTime";
    }
}

// ✅ Step 3: CommunicationService works with proper channel types
public class CommunicationService {
    private UserService userService;
    private Logger logger;
    private Map<String, CommunicationChannel> channels;
    
    public CommunicationService() {
        this.userService = new UserService();
        this.logger = new Logger();
        this.channels = new HashMap<>();
        
        channels.put("email", new EmailChannel());
        channels.put("sms", new SMSChannel());
        channels.put("push", new PushChannel());
    }
    
    public void sendCommunication(String user, String message, String type) {
        userService.addUser(user);
        logger.log("Sending communication to " + user);
        
        CommunicationChannel channel = channels.get(type);
        
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        // ✅ LSP: Check if channel is notification-based or call-based - type-safe check
        if (channel instanceof NotificationChannel) {
            NotificationChannel notificationChannel = (NotificationChannel) channel;
            notificationChannel.send(user, message);  // All NotificationChannel MUST send messages
            logger.log("Message sent to " + user);
        } else if (channel instanceof CallChannel) {
            CallChannel callChannel = (CallChannel) channel;
            callChannel.call(user, message);  // All CallChannel MUST make calls
            logger.log("Call made to " + user + " via " + callChannel.getChannelType() + " (duration: " + callChannel.getCallDuration() + "s)");
        }
    }
    
    public void registerChannel(String type, CommunicationChannel channel) {
        channels.put(type, channel);
    }
}

// ✅ LSP Compliance: 
// - All notification channels are communication channels(no ifs and buts)
// - All call channels are communication channels(no ifs and buts)
// - PhoneCallChannel and FaceTimeChannel don't have to implement send() - they have call() instead
// - No implementations forced to have methods they shouldn't have
```

**Improvement:**
- ✅ All implementations can be substituted without breaking functionality
- ✅ Proper exception handling maintains contract
- ✅ Multi-level inheritance: `NotificationChannel` and `CallChannel` properly separate concerns
- ✅ Real-world example: Phone calls (PhoneCallChannel, FaceTimeChannel) vs sending messages (Email, SMS, Push) - both are valid communication channels but with different behaviors
- ✅ LSP compliance: Child classes do at least what parent classes do
- ✅ **Key Learning:** Don't force a child to implement methods it shouldn't have. If a class makes calls instead of sends, it shouldn't implement `send()`. Use proper inheritance hierarchy to separate different behaviors.

**Still Problems:**
- ❌ CommunicationService depends on concrete UserService and Logger
- ❌ Too many dependencies forced on implementations

---

## Step 4: Interface Segregation Principle (ISP)

**Principle:** Clients should not be forced to depend on interfaces they do not use.

**Refactoring:** Split large interfaces into smaller, focused ones. If an interface can be used in different hierarchies, use this principle.

New requirement: Push Notification, PhoneCall, and FaceTimeCall are Schedulable. SMS is Trackable.

**Solution:** Split the large interface into smaller, focused interfaces. Each implementation only implements what it needs. Do not add schedule or track to existing interfaces.

### UML Diagram - Correct Approach

```
       ┌──────────────────────────────┐
       │  CommunicationChannel        │
       │      <<interface>>           │
       ├──────────────────────────────┤
       │ + getChannelType(): String   │
       └──────────────────────────────┘
                   ▲
                   │ extends
              ┌────┴───────────────────────┐
              │                            │
          ┌───┴──────────┐  ┌──────────────┴──────────┐
          │Notification  │  │      Call               │
          │   Channel    │  │      Channel            │
          │<<interface>> │  │    <<interface>>        │
          ├──────────────┤  ├─────────────────────────┤
          │ + send(...)  │  │ + call(user, message)   │
          └──────────────┘  │ + getCallDuration():int │
              ▲             └─────────────────────────┘
              │ implements           ▲
              │                      │ implements
              │                      │
 ┌────────┌───┴────┬──────────┐  ┌───┴────────┬──────────┐
 │ Email  │ SMS    │   Push   │  │  PhoneCall │ FaceTime │
 │Channel │Channel │ Channel  │  │  Service   │ Service  │
 ├────────├────────┤──────────│  ├────────────┤──────────│
 │+send() |+send() │ +send()  │  │+call()     │ +call()  │
 └────────└────────┴──────────┘  │+getCall    │ +getCall │
              │         │        │Duration()  │Duration()│
              │         │        └────────────┴──────────┘
              │         │              │            │
              │         │implements    │implements  │   
              │         │              │            │ implements
              │         │              │            │
              │         ▼              ▼            ▼
              │        ┌──────────────────────────────┐
              │        │     SchedulableChannel       │
              │        │      <<interface>>           │
              │        ├──────────────────────────────┤
              │        │ + schedule(user, msg, time)  │
              │        └──────────────────────────────┘
              │
              │ implements
              │
              ▼
          ┌──────────────────────────────┐
          │     TrackableChannel         │
          │      <<interface>>           │
          ├──────────────────────────────┤
          │ + trackDelivery(messageId)   │
          │ + getDeliveryStatus(...)     │
          └──────────────────────────────┘

✅ EmailChannel: Only implements NotificationChannel
✅ SMSChannel: Implements NotificationChannel and TrackableNotificationChannel (trackable)
✅ PushChannel: Implements NotificationChannel and SchedulableNotificationChannel (schedulable)
✅ PhoneCallChannel: Implements CallChannel and SchedulableChannel (schedulable)
✅ FaceTimeChannel: Implements CallChannel and SchedulableChannel (schedulable)
```

```java
// ❌ Bad implementation: One large interface forcing all implementations to have everything
// public interface NotificationChannel {
//     void send(String user, String message);
//     void schedule(String user, String message, Date time);
//     void trackDelivery(String messageId);
// }


// ✅ Good Implementation

// ✅ Step 4: Base interface - common contract for all channels
public interface CommunicationChannel {
    String getChannelType();
}

// ✅ Step 4: Interface for notification-based communication channels
public interface NotificationChannel extends CommunicationChannel {
    void send(String user, String message);
}

// ✅ Step 4: Interface for call-based communication channels
public interface CallChannel extends CommunicationChannel {
    void call(String user, String message);
    int getCallDuration();
}

// ✅ Step 4: Single interface for scheduling (works for both notifications and calls)
public interface SchedulableChannel {
    void schedule(String user, String message, Date time);
}

// ✅ Step 4: Interface for tracking
public interface TrackableChannel {
    void trackDelivery(String messageId);
    String getDeliveryStatus(String messageId);
}

// ✅ Step 4: Implementations only implement what they need
public class EmailChannel implements NotificationChannel {
    @Override
    public String getChannelType() {
        return "Email";
    }
    
    @Override
    public void send(String user, String message) {
        System.out.println("Sending email to " + user + ": " + message);
    }
    // No scheduling, no tracking - doesn't need to implement them!
}

// ✅ Step 4: SMS implements NotificationChannel and TrackableChannel (trackable)
public class SMSChannel implements NotificationChannel, TrackableChannel {
    @Override
    public String getChannelType() {
        return "SMS";
    }
    
    @Override
    public void send(String user, String message) {
        System.out.println("Sending SMS to " + user + ": " + message);
    }
    
    // ✅ Only SMSChannel needs tracking
    @Override
    public void trackDelivery(String messageId) {
        System.out.println("Tracking SMS delivery: " + messageId);
    }
    
    @Override
    public String getDeliveryStatus(String messageId) {
        return "delivered";
    }
}

// ✅ Step 4: Push notification with scheduling
public class PushChannel implements NotificationChannel, SchedulableChannel {
    @Override
    public String getChannelType() {
        return "Push";
    }
    
    @Override
    public void send(String user, String message) {
        System.out.println("Sending push to " + user + ": " + message);
    }
    
    // ✅ Only PushChannel needs scheduling
    @Override
    public void schedule(String user, String message, Date time) {
        System.out.println("Scheduling push to " + user + " at " + time);
    }
}

// ✅ Step 4: PhoneCallChannel with scheduling
public class PhoneCallChannel implements CallChannel, SchedulableChannel {
    private int lastCallDuration = 0;
    
    @Override
    public String getChannelType() {
        return "PhoneCall";
    }
    
    @Override
    public void call(String user, String message) {
        System.out.println("Calling " + user + " with message: " + message);
        lastCallDuration = 30;
    }
    
    @Override
    public int getCallDuration() {
        return lastCallDuration;
    }
    
    // ✅ PhoneCallChannel needs scheduling
    @Override
    public void schedule(String user, String message, Date time) {
        System.out.println("Scheduling phone call to " + user + " at " + time);
    }
}

// ✅ Step 4: FaceTimeChannel with scheduling
public class FaceTimeChannel implements CallChannel, SchedulableChannel {
    private int lastCallDuration = 0;
    
    @Override
    public String getChannelType() {
        return "FaceTime";
    }
    
    @Override
    public void call(String user, String message) {
        System.out.println("FaceTime calling " + user + " with message: " + message);
        lastCallDuration = 45;
    }
    
    @Override
    public int getCallDuration() {
        return lastCallDuration;
    }
    
    // ✅ FaceTimeChannel needs scheduling
    @Override
    public void schedule(String user, String message, Date time) {
        System.out.println("Scheduling FaceTime call to " + user + " at " + time);
    }
}

// ✅ Step 4: CommunicationService uses the segregated interfaces
public class CommunicationService {
    private UserService userService;
    private Logger logger;
    private Map<String, CommunicationChannel> channels;
    
    public CommunicationService() {
        this.userService = new UserService();
        this.logger = new Logger();
        this.channels = new HashMap<>();
        
        channels.put("email", new EmailChannel());
        channels.put("sms", new SMSChannel());
        channels.put("push", new PushChannel());
        channels.put("phonecall", new PhoneCallChannel());
        channels.put("facetime", new FaceTimeChannel());
    }
    
    public void sendCommunication(String user, String message, String type) {
        userService.addUser(user);
        logger.log("Sending communication to " + user);
        
        CommunicationChannel channel = channels.get(type);
        
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        // ✅ ISP: Use specific interfaces only when needed
        if (channel instanceof NotificationChannel) {
            NotificationChannel notificationChannel = (NotificationChannel) channel;
            notificationChannel.send(user, message);
            logger.log("Message sent to " + user);
        } else if (channel instanceof CallChannel) {
            CallChannel callChannel = (CallChannel) channel;
            callChannel.call(user, message);
            logger.log("Call made to " + user + " via " + callChannel.getChannelType() + " (duration: " + callChannel.getCallDuration() + "s)");
        }
    }
    
    public void scheduleCommunication(String user, String message, String type, Date time) {
        CommunicationChannel channel = channels.get(type);
        
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        // ✅ ISP: Only channels that implement SchedulableChannel can be scheduled
        if (channel instanceof SchedulableChannel) {
            SchedulableChannel schedulableChannel = (SchedulableChannel) channel;
            schedulableChannel.schedule(user, message, time);
            logger.log("Communication scheduled to " + user + " via " + channel.getChannelType());
        } else {
            logger.logError("Channel " + type + " does not support scheduling");
        }
    }
    
    public void trackDelivery(String messageId, String type) {
        CommunicationChannel channel = channels.get(type);
        
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        // ✅ ISP: Only channels that implement TrackableChannel can be tracked
        if (channel instanceof TrackableChannel) {
            TrackableChannel trackableChannel = (TrackableChannel) channel;
            trackableChannel.trackDelivery(messageId);
            String status = trackableChannel.getDeliveryStatus(messageId);
            logger.log("Delivery status for " + messageId + ": " + status);
        } else {
            logger.logError("Channel " + type + " does not support tracking");
        }
    }
    
    public void registerChannel(String type, CommunicationChannel channel) {
        channels.put(type, channel);
    }
}
```

**Improvement:**
- ✅ Clients only depend on methods they actually use
- ✅ No forced implementation of unused methods
- ✅ More flexible and maintainable
- ✅ Better separation of concerns
- ✅ CommunicationService can work with any combination of interfaces (Schedulable, Trackable) without forcing all channels to implement everything

**Still Problems:**
- ❌ CommunicationService still creates concrete UserService and Logger
- ❌ Hard to test (can't mock dependencies)
- ❌ Tight coupling to concrete implementations
- ❌ Using String for channel type is error-prone (typos cause runtime failures)

---

## Detour Step: Dependency Injection & Type Safety with Enums

**Note:** This is a detour from SOLID to explain Dependency Injection (a technique) before we cover Dependency Inversion Principle (DIP - a principle). They are related but different concepts.

**Dependency Injection:** Instead of creating dependencies inside a class, inject them from outside (via constructor, setter, etc.). This makes code testable and flexible.

**Problem:** Creating dependencies inside the class makes it hard to test and tightly coupled. Mocking the objects instantiated with `new` is very hard and often requires a lot of work.

### ❌ Problem: Creating Dependencies Inside Class

```java
// ❌ BAD: Creating dependencies inside the class
public class CommunicationService {
    private UserService userService;
    private Logger logger;
    
    public CommunicationService() {
        this.userService = new UserService();  // ❌ Hard to test - can't mock
        this.logger = new Logger();            // ❌ Hard to test - can't mock
    }
    
    public void sendCommunication(String user, String message, String type) {
        // Using String for type - error-prone!
        if (type.equals("email")) {  // ❌ Typo: "emial" won't be caught until runtime
            // ...
        }
    }
}

// ❌ Testing is hard - can't mock UserService or Logger
public class CommunicationServiceTest {
    @Test
    public void testSendCommunication() {
        CommunicationService service = new CommunicationService();
        // ❌ Can't control UserService or Logger behavior
        // ❌ Can't verify if UserService.addUser() was called
        service.sendCommunication("user1", "Hello", "email");
    }
}

// ❌ Alternative: Using PowerMockito.whenNew() - NOT RECOMMENDED
// PowerMockito can mock object creation using reflection:
// PowerMockito.whenNew(UserService.class).withNoArguments().thenReturn(mockUserService);
// 
// Why this is bad:
// - Uses reflection (runtime code manipulation) - slow and fragile
// - Requires bytecode manipulation - breaks with Java updates
// - Makes tests brittle - breaks when code structure changes
// - Hard to read and maintain - not clear what's being tested
// - Violates encapsulation - tests depend on internal implementation details
// 
// ✅ Better design: Use Dependency Injection to avoid needing PowerMockito
```

### ✅ Solution: Constructor Injection + Enum for Type Safety

```java
// ✅ GOOD: Dependencies injected via constructor
public class CommunicationService {
    private final UserService userService;
    private final Logger logger;
    private final Map<ChannelType, CommunicationChannel> channels;
    
    // ✅ Constructor Injection: Dependencies provided from outside
    public CommunicationService(UserService userService, Logger logger) {
        this.userService = userService;  // ✅ Injected - can be mocked
        this.logger = logger;            // ✅ Injected - can be mocked
        this.channels = new HashMap<>();
        
        // Register default channels
        channels.put(ChannelType.EMAIL, new EmailChannel());
        channels.put(ChannelType.SMS, new SMSChannel());
        channels.put(ChannelType.PUSH, new PushChannel());
        channels.put(ChannelType.PHONE_CALL, new PhoneCallChannel());
        channels.put(ChannelType.FACETIME, new FaceTimeChannel());
        
        // Note: We're still creating channel objects with 'new' here, which violates Dependency Injection.
        // We can fix this by injecting a ChannelFactory (Factory Design Pattern) that creates channels.
        // However, that's beyond the scope of this discussion - we're focusing on the main dependencies (UserService, Logger).
    }
    
    public void sendCommunication(String user, String message, ChannelType type) {
        userService.addUser(user);
        logger.log("Sending communication to " + user);
        
        CommunicationChannel channel = channels.get(type);
        
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        if (channel instanceof NotificationChannel) {
            NotificationChannel notificationChannel = (NotificationChannel) channel;
            notificationChannel.send(user, message);
            logger.log("Message sent to " + user);
        } else if (channel instanceof CallChannel) {
            CallChannel callChannel = (CallChannel) channel;
            callChannel.call(user, message);
            logger.log("Call made to " + user);
        }
    }
    
    public void registerChannel(ChannelType type, CommunicationChannel channel) {
        channels.put(type, channel);
    }
}

// ✅ Enum for type safety - compile-time checking
public enum ChannelType {
    EMAIL,
    SMS,
    PUSH,
    PHONE_CALL,
    FACETIME
}

// ✅ Usage with enum - type-safe!
public class Main {
    public static void main(String[] args) {
        UserService userService = new UserService();
        Logger logger = new Logger();
        
        // ✅ Dependencies injected via constructor
        CommunicationService service = new CommunicationService(userService, logger);
        
        // ✅ Enum prevents typos - compile-time error if wrong
        service.sendCommunication("user1", "Hello", ChannelType.EMAIL);  // ✅ Compile-time check
        // service.sendCommunication("user1", "Hello", "emial");  // ❌ Runtime error if using String
    }
}
```

### ✅ Testing with Mocks

```java
// ✅ Easy to test - can mock dependencies
public class CommunicationServiceTest {
    @Test
    public void testSendCommunication() {
        // ✅ Create mocks
        UserService mockUserService = new MockUserService();
        Logger mockLogger = new MockLogger();
        
        // ✅ Inject mocks via constructor
        CommunicationService service = new CommunicationService(mockUserService, mockLogger);
        
        // ✅ Test behavior
        service.sendCommunication("user1", "Hello", ChannelType.EMAIL);
        
        // ✅ Verify interactions
        assertTrue(mockUserService.wasAddUserCalled("user1"));
        assertTrue(mockLogger.wasLogCalled("Sending communication to user1"));
    }
}

// Mock implementations for testing
class MockUserService extends UserService {
    private List<String> addedUsers = new ArrayList<>();
    
    @Override
    public void addUser(String user) {
        addedUsers.add(user);
    }
    
    public boolean wasAddUserCalled(String user) {
        return addedUsers.contains(user);
    }
}

class MockLogger extends Logger {
    private List<String> logs = new ArrayList<>();
    
    @Override
    public void log(String message) {
        logs.add(message);
    }
    
    public boolean wasLogCalled(String message) {
        return logs.contains(message);
    }
}
```

**Key Benefits:**
- ✅ **Testable**: Can inject mocks for testing
- ✅ **Flexible**: Can swap implementations easily
- ✅ **Type Safety**: Enum prevents typos - compile-time errors instead of runtime failures
- ✅ **Loose Coupling**: Class doesn't create its dependencies

**String vs Enum:**
- ❌ **String**: `"emial"` typo → Runtime failure (only discovered when code runs)
- ✅ **Enum**: `ChannelType.EMIAL` → Compile-time error (caught before code runs)

---

## Step 5: Dependency Inversion Principle (DIP)

**Principle:** High-level modules should not depend on low-level modules. Both should depend on abstractions. This helps break dependency cycles between modules.

**Problem:** When two modules depend on each other, it creates a cycle that makes the code hard to maintain and test.

**Refactoring:** Create abstraction modules (API modules with interfaces) to break the cycle. Both implementation modules depend on abstraction modules instead of each other.

### ❌ Problem: Dependency Cycle

**Scenario:** Two modules depend on each other, creating a cycle.

Let's say a use case where, we want to send communication when a new User is created. Check `addUser`.

```
┌─────────────────────────────────────┐         ┌─────────────────────────────────────┐
│   Module: user-management           │         │   Module: communications            │
│                                     │         │                                     │
│   ┌──────────────────────────────┐  │         │   ┌──────────────────────────────┐  │
│   │  User (object)               │  │         │   │  ChannelType (enum)          │  │
│   │  - name: String              │  │         │   │  EMAIL, SMS, PUSH, etc.      │  │
│   │  - email: String             │  │         │   └──────────────────────────────┘  │
│   └──────────────────────────────┘  │         │                                     │
│                                     │         │   ┌──────────────────────────────┐  │
│   ┌──────────────────────────────┐  │         │   │  CommunicationService        │  │
│   │  UserService                 │  │         │   │  + sendCommunication(        │  │
│   │  + addUser(User user) {      │  │         │   │      user: User,             │  │
│   │      CommunicationService    │  │         │   │      message: String,        │  │
│   │          commService =       │  │         │   │      type: ChannelType)      │  │
│   │          new Communication   │  │         │   └──────────────────────────────┘  │
│   │          Service(...);       │  │         │                 │                   │
│   │      commService.send        │  │         │                 │                   │
│   │          Communication(      │  │         │                 │                   │
│   │          user, "Welcome!",   │  │         │                 │                   │
│   │          ChannelType.EMAIL); │  │         │                 │                   │
│   │  }                           │  │         │                 │  depends on       │
│   └──────────────────────────────┘  │         │                 │                   │
│           │                         │         │                 │                   │
│           │ depends on              │         │                 │                   │
│           │                         │         │                 │                   │
│           ▼                         │         │                 ▼                   │
│   ┌──────────────────────────────┐  │         │   ┌──────────────────────────────┐  │
│   │  Needs:                      │  │         │   │  Needs:                      │  │
│   │  - CommunicationService      │───────────>│   │  - User (object)             │  │
│   │  - ChannelType (enum)        │  │         │   │                              │  │
│   └──────────────────────────────┘  │         │   └──────────────────────────────┘  │
│                                     │         │           │                         │
│                                     │         │           │                         │
│                                     │         │           │                         │
│                                     │<────────┼───────────┘                         │
│                                     │         │                                     │
└─────────────────────────────────────┘         └─────────────────────────────────────┘

                              ❌ CYCLE
                    user-management → communications
                    communications → user-management
```

**Why this is bad:**
- Code won't even compile! `user-management` module wants `communications` module to be compiled first to use in `UserService`. `communications` module wants `user-management` module to be compiled first to use in `CommunicationService`. Cyclic dependency issue.

### ✅ Solution: Break Cycle with API and Implementation Modules

**Create API modules (with interfaces, objects, enums) and Implementation modules (with concrete classes) to break the cycle:**

```
┌─────────────────────────────────────┐         ┌─────────────────────────────────────┐
│   API Module: user-management-api   │         │   API Module: communications-api    │
│                                     │         │                                     │
│   ┌──────────────────────────────┐  │         │   ┌──────────────────────────────┐  │
│   │  User (object/class)         │  │         │   │  ChannelType (enum)          │  │
│   │  - name: String              │  │         │   │  EMAIL, SMS, PUSH, etc.      │  │
│   │  - email: String             │  │         │   └──────────────────────────────┘  │
│   └──────────────────────────────┘  │         │                                     │
│                                     │         │   ┌──────────────────────────────┐  │
│   ┌──────────────────────────────┐  │         │   │  CommunicationService        │  │
│   │  UserService                 │  │         │   │      <<interface>>           │  │
│   │      <<interface>>           │  │ <-------│   │  + sendCommunication(        │  │
│   │  + addUser(user: User)       │  │  (needs │   │      user: User,             │  │
│   │  + removeUser(user: User)    │  │   User  │   │      message: String,        │  │
│   │  + userExists(user: User)    │  │   class)│   │      type: ChannelType)      │  │
│   └──────────────────────────────┘  │         │   └──────────────────────────────┘  │
│                                     │         │                                     │
│                                     │         │   ┌──────────────────────────────┐  │
│                                     │         │   │  CommunicationChannel        │  │
│                                     │         │   │      <<interface>>           │  │
│                                     │         │   │  + send(user, message)       │  │
│                                     │         │   └──────────────────────────────┘  │
└─────────────────────────────────────┘         └─────────────────────────────────────┘
           ▲                           ^        ^         ▲
           │                            \      /          │
           │ implements                  \    /           │ implements
           │                              \  /            │
┌──────────┴──────────────────────────┐    \/   ┌─────────┴───────────────────────────┐
│ Implementation: user-management-impl│    /\   │  Implementation: communications-impl│
│   ┌──────────────────────────────┐  │   /  \  │   ┌──────────────────────────────┐  │
│   │  Needs:                      │  │  /    \ │   │  Needs:                      │  │
│   │  - CommunicationService      │  │ /      \│___│  - User (object)             │  │
│   │     (from communications-api)│__│/        │   │    (from user-management-api)│  │
│   │  - ChannelType (enum)        │  │         │   │                              │  │
│   │     (from communications-api)│  │         │   │                              │  │
│   └──────────────────────────────┘  │         │   └──────────────────────────────┘  │
│         ^ depends on                │         │            ^ depends on             │
│         │                           │         │            │                        │
│   ┌──────────────────────────────┐  │         │   ┌──────────────────────────────┐  │
│   │  UserServiceImpl             │  │         │   │  CommunicationServiceImpl    │  │
│   │  implements UserService      │  │         │   │  implements Communication    │  │
│   │  - CommunicationService      │  │         │   │      Service                 │  │
│   │      commService             │  │         │   │  - Map<ChannelType,          │  │
│   │  + addUser(user: User) {     │  │         │   │      Channel> channels       │  │
│   │      // add user...          │  │         │   │  + sendCommunication(        │  │
│   │      commService.send        │  │         │   │      user, message, type)    │  │
│   │          Communication(...); │  │         │   └──────────────────────────────┘  │
│   │  }                           │  │         │                                     │
│   └──────────────────────────────┘  │         │                                     │
│                                     │         │                                     │
└─────────────────────────────────────┘         └─────────────────────────────────────┘

✅ NO CYCLE: 
- user-management-impl → communications-api (abstraction, no cycle)
- communications-impl → user-management-api (abstraction, no cycle)
- API modules don't depend on each other
- Implementation modules don't depend on each other
- Dependencies flow: impl → api (one direction only)
```

**Key Question: How does `CommunicationService` get into `UserService`? Won't we still need to create `CommunicationServiceImpl`, causing a cycle?**

**Answer: Dependency Injection happens at the APPLICATION/WIRING level (outside both modules), not inside the modules themselves.**

The wiring can be done via:
1. **Spring Boot DI Container** (most common in real applications)
2. **Manual wiring in Main class** (as shown in code examples)
3. **Factory Pattern**
4. **Any other DI framework** (Guice, Dagger, etc.)

**How it works:**
```
┌──────────────────────────┐   ┌──────────────────────────┐
│  user-management-impl    │   │  communications-impl     │
│  (doesn't create         │   │  (doesn't create         │
│   CommunicationService)  │   │   UserService)           │
└──────────────────────────┘   └──────────────────────────┘
           ^                              ^
           │                              │
           │ depends on                   │ depends on
┌─────────────────────────────────────────────────────────────┐
│   Application Layer (Main class / Spring Boot / Factory)    │
│   - Depends on BOTH user-management-impl AND                │
│     communications-impl                                     │
│   - Creates instances and wires them together               │
│                                                             │
│   1. Create CommunicationServiceImpl                        │
│   2. Create UserService, inject CommunicationServiceImpl    │
│   3. Wire everything together                               │
└─────────────────────────────────────────────────────────────┘


```

**Example with Spring Boot:**
```java
// Spring Boot automatically wires dependencies
@Configuration
public class AppConfig {
    @Bean
    public CommunicationService communicationService() {
        return new CommunicationServiceImpl();  // Created by Spring
    }
    
    @Bean
    public UserService userService(CommunicationService commService) {
        return new UserServiceImpl(commService);  // Spring injects CommunicationService
    }
}
```

**Example with Manual Wiring (Main class):**
```java
// Manual wiring - no cycle because this is outside both modules
public class Main {
    public static void main(String[] args) {
        // Application layer creates and wires
        CommunicationService commService = new CommunicationServiceImpl();
        UserService userService = new UserServiceImpl(commService);  // Injected here
    }
}
```

**Important Points:**
- ✅ **Implementation modules don't create each other** - they only depend on API modules (interfaces)
- ✅ **Application layer creates and wires** - this is where `CommunicationServiceImpl` is instantiated and injected into `UserService`
- ✅ **No cycle at module level** - modules only depend on abstractions (API modules), not on each other
- ✅ **Application layer can depend on both** - it's fine for the application to depend on multiple implementation modules

**Key Insight: DIP vs DI - They are related but separate!**

**DIP (Dependency Inversion Principle)** = Principle: Depend on abstractions (interfaces, enums, objects in API modules) to break cycles

**DI (Dependency Injection)** = Technique: Inject dependencies (for objects/services that need to be instantiated)

**Can you use DIP without DI? YES!**

**Example: If `user-management-impl` only needed `ChannelType` enum (not `CommunicationService`):**

```java
// user-management-impl
import communications.api.ChannelType;  // ✅ Just import the enum - no DI needed!

public class UserService {
    public void addUser(User user, ChannelType preferredChannel) {
        // Use ChannelType directly - it's just an enum, no instantiation needed
        if (preferredChannel == ChannelType.EMAIL) {
            // ...
        }
    }
}
```

**In this case:**
- ✅ **Still need DIP** - Put `ChannelType` in `communications-api` to break the cycle
- ❌ **Don't need DI** - Enums are just imported and used directly, no injection needed
- ✅ **No cycle** - `user-management-impl` depends on `communications-api` (abstraction), not `communications-impl`

**When do you need DI?**
- When you need to inject **objects/services** that need to be instantiated (like `CommunicationService`)
- When you want to make code **testable** (can inject mocks)
- When you want **flexibility** (can swap implementations)

**When do you need DIP?**
- When you have **dependency cycles** between modules
- When you want modules to depend on **abstractions** instead of concrete implementations
- Works with **enums, interfaces, objects** - doesn't require DI

**Can you use DI without DIP? YES!**

**Example: The Detour Step (before DIP) - We used DI without DIP!**

In the "Detour Step: Dependency Injection & Type Safety with Enums" section, we showed:
```java
public class CommunicationService {
    private final UserService userService;
    private final Logger logger;
    
    // ✅ DI: Dependencies injected via constructor
    public CommunicationService(UserService userService, Logger logger) {
        this.userService = userService;
        this.logger = logger;
    }
}
```

**In this case:**
- ✅ **Used DI** - Injected `UserService` and `Logger` for testability and flexibility
- ❌ **Didn't need DIP** - No dependency cycle between modules, just needed to make code testable
- ✅ **No cycle** - `CommunicationService` depends on `UserService` and `Logger`, but they don't depend back on `CommunicationService`

**Summary:**
- **DI without DIP**: Use when you need testability/flexibility, but no module cycle exists
- **DIP without DI**: Use when you need to break cycles, but only need simple types (enums, objects)
- **Both DI and DIP**: Use when you have cycles AND need to inject services/objects (most common in real applications)



```java
// ❌ WRONG APPROACH: Dependency Cycle

// Module: user-management
public class User {
    private String name;
    private String email;
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // getters...
}

public class UserService {
    public void addUser(User user) {
        // Add user to database...
        
        // ❌ Creates dependency on communications module
        CommunicationService commService = new CommunicationService();
        commService.sendCommunication(user, "Welcome!", ChannelType.EMAIL);  // ❌ Depends on CommunicationService
    }
}

// Module: communications
public enum ChannelType {
    EMAIL, SMS, PUSH, PHONE_CALL, FACETIME
}

public class CommunicationService {
    public void sendCommunication(User user, String message, ChannelType type) {
        // ❌ Depends on User from user-management module
        System.out.println("Sending " + type + " to " + user.getName() + ": " + message);
    }
}

// ❌ CYCLE: user-management → communications → user-management
// ❌ Can't compile! Circular dependency!

// ✅ CORRECT APPROACH: Break Cycle with API and Implementation Modules

// ============================================
// API Module: user-management-api
// ============================================
// Contains: Objects, Interfaces, Enums (abstractions)

public class User {
    private String name;
    private String email;
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    public String getName() { return name; }
    public String getEmail() { return email; }
}

public interface UserService {
    void addUser(User user);
    void removeUser(User user);
    boolean userExists(User user);
}

// ============================================
// API Module: communications-api
// ============================================
// Contains: Objects, Interfaces, Enums (abstractions)

public enum ChannelType {
    EMAIL, SMS, PUSH, PHONE_CALL, FACETIME
}

public interface CommunicationService {
    void sendCommunication(User user, String message, ChannelType type);
}

public interface CommunicationChannel {
    void send(User user, String message);
    String getChannelType();
}

// ============================================
// Implementation Module: user-management-impl
// ============================================
// Contains: Concrete implementations

import user.management.api.User;
import user.management.api.UserService;
import communications.api.CommunicationService;
import communications.api.ChannelType;

public class UserServiceImpl implements UserService {
    // ✅ Depends on abstraction (CommunicationService interface from communications-api)
    private final CommunicationService communicationService;
    private final List<User> users = new ArrayList<>();
    
    // ✅ Dependency Injection: CommunicationService injected via constructor
    public UserServiceImpl(CommunicationService communicationService) {
        this.communicationService = communicationService;
    }
    
    @Override
    public void addUser(User user) {
        if (!users.contains(user)) {
            users.add(user);
            // ✅ Uses abstraction from communications-api
            communicationService.sendCommunication(user, "Welcome! Your account has been created.", ChannelType.EMAIL);
        }
    }
    
    @Override
    public void removeUser(User user) {
        users.remove(user);
    }
    
    @Override
    public boolean userExists(User user) {
        return users.contains(user);
    }
}

// ============================================
// Implementation Module: communications-impl
// ============================================
// Contains: Concrete implementations

import communications.api.CommunicationService;
import communications.api.ChannelType;
import communications.api.CommunicationChannel;
import user.management.api.User;  // ✅ Depends on User from user-management-api

public class CommunicationServiceImpl implements CommunicationService {
    private final Map<ChannelType, CommunicationChannel> channels;
    
    public CommunicationServiceImpl() {
        this.channels = new HashMap<>();
        channels.put(ChannelType.EMAIL, new EmailChannel());
        channels.put(ChannelType.SMS, new SMSChannel());
        channels.put(ChannelType.PUSH, new PushChannel());
        channels.put(ChannelType.PHONE_CALL, new PhoneCallChannel());
        channels.put(ChannelType.FACETIME, new FaceTimeChannel());
    }
    
    @Override
    public void sendCommunication(User user, String message, ChannelType type) {
        // ✅ Uses User from user-management-api (abstraction)
        CommunicationChannel channel = channels.get(type);
        if (channel != null) {
            channel.send(user, message);
            System.out.println("Sent " + type + " to " + user.getName() + ": " + message);
        }
    }
}

// Channel implementations
class EmailChannel implements CommunicationChannel {
    @Override
    public void send(User user, String message) {
        System.out.println("Sending email to " + user.getEmail() + ": " + message);
    }
    
    @Override
    public String getChannelType() { return "EMAIL"; }
}

class SMSChannel implements CommunicationChannel {
    @Override
    public void send(User user, String message) {
        System.out.println("Sending SMS to " + user.getName() + ": " + message);
    }
    
    @Override
    public String getChannelType() { return "SMS"; }
}

// ... other channel implementations

// ============================================
// Usage: Wire dependencies together
// ============================================
// Note: In real applications, use a Dependency Injection framework (Spring, Guice, etc.)

import user.management.api.User;
import user.management.api.UserService;
import communications.api.CommunicationService;
import communications.api.ChannelType;
import user.management.impl.UserServiceImpl;
import communications.impl.CommunicationServiceImpl;

public class Main {
    public static void main(String[] args) {
        // Step 1: Create CommunicationService implementation
        CommunicationService commService = new CommunicationServiceImpl();
        
        // Step 2: Create UserServiceImpl with CommunicationService injected
        UserService userService = new UserServiceImpl(commService);
        
        // Step 3: Use the services
        User newUser = new User("John Doe", "john@example.com");
        userService.addUser(newUser);  // This will trigger communication via abstraction
        
        // Can also use CommunicationService directly
        commService.sendCommunication(newUser, "Hello!", ChannelType.SMS);
    }
}

// ✅ Easy to test - can mock abstractions
import user.management.api.User;
import user.management.api.UserService;
import communications.api.CommunicationService;
import communications.api.ChannelType;
import user.management.impl.UserServiceImpl;

public class UserServiceTest {
    @Test
    public void testAddUser() {
        // ✅ Mock the abstraction from communications-api
        CommunicationService mockCommService = new MockCommunicationService();
        UserService userService = new UserServiceImpl(mockCommService);
        
        User user = new User("Test User", "test@example.com");
        userService.addUser(user);
        
        // ✅ Verify behavior
        assertTrue(userService.userExists(user));
        assertTrue(mockCommService.wasSendCommunicationCalled());
    }
}

class MockCommunicationService implements CommunicationService {
    private boolean sendCommunicationCalled = false;
    
    @Override
    public void sendCommunication(User user, String message, ChannelType type) {
        sendCommunicationCalled = true;
    }
    
    public boolean wasSendCommunicationCalled() {
        return sendCommunicationCalled;
    }
}
```

**Module Dependencies:**
```
user-management-impl → user-management-api (implements interfaces)
user-management-impl → communications-api (depends on interfaces)

communications-impl → communications-api (implements interfaces)
communications-impl → user-management-api (depends on User object)

✅ NO CYCLE: Implementation modules depend on API modules, not each other
✅ API modules don't depend on each other
```

**Key Points:**
- ✅ **DIP (Dependency Inversion Principle)**: Break cycles by depending on abstractions (interfaces in separate modules)
- ✅ **Dependency Injection**: Technique to provide dependencies (via constructor/setter) - makes code testable and flexible
- ✅ **No Cycle**: Both modules depend on abstraction modules, not each other
- ✅ **Testable**: Can easily mock abstractions for testing

**Final Improvement:**
- ✅ **No dependency cycles**: Modules depend on abstractions, not each other
- ✅ **Break cycles**: Abstraction modules (interfaces) break circular dependencies
- ✅ **Easy to test**: Can mock abstractions independently
- ✅ **Loose coupling**: Modules are decoupled through abstractions
- ✅ **Follows all SOLID principles!**

---

## Final Refactored Code Summary

```java
// ✅ All SOLID principles applied
public class CommunicationService {
    // ✅ DIP: Depends on abstractions (interfaces), not concrete implementations
    // ✅ DI: Dependencies injected via constructor
    private final UserService userService;        // Interface, not UserServiceImpl
    private final LoggingService logger;          // Interface, not concrete Logger
    private final Map<ChannelType, CommunicationChannel> channels;  // Interface-based, enum for type safety
    
    // ✅ DI: Constructor injection - dependencies provided from outside
    public CommunicationService(UserService userService, LoggingService logger) {
        this.userService = userService;  // ✅ DIP: Abstraction
        this.logger = logger;            // ✅ DIP: Abstraction
        this.channels = new HashMap<>();
    }
    
    // ✅ OCP: Open for extension - can add new channels without modifying this class
    // ✅ OCP: Closed for modification - don't need to change this method to add channels
    public void registerChannel(ChannelType type, CommunicationChannel channel) {
        channels.put(type, channel);  // ✅ LSP: Any CommunicationChannel implementation works
    }
    
    // ✅ SRP: Single responsibility - only handles communication orchestration
    //    (User management is handled by UserService, logging by LoggingService)
    // ✅ ISP: Uses focused interfaces - only depends on methods it actually uses
    // ✅ LSP: Any CommunicationChannel can be substituted without breaking functionality
    // ✅ DIP: Works with any UserService/LoggingService implementation via interfaces
    public void sendCommunication(User user, String message, ChannelType type) {
        userService.addUser(user);  // ✅ DIP: Uses interface method
        logger.log("Sending communication to " + user.getName());  // ✅ DIP: Uses interface method
        
        CommunicationChannel channel = channels.get(type);
        if (channel == null) {
            logger.logError("Channel not found: " + type);
            return;
        }
        
        // ✅ LSP: Can use any CommunicationChannel implementation (EmailChannel, SMSChannel, etc.)
        if (channel instanceof NotificationChannel) {
            NotificationChannel notificationChannel = (NotificationChannel) channel;
            notificationChannel.send(user, message);  // ✅ LSP: Substitutable implementations
            logger.log("Message sent to " + user.getName());
        } else if (channel instanceof CallChannel) {
            CallChannel callChannel = (CallChannel) channel;
            callChannel.call(user, message);  // ✅ LSP: Substitutable implementations
            logger.log("Call made to " + user.getName());
        }
    }
}
```

---

## SOLID Principles Summary

1. **Single Responsibility Principle (SRP)**: Each class has one reason to change
   - **UserService** handles user management (add, remove, check existence)
   - **LoggingService** handles logging operations
   - **CommunicationService** handles communication orchestration only
   - Each class has a single, well-defined responsibility

2. **Open/Closed Principle (OCP)**: Open for extension, closed for modification
   - Can add new **CommunicationChannel** implementations (EmailChannel, SMSChannel, etc.) without modifying CommunicationService
   - `registerChannel()` method allows extension without changing existing code
   - New channel types can be added without touching the core CommunicationService class

3. **Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types
   - Any **CommunicationChannel** implementation can replace another without breaking functionality
   - **NotificationChannel** implementations (EmailChannel, SMSChannel) are fully substitutable
   - **CallChannel** implementations (PhoneCallChannel, FaceTimeChannel) are fully substitutable
   - No implementation is forced to have methods it shouldn't have (e.g., CallChannel doesn't need `send()`)

4. **Interface Segregation Principle (ISP)**: Clients shouldn't depend on interfaces they don't use
   - Separate interfaces for different capabilities: **SchedulableChannel**, **TrackableChannel**
   - Implementations only implement interfaces they actually need
   - EmailChannel doesn't need to implement scheduling or tracking
   - SMSChannel implements TrackableChannel (for delivery tracking)
   - PushChannel and PhoneCallChannel implement SchedulableChannel (for scheduling)

5. **Dependency Inversion Principle (DIP)**: Break dependency cycles by depending on abstractions
   - **Problem**: When Module A depends on Module B, and Module B depends on Module A, you get a cycle
   - **Solution**: Create abstraction modules (API modules with interfaces, enums, objects) that both implementation modules depend on
   - **user-management-impl** depends on **communications-api** (abstraction), not communications-impl
   - **communications-impl** depends on **user-management-api** (abstraction), not user-management-impl
   - API modules don't depend on each other, breaking the cycle
   - **Note**: **Dependency Injection (DI)** is a technique (constructor/setter injection) to provide dependencies - it's separate from DIP but often used together. You can use DIP without DI (e.g., with enums), and DI without DIP (e.g., for testability when there's no cycle).

---

## Benefits of Following SOLID

- ✅ **Maintainable**: Easy to understand and modify
- ✅ **Testable**: Can easily mock dependencies
- ✅ **Extensible**: Easy to add new features
- ✅ **Reusable**: Components can be reused in different contexts
- ✅ **Flexible**: Easy to swap implementations

