# Prototype Pattern

## Category
Creational Pattern

## Intent
Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

## Motivation
Building an editor for music scores by customizing a general framework for graphical editors. The framework provides an abstract Graphic class for graphical components like notes and staffs. The framework also predefines a GraphicTool subclass for tools that create instances of graphical objects and add them to the document. But GraphicTool presents a problem: the class of Graphic is specific to the application, but GraphicTool belongs to the framework. The solution is to make GraphicTool create new Graphic by copying or "cloning" an instance of a Graphic subclass.

## Structure

```
┌────────────┐
│   Client   │
├────────────┤
│            │───────►┌──────────────┐
└────────────┘        │  Prototype   │
                      ├──────────────┤
                      │ + clone()    │
                      └──────────────┘
                             △
                             │
                    ┌────────┴────────┐
                    │                 │
        ┌──────────────────┐  ┌──────────────────┐
        │ConcretePrototype1│  │ConcretePrototype2│
        ├──────────────────┤  ├──────────────────┤
        │ + clone()        │  │ + clone()        │
        └──────────────────┘  └──────────────────┘
```

## Participants

**Prototype**
- Declares an interface for cloning itself

**ConcretePrototype**
- Implements an operation for cloning itself

**Client**
- Creates a new object by asking a prototype to clone itself

## Collaborations
- Client asks a prototype to clone itself

## Consequences

**Benefits:**
- Adding and removing products at runtime
- Specifying new objects by varying values
- Specifying new objects by varying structure
- Reduced subclassing
- Configuring an application with classes dynamically

**Drawbacks:**
- Each subclass must implement clone operation
- Implementing clone can be difficult (circular references, objects that don't support copying)

## Implementation

### Basic Implementation (Java)

```java
// Prototype interface
interface Prototype extends Cloneable {
    Prototype clone();
}

// Concrete Prototypes
class ConcretePrototype1 implements Prototype {
    private String field;
    
    public ConcretePrototype1(String field) {
        this.field = field;
    }
    
    @Override
    public Prototype clone() {
        return new ConcretePrototype1(this.field);
    }
    
    @Override
    public String toString() {
        return "ConcretePrototype1 [field=" + field + "]";
    }
}

class ConcretePrototype2 implements Prototype {
    private int value;
    
    public ConcretePrototype2(int value) {
        this.value = value;
    }
    
    @Override
    public Prototype clone() {
        return new ConcretePrototype2(this.value);
    }
    
    @Override
    public String toString() {
        return "ConcretePrototype2 [value=" + value + "]";
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        Prototype prototype1 = new ConcretePrototype1("Original");
        Prototype clone1 = prototype1.clone();
        
        System.out.println(prototype1);
        System.out.println(clone1);
    }
}
```

### Deep Copy Implementation (Java)

```java
class Address implements Cloneable {
    private String street;
    private String city;
    
    public Address(String street, String city) {
        this.street = street;
        this.city = city;
    }
    
    @Override
    protected Address clone() {
        return new Address(this.street, this.city);
    }
    
    @Override
    public String toString() {
        return street + ", " + city;
    }
}

class Person implements Cloneable {
    private String name;
    private Address address;
    
    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }
    
    @Override
    protected Person clone() {
        return new Person(this.name, this.address.clone());
    }
    
    @Override
    public String toString() {
        return "Person [name=" + name + ", address=" + address + "]";
    }
}
```

### Python Implementation

```python
import copy
from abc import ABC, abstractmethod

class Prototype(ABC):
    @abstractmethod
    def clone(self):
        pass

class ConcretePrototype1(Prototype):
    def __init__(self, field):
        self.field = field
    
    def clone(self):
        return copy.deepcopy(self)
    
    def __str__(self):
        return f"ConcretePrototype1 [field={self.field}]"

class ConcretePrototype2(Prototype):
    def __init__(self, value):
        self.value = value
    
    def clone(self):
        return copy.deepcopy(self)
    
    def __str__(self):
        return f"ConcretePrototype2 [value={self.value}]"

# Usage
prototype1 = ConcretePrototype1("Original")
clone1 = prototype1.clone()

print(prototype1)
print(clone1)
print(f"Same object? {prototype1 is clone1}")
```

### JavaScript Implementation

```javascript
class Prototype {
    clone() {
        throw new Error("Must implement clone");
    }
}

class ConcretePrototype1 extends Prototype {
    constructor(field) {
        super();
        this.field = field;
    }
    
    clone() {
        return new ConcretePrototype1(this.field);
    }
    
    toString() {
        return `ConcretePrototype1 [field=${this.field}]`;
    }
}

class ConcretePrototype2 extends Prototype {
    constructor(value) {
        super();
        this.value = value;
    }
    
    clone() {
        return new ConcretePrototype2(this.value);
    }
    
    toString() {
        return `ConcretePrototype2 [value=${this.value}]`;
    }
}

// Deep copy with nested objects
class Address {
    constructor(street, city) {
        this.street = street;
        this.city = city;
    }
    
    clone() {
        return new Address(this.street, this.city);
    }
}

class Person {
    constructor(name, address) {
        this.name = name;
        this.address = address;
    }
    
    clone() {
        return new Person(this.name, this.address.clone());
    }
}

// Usage
const prototype1 = new ConcretePrototype1("Original");
const clone1 = prototype1.clone();

console.log(prototype1.toString());
console.log(clone1.toString());
console.log("Same object?", prototype1 === clone1);
```

## Implementation Considerations

1. **Using a prototype manager**: Keep a registry of available prototypes. Clients retrieve prototypes from registry and clone them.

2. **Implementing clone operation**: 
   - Shallow copy vs deep copy
   - Handle circular references
   - Consider using serialization for deep copying

3. **Initializing clones**: Some clients want to initialize internal state of cloned objects. Consider passing parameters to clone operation or providing separate initialize operation.

## Known Uses

- **Object.clone()**: Java's cloning mechanism
- **copy module**: Python's copy and deepcopy
- **Prototype-based languages**: JavaScript uses prototypal inheritance
- **Game development**: Cloning game objects
- **Document editors**: Copying graphical elements

## Related Patterns

- **Abstract Factory**: Can use Prototype to implement factories
- **Composite**: Often benefit from Prototype for copying complex structures
- **Decorator**: Often benefit from Prototype

## Real-World Example: Shape Cloning

```java
abstract class Shape implements Cloneable {
    private String id;
    protected String type;
    
    abstract void draw();
    
    public String getType() {
        return type;
    }
    
    public String getId() {
        return id;
    }
    
    public void setId(String id) {
        this.id = id;
    }
    
    @Override
    public Shape clone() {
        Shape clone = null;
        try {
            clone = (Shape) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

class Rectangle extends Shape {
    public Rectangle() {
        type = "Rectangle";
    }
    
    @Override
    void draw() {
        System.out.println("Drawing Rectangle");
    }
}

class Circle extends Shape {
    public Circle() {
        type = "Circle";
    }
    
    @Override
    void draw() {
        System.out.println("Drawing Circle");
    }
}

class Square extends Shape {
    public Square() {
        type = "Square";
    }
    
    @Override
    void draw() {
        System.out.println("Drawing Square");
    }
}

class ShapeCache {
    private static Map<String, Shape> shapeMap = new HashMap<>();
    
    public static Shape getShape(String shapeId) {
        Shape cachedShape = shapeMap.get(shapeId);
        return cachedShape.clone();
    }
    
    public static void loadCache() {
        Circle circle = new Circle();
        circle.setId("1");
        shapeMap.put(circle.getId(), circle);
        
        Square square = new Square();
        square.setId("2");
        shapeMap.put(square.getId(), square);
        
        Rectangle rectangle = new Rectangle();
        rectangle.setId("3");
        shapeMap.put(rectangle.getId(), rectangle);
    }
}

// Usage
public class PrototypeDemo {
    public static void main(String[] args) {
        ShapeCache.loadCache();
        
        Shape clonedShape1 = ShapeCache.getShape("1");
        System.out.println("Shape: " + clonedShape1.getType());
        
        Shape clonedShape2 = ShapeCache.getShape("2");
        System.out.println("Shape: " + clonedShape2.getType());
        
        Shape clonedShape3 = ShapeCache.getShape("3");
        System.out.println("Shape: " + clonedShape3.getType());
    }
}
```

## Real-World Example: Game Characters

```python
import copy

class GameCharacter:
    def __init__(self, name, health, mana, equipment):
        self.name = name
        self.health = health
        self.mana = mana
        self.equipment = equipment
    
    def clone(self):
        return copy.deepcopy(self)
    
    def __str__(self):
        return f"{self.name}: HP={self.health}, MP={self.mana}, Equipment={self.equipment}"

class CharacterRegistry:
    def __init__(self):
        self._characters = {}
    
    def register_character(self, key, character):
        self._characters[key] = character
    
    def get_character(self, key):
        character = self._characters.get(key)
        if character:
            return character.clone()
        return None

# Usage
registry = CharacterRegistry()

# Register prototypes
warrior = GameCharacter("Warrior", 100, 50, ["Sword", "Shield"])
mage = GameCharacter("Mage", 70, 150, ["Staff", "Robe"])

registry.register_character("warrior", warrior)
registry.register_character("mage", mage)

# Clone characters
player1 = registry.get_character("warrior")
player1.name = "Player1 Warrior"

player2 = registry.get_character("mage")
player2.name = "Player2 Mage"

print(player1)
print(player2)
print(warrior)  # Original unchanged
```

## When to Use

- Classes to instantiate are specified at runtime
- Avoid building class hierarchies of factories that parallel class hierarchy of products
- Instances of a class can have one of only a few different combinations of state
- Creating an object is expensive compared to cloning

## Current World Examples & Famous Products

### 1. **Document & Content Editors**
- **Microsoft Word**: Copy/paste functionality, duplicate slides, template cloning
- **Google Docs**: Duplicate document feature
- **Adobe Photoshop**: Layer duplication, style copying
- **Figma**: Component duplication, frame cloning
- **Canva**: Design template cloning

### 2. **Project Management Tools**
- **Jira**: Clone issue/epic functionality
- **Trello**: Copy board/card feature
- **Asana**: Duplicate task/project
- **Monday.com**: Duplicate board templates
- **Notion**: Duplicate page/database

### 3. **E-Commerce Platforms**
- **Shopify**: Product duplication for variants
- **Amazon Seller Central**: Clone product listings
- **eBay**: Copy listing feature
- **WooCommerce**: Duplicate products
- **Etsy**: Copy listing to create similar items

### 4. **Cloud & DevOps**
- **AWS**: AMI (Amazon Machine Image) cloning, snapshot copying
- **Docker**: Container image cloning
- **VMware**: Virtual machine cloning
- **Kubernetes**: Pod template replication
- **GitHub**: Repository forking and template repositories

### 5. **Database Systems**
- **MySQL**: Table cloning, database copying
- **MongoDB**: Document cloning
- **PostgreSQL**: Database template cloning
- **Redis**: Key copying
- **Cassandra**: Snapshot cloning

### 6. **Gaming**
- **Minecraft**: Structure copying with structure blocks
- **The Sims**: Clone Sim feature (in some versions)
- **Spore**: Creature cloning
- **Game engines**: Prefab instantiation in Unity, Blueprint cloning in Unreal
- **Roblox**: Model cloning in Studio

### 7. **Design & CAD Software**
- **AutoCAD**: Block copying, array commands
- **SketchUp**: Component copying
- **Blender**: Object duplication, linked duplicates
- **SolidWorks**: Part copying, configuration cloning
- **Revit**: Family duplication

### 8. **Email & Communication**
- **Gmail**: Email template cloning
- **Outlook**: Copy appointment/meeting
- **Slack**: Message template duplication
- **Microsoft Teams**: Channel template cloning
- **Discord**: Server template cloning

### 9. **CRM & Sales Tools**
- **Salesforce**: Clone record functionality
- **HubSpot**: Deal/contact duplication
- **Zoho CRM**: Record cloning
- **Pipedrive**: Deal duplication
- **Freshsales**: Lead/contact cloning

### 10. **Development & Testing**
- **Postman**: Request duplication
- **Selenium**: Test case cloning
- **Jenkins**: Job copying
- **Git**: Branch cloning, commit cherry-picking
- **Visual Studio**: Project template cloning

### 11. **Social Media**
- **Facebook**: Post sharing (creates copy with reference)
- **Instagram**: Repost functionality (third-party)
- **Pinterest**: Pin copying to different boards
- **LinkedIn**: Duplicate post drafts
- **TikTok**: Duet/Stitch (creates derivative content)

### 12. **Virtualization**
- **VirtualBox**: VM cloning (full and linked clones)
- **Hyper-V**: Virtual machine export/import
- **Parallels**: VM cloning
- **QEMU**: Disk image cloning
- **Proxmox**: Container/VM templates

### 13. **Mobile Apps**
- **iOS Shortcuts**: Shortcut duplication
- **Android**: App cloning features
- **Tasker**: Task/profile cloning
- **IFTTT**: Applet duplication
- **Zapier**: Zap template cloning

### 14. **Scientific & Research Tools**
- **MATLAB**: Object copying
- **R Studio**: Data frame cloning
- **Jupyter Notebooks**: Cell duplication
- **LabVIEW**: VI cloning
- **Origin**: Graph template cloning

### 15. **Programming Languages & Frameworks**
- **JavaScript**: Object.assign(), spread operator, structuredClone()
- **Python**: copy.copy(), copy.deepcopy()
- **Java**: Object.clone(), Cloneable interface
- **C++**: Copy constructors
- **C#**: ICloneable interface, MemberwiseClone()
