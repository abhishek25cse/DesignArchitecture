# Liskov Substitution Principle (LSP)

## Definition

> "Objects of a superclass should be replaceable with objects of a subclass without breaking the application."
> 
> — Barbara Liskov (1987)

**Formal Definition:** "If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program."

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

The Liskov Substitution Principle ensures that derived classes extend base classes without changing their behavior. Subtypes must be **behaviorally compatible** with their base types.

### Core Idea

```
┌─────────────────────────────────────────────────────────┐
│           Liskov Substitution Principle                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐                                       │
│  │  Base Class  │                                       │
│  │              │                                       │
│  │  method()    │                                       │
│  └──────┬───────┘                                       │
│         │                                                │
│         │ extends                                        │
│         │                                                │
│         ▼                                                │
│  ┌──────────────┐                                       │
│  │ Derived Class│                                       │
│  │              │                                       │
│  │  method()    │  ← Must behave consistently          │
│  └──────────────┘     with base class                   │
│                                                          │
│  Substitution Test:                                     │
│  base = new Derived();  ← Should work correctly         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Motivation

### The Problem: Broken Substitution

Without LSP, subclasses can break the expectations set by their base classes:

```
┌─────────────────────────────────────────────────────────┐
│              LSP Violation Problem                       │
└─────────────────────────────────────────────────────────┘

Client Code expects:
┌────────────────┐
│   Rectangle    │
│                │
│ setWidth(5)    │ ← Sets width to 5
│ setHeight(4)   │ ← Sets height to 4
│ getArea()      │ ← Returns 20
└────────────────┘

But Square breaks this:
┌────────────────┐
│     Square     │
│  (Rectangle)   │
│                │
│ setWidth(5)    │ ← Sets both width AND height to 5!
│ setHeight(4)   │ ← Sets both width AND height to 4!
│ getArea()      │ ← Returns 16 (not 20!)
└────────────────┘

Result: Client code breaks when using Square!
```

**Classic Example Violation:**
```java
// BAD: Square violates LSP
class Rectangle {
    protected int width;
    protected int height;
    
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
    int getArea() { return width * height; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) {
        width = w;
        height = w;  // Violates expected behavior!
    }
    
    @Override
    void setHeight(int h) {
        width = h;   // Violates expected behavior!
        height = h;
    }
}

// This breaks!
void testRectangle(Rectangle rect) {
    rect.setWidth(5);
    rect.setHeight(4);
    assert rect.getArea() == 20;  // Fails for Square!
}
```

### The Solution: Proper Abstraction

```
┌─────────────────────────────────────────────────────────┐
│              LSP Solution Pattern                        │
└─────────────────────────────────────────────────────────┘

         ┌──────────────┐
         │  <<interface>>│
         │     Shape    │
         │              │
         │+ getArea()   │
         └──────┬───────┘
                │
    ┌───────────┴───────────┐
    │                       │
┌───▼───────┐       ┌───────▼───┐
│ Rectangle │       │  Square   │
│           │       │           │
│+ getArea()│       │+ getArea()│
└───────────┘       └───────────┘

Both implement Shape correctly without inheritance issues!
```

---

## Structure

### Class Diagram

```
┌─────────────────────────────────────────────────────────┐
│           Liskov Substitution Structure                  │
└─────────────────────────────────────────────────────────┘

Violation (Bad Inheritance):
┌────────────────┐
│   Rectangle    │
├────────────────┤
│+ setWidth()    │
│+ setHeight()   │
│+ getArea()     │
└────────┬───────┘
         │
         │ extends (violates LSP!)
         │
┌────────▼───────┐
│     Square     │
├────────────────┤
│+ setWidth()    │ ← Changes both dimensions
│+ setHeight()   │ ← Changes both dimensions
└────────────────┘

Proper Design (Good Abstraction):
         ┌──────────────┐
         │  <<interface>>│
         │     Shape    │
         ├──────────────┤
         │+ getArea()   │
         └──────┬───────┘
                │
    ┌───────────┼───────────┐
    │           │           │
┌───▼───────┐ ┌─▼────────┐ ┌▼──────────┐
│ Rectangle │ │  Square  │ │  Circle   │
├───────────┤ ├──────────┤ ├───────────┤
│- width    │ │- side    │ │- radius   │
│- height   │ │          │ │           │
│+ getArea()│ │+getArea()│ │+ getArea()│
└───────────┘ └──────────┘ └───────────┘
```

---

## Key Concepts

### 1. Behavioral Subtyping

Subtypes must preserve the behavior expected by clients of the base type:

```java
// Base class contract
class Bird {
    void fly() {
        // Birds can fly
    }
}

// BAD: Penguin violates LSP
class Penguin extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// Client code breaks
void makeBirdFly(Bird bird) {
    bird.fly();  // Throws exception for Penguin!
}
```

### 2. Contract Preservation

Derived classes must honor the contract established by the base class:

**Preconditions** cannot be strengthened:
```java
// Base class
class Account {
    void withdraw(double amount) {
        // Precondition: amount > 0
    }
}

// BAD: Strengthens precondition
class PremiumAccount extends Account {
    @Override
    void withdraw(double amount) {
        // Precondition: amount > 0 AND amount < 10000
        // This violates LSP!
    }
}
```

**Postconditions** cannot be weakened:
```java
// Base class
class Calculator {
    int divide(int a, int b) {
        // Postcondition: returns a/b, throws if b == 0
    }
}

// BAD: Weakens postcondition
class SilentCalculator extends Calculator {
    @Override
    int divide(int a, int b) {
        // Returns 0 instead of throwing
        // This violates LSP!
        return b == 0 ? 0 : a / b;
    }
}
```

### 3. Invariants Must Be Preserved

Class invariants must remain true in derived classes:

```java
// Base class invariant: balance >= 0
class BankAccount {
    private double balance;
    
    void withdraw(double amount) {
        if (balance - amount >= 0) {
            balance -= amount;
        }
    }
}

// BAD: Violates invariant
class OverdraftAccount extends BankAccount {
    @Override
    void withdraw(double amount) {
        balance -= amount;  // Can go negative!
    }
}
```

---

## Examples

### Example 1: Rectangle-Square Problem

#### ❌ Violation

```java
// BAD: Classic LSP violation
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getWidth() {
        return width;
    }
    
    public int getHeight() {
        return height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Violates expected behavior
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;  // Violates expected behavior
        this.height = height;
    }
}

// This test passes for Rectangle but fails for Square
public class GeometryTest {
    @Test
    public void testRectangleArea() {
        Rectangle rect = new Rectangle();
        rect.setWidth(5);
        rect.setHeight(4);
        assertEquals(20, rect.getArea());  // PASS
    }
    
    @Test
    public void testSquareArea() {
        Rectangle rect = new Square();  // Substitution
        rect.setWidth(5);
        rect.setHeight(4);
        assertEquals(20, rect.getArea());  // FAIL! Returns 16
    }
}
```

**Why it violates LSP:**
- Square changes the behavior of `setWidth()` and `setHeight()`
- Client code expecting Rectangle behavior breaks with Square
- The "is-a" relationship is mathematically correct but behaviorally wrong

#### ✅ Proper Implementation

```java
// GOOD: Use abstraction without problematic inheritance
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private final int width;
    private final int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public int getWidth() {
        return width;
    }
    
    public int getHeight() {
        return height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private final int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public int getSide() {
        return side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}

// Client code works with Shape interface
public class AreaCalculator {
    public int calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToInt(Shape::getArea)
                    .sum();
    }
}

// Usage - no substitution issues
List<Shape> shapes = Arrays.asList(
    new Rectangle(5, 4),
    new Square(3)
);
AreaCalculator calc = new AreaCalculator();
int total = calc.calculateTotalArea(shapes);  // Works correctly
```

### Example 2: Bird Hierarchy (Python)

#### ❌ Violation

```python
# BAD: Penguin violates LSP
class Bird:
    def fly(self):
        print("Flying in the sky")
    
    def eat(self):
        print("Eating food")

class Sparrow(Bird):
    def fly(self):
        print("Sparrow flying")

class Penguin(Bird):
    def fly(self):
        # Penguins can't fly!
        raise Exception("Penguins cannot fly")

# Client code breaks
def make_bird_fly(bird: Bird):
    bird.fly()  # Works for Sparrow, breaks for Penguin!

# Usage
sparrow = Sparrow()
make_bird_fly(sparrow)  # OK

penguin = Penguin()
make_bird_fly(penguin)  # Exception! Violates LSP
```

#### ✅ Proper Implementation

```python
# GOOD: Proper abstraction hierarchy
from abc import ABC, abstractmethod

class Bird(ABC):
    @abstractmethod
    def eat(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

class FlyingBird(Bird):
    def move(self):
        self.fly()
    
    def fly(self):
        print("Flying in the sky")
    
    def eat(self):
        print("Eating food")

class FlightlessBird(Bird):
    def move(self):
        self.walk()
    
    def walk(self):
        print("Walking on ground")
    
    def eat(self):
        print("Eating food")

class Sparrow(FlyingBird):
    def fly(self):
        print("Sparrow flying fast")

class Eagle(FlyingBird):
    def fly(self):
        print("Eagle soaring high")

class Penguin(FlightlessBird):
    def walk(self):
        print("Penguin waddling")
    
    def swim(self):
        print("Penguin swimming")

class Ostrich(FlightlessBird):
    def walk(self):
        print("Ostrich running fast")

# Client code works correctly
def make_bird_move(bird: Bird):
    bird.move()  # Works for all birds!

def feed_bird(bird: Bird):
    bird.eat()  # Works for all birds!

# Usage
sparrow = Sparrow()
penguin = Penguin()
ostrich = Ostrich()

make_bird_move(sparrow)  # Sparrow flying fast
make_bird_move(penguin)  # Penguin waddling
make_bird_move(ostrich)  # Ostrich running fast

# All birds can eat
feed_bird(sparrow)  # Eating food
feed_bird(penguin)  # Eating food
```

### Example 3: File Storage (JavaScript)

#### ❌ Violation

```javascript
// BAD: ReadOnlyFile violates LSP
class File {
    constructor(name, content) {
        this.name = name;
        this.content = content;
    }
    
    read() {
        return this.content;
    }
    
    write(content) {
        this.content = content;
    }
    
    delete() {
        this.content = '';
    }
}

class ReadOnlyFile extends File {
    write(content) {
        // Violates LSP - base class allows writing
        throw new Error('Cannot write to read-only file');
    }
    
    delete() {
        // Violates LSP - base class allows deletion
        throw new Error('Cannot delete read-only file');
    }
}

// Client code breaks
function updateFile(file) {
    const content = file.read();
    file.write(content + '\nUpdated');  // Throws for ReadOnlyFile!
}

// Usage
const normalFile = new File('doc.txt', 'Hello');
updateFile(normalFile);  // OK

const readOnlyFile = new ReadOnlyFile('config.txt', 'Settings');
updateFile(readOnlyFile);  // Error! Violates LSP
```

#### ✅ Proper Implementation

```javascript
// GOOD: Proper interface segregation
class ReadableFile {
    constructor(name, content) {
        this.name = name;
        this.content = content;
    }
    
    read() {
        return this.content;
    }
}

class WritableFile extends ReadableFile {
    write(content) {
        this.content = content;
    }
    
    append(content) {
        this.content += content;
    }
}

class DeletableFile extends WritableFile {
    delete() {
        this.content = '';
    }
}

// Or use composition
class FileSystem {
    constructor(storage) {
        this.storage = storage;
    }
    
    read(filename) {
        return this.storage.read(filename);
    }
}

class WritableFileSystem extends FileSystem {
    write(filename, content) {
        if (!this.storage.isWritable()) {
            throw new Error('Storage is read-only');
        }
        this.storage.write(filename, content);
    }
}

// Client code with proper checks
function processFile(file) {
    const content = file.read();
    
    if (file instanceof WritableFile) {
        file.write(content + '\nUpdated');
    } else {
        console.log('File is read-only');
    }
}

// Usage
const normalFile = new DeletableFile('doc.txt', 'Hello');
const readOnlyFile = new ReadableFile('config.txt', 'Settings');

processFile(normalFile);     // Updates file
processFile(readOnlyFile);   // Logs "File is read-only"
```

### Example 4: Collection Operations

#### ❌ Violation

```java
// BAD: ImmutableList violates LSP
public class MutableList<T> {
    private List<T> items = new ArrayList<>();
    
    public void add(T item) {
        items.add(item);
    }
    
    public void remove(T item) {
        items.remove(item);
    }
    
    public T get(int index) {
        return items.get(index);
    }
    
    public int size() {
        return items.size();
    }
}

public class ImmutableList<T> extends MutableList<T> {
    @Override
    public void add(T item) {
        throw new UnsupportedOperationException("Cannot modify immutable list");
    }
    
    @Override
    public void remove(T item) {
        throw new UnsupportedOperationException("Cannot modify immutable list");
    }
}

// Client code breaks
public void processItems(MutableList<String> list) {
    list.add("New Item");  // Throws for ImmutableList!
}
```

#### ✅ Proper Implementation

```java
// GOOD: Proper interface hierarchy
public interface ReadableCollection<T> {
    T get(int index);
    int size();
    boolean contains(T item);
}

public interface ModifiableCollection<T> extends ReadableCollection<T> {
    void add(T item);
    void remove(T item);
    void clear();
}

public class ImmutableList<T> implements ReadableCollection<T> {
    private final List<T> items;
    
    public ImmutableList(List<T> items) {
        this.items = new ArrayList<>(items);
    }
    
    @Override
    public T get(int index) {
        return items.get(index);
    }
    
    @Override
    public int size() {
        return items.size();
    }
    
    @Override
    public boolean contains(T item) {
        return items.contains(item);
    }
}

public class MutableList<T> implements ModifiableCollection<T> {
    private List<T> items = new ArrayList<>();
    
    @Override
    public void add(T item) {
        items.add(item);
    }
    
    @Override
    public void remove(T item) {
        items.remove(item);
    }
    
    @Override
    public void clear() {
        items.clear();
    }
    
    @Override
    public T get(int index) {
        return items.get(index);
    }
    
    @Override
    public int size() {
        return items.size();
    }
    
    @Override
    public boolean contains(T item) {
        return items.contains(item);
    }
}

// Client code with proper types
public void readItems(ReadableCollection<String> collection) {
    for (int i = 0; i < collection.size(); i++) {
        System.out.println(collection.get(i));
    }
}

public void modifyItems(ModifiableCollection<String> collection) {
    collection.add("New Item");
}

// Usage
ImmutableList<String> immutable = new ImmutableList<>(Arrays.asList("A", "B"));
MutableList<String> mutable = new MutableList<>();

readItems(immutable);   // OK
readItems(mutable);     // OK

modifyItems(mutable);   // OK
// modifyItems(immutable);  // Compile error - type safety!
```

---

## Before and After

### Comparison Table

| Aspect | Before LSP | After LSP |
|--------|-----------|-----------|
| **Substitution** | Breaks with subtypes | Works seamlessly |
| **Exceptions** | Unexpected exceptions | Predictable behavior |
| **Type Checking** | Need instanceof checks | Polymorphism works |
| **Client Code** | Fragile | Robust |
| **Testing** | Must test all combinations | Test base contract |
| **Maintenance** | Difficult | Easy |

---

## Real-World Examples

### Example 1: Java Collections

```java
// LSP in action
List<String> list1 = new ArrayList<>();
List<String> list2 = new LinkedList<>();
List<String> list3 = new Vector<>();

// All can be substituted - LSP preserved
void processlist(List<String> list) {
    list.add("item");
    list.remove(0);
    // Works for all implementations!
}
```

### Example 2: Database Connections

```python
# LSP in database drivers
class DatabaseConnection:
    def execute(self, query): pass
    def close(self): pass

class MySQLConnection(DatabaseConnection):
    def execute(self, query):
        # MySQL-specific implementation
        pass

class PostgreSQLConnection(DatabaseConnection):
    def execute(self, query):
        # PostgreSQL-specific implementation
        pass

# Client code works with any connection
def run_query(conn: DatabaseConnection, query: str):
    result = conn.execute(query)
    return result
```

### Famous Products

1. **JDBC**: All database drivers are substitutable
2. **Servlet API**: All servlets follow the same contract
3. **Stream API**: All stream implementations are substitutable
4. **React Components**: All components follow same lifecycle

---

## Benefits

### 1. Reliable Polymorphism
```java
// Can substitute any Shape
void drawShape(Shape shape) {
    shape.draw();  // Always works
}
```

### 2. Reduced Type Checking
```java
// No need for instanceof
if (shape instanceof Circle) {
    // Bad - violates LSP
}
```

### 3. Easier Testing
```java
// Test base contract once
@Test
public void testShapeContract(Shape shape) {
    assertTrue(shape.getArea() >= 0);
}
```

### 4. Better Maintainability
- Predictable behavior
- Clear contracts
- Fewer surprises

---

## Common Violations

### 1. Throwing Unexpected Exceptions

```java
// BAD
class Base {
    void method() { }
}

class Derived extends Base {
    @Override
    void method() {
        throw new UnsupportedOperationException();
    }
}
```

### 2. Strengthening Preconditions

```java
// BAD
class Base {
    void process(int value) {
        // Accepts any int
    }
}

class Derived extends Base {
    @Override
    void process(int value) {
        if (value < 0) throw new IllegalArgumentException();
        // Strengthened precondition!
    }
}
```

### 3. Weakening Postconditions

```java
// BAD
class Base {
    int calculate() {
        // Always returns positive
        return 10;
    }
}

class Derived extends Base {
    @Override
    int calculate() {
        // Can return negative!
        return -5;
    }
}
```

### 4. Changing Return Types Incompatibly

```java
// BAD
class Base {
    Number getValue() {
        return 42;
    }
}

class Derived extends Base {
    @Override
    Integer getValue() {  // Covariant return OK
        return null;  // But returning null violates contract!
    }
}
```

---

## How to Apply

### Step 1: Define Clear Contracts

```java
/**
 * Calculates area of a shape.
 * @return area in square units, always >= 0
 */
double getArea();
```

### Step 2: Ensure Behavioral Compatibility

```java
// All implementations must return non-negative
class Circle implements Shape {
    public double getArea() {
        return Math.PI * radius * radius;  // Always >= 0
    }
}
```

### Step 3: Use "Is-Substitutable-For" Test

Ask: "Can I replace Base with Derived everywhere?"

### Step 4: Avoid Inappropriate Inheritance

```java
// Don't inherit just for code reuse
// Use composition instead
```

### Step 5: Test Substitutability

```java
@Test
public void testLSP() {
    Shape shape1 = new Circle(5);
    Shape shape2 = new Rectangle(4, 6);
    
    // Both should work identically in client code
    assertEquals(true, shape1.getArea() >= 0);
    assertEquals(true, shape2.getArea() >= 0);
}
```

---

## Related Patterns

### Patterns that Support LSP

1. **Template Method**: Defines algorithm skeleton, subclasses fill in steps
2. **Strategy Pattern**: Interchangeable algorithms
3. **Factory Pattern**: Creates substitutable objects
4. **Decorator Pattern**: Adds behavior without changing interface

### Example: Template Method

```java
// LSP-compliant template method
abstract class DataProcessor {
    // Template method
    public final void process() {
        readData();
        processData();
        writeData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();
}

// All subclasses are substitutable
class CSVProcessor extends DataProcessor {
    protected void readData() { /* CSV reading */ }
    protected void processData() { /* CSV processing */ }
    protected void writeData() { /* CSV writing */ }
}
```

---

## Conclusion

The Liskov Substitution Principle ensures that inheritance hierarchies are correct and substitutable. By following LSP, you create:

- **Reliable** polymorphism
- **Predictable** behavior
- **Maintainable** code
- **Testable** systems

**Remember:**
> "Inheritance should be used only for substitutability, not for code reuse."

Design your inheritance hierarchies carefully, ensuring that derived classes truly are substitutable for their base classes.

---

## Quick Reference

### ✅ Do's
- Ensure behavioral compatibility
- Honor base class contracts
- Use "is-substitutable-for" test
- Prefer composition over inheritance
- Document contracts clearly

### ❌ Don'ts
- Throw unexpected exceptions
- Strengthen preconditions
- Weaken postconditions
- Change expected behavior
- Use inheritance for code reuse only

---

**Last Updated:** January 2026  
**Version:** 1.0
