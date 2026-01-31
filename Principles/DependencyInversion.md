# Dependency Inversion Principle (DIP)

## Definition

> "High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."
> 
> — Robert C. Martin

**Simplified:** "Depend on abstractions, not on concretions."

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

The Dependency Inversion Principle inverts the traditional dependency structure where high-level modules depend on low-level modules. Instead, both should depend on abstractions, making the system more flexible and maintainable.

### Core Idea

```
┌─────────────────────────────────────────────────────────┐
│         Dependency Inversion Principle                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Traditional Dependency (BAD):                          │
│  ┌──────────────────┐                                   │
│  │  High-Level      │                                   │
│  │  Module          │                                   │
│  └────────┬─────────┘                                   │
│           │ depends on                                   │
│           ▼                                              │
│  ┌──────────────────┐                                   │
│  │  Low-Level       │                                   │
│  │  Module          │                                   │
│  └──────────────────┘                                   │
│                                                          │
│  Inverted Dependency (GOOD):                            │
│  ┌──────────────────┐                                   │
│  │  High-Level      │                                   │
│  │  Module          │                                   │
│  └────────┬─────────┘                                   │
│           │ depends on                                   │
│           ▼                                              │
│  ┌──────────────────┐                                   │
│  │  <<interface>>   │                                   │
│  │  Abstraction     │                                   │
│  └────────▲─────────┘                                   │
│           │ implements                                   │
│           │                                              │
│  ┌────────┴─────────┐                                   │
│  │  Low-Level       │                                   │
│  │  Module          │                                   │
│  └──────────────────┘                                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Motivation

### The Problem: Tight Coupling

Without DIP, high-level business logic depends directly on low-level implementation details:

```
┌─────────────────────────────────────────────────────────┐
│              Tight Coupling Problem                      │
└─────────────────────────────────────────────────────────┘

┌────────────────────┐
│   UserService      │  ← High-level business logic
│  (Business Logic)  │
└─────────┬──────────┘
          │ depends on (tight coupling)
          ▼
┌────────────────────┐
│  MySQLDatabase     │  ← Low-level implementation
│  (Infrastructure)  │
└────────────────────┘

Problems:
1. Cannot switch to PostgreSQL without changing UserService
2. Cannot test UserService without real database
3. Changes in MySQLDatabase affect UserService
4. High-level logic tied to low-level details
```

**Example Violation:**
```java
// BAD: High-level depends on low-level
public class UserService {
    private MySQLDatabase database;  // Tight coupling!
    
    public UserService() {
        this.database = new MySQLDatabase();  // Direct instantiation
    }
    
    public void saveUser(User user) {
        database.connect("localhost", "root", "password");
        database.executeUpdate("INSERT INTO users...");
        database.disconnect();
    }
}
```

### The Solution: Depend on Abstractions

```
┌─────────────────────────────────────────────────────────┐
│              DIP Solution Pattern                        │
└─────────────────────────────────────────────────────────┘

┌────────────────────┐
│   UserService      │  ← High-level
│  (Business Logic)  │
└─────────┬──────────┘
          │ depends on
          ▼
┌────────────────────┐
│  <<interface>>     │  ← Abstraction
│  Database          │
└─────────▲──────────┘
          │ implements
          │
┌─────────┴──────────┬────────────────┐
│                    │                │
▼                    ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│MySQLDatabase │ │PostgreSQL    │ │MongoDatabase │
└──────────────┘ └──────────────┘ └──────────────┘

Benefits:
1. Can switch databases easily
2. Can test with mock database
3. High-level logic independent of details
4. Flexible and maintainable
```

---

## Structure

### Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│         Dependency Inversion Structure                   │
└─────────────────────────────────────────────────────────┘

Violation (Direct Dependency):
┌────────────────────┐
│   OrderService     │
├────────────────────┤
│- emailSender:      │
│  SMTPEmailSender   │ ← Concrete dependency
├────────────────────┤
│+ placeOrder()      │
└────────────────────┘
         │
         │ creates
         ▼
┌────────────────────┐
│  SMTPEmailSender   │
├────────────────────┤
│+ sendEmail()       │
└────────────────────┘

Proper Design (Dependency Inversion):
┌────────────────────┐
│   OrderService     │
├────────────────────┤
│- emailSender:      │
│  EmailSender       │ ← Interface dependency
├────────────────────┤
│+ placeOrder()      │
└─────────┬──────────┘
          │ depends on
          ▼
┌────────────────────┐
│  <<interface>>     │
│   EmailSender      │
├────────────────────┤
│+ sendEmail()       │
└─────────▲──────────┘
          │ implements
          │
┌─────────┴──────────┬────────────────┐
│                    │                │
▼                    ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│SMTPEmailSender│ │SendGridSender│ │MockEmailSender│
├──────────────┤ ├──────────────┤ ├──────────────┤
│+ sendEmail() │ │+ sendEmail() │ │+ sendEmail() │
└──────────────┘ └──────────────┘ └──────────────┘
```

---

## Key Concepts

### 1. Abstraction vs. Concretion

**Abstraction:** Interface or abstract class defining contract
**Concretion:** Concrete implementation with specific details

```java
// Abstraction
interface PaymentGateway {
    boolean processPayment(double amount);
}

// Concretion
class StripePaymentGateway implements PaymentGateway {
    public boolean processPayment(double amount) {
        // Stripe-specific implementation
        return true;
    }
}
```

### 2. Dependency Injection

DIP is often implemented through Dependency Injection:

```java
// Constructor injection
public class OrderService {
    private final PaymentGateway paymentGateway;
    
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;  // Injected
    }
}

// Usage
PaymentGateway gateway = new StripePaymentGateway();
OrderService service = new OrderService(gateway);
```

### 3. Inversion of Control (IoC)

Control of dependencies is inverted from the class to the caller:

```java
// Without IoC - class controls dependencies
class Service {
    private Database db = new MySQLDatabase();  // Class decides
}

// With IoC - caller controls dependencies
class Service {
    private Database db;
    
    Service(Database db) {  // Caller decides
        this.db = db;
    }
}
```

---

## Examples

### Example 1: Database Access

#### ❌ Violation

```java
// BAD: High-level depends on low-level
public class UserService {
    private MySQLDatabase database;
    
    public UserService() {
        // Direct instantiation - tight coupling
        this.database = new MySQLDatabase();
        this.database.connect("localhost", "root", "password");
    }
    
    public void saveUser(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        database.executeUpdate(sql, user.getName(), user.getEmail());
    }
    
    public User findUser(int id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        ResultSet rs = database.executeQuery(sql, id);
        // Parse ResultSet...
        return null;
    }
}

// Problems:
// 1. Cannot switch to PostgreSQL without changing UserService
// 2. Cannot test without real MySQL database
// 3. Hard to mock for unit tests
// 4. Violates Single Responsibility (knows about SQL)
```

#### ✅ Proper Implementation

```java
// GOOD: Depend on abstraction
public interface UserRepository {
    void save(User user);
    User findById(int id);
    List<User> findAll();
}

public class MySQLUserRepository implements UserRepository {
    private final Connection connection;
    
    public MySQLUserRepository(Connection connection) {
        this.connection = connection;
    }
    
    @Override
    public void save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setString(1, user.getName());
            stmt.setString(2, user.getEmail());
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Failed to save user", e);
        }
    }
    
    @Override
    public User findById(int id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setInt(1, id);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                return new User(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getString("email")
                );
            }
        } catch (SQLException e) {
            throw new RuntimeException("Failed to find user", e);
        }
        return null;
    }
    
    @Override
    public List<User> findAll() {
        // Implementation...
        return new ArrayList<>();
    }
}

public class PostgreSQLUserRepository implements UserRepository {
    private final Connection connection;
    
    public PostgreSQLUserRepository(Connection connection) {
        this.connection = connection;
    }
    
    @Override
    public void save(User user) {
        // PostgreSQL-specific implementation
    }
    
    @Override
    public User findById(int id) {
        // PostgreSQL-specific implementation
        return null;
    }
    
    @Override
    public List<User> findAll() {
        // PostgreSQL-specific implementation
        return new ArrayList<>();
    }
}

// High-level service depends on abstraction
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;  // Dependency injection
    }
    
    public void registerUser(String name, String email) {
        User user = new User(0, name, email);
        repository.save(user);
    }
    
    public User getUser(int id) {
        return repository.findById(id);
    }
}

// Usage - can easily switch implementations
UserRepository mysqlRepo = new MySQLUserRepository(mysqlConnection);
UserService service1 = new UserService(mysqlRepo);

UserRepository postgresRepo = new PostgreSQLUserRepository(postgresConnection);
UserService service2 = new UserService(postgresRepo);

// Testing - easy to mock
UserRepository mockRepo = mock(UserRepository.class);
UserService testService = new UserService(mockRepo);
```

### Example 2: Notification System (Python)

#### ❌ Violation

```python
# BAD: High-level depends on low-level
import smtplib
from email.mime.text import MIMEText

class OrderService:
    def __init__(self):
        # Direct dependency on SMTP
        self.smtp_server = "smtp.gmail.com"
        self.smtp_port = 587
        self.username = "user@gmail.com"
        self.password = "password"
    
    def place_order(self, order):
        # Process order...
        print(f"Order placed: {order}")
        
        # Send confirmation email - tightly coupled to SMTP
        self._send_email(order.customer_email, "Order Confirmation")
    
    def _send_email(self, to, subject):
        msg = MIMEText("Your order has been confirmed")
        msg['Subject'] = subject
        msg['From'] = self.username
        msg['To'] = to
        
        server = smtplib.SMTP(self.smtp_server, self.smtp_port)
        server.starttls()
        server.login(self.username, self.password)
        server.send_message(msg)
        server.quit()

# Problems:
# 1. Cannot switch to SendGrid without changing OrderService
# 2. Cannot test without real SMTP server
# 3. OrderService knows SMTP details (violates SRP)
```

#### ✅ Proper Implementation

```python
# GOOD: Depend on abstraction
from abc import ABC, abstractmethod

class NotificationService(ABC):
    @abstractmethod
    def send(self, recipient, subject, message):
        pass

class EmailNotificationService(NotificationService):
    def __init__(self, smtp_server, smtp_port, username, password):
        self.smtp_server = smtp_server
        self.smtp_port = smtp_port
        self.username = username
        self.password = password
    
    def send(self, recipient, subject, message):
        import smtplib
        from email.mime.text import MIMEText
        
        msg = MIMEText(message)
        msg['Subject'] = subject
        msg['From'] = self.username
        msg['To'] = recipient
        
        server = smtplib.SMTP(self.smtp_server, self.smtp_port)
        server.starttls()
        server.login(self.username, self.password)
        server.send_message(msg)
        server.quit()
        print(f"Email sent to {recipient}")

class SMSNotificationService(NotificationService):
    def __init__(self, api_key):
        self.api_key = api_key
    
    def send(self, recipient, subject, message):
        # SMS API implementation
        print(f"SMS sent to {recipient}: {message}")

class PushNotificationService(NotificationService):
    def __init__(self, api_key):
        self.api_key = api_key
    
    def send(self, recipient, subject, message):
        # Push notification implementation
        print(f"Push notification sent to {recipient}: {message}")

# High-level service depends on abstraction
class OrderService:
    def __init__(self, notification_service: NotificationService):
        self.notification_service = notification_service
    
    def place_order(self, order):
        # Process order...
        print(f"Order placed: {order}")
        
        # Send notification through abstraction
        self.notification_service.send(
            order.customer_email,
            "Order Confirmation",
            f"Your order #{order.id} has been confirmed"
        )

# Usage - can easily switch implementations
email_service = EmailNotificationService(
    "smtp.gmail.com", 587, "user@gmail.com", "password"
)
order_service1 = OrderService(email_service)

sms_service = SMSNotificationService("api_key_123")
order_service2 = OrderService(sms_service)

# Testing - easy to mock
class MockNotificationService(NotificationService):
    def __init__(self):
        self.sent_messages = []
    
    def send(self, recipient, subject, message):
        self.sent_messages.append((recipient, subject, message))

mock_service = MockNotificationService()
test_order_service = OrderService(mock_service)
```

### Example 3: Payment Processing (JavaScript)

#### ❌ Violation

```javascript
// BAD: High-level depends on low-level
class OrderProcessor {
    constructor() {
        // Direct dependency on Stripe
        this.stripe = require('stripe')('sk_test_...');
    }
    
    async processOrder(order) {
        // Process order logic...
        
        // Payment processing - tightly coupled to Stripe
        try {
            const paymentIntent = await this.stripe.paymentIntents.create({
                amount: order.total * 100,
                currency: 'usd',
                payment_method: order.paymentMethodId,
                confirm: true
            });
            
            console.log('Payment successful:', paymentIntent.id);
            return { success: true, transactionId: paymentIntent.id };
        } catch (error) {
            console.error('Payment failed:', error);
            return { success: false, error: error.message };
        }
    }
}

// Problems:
// 1. Cannot switch to PayPal without changing OrderProcessor
// 2. Cannot test without Stripe API
// 3. OrderProcessor knows Stripe API details
```

#### ✅ Proper Implementation

```javascript
// GOOD: Depend on abstraction
class PaymentGateway {
    async processPayment(amount, paymentDetails) {
        throw new Error('processPayment must be implemented');
    }
}

class StripePaymentGateway extends PaymentGateway {
    constructor(apiKey) {
        super();
        this.stripe = require('stripe')(apiKey);
    }
    
    async processPayment(amount, paymentDetails) {
        try {
            const paymentIntent = await this.stripe.paymentIntents.create({
                amount: amount * 100,
                currency: 'usd',
                payment_method: paymentDetails.paymentMethodId,
                confirm: true
            });
            
            return {
                success: true,
                transactionId: paymentIntent.id
            };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
}

class PayPalPaymentGateway extends PaymentGateway {
    constructor(clientId, clientSecret) {
        super();
        this.clientId = clientId;
        this.clientSecret = clientSecret;
    }
    
    async processPayment(amount, paymentDetails) {
        // PayPal API implementation
        console.log(`Processing $${amount} via PayPal`);
        return {
            success: true,
            transactionId: 'paypal_' + Date.now()
        };
    }
}

class MockPaymentGateway extends PaymentGateway {
    async processPayment(amount, paymentDetails) {
        console.log(`Mock payment: $${amount}`);
        return {
            success: true,
            transactionId: 'mock_' + Date.now()
        };
    }
}

// High-level service depends on abstraction
class OrderProcessor {
    constructor(paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
    
    async processOrder(order) {
        console.log(`Processing order #${order.id}`);
        
        // Process payment through abstraction
        const result = await this.paymentGateway.processPayment(
            order.total,
            order.paymentDetails
        );
        
        if (result.success) {
            console.log('Order completed:', result.transactionId);
        } else {
            console.error('Order failed:', result.error);
        }
        
        return result;
    }
}

// Usage - can easily switch implementations
const stripeGateway = new StripePaymentGateway('sk_test_...');
const processor1 = new OrderProcessor(stripeGateway);

const paypalGateway = new PayPalPaymentGateway('client_id', 'secret');
const processor2 = new OrderProcessor(paypalGateway);

// Testing - easy to mock
const mockGateway = new MockPaymentGateway();
const testProcessor = new OrderProcessor(mockGateway);
```

### Example 4: Logging System

#### ❌ Violation

```java
// BAD: High-level depends on low-level
public class UserController {
    public void createUser(User user) {
        // Business logic...
        
        // Logging - tightly coupled to console
        System.out.println("User created: " + user.getName());
        
        // Later need to log to file...
        // Must modify UserController!
    }
}
```

#### ✅ Proper Implementation

```java
// GOOD: Depend on abstraction
public interface Logger {
    void log(String message);
    void error(String message);
    void warn(String message);
}

public class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("[INFO] " + message);
    }
    
    @Override
    public void error(String message) {
        System.err.println("[ERROR] " + message);
    }
    
    @Override
    public void warn(String message) {
        System.out.println("[WARN] " + message);
    }
}

public class FileLogger implements Logger {
    private final String filename;
    
    public FileLogger(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void log(String message) {
        writeToFile("[INFO] " + message);
    }
    
    @Override
    public void error(String message) {
        writeToFile("[ERROR] " + message);
    }
    
    @Override
    public void warn(String message) {
        writeToFile("[WARN] " + message);
    }
    
    private void writeToFile(String message) {
        // File writing implementation
    }
}

public class CompositeLogger implements Logger {
    private final List<Logger> loggers;
    
    public CompositeLogger(Logger... loggers) {
        this.loggers = Arrays.asList(loggers);
    }
    
    @Override
    public void log(String message) {
        loggers.forEach(logger -> logger.log(message));
    }
    
    @Override
    public void error(String message) {
        loggers.forEach(logger -> logger.error(message));
    }
    
    @Override
    public void warn(String message) {
        loggers.forEach(logger -> logger.warn(message));
    }
}

// High-level controller depends on abstraction
public class UserController {
    private final Logger logger;
    
    public UserController(Logger logger) {
        this.logger = logger;
    }
    
    public void createUser(User user) {
        // Business logic...
        
        logger.log("User created: " + user.getName());
    }
}

// Usage - flexible configuration
Logger consoleLogger = new ConsoleLogger();
Logger fileLogger = new FileLogger("app.log");
Logger compositeLogger = new CompositeLogger(consoleLogger, fileLogger);

UserController controller = new UserController(compositeLogger);
```

---

## Before and After

### Comparison Table

| Aspect | Before DIP | After DIP |
|--------|-----------|-----------|
| **Coupling** | Tight | Loose |
| **Flexibility** | Rigid | Flexible |
| **Testability** | Hard to test | Easy to test |
| **Reusability** | Low | High |
| **Maintainability** | Difficult | Easy |
| **Extensibility** | Hard to extend | Easy to extend |

### Metrics

**Before DIP:**
- Unit test coverage: 30% (hard to mock)
- Time to switch implementation: 2 days
- Number of dependencies: 10+ concrete classes
- Risk of change: High

**After DIP:**
- Unit test coverage: 85% (easy to mock)
- Time to switch implementation: 30 minutes
- Number of dependencies: 2-3 interfaces
- Risk of change: Low

---

## Real-World Examples

### Example 1: Spring Framework

```java
// DIP through dependency injection
@Service
public class UserService {
    private final UserRepository repository;
    
    @Autowired  // Spring injects implementation
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

### Example 2: ASP.NET Core

```csharp
// DIP through built-in DI container
public class OrderController : Controller {
    private readonly IOrderService _orderService;
    
    public OrderController(IOrderService orderService) {
        _orderService = orderService;
    }
}
```

### Example 3: Express.js Middleware

```javascript
// DIP through middleware pattern
app.use(logger);        // Abstraction
app.use(authenticate);  // Abstraction
app.use(authorize);     // Abstraction
```

### Famous Products

1. **Spring Framework**: Dependency injection container
2. **Angular**: Dependency injection system
3. **ASP.NET Core**: Built-in DI container
4. **Laravel**: Service container
5. **Django**: Dependency injection (newer versions)

---

## Benefits

### 1. Loose Coupling
```java
// Easy to change implementations
UserService service = new UserService(new MySQLRepository());
// Later...
UserService service = new UserService(new MongoRepository());
```

### 2. Easy Testing
```java
// Easy to mock dependencies
@Test
public void testUserService() {
    UserRepository mockRepo = mock(UserRepository.class);
    UserService service = new UserService(mockRepo);
    // Test without real database
}
```

### 3. Flexibility
```java
// Can configure at runtime
Database db = config.getDatabase().equals("mysql") 
    ? new MySQLDatabase() 
    : new PostgreSQLDatabase();
```

### 4. Parallel Development
```java
// Team A works on interface
interface PaymentGateway { }

// Team B implements high-level
class OrderService {
    OrderService(PaymentGateway gateway) { }
}

// Team C implements low-level
class StripeGateway implements PaymentGateway { }
```

---

## Common Violations

### 1. Direct Instantiation

```java
// BAD: Direct instantiation
class Service {
    private Database db = new MySQLDatabase();
}
```

### 2. Static Dependencies

```java
// BAD: Static dependency
class Service {
    void method() {
        Logger.getInstance().log("message");
    }
}
```

### 3. Service Locator Anti-Pattern

```java
// BAD: Service locator
class Service {
    void method() {
        Database db = ServiceLocator.get(Database.class);
    }
}
```

### 4. Concrete Parameter Types

```java
// BAD: Concrete type in signature
void processOrder(MySQLDatabase db) { }

// GOOD: Abstract type
void processOrder(Database db) { }
```

---

## How to Apply

### Step 1: Identify Dependencies

Look for:
- `new` keyword in business logic
- Direct references to concrete classes
- Static method calls

### Step 2: Extract Interfaces

```java
// Extract interface from concrete class
interface EmailSender {
    void send(String to, String subject, String body);
}
```

### Step 3: Inject Dependencies

```java
// Use constructor injection
class Service {
    private final EmailSender emailSender;
    
    Service(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}
```

### Step 4: Configure Dependencies

```java
// Configure at application startup
EmailSender emailSender = new SMTPEmailSender();
Service service = new Service(emailSender);
```

### Step 5: Use DI Framework (Optional)

```java
// Spring example
@Configuration
public class AppConfig {
    @Bean
    public EmailSender emailSender() {
        return new SMTPEmailSender();
    }
    
    @Bean
    public Service service(EmailSender emailSender) {
        return new Service(emailSender);
    }
}
```

---

## Related Patterns

### Patterns that Support DIP

1. **Factory Pattern**: Creates objects without specifying concrete class
2. **Abstract Factory**: Creates families of related objects
3. **Strategy Pattern**: Encapsulates algorithms behind interface
4. **Dependency Injection**: Injects dependencies from outside
5. **Service Locator**: Locates services (though considered anti-pattern)

### Example: Factory Pattern

```java
// DIP with Factory Pattern
interface Database {
    void connect();
}

interface DatabaseFactory {
    Database createDatabase();
}

class MySQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new MySQLDatabase();
    }
}

class Service {
    private Database db;
    
    Service(DatabaseFactory factory) {
        this.db = factory.createDatabase();
    }
}
```

---

## Conclusion

The Dependency Inversion Principle promotes:
- **Loose coupling** between modules
- **Flexible** architecture that adapts to change
- **Testable** code with easy mocking
- **Maintainable** systems with clear boundaries

**Remember:**
> "Depend on abstractions, not on concretions."

Design your systems so that high-level business logic doesn't depend on low-level implementation details. Both should depend on abstractions, creating a flexible, maintainable architecture.

---

## Quick Reference

### ✅ Do's
- Depend on interfaces/abstractions
- Use dependency injection
- Inject dependencies through constructor
- Program to interfaces, not implementations
- Use DI frameworks when appropriate

### ❌ Don'ts
- Use `new` keyword in business logic
- Depend on concrete classes
- Use static dependencies
- Use service locator pattern
- Hardcode dependencies

---

**Last Updated:** January 2026  
**Version:** 1.0
