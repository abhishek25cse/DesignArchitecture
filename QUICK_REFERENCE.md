# Design Patterns Quick Reference

## Creational Patterns

| Pattern | Intent | When to Use |
|---------|--------|-------------|
| **Singleton** | Ensure a class has only one instance and provide a global point of access | Need exactly one instance of a class (e.g., logging, configuration) |
| **Factory Method** | Define an interface for creating objects, but let subclasses decide which class to instantiate | Class can't anticipate the type of objects it needs to create |
| **Abstract Factory** | Provide an interface for creating families of related objects without specifying concrete classes | System needs to be independent of how its products are created |
| **Builder** | Separate construction of complex object from its representation | Algorithm for creating object should be independent of parts and assembly |
| **Prototype** | Specify kinds of objects to create using a prototypical instance and create new objects by copying | Classes to instantiate are specified at runtime |

## Structural Patterns

| Pattern | Intent | When to Use |
|---------|--------|-------------|
| **Adapter** | Convert interface of a class into another interface clients expect | Want to use existing class with incompatible interface |
| **Bridge** | Decouple abstraction from implementation so both can vary independently | Want to avoid permanent binding between abstraction and implementation |
| **Composite** | Compose objects into tree structures to represent part-whole hierarchies | Want to represent hierarchies of objects and treat them uniformly |
| **Decorator** | Attach additional responsibilities to an object dynamically | Need to add responsibilities to individual objects without affecting others |
| **Facade** | Provide unified interface to a set of interfaces in a subsystem | Want to provide simple interface to complex subsystem |
| **Flyweight** | Use sharing to support large numbers of fine-grained objects efficiently | Application uses large number of objects with shared state |
| **Proxy** | Provide surrogate or placeholder for another object to control access | Need representative object that controls access to another object |

## Behavioral Patterns

| Pattern | Intent | When to Use |
|---------|--------|-------------|
| **Chain of Responsibility** | Avoid coupling sender of request to receiver by giving multiple objects chance to handle | More than one object may handle request, handler not known a priori |
| **Command** | Encapsulate request as an object, letting you parameterize clients with different requests | Need to parameterize objects with actions, queue/log requests, support undo |
| **Iterator** | Provide way to access elements of aggregate object sequentially without exposing representation | Need to access aggregate's contents without exposing internal structure |
| **Mediator** | Define object that encapsulates how a set of objects interact | Set of objects communicate in well-defined but complex ways |
| **Memento** | Capture and externalize object's internal state for later restoration | Need to save/restore object's state without violating encapsulation |
| **Observer** | Define one-to-many dependency so when one object changes state, dependents are notified | Change to one object requires changing others, number unknown |
| **State** | Allow object to alter behavior when internal state changes | Object's behavior depends on its state and must change at runtime |
| **Strategy** | Define family of algorithms, encapsulate each one, and make them interchangeable | Many related classes differ only in behavior, need different variants of algorithm |
| **Template Method** | Define skeleton of algorithm, deferring some steps to subclasses | Want to implement invariant parts of algorithm once and let subclasses define variants |
| **Visitor** | Represent operation to be performed on elements of object structure | Need to perform operations on objects without changing their classes |
| **Interpreter** | Given a language, define representation for its grammar with interpreter | Have language to interpret and can represent statements as abstract syntax trees |

## Pattern Selection Guide

### By Problem Type

**Object Creation**: Singleton, Factory Method, Abstract Factory, Builder, Prototype

**Interface Adaptation**: Adapter, Facade, Bridge

**Object Composition**: Composite, Decorator

**Object Behavior**: Strategy, State, Command, Template Method

**Object Communication**: Observer, Mediator, Chain of Responsibility

**Object Traversal**: Iterator, Visitor

**Performance Optimization**: Flyweight, Proxy

**State Management**: Memento, State

## Complexity Rating

**Simple**: Singleton, Factory Method, Observer, Strategy, Template Method

**Moderate**: Adapter, Decorator, Facade, Command, Iterator, State

**Complex**: Abstract Factory, Builder, Bridge, Composite, Flyweight, Proxy, Chain of Responsibility, Mediator, Memento, Visitor, Interpreter
