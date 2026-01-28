# Factory Method Pattern

## Category
Creational Pattern

## Intent
Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

## Also Known As
Virtual Constructor

## Motivation
Frameworks use abstract classes to define and maintain relationships between objects. A framework often needs to create objects, but it can't anticipate the class of objects to create. The Factory Method pattern provides a solution by defining an interface for creating objects but letting subclasses decide which class to instantiate.

## Structure

```
┌─────────────────────┐
│     Creator         │
├─────────────────────┤
│ + factoryMethod()   │◄─────────┐
│ + operation()       │          │
└─────────────────────┘          │
         △                       │
         │                       │
         │                       │ creates
┌─────────────────────┐          │
│  ConcreteCreator    │          │
├─────────────────────┤          │
│ + factoryMethod()   │──────────┘
└─────────────────────┘
         │
         │ returns
         ▼
┌─────────────────────┐
│     Product         │
└─────────────────────┘
         △
         │
         │
┌─────────────────────┐
│  ConcreteProduct    │
└─────────────────────┘
```

## Participants

**Product**
- Defines the interface of objects the factory method creates

**ConcreteProduct**
- Implements the Product interface

**Creator**
- Declares the factory method, which returns an object of type Product
- May call the factory method to create a Product object

**ConcreteCreator**
- Overrides the factory method to return an instance of a ConcreteProduct

## Collaborations
- Creator relies on its subclasses to define the factory method so that it returns an instance of the appropriate ConcreteProduct

## Consequences

**Benefits:**
- Eliminates need to bind application-specific classes into code
- Provides hooks for subclasses (more flexible than creating objects directly)
- Connects parallel class hierarchies

**Drawbacks:**
- Clients might have to subclass Creator just to create a particular ConcreteProduct
- Can result in many small classes

## Implementation

### Basic Implementation (Java)

```java
// Product interface
interface Product {
    void operation();
}

// Concrete Products
class ConcreteProductA implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductA operation");
    }
}

class ConcreteProductB implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductB operation");
    }
}

// Creator
abstract class Creator {
    // Factory method
    public abstract Product factoryMethod();
    
    // Template method using factory method
    public void operation() {
        Product product = factoryMethod();
        product.operation();
    }
}

// Concrete Creators
class ConcreteCreatorA extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductA();
    }
}

class ConcreteCreatorB extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductB();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Product(ABC):
    @abstractmethod
    def operation(self):
        pass

class ConcreteProductA(Product):
    def operation(self):
        return "ConcreteProductA operation"

class ConcreteProductB(Product):
    def operation(self):
        return "ConcreteProductB operation"

class Creator(ABC):
    @abstractmethod
    def factory_method(self):
        pass
    
    def operation(self):
        product = self.factory_method()
        return product.operation()

class ConcreteCreatorA(Creator):
    def factory_method(self):
        return ConcreteProductA()

class ConcreteCreatorB(Creator):
    def factory_method(self):
        return ConcreteProductB()
```

### JavaScript Implementation

```javascript
class Product {
    operation() {
        throw new Error("Must implement operation");
    }
}

class ConcreteProductA extends Product {
    operation() {
        return "ConcreteProductA operation";
    }
}

class ConcreteProductB extends Product {
    operation() {
        return "ConcreteProductB operation";
    }
}

class Creator {
    factoryMethod() {
        throw new Error("Must implement factoryMethod");
    }
    
    operation() {
        const product = this.factoryMethod();
        return product.operation();
    }
}

class ConcreteCreatorA extends Creator {
    factoryMethod() {
        return new ConcreteProductA();
    }
}

class ConcreteCreatorB extends Creator {
    factoryMethod() {
        return new ConcreteProductB();
    }
}
```

## Implementation Considerations

1. **Two main varieties:**
   - Creator is abstract and requires subclasses to define factory method
   - Creator is concrete and provides default implementation

2. **Parameterized factory methods**: Factory method takes parameter to identify kind of object to create

3. **Language-specific variants**: Some languages use templates/generics to avoid subclassing

4. **Naming conventions**: Use descriptive names like `createProduct()` or `makeProduct()`

## Known Uses

- **Java Collections**: `iterator()` method in Collection interface
- **JDBC**: `Connection.createStatement()`
- **Spring Framework**: `FactoryBean` interface
- **Document frameworks**: Creating different document types
- **GUI toolkits**: Creating platform-specific widgets

## Related Patterns

- **Abstract Factory**: Often implemented using Factory Methods
- **Template Method**: Factory Methods are often called within Template Methods
- **Prototype**: Doesn't require subclassing but requires Initialize operation

## Real-World Example: Document Application

```java
// Product
interface Document {
    void open();
    void save();
    void close();
}

// Concrete Products
class PDFDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening PDF document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving PDF document");
    }
    
    @Override
    public void close() {
        System.out.println("Closing PDF document");
    }
}

class WordDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening Word document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving Word document");
    }
    
    @Override
    public void close() {
        System.out.println("Closing Word document");
    }
}

// Creator
abstract class Application {
    private Document document;
    
    public abstract Document createDocument();
    
    public void newDocument() {
        document = createDocument();
        document.open();
    }
    
    public void saveDocument() {
        if (document != null) {
            document.save();
        }
    }
}

// Concrete Creators
class PDFApplication extends Application {
    @Override
    public Document createDocument() {
        return new PDFDocument();
    }
}

class WordApplication extends Application {
    @Override
    public Document createDocument() {
        return new WordDocument();
    }
}

// Usage
public class Client {
    public static void main(String[] args) {
        Application app = new PDFApplication();
        app.newDocument();
        app.saveDocument();
    }
}
```

## Real-World Example: Logistics System

```python
from abc import ABC, abstractmethod

class Transport(ABC):
    @abstractmethod
    def deliver(self):
        pass

class Truck(Transport):
    def deliver(self):
        return "Deliver by land in a truck"

class Ship(Transport):
    def deliver(self):
        return "Deliver by sea in a ship"

class Logistics(ABC):
    @abstractmethod
    def create_transport(self):
        pass
    
    def plan_delivery(self):
        transport = self.create_transport()
        return transport.deliver()

class RoadLogistics(Logistics):
    def create_transport(self):
        return Truck()

class SeaLogistics(Logistics):
    def create_transport(self):
        return Ship()

# Usage
def client_code(logistics: Logistics):
    print(logistics.plan_delivery())

if __name__ == "__main__":
    client_code(RoadLogistics())
    client_code(SeaLogistics())
```

## Variations

### Parameterized Factory Method

```java
abstract class Creator {
    public Product createProduct(String type) {
        if (type.equals("A")) {
            return new ConcreteProductA();
        } else if (type.equals("B")) {
            return new ConcreteProductB();
        }
        throw new IllegalArgumentException("Unknown type");
    }
}
```

### Static Factory Method

```java
class ProductFactory {
    public static Product createProduct(String type) {
        switch (type) {
            case "A": return new ConcreteProductA();
            case "B": return new ConcreteProductB();
            default: throw new IllegalArgumentException("Unknown type");
        }
    }
}
```

## When to Use

- A class can't anticipate the class of objects it must create
- A class wants its subclasses to specify the objects it creates
- Classes delegate responsibility to one of several helper subclasses
- You want to localize the knowledge of which helper subclass is the delegate
