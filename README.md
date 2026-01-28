# Design Patterns Documentation

A comprehensive guide to software design patterns organized by category.

## Overview

Design patterns are reusable solutions to commonly occurring problems in software design. This documentation covers 23 classic Gang of Four (GoF) design patterns plus additional modern patterns.

## Pattern Categories

### 1. Creational Patterns
Patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

- [Singleton](./Creational/Singleton.md)
- [Factory Method](./Creational/FactoryMethod.md)
- [Abstract Factory](./Creational/AbstractFactory.md)
- [Builder](./Creational/Builder.md)
- [Prototype](./Creational/Prototype.md)

### 2. Structural Patterns
Patterns that deal with object composition and typically identify simple ways to realize relationships between different objects.

- [Adapter](./Structural/Adapter.md)
- [Bridge](./Structural/Bridge.md)
- [Composite](./Structural/Composite.md)
- [Decorator](./Structural/Decorator.md)
- [Facade](./Structural/Facade.md)
- [Flyweight](./Structural/Flyweight.md)
- [Proxy](./Structural/Proxy.md)

### 3. Behavioral Patterns
Patterns that deal with communication between objects, how objects interact and distribute responsibility.

- [Chain of Responsibility](./Behavioral/ChainOfResponsibility.md)
- [Command](./Behavioral/Command.md)
- [Iterator](./Behavioral/Iterator.md)
- [Mediator](./Behavioral/Mediator.md)
- [Memento](./Behavioral/Memento.md)
- [Observer](./Behavioral/Observer.md)
- [State](./Behavioral/State.md)
- [Strategy](./Behavioral/Strategy.md)
- [Template Method](./Behavioral/TemplateMethod.md)
- [Visitor](./Behavioral/Visitor.md)
- [Interpreter](./Behavioral/Interpreter.md)

## Quick Reference

See [QUICK_REFERENCE.md](./QUICK_REFERENCE.md) for a quick lookup table of all patterns with their intent and use cases.

## How to Use This Documentation

Each pattern documentation includes:
- **Intent**: What problem the pattern solves
- **Motivation**: Why you would use this pattern
- **Structure**: UML diagram and component description
- **Participants**: Key classes/objects involved
- **Collaborations**: How participants work together
- **Consequences**: Trade-offs and results
- **Implementation**: Code examples and considerations
- **Known Uses**: Real-world applications
- **Related Patterns**: Similar or complementary patterns

## Learning Path

**Beginners**: Start with Singleton, Factory Method, and Observer
**Intermediate**: Move to Decorator, Strategy, and Adapter
**Advanced**: Explore Visitor, Interpreter, and Flyweight

## Resources

- Original Gang of Four book: "Design Patterns: Elements of Reusable Object-Oriented Software"
- [Pattern Comparison Guide](./PATTERN_COMPARISON.md)
- [Anti-Patterns to Avoid](./ANTI_PATTERNS.md)
