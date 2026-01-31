# SOLID Principles - Complete Summary

## Quick Reference Guide

This document provides a consolidated summary of all five SOLID principles with key concepts, examples, and quick reference tables.

---

## Table of Contents

1. [Overview](#overview)
2. [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
3. [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
4. [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
5. [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
6. [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
7. [Quick Comparison](#quick-comparison)
8. [Decision Matrix](#decision-matrix)
9. [Common Patterns](#common-patterns)

---

## Overview

**SOLID** is an acronym for five design principles that make software designs more understandable, flexible, and maintainable.

| Letter | Principle | Definition |
|--------|-----------|------------|
| **S** | Single Responsibility | A class should have one, and only one, reason to change |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subtypes must be substitutable for their base types |
| **I** | Interface Segregation | Many specific interfaces are better than one general-purpose interface |
| **D** | Dependency Inversion | Depend on abstractions, not on concretions |

---

## Single Responsibility Principle (SRP)

### Definition
> "A class should have one, and only one, reason to change."

### Key Concept
Each class should have only one job or responsibility.

### Visual Summary
```
BAD: God Class                    GOOD: Focused Classes
┌────────────────┐               ┌──────────┐ ┌──────────┐
│   Employee     │               │ Employee │ │PayCalc   │
├────────────────┤               ├──────────┤ ├──────────┤
│- calculatePay()│               │- name    │ │- calc()  │
│- save()        │      ──▶      │- salary  │ │          │
│- report()      │               └──────────┘ └──────────┘
│- validate()    │               ┌──────────┐ ┌──────────┐
└────────────────┘               │EmpRepo   │ │Reporter  │
Multiple reasons                 ├──────────┤ ├──────────┤
to change!                       │- save()  │ │- report()│
                                 └──────────┘ └──────────┘
                                 Single reason to change each!
```

### Example

**❌ Violation:**
```java
class Employee {
    void calculatePay() { }      // Accounting
    void saveToDatabase() { }    // Persistence
    void generateReport() { }    // Reporting
}
```

**✅ Proper:**
```java
class Employee {
    private String name;
    private double salary;
}

class PayCalculator {
    double calculatePay(Employee emp) { }
}

class EmployeeRepository {
    void save(Employee emp) { }
}

class ReportGenerator {
    String generate(Employee emp) { }
}
```

### Benefits
- ✅ Easier to understand
- ✅ Easier to test
- ✅ Less coupling
- ✅ Better reusability

### Code Smells
- ❌ Large classes (>200 lines)
- ❌ Many methods (>10)
- ❌ Multiple import groups
- ❌ Comments like "Section 1", "Section 2"

---

## Open/Closed Principle (OCP)

### Definition
> "Software entities should be open for extension but closed for modification."

### Key Concept
Add new functionality without changing existing code.

### Visual Summary
```
BAD: Modification Required        GOOD: Extension Through Abstraction
┌────────────────┐               ┌──────────────┐
│AreaCalculator  │               │<<interface>> │
├────────────────┤               │    Shape     │
│if (Circle)     │               ├──────────────┤
│  calc circle   │               │+ getArea()   │
│else if (Rect)  │      ──▶      └──────┬───────┘
│  calc rect     │                      │
│else if (Tri)   │               ┌──────┴───────┬────────┐
│  calc triangle │               │              │        │
└────────────────┘           ┌───▼───┐     ┌───▼───┐ ┌──▼──┐
Must modify for              │Circle │     │  Rect │ │ Tri │
each new shape!              └───────┘     └───────┘ └─────┘
                             Add new shapes without modification!
```

### Example

**❌ Violation:**
```java
class AreaCalculator {
    double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            // Circle calculation
        } else if (shape instanceof Rectangle) {
            // Rectangle calculation
        }
        // Must add more if-else for new shapes!
    }
}
```

**✅ Proper:**
```java
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    public double calculateArea() {
        return width * height;
    }
}

// Add new shapes without modifying existing code
class Triangle implements Shape {
    public double calculateArea() {
        return 0.5 * base * height;
    }
}
```

### Benefits
- ✅ Reduced risk of breaking existing code
- ✅ Easier to add features
- ✅ Better testability
- ✅ Promotes code reuse

### Code Smells
- ❌ Type checking (instanceof, typeof)
- ❌ Switch statements on types
- ❌ Frequent modifications to existing code

---

## Liskov Substitution Principle (LSP)

### Definition
> "Objects of a superclass should be replaceable with objects of a subclass without breaking the application."

### Key Concept
Subtypes must be behaviorally compatible with their base types.

### Visual Summary
```
BAD: Broken Substitution          GOOD: Proper Abstraction
┌────────────┐                   ┌──────────────┐
│ Rectangle  │                   │<<interface>> │
├────────────┤                   │    Shape     │
│setWidth(5) │                   ├──────────────┤
│setHeight(4)│                   │+ getArea()   │
│getArea()=20│                   └──────┬───────┘
└──────┬─────┘                          │
       │                          ┌─────┴──────┬────────┐
┌──────▼─────┐                    │            │        │
│   Square   │           ──▶  ┌───▼───┐   ┌───▼───┐ ┌──▼──┐
├────────────┤               │ Rect  │   │Square │ │Circle│
│setWidth(5) │               │(w,h)  │   │(side) │ │(rad) │
│  sets both!│               └───────┘   └───────┘ └──────┘
│setHeight(4)│               Each implements getArea() correctly
│  sets both!│               without breaking substitution!
│getArea()=16│
└────────────┘
Breaks expected
behavior!
```

### Example

**❌ Violation:**
```java
class Rectangle {
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) {
        width = w;
        height = w;  // Violates expected behavior!
    }
}

// This breaks!
void test(Rectangle rect) {
    rect.setWidth(5);
    rect.setHeight(4);
    assert rect.getArea() == 20;  // Fails for Square!
}
```

**✅ Proper:**
```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width, height;
    
    Rectangle(int w, int h) {
        width = w;
        height = h;
    }
    
    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;
    
    Square(int s) {
        side = s;
    }
    
    public int getArea() {
        return side * side;
    }
}
```

### Benefits
- ✅ Ensures correct inheritance
- ✅ Prevents unexpected behavior
- ✅ Enables true polymorphism
- ✅ More reliable code

### Code Smells
- ❌ Throwing UnsupportedOperationException
- ❌ Empty method implementations
- ❌ Strengthening preconditions
- ❌ Weakening postconditions

---

## Interface Segregation Principle (ISP)

### Definition
> "Clients should not be forced to depend on interfaces they do not use."

### Key Concept
Many small, focused interfaces are better than one large interface.

### Visual Summary
```
BAD: Fat Interface                GOOD: Segregated Interfaces
┌────────────────┐               ┌──────────┐ ┌──────────┐
│<<interface>>   │               │Workable  │ │ Eatable  │
│    Worker      │               ├──────────┤ ├──────────┤
├────────────────┤               │+ work()  │ │+ eat()   │
│+ work()        │               └─────┬────┘ └─────┬────┘
│+ eat()         │      ──▶            │            │
│+ sleep()       │               ┌─────▼────┐ ┌─────▼────┐
└────────┬───────┘               │  Human   │ │  Robot   │
         │                       │implements│ │implements│
    ┌────┴────┐                  │both      │ │Workable  │
┌───▼───┐ ┌──▼───┐               └──────────┘ │only      │
│ Human │ │Robot │                             └──────────┘
│all ✓  │ │work✓ │               No forced implementations!
│       │ │eat ✗ │
│       │ │sleep✗│
└───────┘ └──────┘
Forced to implement
unused methods!
```

### Example

**❌ Violation:**
```java
interface Worker {
    void work();
    void eat();
    void sleep();
}

class RobotWorker implements Worker {
    public void work() { /* work */ }
    public void eat() { throw new UnsupportedOperationException(); }
    public void sleep() { throw new UnsupportedOperationException(); }
}
```

**✅ Proper:**
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class HumanWorker implements Workable, Eatable, Sleepable {
    public void work() { /* work */ }
    public void eat() { /* eat */ }
    public void sleep() { /* sleep */ }
}

class RobotWorker implements Workable {
    public void work() { /* work */ }
}
```

### Benefits
- ✅ Reduced coupling
- ✅ Better flexibility
- ✅ Easier to implement
- ✅ Clearer intent

### Code Smells
- ❌ Large interfaces (>5 methods)
- ❌ Empty method implementations
- ❌ UnsupportedOperationException
- ❌ Implementing interfaces with unused methods

---

## Dependency Inversion Principle (DIP)

### Definition
> "Depend on abstractions, not on concretions."

### Key Concept
High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Visual Summary
```
BAD: Direct Dependency            GOOD: Inverted Dependency
┌────────────────┐               ┌────────────────┐
│  UserService   │               │  UserService   │
│  (High-level)  │               │  (High-level)  │
└────────┬───────┘               └────────┬───────┘
         │ depends on                     │ depends on
         ▼                                ▼
┌────────────────┐               ┌────────────────┐
│ MySQLDatabase  │               │<<interface>>   │
│  (Low-level)   │      ──▶      │  Database      │
└────────────────┘               └────────▲───────┘
Tight coupling!                           │ implements
                                          │
                                 ┌────────┴────────┬──────────┐
                                 │                 │          │
                            ┌────▼────┐      ┌────▼────┐ ┌───▼───┐
                            │  MySQL  │      │Postgres │ │ Mongo │
                            └─────────┘      └─────────┘ └───────┘
                            Loose coupling, easy to swap!
```

### Example

**❌ Violation:**
```java
class UserService {
    private MySQLDatabase database;  // Tight coupling!
    
    UserService() {
        this.database = new MySQLDatabase();
    }
    
    void saveUser(User user) {
        database.save(user);
    }
}
```

**✅ Proper:**
```java
interface Database {
    void save(User user);
}

class MySQLDatabase implements Database {
    public void save(User user) {
        // MySQL implementation
    }
}

class PostgreSQLDatabase implements Database {
    public void save(User user) {
        // PostgreSQL implementation
    }
}

class UserService {
    private Database database;  // Depends on abstraction
    
    UserService(Database database) {  // Dependency injection
        this.database = database;
    }
    
    void saveUser(User user) {
        database.save(user);
    }
}

// Usage - easy to switch
UserService service1 = new UserService(new MySQLDatabase());
UserService service2 = new UserService(new PostgreSQLDatabase());
```

### Benefits
- ✅ Loose coupling
- ✅ Easy to test (mocking)
- ✅ Flexible and extensible
- ✅ Better maintainability

### Code Smells
- ❌ `new` keyword in business logic
- ❌ Direct references to concrete classes
- ❌ Static method calls
- ❌ Service locator pattern

---

## Quick Comparison

### At a Glance

| Principle | Focus | Main Goal | Key Technique |
|-----------|-------|-----------|---------------|
| **SRP** | Class responsibility | One reason to change | Extract classes |
| **OCP** | Extension vs modification | Open for extension | Use abstractions |
| **LSP** | Inheritance | Subtypes substitutable | Proper contracts |
| **ISP** | Interface design | Small, focused interfaces | Segregate interfaces |
| **DIP** | Dependencies | Depend on abstractions | Dependency injection |

### When to Apply

| Principle | Apply When | Avoid When |
|-----------|-----------|------------|
| **SRP** | Class has multiple responsibilities | Class is simple and focused |
| **OCP** | Frequent new features | Requirements are stable |
| **LSP** | Using inheritance | Using composition |
| **ISP** | Interface has many methods | Interface is cohesive |
| **DIP** | Need flexibility/testability | Simple, stable dependencies |

### Violation Indicators

| Principle | Red Flags |
|-----------|-----------|
| **SRP** | Large classes, many methods, multiple reasons to change |
| **OCP** | Type checking, switch statements, frequent modifications |
| **LSP** | Empty methods, UnsupportedOperationException, type checking |
| **ISP** | Fat interfaces, forced implementations, unused methods |
| **DIP** | `new` keyword everywhere, tight coupling, hard to test |

---

## Decision Matrix

### Choosing Which Principle to Apply First

```
Start Here: What's the main problem?

├─ Class doing too much?
│  └─> Apply SRP
│     └─> Extract responsibilities into separate classes
│
├─ Need to add new features frequently?
│  └─> Apply OCP
│     └─> Design with abstractions and extension points
│
├─ Inheritance causing issues?
│  └─> Apply LSP
│     └─> Ensure subtypes are truly substitutable
│
├─ Interfaces too large?
│  └─> Apply ISP
│     └─> Split into smaller, focused interfaces
│
└─ Hard to test or swap implementations?
   └─> Apply DIP
      └─> Depend on abstractions, inject dependencies
```

### Priority by Project Phase

**New Project:**
1. **DIP** - Set up dependency injection early
2. **SRP** - Keep classes focused from start
3. **ISP** - Design small interfaces
4. **OCP** - Plan for extension
5. **LSP** - Ensure proper inheritance

**Refactoring Existing Code:**
1. **SRP** - Break up god classes first
2. **DIP** - Introduce abstractions for testability
3. **ISP** - Split fat interfaces
4. **OCP** - Add extension points
5. **LSP** - Fix inheritance issues

---

## Common Patterns

### How SOLID Principles Work Together

```
┌─────────────────────────────────────────────────────────┐
│                  SOLID Synergy                           │
└─────────────────────────────────────────────────────────┘

SRP: Each class has one responsibility
  ↓
OCP: Each class can be extended without modification
  ↓
LSP: Subclasses work correctly with base class contracts
  ↓
ISP: Interfaces are small and focused (supports SRP)
  ↓
DIP: All depend on abstractions (enables OCP and testability)
  ↓
Result: Flexible, maintainable, testable system
```

### Design Pattern Relationships

| Pattern | Primary Principle | Supporting Principles |
|---------|------------------|----------------------|
| **Strategy** | OCP | DIP, SRP |
| **Factory** | DIP | OCP, SRP |
| **Decorator** | OCP | SRP, LSP |
| **Adapter** | ISP | DIP |
| **Template Method** | OCP | LSP |
| **Observer** | OCP | DIP |
| **Facade** | SRP | ISP |
| **Proxy** | SRP | DIP |

---

## Practical Examples Summary

### E-Commerce System Example

```java
// SRP: Separate responsibilities
class Product { }                    // Data
class ProductRepository { }          // Persistence
class ProductValidator { }           // Validation
class PriceCalculator { }           // Business logic

// OCP: Extend without modification
interface PaymentMethod {
    void process(double amount);
}
class CreditCard implements PaymentMethod { }
class PayPal implements PaymentMethod { }
class Bitcoin implements PaymentMethod { }  // New, no changes needed

// LSP: Substitutable implementations
void processPayment(PaymentMethod method, double amount) {
    method.process(amount);  // Works for all implementations
}

// ISP: Focused interfaces
interface Orderable { void placeOrder(); }
interface Cancellable { void cancel(); }
interface Refundable { void refund(); }

// DIP: Depend on abstractions
class OrderService {
    private PaymentMethod payment;
    private OrderRepository repository;
    
    OrderService(PaymentMethod payment, OrderRepository repository) {
        this.payment = payment;
        this.repository = repository;
    }
}
```

---

## Quick Tips

### Do's ✅

1. **SRP**: Keep classes small and focused
2. **OCP**: Design with abstractions and interfaces
3. **LSP**: Ensure behavioral compatibility
4. **ISP**: Create role-based interfaces
5. **DIP**: Inject dependencies, don't create them

### Don'ts ❌

1. **SRP**: Don't create god classes
2. **OCP**: Don't use type checking (instanceof)
3. **LSP**: Don't throw UnsupportedOperationException
4. **ISP**: Don't create fat interfaces
5. **DIP**: Don't use `new` in business logic

### Red Flags 🚩

- Classes > 200 lines
- Methods > 20 lines
- Interfaces > 5 methods
- `instanceof` or `typeof` checks
- `new` keyword in constructors
- Empty method implementations
- UnsupportedOperationException

---

## Learning Path

### Beginner (Weeks 1-4)
1. **Week 1-2**: Study SRP - easiest to grasp
2. **Week 3-4**: Learn ISP - natural extension

### Intermediate (Weeks 5-8)
3. **Week 5-6**: Master DIP - critical for testability
4. **Week 7-8**: Understand OCP - requires abstraction thinking

### Advanced (Weeks 9-10)
5. **Week 9-10**: Deep dive into LSP - most subtle principle

### Practice
- Refactor existing code
- Do code katas
- Review open source projects
- Discuss with team

---

## Conclusion

SOLID principles are guidelines, not absolute rules. Apply them pragmatically:

**Remember:**
- Start with SRP - foundation for all others
- Use OCP for flexibility
- Ensure LSP for correct inheritance
- Apply ISP for focused interfaces
- Implement DIP for testability

**The Goal:**
> Write code that is easy to understand, maintain, and extend.

**Next Steps:**
1. Read individual principle documents for details
2. Practice with code examples
3. Apply to your projects incrementally
4. Review and refactor regularly

---

## Additional Resources

### Documentation in This Repository
- [README.md](./README.md) - Overview and quick reference
- [SOLID_OVERVIEW.md](./SOLID_OVERVIEW.md) - Comprehensive guide
- [SingleResponsibility.md](./SingleResponsibility.md) - SRP details
- [OpenClosed.md](./OpenClosed.md) - OCP details
- [LiskovSubstitution.md](./LiskovSubstitution.md) - LSP details
- [InterfaceSegregation.md](./InterfaceSegregation.md) - ISP details
- [DependencyInversion.md](./DependencyInversion.md) - DIP details

### Books
- "Clean Code" by Robert C. Martin
- "Agile Software Development, Principles, Patterns, and Practices" by Robert C. Martin
- "Design Patterns" by Gang of Four

### Online
- Uncle Bob's Blog: blog.cleancoder.com
- Martin Fowler's Website: martinfowler.com

---

**Last Updated:** January 2026  
**Version:** 1.0  
**Total Principles:** 5  
**Total Examples:** 15+  
**Total Diagrams:** 10+
