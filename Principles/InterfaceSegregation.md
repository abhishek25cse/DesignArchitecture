# Interface Segregation Principle (ISP)

## Definition

> "Clients should not be forced to depend on interfaces they do not use."
> 
> — Robert C. Martin

**Alternative Definition:** "Many client-specific interfaces are better than one general-purpose interface."

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

The Interface Segregation Principle states that no client should be forced to depend on methods it does not use. Instead of one fat interface, many small, focused interfaces are preferred.

### Core Idea

```
┌─────────────────────────────────────────────────────────┐
│         Interface Segregation Principle                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Fat Interface (BAD):                                   │
│  ┌────────────────────────────────┐                    │
│  │      <<interface>>             │                    │
│  │         Worker                 │                    │
│  ├────────────────────────────────┤                    │
│  │ + work()                       │                    │
│  │ + eat()                        │                    │
│  │ + sleep()                      │                    │
│  │ + attendMeeting()              │                    │
│  │ + takeBreak()                  │                    │
│  └────────────────────────────────┘                    │
│           ▲                                             │
│           │ implements all methods                      │
│           │ (even unused ones!)                         │
│                                                          │
│  Segregated Interfaces (GOOD):                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │Workable  │  │ Eatable  │  │Sleepable │            │
│  ├──────────┤  ├──────────┤  ├──────────┤            │
│  │+ work()  │  │+ eat()   │  │+ sleep() │            │
│  └──────────┘  └──────────┘  └──────────┘            │
│                                                          │
│  Clients implement only what they need!                 │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Motivation

### The Problem: Fat Interfaces

Without ISP, interfaces become bloated with methods that not all implementers need:

```
┌─────────────────────────────────────────────────────────┐
│              Fat Interface Problem                       │
└─────────────────────────────────────────────────────────┘

┌────────────────────────────────┐
│      <<interface>>             │
│      MultiFunctionDevice       │
├────────────────────────────────┤
│ + print(document)              │
│ + scan(document)               │
│ + fax(document)                │
│ + staple(document)             │
│ + email(document)              │
└────────────────────────────────┘
         ▲              ▲
         │              │
         │              │
┌────────┴─────┐  ┌────┴──────────┐
│AllInOnePrinter│  │SimplePrinter  │
├──────────────┤  ├───────────────┤
│+ print() ✓   │  │+ print() ✓    │
│+ scan() ✓    │  │+ scan() ✗     │ ← Must implement
│+ fax() ✓     │  │+ fax() ✗      │   but doesn't
│+ staple() ✓  │  │+ staple() ✗   │   support!
│+ email() ✓   │  │+ email() ✗    │
└──────────────┘  └───────────────┘

SimplePrinter forced to implement methods it doesn't support!
```

**Example Violation:**
```java
// BAD: Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

class HumanWorker implements Worker {
    public void work() { /* work */ }
    public void eat() { /* eat */ }
    public void sleep() { /* sleep */ }
}

class RobotWorker implements Worker {
    public void work() { /* work */ }
    public void eat() { /* not applicable! */ }
    public void sleep() { /* not applicable! */ }
}
```

### The Solution: Segregated Interfaces

```
┌─────────────────────────────────────────────────────────┐
│              ISP Solution Pattern                        │
└─────────────────────────────────────────────────────────┘

┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│Printable │  │Scannable │  │ Faxable  │  │Emailable │
├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤
│+ print() │  │+ scan()  │  │+ fax()   │  │+ email() │
└─────┬────┘  └─────┬────┘  └─────┬────┘  └─────┬────┘
      │             │             │             │
      └─────────────┼─────────────┼─────────────┘
                    │             │
          ┌─────────┴──────┐  ┌──┴──────────┐
          │AllInOnePrinter │  │SimplePrinter│
          ├────────────────┤  ├─────────────┤
          │Implements:     │  │Implements:  │
          │- Printable     │  │- Printable  │
          │- Scannable     │  │             │
          │- Faxable       │  │             │
          │- Emailable     │  │             │
          └────────────────┘  └─────────────┘

Each class implements only what it needs!
```

---

## Structure

### Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│         Interface Segregation Structure                  │
└─────────────────────────────────────────────────────────┘

Violation (Fat Interface):
┌────────────────────────────────┐
│      <<interface>>             │
│         Vehicle                │
├────────────────────────────────┤
│ + drive()                      │
│ + fly()                        │
│ + sail()                       │
└────────────────────────────────┘
         ▲              ▲
         │              │
┌────────┴─────┐  ┌────┴──────┐
│     Car      │  │  Airplane  │
├──────────────┤  ├────────────┤
│+ drive() ✓   │  │+ drive() ✗ │
│+ fly() ✗     │  │+ fly() ✓   │
│+ sail() ✗    │  │+ sail() ✗  │
└──────────────┘  └────────────┘

Proper Design (Segregated Interfaces):
┌──────────┐  ┌──────────┐  ┌──────────┐
│Drivable  │  │ Flyable  │  │Sailable  │
├──────────┤  ├──────────┤  ├──────────┤
│+ drive() │  │+ fly()   │  │+ sail()  │
└─────┬────┘  └─────┬────┘  └─────┬────┘
      │             │             │
      │             │             │
┌─────▼────┐  ┌─────▼────┐  ┌────▼─────┐
│   Car    │  │ Airplane │  │   Boat   │
├──────────┤  ├──────────┤  ├──────────┤
│+ drive() │  │+ fly()   │  │+ sail()  │
└──────────┘  └──────────┘  └──────────┘

┌──────────────────┐
│  Amphibious Car  │
├──────────────────┤
│Implements:       │
│- Drivable        │
│- Sailable        │
└──────────────────┘
```

---

## Key Concepts

### 1. Client-Specific Interfaces

Design interfaces based on client needs, not implementer capabilities:

```java
// BAD: Interface based on implementer
interface Database {
    void connect();
    void disconnect();
    void executeQuery();
    void executeUpdate();
    void beginTransaction();
    void commit();
    void rollback();
    void backup();
    void restore();
}

// GOOD: Client-specific interfaces
interface QueryExecutor {
    void executeQuery(String sql);
}

interface TransactionManager {
    void beginTransaction();
    void commit();
    void rollback();
}

interface DatabaseBackup {
    void backup();
    void restore();
}
```

### 2. Role Interfaces

Create interfaces that represent specific roles or capabilities:

```java
// Role-based interfaces
interface Readable {
    String read();
}

interface Writable {
    void write(String data);
}

interface Closeable {
    void close();
}

// Combine as needed
class File implements Readable, Writable, Closeable {
    public String read() { /* read */ }
    public void write(String data) { /* write */ }
    public void close() { /* close */ }
}

class ReadOnlyFile implements Readable, Closeable {
    public String read() { /* read */ }
    public void close() { /* close */ }
    // No write() method!
}
```

### 3. Interface Pollution

Avoid adding methods to interfaces that only some implementers need:

```java
// BAD: Interface pollution
interface Shape {
    double getArea();
    double getVolume();  // Not all shapes have volume!
}

// GOOD: Separate concerns
interface Shape2D {
    double getArea();
}

interface Shape3D extends Shape2D {
    double getVolume();
}
```

---

## Examples

### Example 1: Worker Hierarchy

#### ❌ Violation

```java
// BAD: Fat interface forces unnecessary implementations
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void takeVacation();
}

public class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    @Override
    public void attendMeeting() {
        System.out.println("Human attending meeting");
    }
    
    @Override
    public void takeVacation() {
        System.out.println("Human on vacation");
    }
}

public class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
    
    @Override
    public void eat() {
        // Robots don't eat!
        throw new UnsupportedOperationException("Robots don't eat");
    }
    
    @Override
    public void sleep() {
        // Robots don't sleep!
        throw new UnsupportedOperationException("Robots don't sleep");
    }
    
    @Override
    public void attendMeeting() {
        // Robots don't attend meetings!
        throw new UnsupportedOperationException("Robots don't attend meetings");
    }
    
    @Override
    public void takeVacation() {
        // Robots don't take vacations!
        throw new UnsupportedOperationException("Robots don't take vacations");
    }
}
```

**Problems:**
- RobotWorker forced to implement methods it doesn't support
- Throws exceptions at runtime (violates LSP too!)
- Interface too broad for all implementers
- Hard to extend with new worker types

#### ✅ Proper Implementation

```java
// GOOD: Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public interface Attendable {
    void attendMeeting();
}

public interface Vacationable {
    void takeVacation();
}

// Human implements all relevant interfaces
public class HumanWorker implements Workable, Eatable, Sleepable, Attendable, Vacationable {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    @Override
    public void attendMeeting() {
        System.out.println("Human attending meeting");
    }
    
    @Override
    public void takeVacation() {
        System.out.println("Human on vacation");
    }
}

// Robot implements only what it needs
public class RobotWorker implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
}

// Manager for different types of workers
public class WorkManager {
    public void manageWork(Workable worker) {
        worker.work();
    }
    
    public void manageBreak(Eatable worker) {
        worker.eat();
    }
    
    public void manageRest(Sleepable worker) {
        worker.sleep();
    }
}

// Usage
WorkManager manager = new WorkManager();
HumanWorker human = new HumanWorker();
RobotWorker robot = new RobotWorker();

manager.manageWork(human);  // OK
manager.manageWork(robot);  // OK
manager.manageBreak(human); // OK
// manager.manageBreak(robot);  // Compile error - type safe!
```

### Example 2: Multi-Function Device (Python)

#### ❌ Violation

```python
# BAD: Fat interface
from abc import ABC, abstractmethod

class MultiFunctionDevice(ABC):
    @abstractmethod
    def print(self, document):
        pass
    
    @abstractmethod
    def scan(self, document):
        pass
    
    @abstractmethod
    def fax(self, document):
        pass
    
    @abstractmethod
    def photocopy(self, document):
        pass

class AllInOnePrinter(MultiFunctionDevice):
    def print(self, document):
        print(f"Printing: {document}")
    
    def scan(self, document):
        print(f"Scanning: {document}")
    
    def fax(self, document):
        print(f"Faxing: {document}")
    
    def photocopy(self, document):
        print(f"Photocopying: {document}")

class SimplePrinter(MultiFunctionDevice):
    def print(self, document):
        print(f"Printing: {document}")
    
    def scan(self, document):
        # Doesn't support scanning!
        raise NotImplementedError("This printer doesn't support scanning")
    
    def fax(self, document):
        # Doesn't support faxing!
        raise NotImplementedError("This printer doesn't support faxing")
    
    def photocopy(self, document):
        # Doesn't support photocopying!
        raise NotImplementedError("This printer doesn't support photocopying")
```

#### ✅ Proper Implementation

```python
# GOOD: Segregated interfaces
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

class Scanner(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class Fax(ABC):
    @abstractmethod
    def fax(self, document):
        pass

class Photocopier(ABC):
    @abstractmethod
    def photocopy(self, document):
        pass

# Simple printer implements only what it needs
class SimplePrinter(Printer):
    def print(self, document):
        print(f"Printing: {document}")

# All-in-one implements multiple interfaces
class AllInOnePrinter(Printer, Scanner, Fax, Photocopier):
    def print(self, document):
        print(f"Printing: {document}")
    
    def scan(self, document):
        print(f"Scanning: {document}")
        return f"Scanned: {document}"
    
    def fax(self, document):
        print(f"Faxing: {document}")
    
    def photocopy(self, document):
        print(f"Photocopying: {document}")

# Scanner with printer
class ScannerPrinter(Printer, Scanner):
    def print(self, document):
        print(f"Printing: {document}")
    
    def scan(self, document):
        print(f"Scanning: {document}")
        return f"Scanned: {document}"

# Client code uses specific interfaces
class DocumentProcessor:
    def __init__(self, printer: Printer):
        self.printer = printer
    
    def process_document(self, document):
        self.printer.print(document)

class ScanService:
    def __init__(self, scanner: Scanner):
        self.scanner = scanner
    
    def scan_document(self, document):
        return self.scanner.scan(document)

# Usage
simple = SimplePrinter()
all_in_one = AllInOnePrinter()

doc_processor = DocumentProcessor(simple)  # OK
doc_processor.process_document("Report")

scan_service = ScanService(all_in_one)  # OK
scan_service.scan_document("Invoice")

# scan_service = ScanService(simple)  # Type error - safe!
```

### Example 3: Payment Gateway (JavaScript)

#### ❌ Violation

```javascript
// BAD: Fat interface
class PaymentGateway {
    processPayment(amount) {
        throw new Error('Must implement processPayment');
    }
    
    refundPayment(transactionId) {
        throw new Error('Must implement refundPayment');
    }
    
    subscribeRecurring(amount, interval) {
        throw new Error('Must implement subscribeRecurring');
    }
    
    cancelSubscription(subscriptionId) {
        throw new Error('Must implement cancelSubscription');
    }
    
    verifyPayment(transactionId) {
        throw new Error('Must implement verifyPayment');
    }
    
    generateInvoice(transactionId) {
        throw new Error('Must implement generateInvoice');
    }
}

class SimplePaymentGateway extends PaymentGateway {
    processPayment(amount) {
        console.log(`Processing payment: $${amount}`);
        return { success: true, transactionId: '123' };
    }
    
    refundPayment(transactionId) {
        // Doesn't support refunds!
        throw new Error('Refunds not supported');
    }
    
    subscribeRecurring(amount, interval) {
        // Doesn't support subscriptions!
        throw new Error('Subscriptions not supported');
    }
    
    cancelSubscription(subscriptionId) {
        // Doesn't support subscriptions!
        throw new Error('Subscriptions not supported');
    }
    
    verifyPayment(transactionId) {
        // Doesn't support verification!
        throw new Error('Verification not supported');
    }
    
    generateInvoice(transactionId) {
        // Doesn't support invoices!
        throw new Error('Invoices not supported');
    }
}
```

#### ✅ Proper Implementation

```javascript
// GOOD: Segregated interfaces
class PaymentProcessor {
    processPayment(amount) {
        throw new Error('Must implement processPayment');
    }
}

class RefundablePayment {
    refundPayment(transactionId) {
        throw new Error('Must implement refundPayment');
    }
}

class RecurringPayment {
    subscribeRecurring(amount, interval) {
        throw new Error('Must implement subscribeRecurring');
    }
    
    cancelSubscription(subscriptionId) {
        throw new Error('Must implement cancelSubscription');
    }
}

class PaymentVerification {
    verifyPayment(transactionId) {
        throw new Error('Must implement verifyPayment');
    }
}

class InvoiceGenerator {
    generateInvoice(transactionId) {
        throw new Error('Must implement generateInvoice');
    }
}

// Simple gateway implements only basic payment
class SimplePaymentGateway extends PaymentProcessor {
    processPayment(amount) {
        console.log(`Processing payment: $${amount}`);
        return { success: true, transactionId: '123' };
    }
}

// Full-featured gateway implements multiple interfaces
class FullPaymentGateway extends PaymentProcessor {
    constructor() {
        super();
        this.refundService = new RefundService();
        this.subscriptionService = new SubscriptionService();
        this.verificationService = new VerificationService();
        this.invoiceService = new InvoiceService();
    }
    
    processPayment(amount) {
        console.log(`Processing payment: $${amount}`);
        return { success: true, transactionId: '123' };
    }
}

// Composition-based approach
class RefundService extends RefundablePayment {
    refundPayment(transactionId) {
        console.log(`Refunding transaction: ${transactionId}`);
        return { success: true };
    }
}

class SubscriptionService extends RecurringPayment {
    subscribeRecurring(amount, interval) {
        console.log(`Creating subscription: $${amount} every ${interval}`);
        return { subscriptionId: 'sub_123' };
    }
    
    cancelSubscription(subscriptionId) {
        console.log(`Canceling subscription: ${subscriptionId}`);
        return { success: true };
    }
}

class VerificationService extends PaymentVerification {
    verifyPayment(transactionId) {
        console.log(`Verifying transaction: ${transactionId}`);
        return { verified: true };
    }
}

class InvoiceService extends InvoiceGenerator {
    generateInvoice(transactionId) {
        console.log(`Generating invoice for: ${transactionId}`);
        return { invoiceId: 'inv_123', pdf: 'invoice.pdf' };
    }
}

// Client code uses specific capabilities
class PaymentService {
    constructor(processor) {
        this.processor = processor;
    }
    
    makePayment(amount) {
        return this.processor.processPayment(amount);
    }
}

class RefundService {
    constructor(refundable) {
        this.refundable = refundable;
    }
    
    processRefund(transactionId) {
        return this.refundable.refundPayment(transactionId);
    }
}

// Usage
const simpleGateway = new SimplePaymentGateway();
const paymentService = new PaymentService(simpleGateway);
paymentService.makePayment(100);  // OK

// const refundService = new RefundService(simpleGateway);  // Type error!
```

### Example 4: Database Operations

#### ❌ Violation

```java
// BAD: Fat interface
public interface DatabaseOperations {
    void connect();
    void disconnect();
    ResultSet executeQuery(String sql);
    int executeUpdate(String sql);
    void beginTransaction();
    void commit();
    void rollback();
    void createBackup();
    void restoreBackup(String backupFile);
    void optimizeDatabase();
    void repairDatabase();
    List<String> listTables();
    void createIndex(String table, String column);
}

public class ReadOnlyDatabase implements DatabaseOperations {
    @Override
    public void connect() { /* connect */ }
    
    @Override
    public void disconnect() { /* disconnect */ }
    
    @Override
    public ResultSet executeQuery(String sql) {
        // Only this is needed!
        return null;
    }
    
    @Override
    public int executeUpdate(String sql) {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void beginTransaction() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void commit() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void rollback() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void createBackup() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void restoreBackup(String backupFile) {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void optimizeDatabase() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public void repairDatabase() {
        throw new UnsupportedOperationException("Read-only database");
    }
    
    @Override
    public List<String> listTables() {
        return new ArrayList<>();
    }
    
    @Override
    public void createIndex(String table, String column) {
        throw new UnsupportedOperationException("Read-only database");
    }
}
```

#### ✅ Proper Implementation

```java
// GOOD: Segregated interfaces
public interface DatabaseConnection {
    void connect();
    void disconnect();
}

public interface QueryExecutor {
    ResultSet executeQuery(String sql);
}

public interface DataManipulator {
    int executeUpdate(String sql);
    int executeInsert(String sql);
    int executeDelete(String sql);
}

public interface TransactionManager {
    void beginTransaction();
    void commit();
    void rollback();
}

public interface DatabaseBackup {
    void createBackup();
    void restoreBackup(String backupFile);
}

public interface DatabaseMaintenance {
    void optimizeDatabase();
    void repairDatabase();
    void createIndex(String table, String column);
}

public interface SchemaInspector {
    List<String> listTables();
    List<String> listColumns(String table);
}

// Read-only database implements only what it needs
public class ReadOnlyDatabase implements DatabaseConnection, QueryExecutor, SchemaInspector {
    @Override
    public void connect() {
        System.out.println("Connecting to read-only database");
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from database");
    }
    
    @Override
    public ResultSet executeQuery(String sql) {
        System.out.println("Executing query: " + sql);
        return null;
    }
    
    @Override
    public List<String> listTables() {
        return Arrays.asList("users", "products", "orders");
    }
    
    @Override
    public List<String> listColumns(String table) {
        return Arrays.asList("id", "name", "created_at");
    }
}

// Full-featured database implements all interfaces
public class FullDatabase implements 
    DatabaseConnection, 
    QueryExecutor, 
    DataManipulator, 
    TransactionManager,
    DatabaseBackup,
    DatabaseMaintenance,
    SchemaInspector {
    
    // Implement all methods...
}

// Client code uses specific interfaces
public class ReportGenerator {
    private QueryExecutor queryExecutor;
    private SchemaInspector schemaInspector;
    
    public ReportGenerator(QueryExecutor queryExecutor, SchemaInspector schemaInspector) {
        this.queryExecutor = queryExecutor;
        this.schemaInspector = schemaInspector;
    }
    
    public void generateReport() {
        List<String> tables = schemaInspector.listTables();
        for (String table : tables) {
            ResultSet rs = queryExecutor.executeQuery("SELECT * FROM " + table);
            // Process results
        }
    }
}

// Usage
ReadOnlyDatabase readOnlyDb = new ReadOnlyDatabase();
readOnlyDb.connect();

ReportGenerator generator = new ReportGenerator(readOnlyDb, readOnlyDb);
generator.generateReport();  // Works perfectly!
```

---

## Before and After

### Comparison Table

| Aspect | Before ISP | After ISP |
|--------|-----------|-----------|
| **Interface Size** | Large (10+ methods) | Small (1-3 methods) |
| **Unused Methods** | Many | None |
| **Implementation** | Forced to implement all | Implement only needed |
| **Flexibility** | Low | High |
| **Coupling** | High | Low |
| **Maintainability** | Difficult | Easy |
| **Type Safety** | Runtime errors | Compile-time safety |

---

## Real-World Examples

### Example 1: Java I/O Streams

```java
// ISP in action - segregated interfaces
InputStream input = new FileInputStream("file.txt");  // Only reading
OutputStream output = new FileOutputStream("out.txt"); // Only writing

// Not forced to implement both read and write
```

### Example 2: Spring Framework

```java
// Segregated repository interfaces
public interface CrudRepository<T, ID> {
    T save(T entity);
    Optional<T> findById(ID id);
    void deleteById(ID id);
}

public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Page<T> findAll(Pageable pageable);
}

// Use only what you need
public interface UserRepository extends CrudRepository<User, Long> {
    // Only CRUD operations
}
```

### Example 3: JavaScript Events

```javascript
// ISP in event handling
element.addEventListener('click', handler);      // Only click
element.addEventListener('mouseover', handler);  // Only mouseover

// Not forced to handle all events
```

### Famous Products

1. **Java Collections**: Separate interfaces (List, Set, Map, Queue)
2. **JDBC**: Segregated interfaces (Connection, Statement, ResultSet)
3. **Servlet API**: Role-based interfaces (Servlet, Filter, Listener)
4. **React**: Focused component interfaces

---

## Benefits

### 1. Reduced Coupling
```java
// Depend only on what you need
class ReportGenerator {
    private QueryExecutor executor;  // Not full database interface
}
```

### 2. Better Flexibility
```java
// Easy to create new implementations
class CachedQueryExecutor implements QueryExecutor {
    // Only implement query execution
}
```

### 3. Improved Testability
```java
// Easy to mock
@Test
public void testReport() {
    QueryExecutor mockExecutor = mock(QueryExecutor.class);
    ReportGenerator generator = new ReportGenerator(mockExecutor);
    // Test only query execution
}
```

### 4. Clearer Intent
```java
// Interface name shows exact capability
void process(Readable readable) {
    // Clear: only reading
}
```

---

## Common Violations

### 1. God Interfaces

```java
// BAD: Interface does everything
interface Service {
    void method1();
    void method2();
    void method3();
    // ... 20 more methods
}
```

### 2. Forced Empty Implementations

```java
// BAD: Must implement but doesn't use
class SimpleImpl implements ComplexInterface {
    public void method1() { /* used */ }
    public void method2() { /* empty */ }
    public void method3() { /* empty */ }
}
```

### 3. Throwing UnsupportedOperationException

```java
// BAD: Runtime error instead of compile-time safety
public void unsupportedMethod() {
    throw new UnsupportedOperationException();
}
```

---

## How to Apply

### Step 1: Identify Fat Interfaces

Look for:
- Interfaces with many methods (>5)
- Implementations with empty methods
- UnsupportedOperationException throws

### Step 2: Group Related Methods

```java
// Group by capability
interface Worker {
    void work();      // Work capability
    void eat();       // Eating capability
    void sleep();     // Sleeping capability
}
```

### Step 3: Extract Interfaces

```java
// Extract separate interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }
```

### Step 4: Update Implementations

```java
// Implement only what's needed
class Robot implements Workable {
    public void work() { /* work */ }
}
```

### Step 5: Update Client Code

```java
// Use specific interfaces
void manageWork(Workable worker) {
    worker.work();
}
```

---

## Related Patterns

### Patterns that Support ISP

1. **Adapter Pattern**: Adapts fat interface to thin interface
2. **Facade Pattern**: Provides simplified interface
3. **Proxy Pattern**: Controls access through focused interface
4. **Decorator Pattern**: Adds capabilities through composition

### Example: Adapter Pattern

```java
// Adapt fat interface to thin interface
interface LegacyPrinter {
    void print();
    void scan();
    void fax();
}

interface ModernPrinter {
    void print();
}

class PrinterAdapter implements ModernPrinter {
    private LegacyPrinter legacy;
    
    public void print() {
        legacy.print();  // Only expose print
    }
}
```

---

## Conclusion

The Interface Segregation Principle promotes:
- **Focused** interfaces with single responsibilities
- **Flexible** designs that are easy to extend
- **Maintainable** code with clear contracts
- **Type-safe** implementations

**Remember:**
> "No client should be forced to depend on methods it does not use."

Design small, focused interfaces that serve specific client needs, and your code will be more flexible, maintainable, and easier to understand.

---

## Quick Reference

### ✅ Do's
- Create small, focused interfaces
- Design interfaces based on client needs
- Use role interfaces
- Prefer composition over fat interfaces
- Keep interfaces cohesive

### ❌ Don'ts
- Create god interfaces
- Force implementations of unused methods
- Throw UnsupportedOperationException
- Add methods that only some clients need
- Pollute interfaces with unrelated methods

---

**Last Updated:** January 2026  
**Version:** 1.0
