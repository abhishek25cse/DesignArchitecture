# Abstract Factory Pattern

## Category
Creational Pattern

## Intent
Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

## Also Known As
Kit

## Motivation
Consider a user interface toolkit that supports multiple look-and-feel standards (Windows, macOS, Linux). Different look-and-feels define different appearances and behaviors for widgets like buttons, scrollbars, and windows. To be portable, an application should not hardcode widgets for a specific look-and-feel. The Abstract Factory pattern solves this by defining an abstract factory interface for creating each basic widget, with concrete factories for each look-and-feel standard.

## Structure

```
┌──────────────────────┐
│  AbstractFactory     │
├──────────────────────┤
│ + createProductA()   │
│ + createProductB()   │
└──────────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───────────────────┐  ┌───────────────────┐
│ ConcreteFactory1  │  │ ConcreteFactory2  │
├───────────────────┤  ├───────────────────┤
│ + createProductA()│  │ + createProductA()│
│ + createProductB()│  │ + createProductB()│
└───────────────────┘  └───────────────────┘
    │         │            │         │
    │         │            │         │
    ▼         ▼            ▼         ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ProductA1│ │ProductB1│ │ProductA2│ │ProductB2│
└─────────┘ └─────────┘ └─────────┘ └─────────┘
```

## Participants

**AbstractFactory**
- Declares an interface for operations that create abstract product objects

**ConcreteFactory**
- Implements the operations to create concrete product objects

**AbstractProduct**
- Declares an interface for a type of product object

**ConcreteProduct**
- Defines a product object to be created by the corresponding concrete factory
- Implements the AbstractProduct interface

**Client**
- Uses only interfaces declared by AbstractFactory and AbstractProduct classes

## Collaborations
- A single instance of a ConcreteFactory class is created at runtime
- This concrete factory creates product objects having a particular implementation
- To create different product objects, clients use a different concrete factory

## Consequences

**Benefits:**
- Isolates concrete classes
- Makes exchanging product families easy
- Promotes consistency among products

**Drawbacks:**
- Supporting new kinds of products is difficult (requires extending factory interface)

## Implementation

### Basic Implementation (Java)

```java
// Abstract Products
interface Button {
    void paint();
}

interface Checkbox {
    void paint();
}

// Concrete Products - Windows
class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering Windows button");
    }
}

class WindowsCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete Products - macOS
class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering macOS button");
    }
}

class MacCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Rendering macOS checkbox");
    }
}

// Abstract Factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factories
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client
class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void paint() {
        button.paint();
        checkbox.paint();
    }
}

// Usage
public class Demo {
    private static Application configureApplication() {
        Application app;
        GUIFactory factory;
        
        String osName = System.getProperty("os.name").toLowerCase();
        if (osName.contains("mac")) {
            factory = new MacFactory();
        } else {
            factory = new WindowsFactory();
        }
        
        app = new Application(factory);
        return app;
    }
    
    public static void main(String[] args) {
        Application app = configureApplication();
        app.paint();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Button(ABC):
    @abstractmethod
    def paint(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def paint(self):
        pass

class WindowsButton(Button):
    def paint(self):
        return "Rendering Windows button"

class WindowsCheckbox(Checkbox):
    def paint(self):
        return "Rendering Windows checkbox"

class MacButton(Button):
    def paint(self):
        return "Rendering macOS button"

class MacCheckbox(Checkbox):
    def paint(self):
        return "Rendering macOS checkbox"

class GUIFactory(ABC):
    @abstractmethod
    def create_button(self):
        pass
    
    @abstractmethod
    def create_checkbox(self):
        pass

class WindowsFactory(GUIFactory):
    def create_button(self):
        return WindowsButton()
    
    def create_checkbox(self):
        return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self):
        return MacButton()
    
    def create_checkbox(self):
        return MacCheckbox()

class Application:
    def __init__(self, factory: GUIFactory):
        self.button = factory.create_button()
        self.checkbox = factory.create_checkbox()
    
    def paint(self):
        print(self.button.paint())
        print(self.checkbox.paint())

# Usage
def main():
    import platform
    
    if platform.system() == "Darwin":
        factory = MacFactory()
    else:
        factory = WindowsFactory()
    
    app = Application(factory)
    app.paint()

if __name__ == "__main__":
    main()
```

### JavaScript Implementation

```javascript
class Button {
    paint() {
        throw new Error("Must implement paint");
    }
}

class Checkbox {
    paint() {
        throw new Error("Must implement paint");
    }
}

class WindowsButton extends Button {
    paint() {
        return "Rendering Windows button";
    }
}

class WindowsCheckbox extends Checkbox {
    paint() {
        return "Rendering Windows checkbox";
    }
}

class MacButton extends Button {
    paint() {
        return "Rendering macOS button";
    }
}

class MacCheckbox extends Checkbox {
    paint() {
        return "Rendering macOS checkbox";
    }
}

class GUIFactory {
    createButton() {
        throw new Error("Must implement createButton");
    }
    
    createCheckbox() {
        throw new Error("Must implement createCheckbox");
    }
}

class WindowsFactory extends GUIFactory {
    createButton() {
        return new WindowsButton();
    }
    
    createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory extends GUIFactory {
    createButton() {
        return new MacButton();
    }
    
    createCheckbox() {
        return new MacCheckbox();
    }
}

class Application {
    constructor(factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }
    
    paint() {
        console.log(this.button.paint());
        console.log(this.checkbox.paint());
    }
}

// Usage
const os = process.platform;
const factory = os === 'darwin' ? new MacFactory() : new WindowsFactory();
const app = new Application(factory);
app.paint();
```

## Implementation Considerations

1. **Factories as singletons**: Typically only one instance of ConcreteFactory per product family is needed

2. **Creating the products**: AbstractFactory only declares interface for creating products. ConcreteProduct subclasses actually create them. Most common way is to define a factory method for each product.

3. **Defining extensible factories**: Add parameter to operations that create objects (type of object to be created)

## Known Uses

- **Java AWT/Swing**: Different look-and-feel implementations
- **Database drivers**: JDBC provides abstract factory for different database vendors
- **UI frameworks**: Creating platform-specific UI components
- **Game development**: Creating different themes or levels with consistent object families

## Related Patterns

- **Factory Method**: Abstract Factory is often implemented with Factory Methods
- **Singleton**: Concrete factories are often Singletons
- **Prototype**: Concrete factories can use Prototype to create products

## Real-World Example: Furniture Shop

```java
// Abstract Products
interface Chair {
    void sitOn();
}

interface Sofa {
    void lieOn();
}

interface CoffeeTable {
    void putCup();
}

// Victorian Style
class VictorianChair implements Chair {
    public void sitOn() {
        System.out.println("Sitting on Victorian chair");
    }
}

class VictorianSofa implements Sofa {
    public void lieOn() {
        System.out.println("Lying on Victorian sofa");
    }
}

class VictorianCoffeeTable implements CoffeeTable {
    public void putCup() {
        System.out.println("Putting cup on Victorian coffee table");
    }
}

// Modern Style
class ModernChair implements Chair {
    public void sitOn() {
        System.out.println("Sitting on Modern chair");
    }
}

class ModernSofa implements Sofa {
    public void lieOn() {
        System.out.println("Lying on Modern sofa");
    }
}

class ModernCoffeeTable implements CoffeeTable {
    public void putCup() {
        System.out.println("Putting cup on Modern coffee table");
    }
}

// Abstract Factory
interface FurnitureFactory {
    Chair createChair();
    Sofa createSofa();
    CoffeeTable createCoffeeTable();
}

// Concrete Factories
class VictorianFurnitureFactory implements FurnitureFactory {
    public Chair createChair() {
        return new VictorianChair();
    }
    
    public Sofa createSofa() {
        return new VictorianSofa();
    }
    
    public CoffeeTable createCoffeeTable() {
        return new VictorianCoffeeTable();
    }
}

class ModernFurnitureFactory implements FurnitureFactory {
    public Chair createChair() {
        return new ModernChair();
    }
    
    public Sofa createSofa() {
        return new ModernSofa();
    }
    
    public CoffeeTable createCoffeeTable() {
        return new ModernCoffeeTable();
    }
}

// Client
class FurnitureShop {
    private Chair chair;
    private Sofa sofa;
    private CoffeeTable coffeeTable;
    
    public FurnitureShop(FurnitureFactory factory) {
        chair = factory.createChair();
        sofa = factory.createSofa();
        coffeeTable = factory.createCoffeeTable();
    }
    
    public void furnish() {
        chair.sitOn();
        sofa.lieOn();
        coffeeTable.putCup();
    }
}
```

## When to Use

- System should be independent of how its products are created, composed, and represented
- System should be configured with one of multiple families of products
- Family of related product objects is designed to be used together
- You want to provide a class library of products and reveal only their interfaces, not implementations

## Current World Examples & Famous Products

### 1. **UI Frameworks & Design Systems**
- **Material Design (Google)**: Factory creates buttons, cards, dialogs for Android/Web/iOS
- **Fluent Design (Microsoft)**: Creates consistent UI components across Windows, Office, Xbox
- **Human Interface Guidelines (Apple)**: Factory for iOS, macOS, watchOS, tvOS components
- **Ant Design**: Creates consistent React components for enterprise applications
- **Bootstrap**: Theme factory creates themed components (buttons, forms, navbars)

### 2. **Cross-Platform Development**
- **React Native**: Platform-specific component factory (iOS vs Android components)
- **Flutter**: Widget factory creates Material or Cupertino widgets
- **Xamarin**: UI factory creates native controls for iOS, Android, Windows
- **Electron**: Window factory creates platform-specific windows and menus
- **Unity**: Platform factory creates platform-specific input, graphics, audio systems

### 3. **Database Drivers**
- **JDBC**: Driver factory creates connections, statements, result sets for different databases
- **ODBC**: Factory for database-specific implementations (MySQL, PostgreSQL, SQL Server)
- **Hibernate**: Dialect factory creates SQL for different database vendors
- **Entity Framework**: Provider factory for SQL Server, MySQL, PostgreSQL, SQLite

### 4. **Cloud Service Providers**
- **Terraform**: Provider factory creates resources for AWS, Azure, GCP
- **Kubernetes**: Cloud provider factory for different cloud implementations
- **Docker**: Container runtime factory for different operating systems
- **Ansible**: Module factory for different infrastructure providers

### 5. **Gaming Engines**
- **Unity**: Platform factory creates input, rendering, audio for different platforms
- **Unreal Engine**: Target platform factory (PC, Console, Mobile, VR)
- **Godot**: Export preset factory for different platforms
- **CryEngine**: Renderer factory for DirectX, Vulkan, Metal

### 6. **Office Suites**
- **Microsoft Office**: Document factory creates Word, Excel, PowerPoint documents
- **Google Workspace**: Factory creates Docs, Sheets, Slides with consistent styling
- **LibreOffice**: Document type factory for Writer, Calc, Impress
- **Apple iWork**: Factory creates Pages, Numbers, Keynote documents

### 7. **E-Commerce Platforms**
- **Magento**: Theme factory creates consistent storefront components
- **WooCommerce**: Payment gateway factory creates payment forms and processors
- **Shopify**: Theme engine factory creates product cards, checkout forms
- **BigCommerce**: Widget factory for different store components

### 8. **Automotive Software**
- **Tesla**: UI factory creates consistent controls across Model S, 3, X, Y
- **Android Automotive**: Factory creates car-specific UI components
- **CarPlay (Apple)**: App factory creates consistent car interfaces
- **AUTOSAR**: Software component factory for different ECUs

### 9. **Smart Home Systems**
- **Google Home**: Device factory creates controls for lights, thermostats, cameras
- **Amazon Alexa**: Skill factory creates consistent voice interfaces
- **Apple HomeKit**: Accessory factory for different device types
- **Samsung SmartThings**: Device handler factory for various smart devices

### 10. **Content Management Systems**
- **WordPress**: Theme factory creates headers, footers, sidebars, content areas
- **Drupal**: Block factory creates consistent content blocks
- **Joomla**: Module factory for different component types
- **Adobe Experience Manager**: Component factory for web content

### 11. **Financial Software**
- **Bloomberg Terminal**: Widget factory creates charts, news feeds, trading panels
- **Trading platforms**: Order type factory (market, limit, stop, trailing stop)
- **Banking apps**: Transaction factory for different account types
- **QuickBooks**: Report factory creates consistent financial reports

### 12. **Healthcare Systems**
- **Epic Systems**: Form factory creates patient intake, clinical notes, prescriptions
- **Cerner**: UI factory for different clinical workflows
- **MEDITECH**: Screen factory for various healthcare modules
- **Allscripts**: Document factory for medical records
