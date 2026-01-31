# Single Responsibility Principle (SRP)

## Definition

> "A class should have one, and only one, reason to change."
> 
> — Robert C. Martin

**Alternative Definition:** "A class should have only one responsibility."

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

The Single Responsibility Principle states that a class should have **only one reason to change**. This means that a class should have only one job or responsibility.

### What is a "Responsibility"?

A responsibility is a **reason to change**. If you can think of more than one motive for changing a class, then that class has more than one responsibility.

### Core Idea

```
┌─────────────────────────────────────────────────────────┐
│              Single Responsibility Principle             │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  One Class = One Responsibility = One Reason to Change  │
│                                                          │
│  ┌──────────────┐                                       │
│  │    Class     │                                       │
│  │              │                                       │
│  │  ┌────────┐  │                                       │
│  │  │ Single │  │                                       │
│  │  │Responsi│  │                                       │
│  │  │ bility │  │                                       │
│  │  └────────┘  │                                       │
│  │              │                                       │
│  └──────────────┘                                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Motivation

### The Problem: God Classes

Without SRP, classes tend to accumulate responsibilities over time, becoming "God Classes" that do too much:

```
┌─────────────────────────────────────────────────────────┐
│                    God Class Problem                     │
└─────────────────────────────────────────────────────────┘

        ┌────────────────────────────────┐
        │      Employee Class            │
        ├────────────────────────────────┤
        │ - name, salary, department     │
        │ - calculatePay()               │  ← Accounting
        │ - saveToDatabase()             │  ← Persistence
        │ - generateReport()             │  ← Reporting
        │ - sendEmail()                  │  ← Notification
        │ - validateData()               │  ← Validation
        │ - exportToXML()                │  ← Serialization
        └────────────────────────────────┘
                    ▲
                    │
        Multiple reasons to change!
```

**Problems:**
1. **Hard to understand**: Too many responsibilities
2. **Hard to test**: Must test all responsibilities together
3. **High coupling**: Changes affect multiple areas
4. **Difficult to reuse**: Can't use one responsibility without others
5. **Merge conflicts**: Multiple developers changing same class

### The Solution: Focused Classes

```
┌─────────────────────────────────────────────────────────┐
│                  SRP Solution                            │
└─────────────────────────────────────────────────────────┘

┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Employee   │    │PayCalculator │    │EmployeeRepo │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ - name       │    │calculatePay()│    │ save()       │
│ - salary     │    │              │    │ find()       │
│ - department │    │              │    │ delete()     │
└──────────────┘    └──────────────┘    └──────────────┘
     Data               Accounting         Persistence

┌──────────────┐    ┌──────────────┐
│ReportGen     │    │EmailService  │
├──────────────┤    ├──────────────┤
│ generate()   │    │ send()       │
│              │    │              │
└──────────────┘    └──────────────┘
   Reporting         Notification
```

---

## Structure

### Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│           Single Responsibility Structure                │
└─────────────────────────────────────────────────────────┘

Before SRP (Violation):
┌────────────────────────────────┐
│         UserManager            │
├────────────────────────────────┤
│ + createUser()                 │
│ + validateUser()               │
│ + saveToDatabase()             │
│ + sendWelcomeEmail()           │
│ + generateUserReport()         │
│ + exportToJSON()               │
└────────────────────────────────┘

After SRP (Proper):
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  UserService   │  │UserValidator   │  │UserRepository  │
├────────────────┤  ├────────────────┤  ├────────────────┤
│+ createUser()  │  │+ validate()    │  │+ save()        │
│                │  │                │  │+ find()        │
└────────────────┘  └────────────────┘  └────────────────┘

┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│EmailService    │  │ReportGenerator │  │JSONExporter    │
├────────────────┤  ├────────────────┤  ├────────────────┤
│+ send()        │  │+ generate()    │  │+ export()      │
└────────────────┘  └────────────────┘  └────────────────┘
```

---

## Key Concepts

### 1. Cohesion

**High Cohesion** = All methods and properties work together toward a single purpose

```java
// HIGH COHESION - All methods relate to order calculation
class OrderCalculator {
    double calculateSubtotal(List<Item> items) { }
    double calculateTax(double subtotal) { }
    double calculateTotal(double subtotal, double tax) { }
}

// LOW COHESION - Methods serve different purposes
class OrderManager {
    double calculateTotal() { }      // Calculation
    void saveToDatabase() { }        // Persistence
    void sendEmail() { }             // Notification
    String generateInvoice() { }     // Reporting
}
```

### 2. Coupling

**Low Coupling** = Classes depend on few other classes

```java
// TIGHT COUPLING - Depends on concrete implementations
class OrderService {
    private MySQLDatabase db = new MySQLDatabase();
    private SMTPEmailer email = new SMTPEmailer();
    private PDFGenerator pdf = new PDFGenerator();
}

// LOOSE COUPLING - Depends on abstractions
class OrderService {
    private Database db;
    private EmailService email;
    private ReportGenerator report;
    
    OrderService(Database db, EmailService email, ReportGenerator report) {
        this.db = db;
        this.email = email;
        this.report = report;
    }
}
```

### 3. Separation of Concerns

Different concerns should be in different classes:

- **Business Logic**: Core domain rules
- **Data Access**: Database operations
- **Presentation**: UI/formatting
- **Infrastructure**: External services
- **Validation**: Input checking

---

## Examples

### Example 1: Employee Management

#### ❌ Violation

```java
// BAD: Multiple responsibilities
public class Employee {
    private String name;
    private double salary;
    private String department;
    
    // Responsibility 1: Business Logic
    public double calculatePay() {
        // Complex pay calculation logic
        return salary * 1.1;
    }
    
    // Responsibility 2: Persistence
    public void saveToDatabase() {
        // Database connection and save logic
        Connection conn = DriverManager.getConnection("jdbc:mysql://...");
        PreparedStatement stmt = conn.prepareStatement("INSERT INTO employees...");
        stmt.setString(1, name);
        stmt.setDouble(2, salary);
        stmt.executeUpdate();
    }
    
    // Responsibility 3: Reporting
    public String generateReport() {
        // Report generation logic
        return "Employee Report:\n" +
               "Name: " + name + "\n" +
               "Salary: " + salary + "\n" +
               "Department: " + department;
    }
    
    // Responsibility 4: Validation
    public boolean isValid() {
        return name != null && !name.isEmpty() && salary > 0;
    }
}
```

**Problems:**
- Changes to database schema affect Employee class
- Changes to report format affect Employee class
- Changes to pay calculation affect Employee class
- Hard to test each responsibility independently
- Violates Open/Closed Principle

#### ✅ Proper Implementation

```java
// GOOD: Single responsibility - Data holder
public class Employee {
    private String name;
    private double salary;
    private String department;
    
    public Employee(String name, double salary, String department) {
        this.name = name;
        this.salary = salary;
        this.department = department;
    }
    
    // Getters and setters only
    public String getName() { return name; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }
}

// Single responsibility - Pay calculation
public class PayCalculator {
    public double calculatePay(Employee employee) {
        // Complex pay calculation logic
        return employee.getSalary() * 1.1;
    }
}

// Single responsibility - Persistence
public class EmployeeRepository {
    private Connection connection;
    
    public EmployeeRepository(Connection connection) {
        this.connection = connection;
    }
    
    public void save(Employee employee) {
        try {
            PreparedStatement stmt = connection.prepareStatement(
                "INSERT INTO employees (name, salary, department) VALUES (?, ?, ?)"
            );
            stmt.setString(1, employee.getName());
            stmt.setDouble(2, employee.getSalary());
            stmt.setString(3, employee.getDepartment());
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Failed to save employee", e);
        }
    }
    
    public Employee findById(int id) {
        // Find logic
        return null;
    }
}

// Single responsibility - Reporting
public class EmployeeReportGenerator {
    public String generate(Employee employee) {
        return "Employee Report:\n" +
               "Name: " + employee.getName() + "\n" +
               "Salary: " + employee.getSalary() + "\n" +
               "Department: " + employee.getDepartment();
    }
}

// Single responsibility - Validation
public class EmployeeValidator {
    public boolean isValid(Employee employee) {
        return employee.getName() != null && 
               !employee.getName().isEmpty() && 
               employee.getSalary() > 0;
    }
}
```

### Example 2: User Authentication (Python)

#### ❌ Violation

```python
# BAD: Multiple responsibilities
class UserAuthenticator:
    def __init__(self):
        self.db_connection = self._connect_to_database()
    
    # Responsibility 1: Database connection
    def _connect_to_database(self):
        return pymysql.connect(host='localhost', user='root', password='pass')
    
    # Responsibility 2: Authentication logic
    def authenticate(self, username, password):
        hashed_password = self._hash_password(password)
        user = self._find_user(username)
        if user and user['password'] == hashed_password:
            self._log_login(username)
            self._send_notification(username)
            return True
        return False
    
    # Responsibility 3: Password hashing
    def _hash_password(self, password):
        return hashlib.sha256(password.encode()).hexdigest()
    
    # Responsibility 4: Database query
    def _find_user(self, username):
        cursor = self.db_connection.cursor()
        cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
        return cursor.fetchone()
    
    # Responsibility 5: Logging
    def _log_login(self, username):
        with open('login.log', 'a') as f:
            f.write(f"{datetime.now()}: {username} logged in\n")
    
    # Responsibility 6: Notification
    def _send_notification(self, username):
        # Send email notification
        pass
```

#### ✅ Proper Implementation

```python
# GOOD: Single responsibility - User data
class User:
    def __init__(self, username, password_hash):
        self.username = username
        self.password_hash = password_hash

# Single responsibility - Password hashing
class PasswordHasher:
    def hash(self, password):
        return hashlib.sha256(password.encode()).hexdigest()
    
    def verify(self, password, password_hash):
        return self.hash(password) == password_hash

# Single responsibility - User repository
class UserRepository:
    def __init__(self, db_connection):
        self.db = db_connection
    
    def find_by_username(self, username):
        cursor = self.db.cursor()
        cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
        row = cursor.fetchone()
        if row:
            return User(row['username'], row['password_hash'])
        return None

# Single responsibility - Authentication logic
class AuthenticationService:
    def __init__(self, user_repo, password_hasher):
        self.user_repo = user_repo
        self.password_hasher = password_hasher
    
    def authenticate(self, username, password):
        user = self.user_repo.find_by_username(username)
        if user and self.password_hasher.verify(password, user.password_hash):
            return True
        return False

# Single responsibility - Login logging
class LoginLogger:
    def __init__(self, log_file='login.log'):
        self.log_file = log_file
    
    def log(self, username):
        with open(self.log_file, 'a') as f:
            f.write(f"{datetime.now()}: {username} logged in\n")

# Single responsibility - Notification
class NotificationService:
    def send_login_notification(self, username):
        # Send email notification
        pass

# Usage
db = connect_to_database()
user_repo = UserRepository(db)
password_hasher = PasswordHasher()
auth_service = AuthenticationService(user_repo, password_hasher)
logger = LoginLogger()
notifier = NotificationService()

if auth_service.authenticate(username, password):
    logger.log(username)
    notifier.send_login_notification(username)
```

### Example 3: Order Processing (JavaScript)

#### ❌ Violation

```javascript
// BAD: Multiple responsibilities
class OrderProcessor {
    constructor() {
        this.db = new Database();
        this.emailService = new EmailService();
    }
    
    // Responsibility 1: Order validation
    validateOrder(order) {
        if (!order.items || order.items.length === 0) {
            throw new Error('Order must have items');
        }
        if (!order.customer || !order.customer.email) {
            throw new Error('Customer email required');
        }
        return true;
    }
    
    // Responsibility 2: Price calculation
    calculateTotal(order) {
        let subtotal = 0;
        for (const item of order.items) {
            subtotal += item.price * item.quantity;
        }
        const tax = subtotal * 0.1;
        const shipping = this.calculateShipping(order);
        return subtotal + tax + shipping;
    }
    
    // Responsibility 3: Shipping calculation
    calculateShipping(order) {
        if (order.total > 100) return 0;
        return 10;
    }
    
    // Responsibility 4: Payment processing
    processPayment(order, paymentInfo) {
        // Payment gateway integration
        const gateway = new PaymentGateway();
        return gateway.charge(paymentInfo, order.total);
    }
    
    // Responsibility 5: Database operations
    saveOrder(order) {
        this.db.insert('orders', order);
    }
    
    // Responsibility 6: Email notification
    sendConfirmation(order) {
        const email = {
            to: order.customer.email,
            subject: 'Order Confirmation',
            body: `Your order #${order.id} has been confirmed`
        };
        this.emailService.send(email);
    }
    
    // Responsibility 7: Orchestration
    processOrder(order, paymentInfo) {
        this.validateOrder(order);
        order.total = this.calculateTotal(order);
        this.processPayment(order, paymentInfo);
        this.saveOrder(order);
        this.sendConfirmation(order);
    }
}
```

#### ✅ Proper Implementation

```javascript
// GOOD: Single responsibility - Order data
class Order {
    constructor(id, customer, items) {
        this.id = id;
        this.customer = customer;
        this.items = items;
        this.total = 0;
    }
}

// Single responsibility - Order validation
class OrderValidator {
    validate(order) {
        if (!order.items || order.items.length === 0) {
            throw new Error('Order must have items');
        }
        if (!order.customer || !order.customer.email) {
            throw new Error('Customer email required');
        }
        return true;
    }
}

// Single responsibility - Price calculation
class PriceCalculator {
    calculateSubtotal(items) {
        return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    }
    
    calculateTax(subtotal) {
        return subtotal * 0.1;
    }
    
    calculateTotal(subtotal, tax, shipping) {
        return subtotal + tax + shipping;
    }
}

// Single responsibility - Shipping calculation
class ShippingCalculator {
    calculate(order) {
        const subtotal = new PriceCalculator().calculateSubtotal(order.items);
        return subtotal > 100 ? 0 : 10;
    }
}

// Single responsibility - Payment processing
class PaymentProcessor {
    constructor(paymentGateway) {
        this.gateway = paymentGateway;
    }
    
    process(paymentInfo, amount) {
        return this.gateway.charge(paymentInfo, amount);
    }
}

// Single responsibility - Order persistence
class OrderRepository {
    constructor(database) {
        this.db = database;
    }
    
    save(order) {
        return this.db.insert('orders', order);
    }
    
    findById(id) {
        return this.db.findOne('orders', { id });
    }
}

// Single responsibility - Email notification
class OrderNotificationService {
    constructor(emailService) {
        this.emailService = emailService;
    }
    
    sendConfirmation(order) {
        const email = {
            to: order.customer.email,
            subject: 'Order Confirmation',
            body: `Your order #${order.id} has been confirmed`
        };
        return this.emailService.send(email);
    }
}

// Single responsibility - Orchestration
class OrderService {
    constructor(validator, priceCalc, shippingCalc, paymentProc, orderRepo, notifier) {
        this.validator = validator;
        this.priceCalculator = priceCalc;
        this.shippingCalculator = shippingCalc;
        this.paymentProcessor = paymentProc;
        this.orderRepository = orderRepo;
        this.notificationService = notifier;
    }
    
    processOrder(order, paymentInfo) {
        // Validate
        this.validator.validate(order);
        
        // Calculate prices
        const subtotal = this.priceCalculator.calculateSubtotal(order.items);
        const tax = this.priceCalculator.calculateTax(subtotal);
        const shipping = this.shippingCalculator.calculate(order);
        order.total = this.priceCalculator.calculateTotal(subtotal, tax, shipping);
        
        // Process payment
        this.paymentProcessor.process(paymentInfo, order.total);
        
        // Save and notify
        this.orderRepository.save(order);
        this.notificationService.sendConfirmation(order);
        
        return order;
    }
}

// Usage
const orderService = new OrderService(
    new OrderValidator(),
    new PriceCalculator(),
    new ShippingCalculator(),
    new PaymentProcessor(new PaymentGateway()),
    new OrderRepository(new Database()),
    new OrderNotificationService(new EmailService())
);

orderService.processOrder(order, paymentInfo);
```

---

## Before and After

### Comparison Table

| Aspect | Before SRP | After SRP |
|--------|-----------|-----------|
| **Class Size** | 500+ lines | 50-100 lines per class |
| **Methods per Class** | 15-20 methods | 3-5 methods |
| **Testability** | Hard (must mock everything) | Easy (focused tests) |
| **Reusability** | Low (tightly coupled) | High (independent) |
| **Maintainability** | Difficult | Easy |
| **Team Collaboration** | Conflicts common | Minimal conflicts |
| **Understanding** | Takes hours | Takes minutes |

---

## Real-World Examples

### Example 1: Spring Framework

**SRP in Action:**
```java
// Separate responsibilities
@Controller  // Handles HTTP requests
public class UserController { }

@Service     // Business logic
public class UserService { }

@Repository  // Data access
public class UserRepository { }

@Component   // Validation
public class UserValidator { }
```

### Example 2: Express.js Middleware

```javascript
// Each middleware has single responsibility
app.use(bodyParser.json());           // Parse JSON
app.use(cors());                       // Handle CORS
app.use(authenticate);                 // Authentication
app.use(authorize);                    // Authorization
app.use(logger);                       // Logging
app.use(errorHandler);                 // Error handling
```

### Example 3: React Components

```javascript
// Separate concerns
function UserProfile() {              // Presentation
    const user = useUser();           // Data fetching (custom hook)
    const validator = useValidator(); // Validation (custom hook)
    
    return <UserView user={user} />;  // Pure presentation
}
```

### Famous Products

1. **Linux Kernel**: Each module has single responsibility
2. **PostgreSQL**: Separate modules for parsing, planning, execution
3. **Kubernetes**: Controllers each manage one resource type
4. **React**: Components, hooks, and utilities are separated

---

## Benefits

### 1. Easier to Understand
- Smaller classes are easier to comprehend
- Clear purpose and responsibility
- Less cognitive load

### 2. Easier to Test
```java
// Easy to test single responsibility
@Test
public void testPayCalculation() {
    PayCalculator calc = new PayCalculator();
    Employee emp = new Employee("John", 50000, "IT");
    assertEquals(55000, calc.calculatePay(emp));
}
```

### 3. Easier to Maintain
- Changes are localized
- Less risk of breaking other functionality
- Easier to find and fix bugs

### 4. Better Reusability
```java
// Can reuse in different contexts
PayCalculator calc = new PayCalculator();
calc.calculatePay(employee);
calc.calculatePay(contractor);
calc.calculatePay(consultant);
```

### 5. Reduced Coupling
- Classes depend on fewer things
- Changes propagate less
- More flexible architecture

### 6. Improved Collaboration
- Multiple developers can work on different classes
- Fewer merge conflicts
- Clearer ownership

---

## Common Violations

### 1. God Classes

```java
// BAD: Does everything
class Application {
    void handleRequest() { }
    void connectDatabase() { }
    void sendEmail() { }
    void generateReport() { }
    void processPayment() { }
    // ... 50 more methods
}
```

### 2. Mixed Concerns

```java
// BAD: UI and business logic mixed
class LoginForm extends JFrame {
    void onLoginButtonClick() {
        String username = usernameField.getText();
        String password = passwordField.getText();
        
        // Business logic in UI!
        Connection conn = DriverManager.getConnection(...);
        PreparedStatement stmt = conn.prepareStatement(...);
        ResultSet rs = stmt.executeQuery();
        
        if (rs.next()) {
            // More business logic
        }
    }
}
```

### 3. Utility Classes

```java
// BAD: Dumping ground for unrelated methods
class Utils {
    static String formatDate(Date date) { }
    static void sendEmail(String to, String subject) { }
    static double calculateTax(double amount) { }
    static void logError(Exception e) { }
}
```

### 4. Manager/Controller Classes

```java
// BAD: Manages too much
class UserManager {
    void createUser() { }
    void updateUser() { }
    void deleteUser() { }
    void validateUser() { }
    void saveUser() { }
    void sendWelcomeEmail() { }
    void generateUserReport() { }
}
```

---

## How to Apply

### Step 1: Identify Responsibilities

Ask these questions:
1. What does this class do?
2. Why would this class change?
3. Who would request changes to this class?

### Step 2: Look for Code Smells

- Large classes (>200 lines)
- Many methods (>10)
- Multiple import statements
- Mixed levels of abstraction
- Comments like "// Section 1", "// Section 2"

### Step 3: Extract Classes

```java
// Before
class Employee {
    void calculatePay() { }
    void save() { }
    void generateReport() { }
}

// After - Extract responsibilities
class Employee { }                    // Data
class PayCalculator { }               // Calculation
class EmployeeRepository { }          // Persistence
class EmployeeReportGenerator { }     // Reporting
```

### Step 4: Test Each Class

Write focused unit tests for each responsibility:

```java
@Test
public void testPayCalculation() {
    // Test only pay calculation
}

@Test
public void testEmployeeSave() {
    // Test only persistence
}
```

### Step 5: Refactor Incrementally

Don't refactor everything at once:
1. Start with most problematic class
2. Extract one responsibility at a time
3. Ensure tests pass after each extraction
4. Commit frequently

---

## Related Patterns

### Patterns that Support SRP

1. **Facade Pattern**: Provides simple interface to complex subsystem
2. **Proxy Pattern**: Separates access control from business logic
3. **Decorator Pattern**: Adds responsibilities without modifying class
4. **Strategy Pattern**: Separates algorithm from context
5. **Command Pattern**: Encapsulates request as object

### Example: Strategy Pattern + SRP

```java
// Each strategy has single responsibility
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) {
        // Only credit card payment logic
    }
}

class PayPalPayment implements PaymentStrategy {
    public void pay(double amount) {
        // Only PayPal payment logic
    }
}
```

---

## Conclusion

The Single Responsibility Principle is the foundation of good object-oriented design. By ensuring each class has only one reason to change, you create:

- **Modular** code that's easy to understand
- **Testable** code with focused unit tests
- **Maintainable** code that's easy to change
- **Reusable** code that works in different contexts

**Remember:**
> "Do one thing and do it well."
> 
> — Unix Philosophy

Start applying SRP today by identifying classes with multiple responsibilities and extracting them into focused, cohesive classes.

---

## Quick Reference

### ✅ Do's
- Keep classes small and focused
- Extract responsibilities into separate classes
- Use composition over inheritance
- Write focused unit tests
- Follow the "one reason to change" rule

### ❌ Don'ts
- Create God classes
- Mix different levels of abstraction
- Put business logic in UI classes
- Create utility classes as dumping grounds
- Ignore code smells

---

**Last Updated:** January 2026  
**Version:** 1.0
