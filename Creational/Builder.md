# Builder Pattern

## Category
Creational Pattern

## Intent
Separate the construction of a complex object from its representation so that the same construction process can create different representations.

## Motivation
A reader for the RTF (Rich Text Format) document exchange format should be able to convert RTF to many text formats. The problem is that the number of possible conversions is open-ended. So it should be easy to add a new conversion without modifying the reader. The Builder pattern separates the algorithm for interpreting RTF from how a converted format gets created and represented.

## Structure

```
┌─────────────┐
│   Director  │
├─────────────┤
│ + construct()│───────►┌──────────────┐
└─────────────┘        │   Builder    │
                       ├──────────────┤
                       │ + buildPart()│
                       └──────────────┘
                              △
                              │
                    ┌─────────┴─────────┐
                    │                   │
          ┌──────────────────┐  ┌──────────────────┐
          │ ConcreteBuilder1 │  │ ConcreteBuilder2 │
          ├──────────────────┤  ├──────────────────┤
          │ + buildPart()    │  │ + buildPart()    │
          │ + getResult()    │  │ + getResult()    │
          └──────────────────┘  └──────────────────┘
                  │                     │
                  ▼                     ▼
            ┌─────────┐           ┌─────────┐
            │Product1 │           │Product2 │
            └─────────┘           └─────────┘
```

## Participants

**Builder**
- Specifies an abstract interface for creating parts of a Product object

**ConcreteBuilder**
- Constructs and assembles parts of the product by implementing the Builder interface
- Defines and keeps track of the representation it creates
- Provides an interface for retrieving the product

**Director**
- Constructs an object using the Builder interface

**Product**
- Represents the complex object under construction
- Includes classes that define the constituent parts

## Collaborations
- Client creates Director object and configures it with desired Builder object
- Director notifies builder whenever a part of product should be built
- Builder handles requests from director and adds parts to product
- Client retrieves product from builder

## Consequences

**Benefits:**
- Lets you vary a product's internal representation
- Isolates code for construction and representation
- Gives finer control over the construction process

**Drawbacks:**
- Requires creating a separate ConcreteBuilder for each type of product
- Builder classes must be mutable
- May hamper dependency injection

## Implementation

### Basic Implementation (Java)

```java
// Product
class House {
    private String foundation;
    private String structure;
    private String roof;
    private String interior;
    
    public void setFoundation(String foundation) {
        this.foundation = foundation;
    }
    
    public void setStructure(String structure) {
        this.structure = structure;
    }
    
    public void setRoof(String roof) {
        this.roof = roof;
    }
    
    public void setInterior(String interior) {
        this.interior = interior;
    }
    
    @Override
    public String toString() {
        return "House [foundation=" + foundation + ", structure=" + structure +
               ", roof=" + roof + ", interior=" + interior + "]";
    }
}

// Builder interface
interface HouseBuilder {
    void buildFoundation();
    void buildStructure();
    void buildRoof();
    void buildInterior();
    House getHouse();
}

// Concrete Builders
class ConcreteHouseBuilder implements HouseBuilder {
    private House house;
    
    public ConcreteHouseBuilder() {
        this.house = new House();
    }
    
    @Override
    public void buildFoundation() {
        house.setFoundation("Concrete foundation");
    }
    
    @Override
    public void buildStructure() {
        house.setStructure("Concrete structure");
    }
    
    @Override
    public void buildRoof() {
        house.setRoof("Concrete roof");
    }
    
    @Override
    public void buildInterior() {
        house.setInterior("Concrete interior");
    }
    
    @Override
    public House getHouse() {
        return this.house;
    }
}

class WoodenHouseBuilder implements HouseBuilder {
    private House house;
    
    public WoodenHouseBuilder() {
        this.house = new House();
    }
    
    @Override
    public void buildFoundation() {
        house.setFoundation("Wooden foundation");
    }
    
    @Override
    public void buildStructure() {
        house.setStructure("Wooden structure");
    }
    
    @Override
    public void buildRoof() {
        house.setRoof("Wooden roof");
    }
    
    @Override
    public void buildInterior() {
        house.setInterior("Wooden interior");
    }
    
    @Override
    public House getHouse() {
        return this.house;
    }
}

// Director
class ConstructionEngineer {
    private HouseBuilder houseBuilder;
    
    public ConstructionEngineer(HouseBuilder houseBuilder) {
        this.houseBuilder = houseBuilder;
    }
    
    public House constructHouse() {
        houseBuilder.buildFoundation();
        houseBuilder.buildStructure();
        houseBuilder.buildRoof();
        houseBuilder.buildInterior();
        return houseBuilder.getHouse();
    }
}

// Usage
public class Client {
    public static void main(String[] args) {
        HouseBuilder concreteBuilder = new ConcreteHouseBuilder();
        ConstructionEngineer engineer = new ConstructionEngineer(concreteBuilder);
        House house = engineer.constructHouse();
        System.out.println(house);
    }
}
```

### Fluent Builder (Java)

```java
class Pizza {
    private String dough;
    private String sauce;
    private String topping;
    
    private Pizza(Builder builder) {
        this.dough = builder.dough;
        this.sauce = builder.sauce;
        this.topping = builder.topping;
    }
    
    public static class Builder {
        private String dough;
        private String sauce;
        private String topping;
        
        public Builder dough(String dough) {
            this.dough = dough;
            return this;
        }
        
        public Builder sauce(String sauce) {
            this.sauce = sauce;
            return this;
        }
        
        public Builder topping(String topping) {
            this.topping = topping;
            return this;
        }
        
        public Pizza build() {
            return new Pizza(this);
        }
    }
    
    @Override
    public String toString() {
        return "Pizza [dough=" + dough + ", sauce=" + sauce + ", topping=" + topping + "]";
    }
}

// Usage
Pizza pizza = new Pizza.Builder()
    .dough("thin crust")
    .sauce("tomato")
    .topping("mozzarella")
    .build();
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class House:
    def __init__(self):
        self.foundation = None
        self.structure = None
        self.roof = None
        self.interior = None
    
    def __str__(self):
        return f"House [foundation={self.foundation}, structure={self.structure}, " \
               f"roof={self.roof}, interior={self.interior}]"

class HouseBuilder(ABC):
    @abstractmethod
    def build_foundation(self):
        pass
    
    @abstractmethod
    def build_structure(self):
        pass
    
    @abstractmethod
    def build_roof(self):
        pass
    
    @abstractmethod
    def build_interior(self):
        pass
    
    @abstractmethod
    def get_house(self):
        pass

class ConcreteHouseBuilder(HouseBuilder):
    def __init__(self):
        self.house = House()
    
    def build_foundation(self):
        self.house.foundation = "Concrete foundation"
    
    def build_structure(self):
        self.house.structure = "Concrete structure"
    
    def build_roof(self):
        self.house.roof = "Concrete roof"
    
    def build_interior(self):
        self.house.interior = "Concrete interior"
    
    def get_house(self):
        return self.house

class ConstructionEngineer:
    def __init__(self, builder: HouseBuilder):
        self.builder = builder
    
    def construct_house(self):
        self.builder.build_foundation()
        self.builder.build_structure()
        self.builder.build_roof()
        self.builder.build_interior()
        return self.builder.get_house()

# Fluent Builder in Python
class Pizza:
    def __init__(self):
        self.dough = None
        self.sauce = None
        self.topping = None
    
    class Builder:
        def __init__(self):
            self._pizza = Pizza()
        
        def dough(self, dough):
            self._pizza.dough = dough
            return self
        
        def sauce(self, sauce):
            self._pizza.sauce = sauce
            return self
        
        def topping(self, topping):
            self._pizza.topping = topping
            return self
        
        def build(self):
            return self._pizza

# Usage
pizza = Pizza.Builder() \
    .dough("thin crust") \
    .sauce("tomato") \
    .topping("mozzarella") \
    .build()
```

### JavaScript Implementation

```javascript
class House {
    constructor() {
        this.foundation = null;
        this.structure = null;
        this.roof = null;
        this.interior = null;
    }
    
    toString() {
        return `House [foundation=${this.foundation}, structure=${this.structure}, ` +
               `roof=${this.roof}, interior=${this.interior}]`;
    }
}

class HouseBuilder {
    buildFoundation() {
        throw new Error("Must implement buildFoundation");
    }
    
    buildStructure() {
        throw new Error("Must implement buildStructure");
    }
    
    buildRoof() {
        throw new Error("Must implement buildRoof");
    }
    
    buildInterior() {
        throw new Error("Must implement buildInterior");
    }
    
    getHouse() {
        throw new Error("Must implement getHouse");
    }
}

class ConcreteHouseBuilder extends HouseBuilder {
    constructor() {
        super();
        this.house = new House();
    }
    
    buildFoundation() {
        this.house.foundation = "Concrete foundation";
    }
    
    buildStructure() {
        this.house.structure = "Concrete structure";
    }
    
    buildRoof() {
        this.house.roof = "Concrete roof";
    }
    
    buildInterior() {
        this.house.interior = "Concrete interior";
    }
    
    getHouse() {
        return this.house;
    }
}

// Fluent Builder
class Pizza {
    constructor() {
        this.dough = null;
        this.sauce = null;
        this.topping = null;
    }
    
    static get Builder() {
        class Builder {
            constructor() {
                this.pizza = new Pizza();
            }
            
            dough(dough) {
                this.pizza.dough = dough;
                return this;
            }
            
            sauce(sauce) {
                this.pizza.sauce = sauce;
                return this;
            }
            
            topping(topping) {
                this.pizza.topping = topping;
                return this;
            }
            
            build() {
                return this.pizza;
            }
        }
        return Builder;
    }
}

// Usage
const pizza = new Pizza.Builder()
    .dough("thin crust")
    .sauce("tomato")
    .topping("mozzarella")
    .build();
```

## Implementation Considerations

1. **Assembly and construction interface**: Builders construct products step by step. Builder class interface must be general enough to allow construction of products for all kinds of concrete builders.

2. **Why no abstract class for products**: Products constructed by concrete builders differ so greatly in representation that there's little to gain from giving different products a common parent class.

3. **Empty methods as default in Builder**: Methods in Builder are often defined as empty methods instead of abstract methods to let clients override only operations they're interested in.

## Known Uses

- **StringBuilder/StringBuffer**: Java's string building classes
- **DocumentBuilder**: XML parsing in Java
- **Lombok @Builder**: Annotation-based builder generation
- **Protocol Buffers**: Google's data serialization format
- **SQL Query Builders**: Constructing complex SQL queries

## Related Patterns

- **Abstract Factory**: Similar to Builder in that it may construct complex objects. Primary difference is that Builder focuses on constructing complex object step by step, while Abstract Factory emphasizes families of product objects.
- **Composite**: What the builder often builds
- **Template Method**: Builder pattern can use Template Method to define construction steps

## Real-World Example: HTTP Request Builder

```java
class HttpRequest {
    private String method;
    private String url;
    private Map<String, String> headers;
    private Map<String, String> parameters;
    private String body;
    
    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = builder.headers;
        this.parameters = builder.parameters;
        this.body = builder.body;
    }
    
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private Map<String, String> parameters = new HashMap<>();
        private String body;
        
        public Builder(String url) {
            this.url = url;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public Builder parameter(String key, String value) {
            this.parameters.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// Usage
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body("{\"name\":\"John\"}")
    .build();
```

## When to Use

- Algorithm for creating complex object should be independent of parts that make up object and how they're assembled
- Construction process must allow different representations for object that's constructed
- Need to construct objects with many optional parameters (telescoping constructor anti-pattern)
