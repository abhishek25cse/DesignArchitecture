# SOLID Principles

## Overview

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (Uncle Bob) and are fundamental to object-oriented design.

## The Five SOLID Principles

### 1. [Single Responsibility Principle (SRP)](./SingleResponsibility.md)
**"A class should have one, and only one, reason to change."**

A class should have only one job or responsibility. When a class has multiple responsibilities, changes to one responsibility may affect the others.

**Key Benefits:**
- Easier to understand and maintain
- Reduces coupling
- Improves testability
- Facilitates code reuse

---

### 2. [Open/Closed Principle (OCP)](./OpenClosed.md)
**"Software entities should be open for extension but closed for modification."**

You should be able to extend a class's behavior without modifying its source code. This is achieved through abstraction and polymorphism.

**Key Benefits:**
- Reduces risk of breaking existing code
- Promotes code reusability
- Encourages modular design
- Easier to add new features

---

### 3. [Liskov Substitution Principle (LSP)](./LiskovSubstitution.md)
**"Objects of a superclass should be replaceable with objects of a subclass without breaking the application."**

Derived classes must be substitutable for their base classes. Subtypes must be behaviorally compatible with their base types.

**Key Benefits:**
- Ensures proper inheritance hierarchies
- Prevents unexpected behavior
- Improves code reliability
- Enables polymorphism

---

### 4. [Interface Segregation Principle (ISP)](./InterfaceSegregation.md)
**"Clients should not be forced to depend on interfaces they do not use."**

Many specific interfaces are better than one general-purpose interface. Classes should not be forced to implement methods they don't need.

**Key Benefits:**
- Reduces unnecessary dependencies
- Improves code clarity
- Enhances flexibility
- Easier to refactor

---

### 5. [Dependency Inversion Principle (DIP)](./DependencyInversion.md)
**"Depend on abstractions, not on concretions."**

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

**Key Benefits:**
- Reduces coupling between modules
- Improves testability (easier mocking)
- Enhances flexibility and maintainability
- Facilitates dependency injection

---

## Quick Reference

| Principle | Focus | Main Goal |
|-----------|-------|-----------|
| **SRP** | Class responsibility | One reason to change |
| **OCP** | Extension vs. modification | Open for extension, closed for modification |
| **LSP** | Inheritance | Subtypes must be substitutable |
| **ISP** | Interface design | Small, focused interfaces |
| **DIP** | Dependencies | Depend on abstractions |

---

## How SOLID Principles Work Together

```
┌─────────────────────────────────────────────────────────┐
│                    SOLID Principles                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│  │   SRP    │───▶│   OCP    │───▶│   LSP    │         │
│  │ Single   │    │  Open/   │    │  Liskov  │         │
│  │ Responsi-│    │  Closed  │    │  Substi- │         │
│  │  bility  │    │          │    │  tution  │         │
│  └──────────┘    └──────────┘    └──────────┘         │
│       │               │                │                │
│       │               │                │                │
│       ▼               ▼                ▼                │
│  ┌─────────────────────────────────────────┐           │
│  │         Better Design Quality            │           │
│  └─────────────────────────────────────────┘           │
│       │               │                │                │
│       ▼               ▼                ▼                │
│  ┌──────────┐    ┌──────────┐                          │
│  │   ISP    │    │   DIP    │                          │
│  │Interface │    │Dependency│                          │
│  │Segrega-  │    │Inversion │                          │
│  │  tion    │    │          │                          │
│  └──────────┘    └──────────┘                          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Relationships:

1. **SRP** lays the foundation by ensuring each class has a single responsibility
2. **OCP** builds on SRP by allowing extension without modification
3. **LSP** ensures that inheritance hierarchies work correctly with OCP
4. **ISP** complements SRP by ensuring interfaces are focused
5. **DIP** ties everything together by inverting dependencies to abstractions

---

## When to Apply SOLID Principles

### ✅ Apply When:
- Building long-term, maintainable systems
- Working in teams
- Code is likely to change or extend
- Building reusable libraries or frameworks
- Complexity is increasing

### ⚠️ Be Pragmatic When:
- Building quick prototypes or MVPs
- Working on small, simple projects
- Deadlines are extremely tight
- Over-engineering would add unnecessary complexity

**Remember:** SOLID principles are guidelines, not absolute rules. Apply them judiciously based on your context.

---

## Common Violations and Code Smells

| Principle | Common Violations | Code Smells |
|-----------|------------------|-------------|
| **SRP** | God classes, classes doing too much | Large classes, many methods, multiple reasons to change |
| **OCP** | Modifying existing code for new features | Switch statements, type checking, frequent modifications |
| **LSP** | Subclasses breaking parent contracts | Throwing exceptions in overrides, empty implementations |
| **ISP** | Fat interfaces with many methods | Implementing interfaces with empty methods |
| **DIP** | Direct instantiation of dependencies | `new` keyword everywhere, tight coupling |

---

## SOLID and Design Patterns

Many design patterns embody SOLID principles:

| Principle | Related Patterns |
|-----------|-----------------|
| **SRP** | Facade, Proxy, Decorator |
| **OCP** | Strategy, Template Method, Decorator, Observer |
| **LSP** | Factory Method, Abstract Factory, Template Method |
| **ISP** | Adapter, Facade |
| **DIP** | Factory, Abstract Factory, Dependency Injection, Service Locator |

---

## Learning Path

### Beginner
1. Start with **SRP** - easiest to understand and apply
2. Move to **ISP** - natural extension of SRP for interfaces
3. Learn **DIP** - fundamental for testable code

### Intermediate
4. Study **OCP** - requires understanding of abstraction
5. Master **LSP** - most subtle and nuanced principle

### Advanced
6. Understand how all principles work together
7. Learn when to apply and when to be pragmatic
8. Study real-world examples and refactoring

---

## Resources

### Books
- **"Clean Code"** by Robert C. Martin
- **"Agile Software Development, Principles, Patterns, and Practices"** by Robert C. Martin
- **"Design Patterns: Elements of Reusable Object-Oriented Software"** by Gang of Four

### Online
- [SOLID Principles Overview](./SOLID_OVERVIEW.md) - Comprehensive guide in this repository
- Uncle Bob's Blog: blog.cleancoder.com
- Martin Fowler's Website: martinfowler.com

---

## Document Structure

Each principle document in this folder contains:

1. **Intent** - What the principle aims to achieve
2. **Motivation** - Why it's important
3. **Structure** - Visual diagrams
4. **Examples** - Code in Java, Python, JavaScript
5. **Before/After** - Violations vs. proper implementation
6. **Real-World Examples** - Famous products using the principle
7. **Common Pitfalls** - What to avoid
8. **Related Patterns** - Design patterns that embody the principle

---

## Quick Start

1. Read [SOLID_OVERVIEW.md](./SOLID_OVERVIEW.md) for a comprehensive introduction
2. Study each principle individually:
   - [Single Responsibility Principle](./SingleResponsibility.md)
   - [Open/Closed Principle](./OpenClosed.md)
   - [Liskov Substitution Principle](./LiskovSubstitution.md)
   - [Interface Segregation Principle](./InterfaceSegregation.md)
   - [Dependency Inversion Principle](./DependencyInversion.md)
3. Practice with the provided examples
4. Apply to your own projects incrementally

---

**Remember:** The goal is not to follow these principles blindly, but to write better, more maintainable code. Use them as guidelines to inform your design decisions.

---

**Last Updated:** January 2026  
**Version:** 1.0
