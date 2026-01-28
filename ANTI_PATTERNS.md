# Anti-Patterns to Avoid

## What are Anti-Patterns?

Anti-patterns are common responses to recurring problems that are usually ineffective and risk being counterproductive. They represent common mistakes in software design and implementation.

## Common Anti-Patterns

### 1. God Object (The Blob)

**Description**: A single class that does too much, knows too much, or controls too much.

**Symptoms**:
- Class has too many responsibilities
- Class has excessive number of methods and attributes
- Other classes are just data holders
- Difficult to maintain and test

**Example**:
```java
class SystemManager {
    void manageUsers() { }
    void manageDatabase() { }
    void manageNetwork() { }
    void manageUI() { }
    void manageLogging() { }
    void manageConfiguration() { }
    void manageSecurity() { }
    // ... hundreds more methods
}
```

**Solution**:
- Apply Single Responsibility Principle
- Break into smaller, focused classes
- Use Facade pattern if needed for simplified interface

### 2. Spaghetti Code

**Description**: Code with complex and tangled control structure, making it difficult to follow.

**Symptoms**:
- Excessive use of GOTO statements (in languages that support it)
- Deeply nested conditionals
- No clear structure or organization
- Difficult to understand flow

**Solution**:
- Refactor into smaller methods
- Use design patterns appropriately
- Apply SOLID principles
- Improve code organization

### 3. Lava Flow

**Description**: Dead code and forgotten design information frozen in an ever-changing design.

**Symptoms**:
- Code that nobody dares to remove
- Unclear purpose of certain code sections
- "It might be needed someday" mentality
- Accumulation of obsolete code

**Solution**:
- Regular code reviews
- Remove unused code
- Use version control (can always retrieve old code)
- Document code purpose clearly

### 4. Golden Hammer

**Description**: Over-reliance on a familiar technology or pattern for all problems.

**Symptoms**:
- "We always use X for everything"
- Forcing inappropriate solutions
- Ignoring better alternatives
- Technology-driven rather than problem-driven

**Example**:
```java
// Using Singleton for everything
class ConfigurationSingleton { }
class LoggerSingleton { }
class DatabaseSingleton { }
class CacheSingleton { }
// Everything is a Singleton!
```

**Solution**:
- Evaluate each problem independently
- Learn multiple approaches
- Choose appropriate tool for the job
- Stay open to new technologies

### 5. Premature Optimization

**Description**: Optimizing code before knowing if optimization is needed.

**Symptoms**:
- Complex code for marginal performance gains
- Sacrificing readability for performance
- Optimizing without profiling
- "This might be slow" assumptions

**Solution**:
- Write clear code first
- Profile before optimizing
- Optimize only bottlenecks
- Measure impact of optimizations

### 6. Copy-Paste Programming

**Description**: Copying and pasting code instead of creating reusable abstractions.

**Symptoms**:
- Similar code blocks repeated
- Bug fixes need to be applied in multiple places
- Inconsistent implementations
- Difficult to maintain

**Solution**:
- Extract common code into methods/classes
- Use inheritance or composition
- Apply DRY (Don't Repeat Yourself) principle
- Create reusable components

### 7. Magic Numbers and Strings

**Description**: Using literal values directly in code without explanation.

**Symptoms**:
- Unexplained numeric or string literals
- Same value repeated throughout code
- Unclear meaning of values
- Difficult to change values

**Example**:
```java
if (status == 3) {  // What does 3 mean?
    processOrder(amount * 0.15);  // What is 0.15?
}
```

**Solution**:
```java
private static final int STATUS_APPROVED = 3;
private static final double TAX_RATE = 0.15;

if (status == STATUS_APPROVED) {
    processOrder(amount * TAX_RATE);
}
```

### 8. Yo-Yo Problem

**Description**: Excessive inheritance hierarchy requiring navigation up and down to understand behavior.

**Symptoms**:
- Deep inheritance hierarchies
- Need to check multiple parent classes
- Difficult to understand complete behavior
- Fragile base class problem

**Solution**:
- Favor composition over inheritance
- Keep inheritance hierarchies shallow
- Use interfaces for contracts
- Apply Liskov Substitution Principle

### 9. Poltergeists (Gypsy Wagon)

**Description**: Classes with limited responsibilities and short lifecycles.

**Symptoms**:
- Classes that only initialize other classes
- Temporary objects with no real purpose
- Classes that exist only to pass data
- Unnecessary abstraction layers

**Solution**:
- Remove unnecessary classes
- Merge functionality into appropriate classes
- Simplify object interactions
- Question the need for each class

### 10. Circular Dependencies

**Description**: Two or more modules depending on each other.

**Symptoms**:
- Module A depends on Module B, which depends on Module A
- Difficult to test modules independently
- Compilation/build issues
- Tight coupling

**Solution**:
- Introduce interfaces to break cycles
- Refactor to remove dependencies
- Use dependency injection
- Apply Dependency Inversion Principle

### 11. Sequential Coupling

**Description**: Methods must be called in specific order, but order isn't enforced.

**Symptoms**:
- Methods depend on previous method calls
- No compile-time enforcement of order
- Easy to misuse API
- Runtime errors from incorrect usage

**Example**:
```java
Connection conn = new Connection();
conn.open();        // Must call first
conn.execute(sql);  // Must call second
conn.close();       // Must call last
// What if someone forgets open() or close()?
```

**Solution**:
- Use Builder pattern
- Use State pattern to enforce order
- Make invalid states unrepresentable
- Use method chaining with clear flow

### 12. Shotgun Surgery

**Description**: Single change requires modifications in many classes.

**Symptoms**:
- One feature change touches many files
- Related code scattered across system
- High coupling between components
- Difficult to make changes

**Solution**:
- Group related functionality
- Apply Single Responsibility Principle
- Reduce coupling between components
- Use appropriate design patterns

### 13. Feature Envy

**Description**: Method uses more features of another class than its own.

**Symptoms**:
- Method accesses other object's data extensively
- Method belongs more to another class
- Violation of encapsulation
- Poor cohesion

**Example**:
```java
class Order {
    void calculateTotal(Customer customer) {
        double total = 0;
        total += customer.getBasePrice();
        total -= customer.getDiscount();
        total += customer.getTax();
        return total;  // Should be in Customer class
    }
}
```

**Solution**:
- Move method to appropriate class
- Improve encapsulation
- Follow "Tell, Don't Ask" principle

### 14. Inappropriate Intimacy

**Description**: Classes that know too much about each other's internal details.

**Symptoms**:
- Excessive coupling between classes
- Classes accessing each other's private data
- Changes in one class require changes in another
- Difficult to reuse classes independently

**Solution**:
- Reduce coupling
- Improve encapsulation
- Use interfaces for communication
- Apply Law of Demeter

### 15. Refused Bequest

**Description**: Subclass doesn't use inherited methods/properties.

**Symptoms**:
- Subclass overrides methods to do nothing
- Subclass doesn't need parent functionality
- Inappropriate inheritance hierarchy
- Violation of Liskov Substitution Principle

**Solution**:
- Use composition instead of inheritance
- Refactor inheritance hierarchy
- Extract common behavior to separate class
- Use interfaces for contracts

## Pattern-Specific Anti-Patterns

### Singleton Abuse

**Problem**: Using Singleton for everything
**Solution**: Use dependency injection, consider if single instance is really needed

### Decorator Overload

**Problem**: Too many decorator layers
**Solution**: Limit decorator depth, consider Composite pattern

### Observer Memory Leaks

**Problem**: Observers not detached, causing memory leaks
**Solution**: Always detach observers, use weak references

### Factory Complexity

**Problem**: Overly complex factory hierarchies
**Solution**: Keep factories simple, use only when needed

### Strategy Explosion

**Problem**: Too many strategy classes
**Solution**: Consider if strategies can be combined or simplified

## How to Avoid Anti-Patterns

### 1. Code Reviews
- Regular peer reviews
- Identify anti-patterns early
- Share knowledge across team

### 2. Refactoring
- Continuous improvement
- Address technical debt
- Don't let anti-patterns accumulate

### 3. Design Principles
- Follow SOLID principles
- Apply appropriate design patterns
- Keep it simple (KISS)
- Don't repeat yourself (DRY)

### 4. Testing
- Write tests to catch issues
- Test-driven development
- Ensure code is testable

### 5. Education
- Learn common anti-patterns
- Understand why they're problematic
- Study better alternatives

### 6. Tools
- Use static analysis tools
- Code quality metrics
- Automated code reviews

## Refactoring Anti-Patterns

### When to Refactor
- Code smells detected
- Difficulty adding features
- High bug rate in area
- Poor test coverage

### How to Refactor
1. Identify the anti-pattern
2. Write tests for current behavior
3. Refactor incrementally
4. Verify tests still pass
5. Review and iterate

### Refactoring Priorities
1. **High Impact, Low Effort**: Do first
2. **High Impact, High Effort**: Plan carefully
3. **Low Impact, Low Effort**: Do when convenient
4. **Low Impact, High Effort**: Consider not doing

## Conclusion

Anti-patterns are common pitfalls in software development. Recognizing them is the first step to avoiding them. Always:
- Question your design decisions
- Seek feedback from peers
- Continuously learn and improve
- Refactor when needed
- Apply appropriate design patterns
- Follow established principles

Remember: The goal is not to avoid all anti-patterns perfectly, but to recognize them and address them appropriately.
