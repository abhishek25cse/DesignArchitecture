# Visitor Pattern

## Category
Behavioral Pattern

## Intent
Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

## Motivation
Consider a compiler that represents programs as abstract syntax trees. It will need to perform operations on abstract syntax trees for static semantic analyses. The Visitor pattern lets you define new operations without changing the classes of the elements on which it operates.

## Participants

**Visitor**
- Declares a Visit operation for each class of ConcreteElement in the object structure

**ConcreteVisitor**
- Implements each operation declared by Visitor

**Element**
- Defines an Accept operation that takes a visitor as an argument

**ConcreteElement**
- Implements an Accept operation

**ObjectStructure**
- Can enumerate its elements
- May provide high-level interface to allow visitor to visit its elements

## Consequences

**Benefits:**
- Makes adding new operations easy
- Gathers related operations and separates unrelated ones
- Visiting across class hierarchies

**Drawbacks:**
- Adding new ConcreteElement classes is hard
- Breaking encapsulation

## Implementation

### Basic Implementation (Java)

```java
interface Visitor {
    void visit(ConcreteElementA element);
    void visit(ConcreteElementB element);
}

interface Element {
    void accept(Visitor visitor);
}

class ConcreteElementA implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    
    public String operationA() {
        return "ConcreteElementA";
    }
}

class ConcreteElementB implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    
    public String operationB() {
        return "ConcreteElementB";
    }
}

class ConcreteVisitor1 implements Visitor {
    @Override
    public void visit(ConcreteElementA element) {
        System.out.println("ConcreteVisitor1 visiting " + element.operationA());
    }
    
    @Override
    public void visit(ConcreteElementB element) {
        System.out.println("ConcreteVisitor1 visiting " + element.operationB());
    }
}

class ConcreteVisitor2 implements Visitor {
    @Override
    public void visit(ConcreteElementA element) {
        System.out.println("ConcreteVisitor2 visiting " + element.operationA());
    }
    
    @Override
    public void visit(ConcreteElementB element) {
        System.out.println("ConcreteVisitor2 visiting " + element.operationB());
    }
}

public class VisitorDemo {
    public static void main(String[] args) {
        List<Element> elements = Arrays.asList(
            new ConcreteElementA(),
            new ConcreteElementB()
        );
        
        Visitor visitor1 = new ConcreteVisitor1();
        Visitor visitor2 = new ConcreteVisitor2();
        
        for (Element element : elements) {
            element.accept(visitor1);
        }
        
        for (Element element : elements) {
            element.accept(visitor2);
        }
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Visitor(ABC):
    @abstractmethod
    def visit_concrete_element_a(self, element):
        pass
    
    @abstractmethod
    def visit_concrete_element_b(self, element):
        pass

class Element(ABC):
    @abstractmethod
    def accept(self, visitor):
        pass

class ConcreteElementA(Element):
    def accept(self, visitor):
        visitor.visit_concrete_element_a(self)
    
    def operation_a(self):
        return "ConcreteElementA"

class ConcreteElementB(Element):
    def accept(self, visitor):
        visitor.visit_concrete_element_b(self)
    
    def operation_b(self):
        return "ConcreteElementB"

class ConcreteVisitor1(Visitor):
    def visit_concrete_element_a(self, element):
        print(f"ConcreteVisitor1 visiting {element.operation_a()}")
    
    def visit_concrete_element_b(self, element):
        print(f"ConcreteVisitor1 visiting {element.operation_b()}")

class ConcreteVisitor2(Visitor):
    def visit_concrete_element_a(self, element):
        print(f"ConcreteVisitor2 visiting {element.operation_a()}")
    
    def visit_concrete_element_b(self, element):
        print(f"ConcreteVisitor2 visiting {element.operation_b()}")

# Usage
elements = [ConcreteElementA(), ConcreteElementB()]
visitor1 = ConcreteVisitor1()
visitor2 = ConcreteVisitor2()

for element in elements:
    element.accept(visitor1)

for element in elements:
    element.accept(visitor2)
```

### JavaScript Implementation

```javascript
class Visitor {
    visitConcreteElementA(element) {
        throw new Error("Must implement visitConcreteElementA");
    }
    
    visitConcreteElementB(element) {
        throw new Error("Must implement visitConcreteElementB");
    }
}

class Element {
    accept(visitor) {
        throw new Error("Must implement accept");
    }
}

class ConcreteElementA extends Element {
    accept(visitor) {
        visitor.visitConcreteElementA(this);
    }
    
    operationA() {
        return "ConcreteElementA";
    }
}

class ConcreteElementB extends Element {
    accept(visitor) {
        visitor.visitConcreteElementB(this);
    }
    
    operationB() {
        return "ConcreteElementB";
    }
}

class ConcreteVisitor1 extends Visitor {
    visitConcreteElementA(element) {
        console.log(`ConcreteVisitor1 visiting ${element.operationA()}`);
    }
    
    visitConcreteElementB(element) {
        console.log(`ConcreteVisitor1 visiting ${element.operationB()}`);
    }
}

// Usage
const elements = [new ConcreteElementA(), new ConcreteElementB()];
const visitor1 = new ConcreteVisitor1();

elements.forEach(element => element.accept(visitor1));
```

## Known Uses

- **Compilers**: AST traversal
- **Document processing**: Different export formats
- **Graphics**: Rendering different shapes

## Related Patterns

- **Composite**: Visitor can apply operation over Composite
- **Interpreter**: Visitor can apply interpretation

## When to Use

- Object structure contains many classes with differing interfaces
- Many distinct and unrelated operations need to be performed
- Classes defining object structure rarely change
