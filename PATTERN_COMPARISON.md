# Design Pattern Comparison Guide

## Similar Patterns Comparison

### Adapter vs Bridge vs Facade vs Proxy

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Adapter** | Convert interface of existing class | Works with existing classes, changes interface |
| **Bridge** | Separate abstraction from implementation | Designed upfront, both can vary independently |
| **Facade** | Simplify complex subsystem | Provides simplified interface to complex system |
| **Proxy** | Control access to object | Same interface, adds control/lazy loading |

**When to use:**
- **Adapter**: Need to use existing class with incompatible interface
- **Bridge**: Want abstraction and implementation to vary independently
- **Facade**: Want to simplify complex subsystem interface
- **Proxy**: Need to control access or add functionality without changing interface

### Strategy vs State vs Template Method

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Strategy** | Encapsulate interchangeable algorithms | Client chooses algorithm, algorithms are independent |
| **State** | Change behavior based on state | Object changes behavior automatically, states know about each other |
| **Template Method** | Define algorithm skeleton | Uses inheritance, subclasses fill in steps |

**When to use:**
- **Strategy**: Need to switch between algorithms at runtime
- **State**: Object behavior changes based on internal state
- **Template Method**: Want to define algorithm structure, let subclasses implement steps

### Decorator vs Proxy vs Adapter

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Decorator** | Add responsibilities dynamically | Enhances object, can stack multiple decorators |
| **Proxy** | Control access to object | Same interface, controls access/lazy loading |
| **Adapter** | Convert interface | Changes interface to match client expectations |

**When to use:**
- **Decorator**: Need to add responsibilities to objects dynamically
- **Proxy**: Need to control access or defer object creation
- **Adapter**: Need to make incompatible interfaces work together

### Factory Method vs Abstract Factory vs Builder

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Factory Method** | Define interface for creating object | One product, subclasses decide which class to instantiate |
| **Abstract Factory** | Create families of related objects | Multiple related products, ensures consistency |
| **Builder** | Construct complex object step by step | Complex object construction, same process different representations |

**When to use:**
- **Factory Method**: Need to defer instantiation to subclasses
- **Abstract Factory**: Need to create families of related objects
- **Builder**: Need to construct complex objects step by step

### Composite vs Decorator

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Composite** | Compose objects into tree structures | Represents part-whole hierarchies, uniform treatment |
| **Decorator** | Add responsibilities to objects | Adds behavior, wraps single object |

**When to use:**
- **Composite**: Need to represent part-whole hierarchies
- **Decorator**: Need to add responsibilities to individual objects

### Observer vs Mediator vs Chain of Responsibility

| Pattern | Intent | Key Difference |
|---------|--------|----------------|
| **Observer** | Notify dependents of state changes | One-to-many, subject notifies all observers |
| **Mediator** | Encapsulate object interactions | Many-to-many, centralizes complex communications |
| **Chain of Responsibility** | Pass request along chain | One-to-one (eventually), decouples sender from receiver |

**When to use:**
- **Observer**: One object needs to notify many others of changes
- **Mediator**: Many objects need to communicate in complex ways
- **Chain of Responsibility**: Request should be handled by one of several objects

## Pattern Relationships

### Patterns that Work Well Together

**Composite + Iterator + Visitor**
- Composite creates tree structure
- Iterator traverses structure
- Visitor performs operations on structure

**Abstract Factory + Singleton**
- Abstract Factory creates product families
- Concrete factories are often Singletons

**Command + Memento**
- Command encapsulates requests
- Memento stores state for undo

**Strategy + Factory Method**
- Factory Method creates Strategy objects
- Strategy encapsulates algorithms

**Decorator + Strategy**
- Decorator changes skin (appearance)
- Strategy changes guts (algorithm)

**Prototype + Abstract Factory**
- Abstract Factory can use Prototype to create products
- Prototype provides alternative to Factory Method

**Builder + Composite**
- Builder constructs complex objects
- Often builds Composite structures

**Facade + Singleton**
- Facade simplifies subsystem interface
- Facade is often a Singleton

## Choosing the Right Pattern

### By Problem Type

**Object Creation Problems**
- Need single instance? → **Singleton**
- Defer instantiation to subclasses? → **Factory Method**
- Create families of objects? → **Abstract Factory**
- Complex object construction? → **Builder**
- Clone existing objects? → **Prototype**

**Interface Problems**
- Incompatible interfaces? → **Adapter**
- Simplify complex system? → **Facade**
- Separate abstraction from implementation? → **Bridge**

**Behavior Problems**
- Encapsulate algorithms? → **Strategy**
- Behavior depends on state? → **State**
- Define algorithm skeleton? → **Template Method**
- Notify multiple objects? → **Observer**
- Encapsulate requests? → **Command**

**Structure Problems**
- Add responsibilities dynamically? → **Decorator**
- Part-whole hierarchies? → **Composite**
- Control access to object? → **Proxy**
- Share objects efficiently? → **Flyweight**

## Common Misconceptions

### Adapter vs Decorator
**Misconception**: They're the same because both wrap objects
**Reality**: Adapter changes interface, Decorator enhances functionality

### Strategy vs State
**Misconception**: They have the same structure
**Reality**: Strategy is about choosing algorithm, State is about changing behavior based on state

### Factory Method vs Abstract Factory
**Misconception**: Abstract Factory is just multiple Factory Methods
**Reality**: Abstract Factory creates families of related objects, Factory Method creates one product

### Singleton vs Static Class
**Misconception**: Singleton is just a static class
**Reality**: Singleton can implement interfaces, be subclassed, and control instantiation

### Facade vs Adapter
**Misconception**: Both simplify interfaces
**Reality**: Facade simplifies many interfaces, Adapter converts one interface to another

## Anti-Pattern Warnings

### Overusing Singleton
**Problem**: Global state, tight coupling, difficult testing
**Solution**: Use dependency injection instead

### Deep Decorator Chains
**Problem**: Complex, hard to debug
**Solution**: Limit decorator depth, consider Composite

### God Object Facade
**Problem**: Facade becomes too large and complex
**Solution**: Break into multiple facades

### Excessive Factory Layers
**Problem**: Too many abstraction layers
**Solution**: Use factories only when needed

### State Explosion
**Problem**: Too many state classes
**Solution**: Consider State pattern alternatives or combine states

## Pattern Selection Flowchart

```
Need to create objects?
├─ Yes → Need single instance? → Singleton
│      → Need to defer to subclasses? → Factory Method
│      → Need families of objects? → Abstract Factory
│      → Complex construction? → Builder
│      → Clone objects? → Prototype
│
├─ Need to adapt interfaces?
│  ├─ Convert interface? → Adapter
│  ├─ Simplify complex system? → Facade
│  └─ Separate abstraction/implementation? → Bridge
│
├─ Need to add behavior?
│  ├─ Add dynamically? → Decorator
│  ├─ Control access? → Proxy
│  └─ Part-whole hierarchy? → Composite
│
└─ Need to manage behavior?
   ├─ Encapsulate algorithms? → Strategy
   ├─ Behavior depends on state? → State
   ├─ Define algorithm skeleton? → Template Method
   ├─ Notify dependents? → Observer
   └─ Encapsulate requests? → Command
```

## Performance Considerations

### Memory Impact
- **Flyweight**: Reduces memory usage significantly
- **Prototype**: Can increase memory if cloning is expensive
- **Singleton**: Minimal memory overhead

### Runtime Performance
- **Proxy**: May add latency for lazy loading
- **Decorator**: Slight overhead per decorator layer
- **Chain of Responsibility**: Performance depends on chain length

### Scalability
- **Observer**: Can become bottleneck with many observers
- **Mediator**: Centralizes communication, can become bottleneck
- **Flyweight**: Improves scalability with many objects

## Testing Considerations

### Easy to Test
- **Strategy**: Easy to test different algorithms
- **Command**: Easy to test commands in isolation
- **Factory Method**: Easy to inject test implementations

### Challenging to Test
- **Singleton**: Global state makes testing difficult
- **Observer**: Need to verify all observers notified
- **Mediator**: Complex interactions to test

### Testing Strategies
- Use dependency injection instead of Singleton
- Mock observers in Observer pattern tests
- Test each state independently in State pattern
- Test decorators individually and in combination
