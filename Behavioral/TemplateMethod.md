# Template Method Pattern

## Category
Behavioral Pattern

## Intent
Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

## Motivation
Consider an application framework that provides Application and Document classes. The Application class is responsible for opening existing documents stored in an external format, such as a file. A Document object represents the information in a document once it's read from the file. Applications built with the framework can subclass Application and Document to suit specific needs. For example, a drawing application defines DrawApplication and DrawDocument subclasses; a spreadsheet application defines SpreadsheetApplication and SpreadsheetDocument subclasses. The abstract Application class defines the algorithm for opening and reading a document in its OpenDocument operation. OpenDocument defines each step for opening a document. It checks if the document can be opened, creates the application-specific Document object, adds it to its set of documents, and reads the Document from a file. We call OpenDocument a template method. A template method defines an algorithm in terms of abstract operations that subclasses override to provide concrete behavior.

## Structure

```
┌──────────────────┐
│ AbstractClass    │
├──────────────────┤
│ + templateMethod()│
│ + primitiveOp1() │
│ + primitiveOp2() │
└──────────────────┘
         △
         │
         │
┌──────────────────┐
│  ConcreteClass   │
├──────────────────┤
│ + primitiveOp1() │
│ + primitiveOp2() │
└──────────────────┘
```

## Participants

**AbstractClass**
- Defines abstract primitive operations that concrete subclasses define to implement steps of an algorithm
- Implements a template method defining the skeleton of an algorithm

**ConcreteClass**
- Implements the primitive operations to carry out subclass-specific steps of the algorithm

## Consequences

**Benefits:**
- Code reuse (common behavior in parent class)
- Controls subclassing (only specific operations can be overridden)
- Inverted control structure (Hollywood Principle: "Don't call us, we'll call you")

**Drawbacks:**
- Can be difficult to maintain as number of steps increases
- Subclasses must implement all abstract methods

## Implementation

### Basic Implementation (Java)

```java
abstract class DataProcessor {
    public final void process() {
        readData();
        processData();
        writeData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();
}

class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from CSV file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing data to CSV file");
    }
}

class XMLDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from XML file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing XML data");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing data to XML file");
    }
}

public class TemplateMethodDemo {
    public static void main(String[] args) {
        DataProcessor csvProcessor = new CSVDataProcessor();
        csvProcessor.process();
        
        System.out.println();
        
        DataProcessor xmlProcessor = new XMLDataProcessor();
        xmlProcessor.process();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class AbstractClass(ABC):
    def template_method(self):
        self.base_operation1()
        self.required_operation1()
        self.base_operation2()
        self.hook()
        self.required_operation2()
        self.base_operation3()
    
    def base_operation1(self):
        print("AbstractClass: I am doing the bulk of the work")
    
    def base_operation2(self):
        print("AbstractClass: But I let subclasses override some operations")
    
    def base_operation3(self):
        print("AbstractClass: But I am doing the bulk of the work anyway")
    
    @abstractmethod
    def required_operation1(self):
        pass
    
    @abstractmethod
    def required_operation2(self):
        pass
    
    def hook(self):
        pass

class ConcreteClass1(AbstractClass):
    def required_operation1(self):
        print("ConcreteClass1: Implemented Operation1")
    
    def required_operation2(self):
        print("ConcreteClass1: Implemented Operation2")

class ConcreteClass2(AbstractClass):
    def required_operation1(self):
        print("ConcreteClass2: Implemented Operation1")
    
    def required_operation2(self):
        print("ConcreteClass2: Implemented Operation2")
    
    def hook(self):
        print("ConcreteClass2: Overridden Hook")

def client_code(abstract_class: AbstractClass):
    abstract_class.template_method()

print("Same client code can work with different subclasses:")
client_code(ConcreteClass1())
print()

print("Same client code can work with different subclasses:")
client_code(ConcreteClass2())
```

### JavaScript Implementation

```javascript
class AbstractClass {
    templateMethod() {
        this.baseOperation1();
        this.requiredOperation1();
        this.baseOperation2();
        this.hook();
        this.requiredOperation2();
        this.baseOperation3();
    }
    
    baseOperation1() {
        console.log("AbstractClass: I am doing the bulk of the work");
    }
    
    baseOperation2() {
        console.log("AbstractClass: But I let subclasses override some operations");
    }
    
    baseOperation3() {
        console.log("AbstractClass: But I am doing the bulk of the work anyway");
    }
    
    requiredOperation1() {
        throw new Error("Must implement requiredOperation1");
    }
    
    requiredOperation2() {
        throw new Error("Must implement requiredOperation2");
    }
    
    hook() {}
}

class ConcreteClass1 extends AbstractClass {
    requiredOperation1() {
        console.log("ConcreteClass1: Implemented Operation1");
    }
    
    requiredOperation2() {
        console.log("ConcreteClass1: Implemented Operation2");
    }
}

class ConcreteClass2 extends AbstractClass {
    requiredOperation1() {
        console.log("ConcreteClass2: Implemented Operation1");
    }
    
    requiredOperation2() {
        console.log("ConcreteClass2: Implemented Operation2");
    }
    
    hook() {
        console.log("ConcreteClass2: Overridden Hook");
    }
}

function clientCode(abstractClass) {
    abstractClass.templateMethod();
}

console.log("Same client code can work with different subclasses:");
clientCode(new ConcreteClass1());
console.log();

console.log("Same client code can work with different subclasses:");
clientCode(new ConcreteClass2());
```

## Known Uses

- **Framework classes**: Application frameworks define template methods
- **Servlet lifecycle**: HttpServlet defines template methods
- **JUnit test cases**: setUp(), tearDown() are hooks
- **Spring Template classes**: JdbcTemplate, RestTemplate

## Related Patterns

- **Factory Method**: Often called by template methods
- **Strategy**: Template Method uses inheritance, Strategy uses delegation

## Real-World Example: Beverage Preparation

```java
abstract class Beverage {
    public final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        if (customerWantsCondiments()) {
            addCondiments();
        }
    }
    
    protected abstract void brew();
    protected abstract void addCondiments();
    
    private void boilWater() {
        System.out.println("Boiling water");
    }
    
    private void pourInCup() {
        System.out.println("Pouring into cup");
    }
    
    protected boolean customerWantsCondiments() {
        return true;
    }
}

class Tea extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Steeping the tea");
    }
    
    @Override
    protected void addCondiments() {
        System.out.println("Adding lemon");
    }
}

class Coffee extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Dripping coffee through filter");
    }
    
    @Override
    protected void addCondiments() {
        System.out.println("Adding sugar and milk");
    }
    
    @Override
    protected boolean customerWantsCondiments() {
        return true;
    }
}

public class BeverageDemo {
    public static void main(String[] args) {
        Beverage tea = new Tea();
        System.out.println("Making tea:");
        tea.prepareRecipe();
        
        System.out.println("\nMaking coffee:");
        Beverage coffee = new Coffee();
        coffee.prepareRecipe();
    }
}
```

## When to Use

- Implement invariant parts of an algorithm once and leave it to subclasses to implement behavior that can vary
- Common behavior among subclasses should be factored and localized in a common class
- Control subclass extensions (template method calls hook operations at specific points)
