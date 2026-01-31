# Open/Closed Principle (OCP)

## Definition

> "Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification."
> 
> — Bertrand Meyer (1988)

**Alternative Definition:** "You should be able to extend a class's behavior without modifying it."

---

## Table of Contents

1. [Intent](#intent)
2. [Motivation](#motivation)
3. [Structure](#structure)
4. [Key Concepts](#key-concepts)
5. [Examples](#examples)
6. [Before and After](#before-and-after)
7. [Real-World Examples](#real-world-examples)
8. [Benefits](#benefits)
9. [Common Violations](#common-violations)
10. [How to Apply](#how-to-apply)
11. [Related Patterns](#related-patterns)

---

## Intent

The Open/Closed Principle states that software entities should be:
- **Open for extension**: You can add new functionality
- **Closed for modification**: You don't change existing code

### Core Idea

```
┌─────────────────────────────────────────────────────────┐
│              Open/Closed Principle                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────┐                                     │
│  │  Existing Code │  ← CLOSED for modification          │
│  │   (Stable)     │                                     │
│  └────────┬───────┘                                     │
│           │                                              │
│           │ extends/implements                           │
│           │                                              │
│           ▼                                              │
│  ┌────────────────┐                                     │
│  │   New Code     │  ← OPEN for extension               │
│  │  (Extension)   │                                     │
│  └────────────────┘                                     │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Motivation

### The Problem: Modification Cascade

Without OCP, adding new features requires modifying existing code, which:

```
┌─────────────────────────────────────────────────────────┐
│           Modification Cascade Problem                   │
└─────────────────────────────────────────────────────────┘

New Feature Request
        │
        ▼
Modify Existing Class
        │
        ├─────────────────────────────────┐
        │                                 │
        ▼                                 ▼
  Break Existing        Introduce New Bugs
    Features                  │
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
          Regression Testing
                   │
                   ▼
            More Bugs Found
                   │
                   ▼
              Fix Bugs
                   │
                   ▼
          Deploy with Risk
```

**Example Violation:**
```java
// BAD: Must modify for each new shape
class AreaCalculator {
    double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.radius * c.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.width * r.height;
        } else if (shape instanceof Triangle) {  // Added new shape
            Triangle t = (Triangle) shape;
            return 0.5 * t.base * t.height;
        }
        // Must keep adding more if-else!
        return 0;
    }
}
```

### The Solution: Extension Through Abstraction

```
┌─────────────────────────────────────────────────────────┐
│              OCP Solution Pattern                        │
└─────────────────────────────────────────────────────────┘

         ┌──────────────┐
         │  <<interface>>│
         │     Shape    │
         │              │
         │+ area()      │
         └──────┬───────┘
                │
    ┌───────────┼───────────┬───────────┐
    │           │           │           │
┌───▼───┐  ┌───▼───┐  ┌───▼───┐  ┌───▼───┐
│Circle │  │Rectangle│ │Triangle│ │ New!  │
│       │  │       │  │       │  │Hexagon│
│area() │  │area() │  │area() │  │area() │
└───────┘  └───────┘  └───────┘  └───────┘

No modification needed to add new shapes!
```

---

## Structure

### Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│           Open/Closed Structure                          │
└─────────────────────────────────────────────────────────┘

Before OCP (Violation):
┌────────────────────────────────┐
│      PaymentProcessor          │
├────────────────────────────────┤
│+ processPayment(type, amount)  │
│  {                             │
│    if (type == "credit")       │
│      processCreditCard()       │
│    else if (type == "paypal")  │
│      processPayPal()           │
│    else if (type == "crypto")  │ ← Must modify!
│      processCrypto()           │
│  }                             │
└────────────────────────────────┘

After OCP (Proper):
         ┌──────────────────┐
         │  <<interface>>   │
         │PaymentMethod     │
         ├──────────────────┤
         │+ process(amount) │
         └────────┬─────────┘
                  │
      ┌───────────┼───────────┬──────────────┐
      │           │           │              │
┌─────▼─────┐ ┌──▼──────┐ ┌──▼────────┐ ┌──▼────────┐
│CreditCard │ │ PayPal  │ │  Crypto   │ │  New!     │
│Payment    │ │ Payment │ │  Payment  │ │  Bitcoin  │
├───────────┤ ├─────────┤ ├───────────┤ ├───────────┤
│+ process()│ │+process()│ │+ process()│ │+ process()│
└───────────┘ └─────────┘ └───────────┘ └───────────┘

┌────────────────────────────────┐
│      PaymentProcessor          │
├────────────────────────────────┤
│- method: PaymentMethod         │
├────────────────────────────────┤
│+ processPayment(amount)        │
│  {                             │
│    method.process(amount)      │ ← No modification!
│  }                             │
└────────────────────────────────┘
```

---

## Key Concepts

### 1. Abstraction

Use interfaces or abstract classes to define contracts:

```java
// Define abstraction
interface Shape {
    double calculateArea();
}

// Concrete implementations
class Circle implements Shape {
    private double radius;
    
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    private double width, height;
    
    public double calculateArea() {
        return width * height;
    }
}
```

### 2. Polymorphism

Use polymorphism to work with abstractions:

```java
// Works with any Shape implementation
class AreaCalculator {
    double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::calculateArea)
                    .sum();
    }
}
```

### 3. Extension Points

Design classes with clear extension points:

```java
// Extension point through abstract method
abstract class ReportGenerator {
    // Template method (closed for modification)
    public final String generate() {
        String header = generateHeader();
        String body = generateBody();      // Extension point
        String footer = generateFooter();
        return header + body + footer;
    }
    
    // Extension point (open for extension)
    protected abstract String generateBody();
    
    private String generateHeader() { return "Header\n"; }
    private String generateFooter() { return "\nFooter"; }
}

// Extend without modifying base class
class SalesReport extends ReportGenerator {
    protected String generateBody() {
        return "Sales data...";
    }
}
```

---

## Examples

### Example 1: Shape Calculator

#### ❌ Violation

```java
// BAD: Must modify for each new shape
public class AreaCalculator {
    public double calculateArea(Object shape) {
        double area = 0;
        
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            area = Math.PI * circle.getRadius() * circle.getRadius();
        } 
        else if (shape instanceof Rectangle) {
            Rectangle rect = (Rectangle) shape;
            area = rect.getWidth() * rect.getHeight();
        }
        else if (shape instanceof Triangle) {
            Triangle tri = (Triangle) shape;
            area = 0.5 * tri.getBase() * tri.getHeight();
        }
        // Must add more if-else for new shapes!
        
        return area;
    }
}
```

**Problems:**
- Must modify `AreaCalculator` for each new shape
- Violates Single Responsibility (knows about all shapes)
- Type checking with `instanceof` is a code smell
- Cannot add shapes without changing existing code
- High risk of breaking existing functionality

#### ✅ Proper Implementation

```java
// GOOD: Open for extension, closed for modification
public interface Shape {
    double calculateArea();
}

public class Circle implements Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    private double width;
    private double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Triangle implements Shape {
    private double base;
    private double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// Can add new shapes without modifying existing code!
public class Hexagon implements Shape {
    private double side;
    
    public Hexagon(double side) {
        this.side = side;
    }
    
    @Override
    public double calculateArea() {
        return (3 * Math.sqrt(3) / 2) * side * side;
    }
}

// Calculator never needs to change
public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
    
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::calculateArea)
                    .sum();
    }
}

// Usage
List<Shape> shapes = Arrays.asList(
    new Circle(5),
    new Rectangle(4, 6),
    new Triangle(3, 4),
    new Hexagon(2)  // New shape, no changes needed!
);

AreaCalculator calculator = new AreaCalculator();
double totalArea = calculator.calculateTotalArea(shapes);
```

### Example 2: Payment Processing (Python)

#### ❌ Violation

```python
# BAD: Must modify for each new payment method
class PaymentProcessor:
    def process_payment(self, payment_type, amount):
        if payment_type == "credit_card":
            # Credit card processing logic
            print(f"Processing ${amount} via Credit Card")
            # Connect to credit card gateway
            # Validate card
            # Charge card
            
        elif payment_type == "paypal":
            # PayPal processing logic
            print(f"Processing ${amount} via PayPal")
            # Connect to PayPal API
            # Authenticate
            # Transfer funds
            
        elif payment_type == "crypto":
            # Cryptocurrency processing logic
            print(f"Processing ${amount} via Crypto")
            # Connect to blockchain
            # Create transaction
            # Confirm transaction
            
        # Must add more elif for new payment methods!
        else:
            raise ValueError(f"Unknown payment type: {payment_type}")
```

#### ✅ Proper Implementation

```python
# GOOD: Open for extension, closed for modification
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentMethod):
    def __init__(self, card_number, cvv):
        self.card_number = card_number
        self.cvv = cvv
    
    def process(self, amount):
        print(f"Processing ${amount} via Credit Card")
        # Credit card specific logic
        self._validate_card()
        self._charge_card(amount)
        return True
    
    def _validate_card(self):
        # Validation logic
        pass
    
    def _charge_card(self, amount):
        # Charging logic
        pass

class PayPalPayment(PaymentMethod):
    def __init__(self, email):
        self.email = email
    
    def process(self, amount):
        print(f"Processing ${amount} via PayPal")
        # PayPal specific logic
        self._authenticate()
        self._transfer_funds(amount)
        return True
    
    def _authenticate(self):
        # Authentication logic
        pass
    
    def _transfer_funds(self, amount):
        # Transfer logic
        pass

class CryptoPayment(PaymentMethod):
    def __init__(self, wallet_address):
        self.wallet_address = wallet_address
    
    def process(self, amount):
        print(f"Processing ${amount} via Cryptocurrency")
        # Crypto specific logic
        self._create_transaction(amount)
        self._confirm_transaction()
        return True
    
    def _create_transaction(self, amount):
        # Transaction creation logic
        pass
    
    def _confirm_transaction(self):
        # Confirmation logic
        pass

# Can add new payment methods without modifying existing code!
class ApplePayPayment(PaymentMethod):
    def __init__(self, device_id):
        self.device_id = device_id
    
    def process(self, amount):
        print(f"Processing ${amount} via Apple Pay")
        # Apple Pay specific logic
        return True

# Processor never needs to change
class PaymentProcessor:
    def __init__(self, payment_method: PaymentMethod):
        self.payment_method = payment_method
    
    def process_payment(self, amount):
        return self.payment_method.process(amount)

# Usage
credit_card = CreditCardPayment("1234-5678-9012-3456", "123")
processor = PaymentProcessor(credit_card)
processor.process_payment(100.00)

# Switch to different payment method
paypal = PayPalPayment("user@example.com")
processor = PaymentProcessor(paypal)
processor.process_payment(50.00)

# Add new payment method without changing existing code
apple_pay = ApplePayPayment("device-123")
processor = PaymentProcessor(apple_pay)
processor.process_payment(75.00)
```

### Example 3: Notification System (JavaScript)

#### ❌ Violation

```javascript
// BAD: Must modify for each new notification channel
class NotificationService {
    sendNotification(type, message, recipient) {
        if (type === 'email') {
            // Email logic
            console.log(`Sending email to ${recipient}: ${message}`);
            const transporter = createEmailTransporter();
            transporter.sendMail({
                to: recipient,
                subject: 'Notification',
                text: message
            });
        } 
        else if (type === 'sms') {
            // SMS logic
            console.log(`Sending SMS to ${recipient}: ${message}`);
            const smsClient = createSMSClient();
            smsClient.send({
                to: recipient,
                body: message
            });
        }
        else if (type === 'push') {
            // Push notification logic
            console.log(`Sending push to ${recipient}: ${message}`);
            const pushService = createPushService();
            pushService.send({
                deviceId: recipient,
                message: message
            });
        }
        // Must add more if-else for new channels!
        else {
            throw new Error(`Unknown notification type: ${type}`);
        }
    }
}
```

#### ✅ Proper Implementation

```javascript
// GOOD: Open for extension, closed for modification
class NotificationChannel {
    send(message, recipient) {
        throw new Error('send() must be implemented');
    }
}

class EmailNotification extends NotificationChannel {
    constructor(transporter) {
        super();
        this.transporter = transporter;
    }
    
    send(message, recipient) {
        console.log(`Sending email to ${recipient}: ${message}`);
        return this.transporter.sendMail({
            to: recipient,
            subject: 'Notification',
            text: message
        });
    }
}

class SMSNotification extends NotificationChannel {
    constructor(smsClient) {
        super();
        this.smsClient = smsClient;
    }
    
    send(message, recipient) {
        console.log(`Sending SMS to ${recipient}: ${message}`);
        return this.smsClient.send({
            to: recipient,
            body: message
        });
    }
}

class PushNotification extends NotificationChannel {
    constructor(pushService) {
        super();
        this.pushService = pushService;
    }
    
    send(message, recipient) {
        console.log(`Sending push to ${recipient}: ${message}`);
        return this.pushService.send({
            deviceId: recipient,
            message: message
        });
    }
}

// Can add new channels without modifying existing code!
class SlackNotification extends NotificationChannel {
    constructor(slackClient) {
        super();
        this.slackClient = slackClient;
    }
    
    send(message, recipient) {
        console.log(`Sending Slack message to ${recipient}: ${message}`);
        return this.slackClient.postMessage({
            channel: recipient,
            text: message
        });
    }
}

class WhatsAppNotification extends NotificationChannel {
    constructor(whatsappClient) {
        super();
        this.whatsappClient = whatsappClient;
    }
    
    send(message, recipient) {
        console.log(`Sending WhatsApp to ${recipient}: ${message}`);
        return this.whatsappClient.sendMessage({
            to: recipient,
            body: message
        });
    }
}

// Service never needs to change
class NotificationService {
    constructor(channel) {
        this.channel = channel;
    }
    
    setChannel(channel) {
        this.channel = channel;
    }
    
    sendNotification(message, recipient) {
        return this.channel.send(message, recipient);
    }
}

// Multi-channel support
class MultiChannelNotificationService {
    constructor() {
        this.channels = [];
    }
    
    addChannel(channel) {
        this.channels.push(channel);
    }
    
    async sendNotification(message, recipient) {
        const promises = this.channels.map(channel => 
            channel.send(message, recipient)
        );
        return Promise.all(promises);
    }
}

// Usage
const emailService = new NotificationService(
    new EmailNotification(emailTransporter)
);
emailService.sendNotification('Hello!', 'user@example.com');

// Switch channels easily
emailService.setChannel(new SMSNotification(smsClient));
emailService.sendNotification('Hello!', '+1234567890');

// Multi-channel
const multiService = new MultiChannelNotificationService();
multiService.addChannel(new EmailNotification(emailTransporter));
multiService.addChannel(new SMSNotification(smsClient));
multiService.addChannel(new SlackNotification(slackClient));
multiService.sendNotification('Important update!', 'user@example.com');
```

### Example 4: Logging System

#### ❌ Violation

```java
// BAD: Must modify for each new log destination
public class Logger {
    public void log(String message, String destination) {
        if (destination.equals("console")) {
            System.out.println(message);
        } 
        else if (destination.equals("file")) {
            try (FileWriter fw = new FileWriter("app.log", true)) {
                fw.write(message + "\n");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        else if (destination.equals("database")) {
            Connection conn = getConnection();
            PreparedStatement stmt = conn.prepareStatement(
                "INSERT INTO logs (message) VALUES (?)"
            );
            stmt.setString(1, message);
            stmt.executeUpdate();
        }
        // Must add more if-else!
    }
}
```

#### ✅ Proper Implementation

```java
// GOOD: Open for extension, closed for modification
public interface LogDestination {
    void write(String message);
}

public class ConsoleLogger implements LogDestination {
    @Override
    public void write(String message) {
        System.out.println(message);
    }
}

public class FileLogger implements LogDestination {
    private String filename;
    
    public FileLogger(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void write(String message) {
        try (FileWriter fw = new FileWriter(filename, true)) {
            fw.write(message + "\n");
        } catch (IOException e) {
            throw new RuntimeException("Failed to write to file", e);
        }
    }
}

public class DatabaseLogger implements LogDestination {
    private Connection connection;
    
    public DatabaseLogger(Connection connection) {
        this.connection = connection;
    }
    
    @Override
    public void write(String message) {
        try {
            PreparedStatement stmt = connection.prepareStatement(
                "INSERT INTO logs (message, timestamp) VALUES (?, ?)"
            );
            stmt.setString(1, message);
            stmt.setTimestamp(2, new Timestamp(System.currentTimeMillis()));
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Failed to write to database", e);
        }
    }
}

// Can add new destinations without modifying existing code!
public class CloudLogger implements LogDestination {
    private CloudService cloudService;
    
    public CloudLogger(CloudService cloudService) {
        this.cloudService = cloudService;
    }
    
    @Override
    public void write(String message) {
        cloudService.sendLog(message);
    }
}

public class ElasticsearchLogger implements LogDestination {
    private RestHighLevelClient client;
    
    public ElasticsearchLogger(RestHighLevelClient client) {
        this.client = client;
    }
    
    @Override
    public void write(String message) {
        // Elasticsearch indexing logic
    }
}

// Logger never needs to change
public class Logger {
    private List<LogDestination> destinations;
    
    public Logger() {
        this.destinations = new ArrayList<>();
    }
    
    public void addDestination(LogDestination destination) {
        destinations.add(destination);
    }
    
    public void log(String message) {
        String timestamp = LocalDateTime.now().toString();
        String formattedMessage = String.format("[%s] %s", timestamp, message);
        
        for (LogDestination destination : destinations) {
            destination.write(formattedMessage);
        }
    }
}

// Usage
Logger logger = new Logger();
logger.addDestination(new ConsoleLogger());
logger.addDestination(new FileLogger("app.log"));
logger.addDestination(new DatabaseLogger(connection));
logger.addDestination(new CloudLogger(cloudService));

logger.log("Application started");  // Logs to all destinations
```

---

## Before and After

### Comparison Table

| Aspect | Before OCP | After OCP |
|--------|-----------|-----------|
| **Adding Features** | Modify existing code | Add new classes |
| **Risk** | High (can break existing) | Low (existing code untouched) |
| **Testing** | Must retest everything | Test only new code |
| **Code Changes** | Scattered modifications | Localized additions |
| **Flexibility** | Rigid | Flexible |
| **Maintenance** | Difficult | Easy |

### Metrics

**Before OCP:**
- Time to add feature: 2 days (including testing)
- Lines changed: 50-100
- Risk of regression: 40%
- Test coverage needed: 100%

**After OCP:**
- Time to add feature: 4 hours
- Lines changed: 0 (only additions)
- Risk of regression: 5%
- Test coverage needed: Only new code

---

## Real-World Examples

### Example 1: Middleware Pattern (Express.js)

```javascript
// OCP in action - add middleware without modifying framework
app.use(bodyParser.json());           // Extension
app.use(cors());                       // Extension
app.use(authenticate);                 // Extension
app.use(rateLimit);                    // Extension
app.use(customMiddleware);             // Extension

// Framework code never changes!
```

### Example 2: Plugin Architecture (WordPress)

```php
// Add functionality through plugins (extensions)
add_action('init', 'my_custom_function');
add_filter('the_content', 'modify_content');

// WordPress core remains unchanged
```

### Example 3: Strategy Pattern (Java Collections)

```java
// Sort with different strategies (extensions)
Collections.sort(list, Comparator.naturalOrder());
Collections.sort(list, Comparator.reverseOrder());
Collections.sort(list, (a, b) -> a.length() - b.length());

// Collections.sort() never changes!
```

### Famous Products

1. **Spring Framework**: Dependency injection, AOP
2. **Eclipse IDE**: Plugin architecture
3. **Visual Studio Code**: Extension marketplace
4. **Jenkins**: Plugin system
5. **Kubernetes**: Operator pattern

---

## Benefits

### 1. Reduced Risk
- Existing code remains stable
- No regression in tested functionality
- Changes are isolated

### 2. Easier Testing
```java
// Only test new implementation
@Test
public void testNewShape() {
    Shape hexagon = new Hexagon(5);
    assertEquals(64.95, hexagon.calculateArea(), 0.01);
}
// No need to retest Circle, Rectangle, etc.
```

### 3. Better Maintainability
- Clear extension points
- Predictable structure
- Easy to understand

### 4. Increased Flexibility
- Easy to add features
- Easy to swap implementations
- Supports plugin architecture

### 5. Parallel Development
- Multiple developers can add features simultaneously
- No merge conflicts in existing code
- Faster development

---

## Common Violations

### 1. Type Checking

```java
// BAD: Type checking violates OCP
if (shape instanceof Circle) {
    // ...
} else if (shape instanceof Rectangle) {
    // ...
}
```

### 2. Switch Statements

```java
// BAD: Switch on type
switch (paymentType) {
    case "credit": processCreditCard(); break;
    case "paypal": processPayPal(); break;
    case "crypto": processCrypto(); break;
}
```

### 3. Configuration-Based Branching

```java
// BAD: Configuration determines behavior
if (config.get("payment.type").equals("credit")) {
    // credit card logic
} else if (config.get("payment.type").equals("paypal")) {
    // paypal logic
}
```

### 4. Modifying Existing Methods

```java
// BAD: Adding parameters to existing methods
// Before
void process(Order order) { }

// After - breaks existing callers!
void process(Order order, boolean sendEmail, boolean logToDatabase) { }
```

---

## How to Apply

### Step 1: Identify Extension Points

Look for code that changes frequently:
- Adding new types/categories
- Adding new behaviors
- Adding new algorithms

### Step 2: Create Abstractions

```java
// Extract interface
interface PaymentMethod {
    void process(double amount);
}
```

### Step 3: Implement Variations

```java
// Create concrete implementations
class CreditCardPayment implements PaymentMethod { }
class PayPalPayment implements PaymentMethod { }
```

### Step 4: Use Polymorphism

```java
// Work with abstraction
class PaymentProcessor {
    void process(PaymentMethod method, double amount) {
        method.process(amount);
    }
}
```

### Step 5: Add New Features by Extension

```java
// Add new implementation without modifying existing code
class BitcoinPayment implements PaymentMethod {
    public void process(double amount) {
        // Bitcoin logic
    }
}
```

---

## Related Patterns

### Patterns that Embody OCP

1. **Strategy Pattern**: Encapsulate algorithms
2. **Template Method**: Define algorithm skeleton
3. **Decorator Pattern**: Add responsibilities dynamically
4. **Observer Pattern**: Subscribe to events
5. **Factory Pattern**: Create objects without specifying class
6. **Command Pattern**: Encapsulate requests

### Example: Strategy Pattern

```java
// OCP through Strategy Pattern
interface SortStrategy {
    void sort(int[] array);
}

class QuickSort implements SortStrategy {
    public void sort(int[] array) { /* quicksort */ }
}

class MergeSort implements SortStrategy {
    public void sort(int[] array) { /* mergesort */ }
}

// Add new strategy without modifying Sorter
class HeapSort implements SortStrategy {
    public void sort(int[] array) { /* heapsort */ }
}

class Sorter {
    private SortStrategy strategy;
    
    void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    void sort(int[] array) {
        strategy.sort(array);  // Never changes!
    }
}
```

---

## Conclusion

The Open/Closed Principle promotes:
- **Stability**: Existing code remains unchanged
- **Extensibility**: Easy to add new features
- **Maintainability**: Clear structure and responsibilities
- **Testability**: Test only new code

**Remember:**
> "The best way to predict the future is to design for it."

Design your classes with extension in mind, and you'll build systems that gracefully adapt to change.

---

## Quick Reference

### ✅ Do's
- Design with abstractions (interfaces/abstract classes)
- Use polymorphism
- Create clear extension points
- Favor composition over modification
- Think about future extensions

### ❌ Don'ts
- Use type checking (instanceof, typeof)
- Use switch statements on types
- Modify existing methods for new features
- Hardcode behavior
- Ignore extension points

---

**Last Updated:** January 2026  
**Version:** 1.0
