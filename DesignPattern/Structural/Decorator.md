# Decorator Pattern

## Category
Structural Pattern

## Intent
Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

## Also Known As
Wrapper

## Motivation
Sometimes we want to add responsibilities to individual objects, not to an entire class. A graphical user interface toolkit should let you add properties like borders or behaviors like scrolling to any user interface component. One way to add responsibilities is with inheritance. Inheriting a border from another class puts a border around every subclass instance. This is inflexible, however, because the choice of border is made statically. A client can't control how and when to decorate the component with a border. A more flexible approach is to enclose the component in another object that adds the border. The enclosing object is called a decorator. The decorator conforms to the interface of the component it decorates so that its presence is transparent to the component's clients. The decorator forwards requests to the component and may perform additional actions before or after forwarding.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐
│  Component   │
├──────────────┤
│ + operation()│
└──────────────┘
      △
      │
      ├────────────────────┬────────────────────┐
      │                    │                    │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ConcreteComp  │    │  Decorator   │    │              │
├──────────────┤    ├──────────────┤    │              │
│ + operation()│    │ + operation()│    │              │
└──────────────┘    └──────────────┘    │              │
                           △             │              │
                           │             │              │
                    ┌──────┴──────┐      │              │
                    │             │      │              │
            ┌───────────────┐ ┌───────────────┐        │
            │ConcreteDecorA │ │ConcreteDecorB │        │
            ├───────────────┤ ├───────────────┤        │
            │ + operation() │ │ + operation() │        │
            │ + addedBehav()│ └───────────────┘        │
            └───────────────┘                          │
                    │                                   │
                    └───────────────────────────────────┘
```

## Participants

**Component**
- Defines the interface for objects that can have responsibilities added to them dynamically

**ConcreteComponent**
- Defines an object to which additional responsibilities can be attached

**Decorator**
- Maintains a reference to a Component object and defines an interface that conforms to Component's interface

**ConcreteDecorator**
- Adds responsibilities to the component

## Collaborations
- Decorator forwards requests to its Component object. It may optionally perform additional operations before and after forwarding the request

## Consequences

**Benefits:**
- More flexibility than static inheritance
- Avoids feature-laden classes high up in the hierarchy
- Decorator and its component aren't identical (decorator is transparent enclosure)
- Lots of little objects (can be hard to learn and debug)

**Drawbacks:**
- Decorator and component aren't identical from object identity point of view
- Lots of small objects that differ only in how they're interconnected

## Implementation

### Basic Implementation (Java)

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete Component
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.0;
    }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.2;
    }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Whip";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.7;
    }
}

// Usage
public class DecoratorDemo {
    public static void main(String[] args) {
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        
        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
        
        coffee = new WhipDecorator(coffee);
        System.out.println(coffee.getDescription() + " $" + coffee.getCost());
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Component(ABC):
    @abstractmethod
    def operation(self):
        pass

class ConcreteComponent(Component):
    def operation(self):
        return "ConcreteComponent"

class Decorator(Component):
    def __init__(self, component: Component):
        self._component = component
    
    @property
    def component(self):
        return self._component
    
    def operation(self):
        return self._component.operation()

class ConcreteDecoratorA(Decorator):
    def operation(self):
        return f"ConcreteDecoratorA({self.component.operation()})"

class ConcreteDecoratorB(Decorator):
    def operation(self):
        return f"ConcreteDecoratorB({self.component.operation()})"

# Usage
def client_code(component: Component):
    print(f"RESULT: {component.operation()}")

simple = ConcreteComponent()
print("Client: Simple component:")
client_code(simple)

decorator1 = ConcreteDecoratorA(simple)
decorator2 = ConcreteDecoratorB(decorator1)
print("\nClient: Decorated component:")
client_code(decorator2)
```

### JavaScript Implementation

```javascript
class Component {
    operation() {
        throw new Error("Must implement operation");
    }
}

class ConcreteComponent extends Component {
    operation() {
        return "ConcreteComponent";
    }
}

class Decorator extends Component {
    constructor(component) {
        super();
        this.component = component;
    }
    
    operation() {
        return this.component.operation();
    }
}

class ConcreteDecoratorA extends Decorator {
    operation() {
        return `ConcreteDecoratorA(${this.component.operation()})`;
    }
}

class ConcreteDecoratorB extends Decorator {
    operation() {
        return `ConcreteDecoratorB(${this.component.operation()})`;
    }
}

// Usage
function clientCode(component) {
    console.log(`RESULT: ${component.operation()}`);
}

const simple = new ConcreteComponent();
console.log("Client: Simple component:");
clientCode(simple);

const decorator1 = new ConcreteDecoratorA(simple);
const decorator2 = new ConcreteDecoratorB(decorator1);
console.log("\nClient: Decorated component:");
clientCode(decorator2);
```

## Implementation Considerations

1. **Interface conformance**: Decorator object's interface must conform to interface of component it decorates

2. **Omitting abstract Decorator class**: No need for abstract Decorator class when you only need to add one responsibility

3. **Keeping Component classes lightweight**: Focus on defining interface, not storing data

4. **Changing the skin vs changing the guts**: Decorator changes skin, Strategy changes guts

## Known Uses

- **Java I/O Streams**: BufferedInputStream, DataInputStream wrap InputStream
- **Java Collections**: Collections.synchronizedList(), Collections.unmodifiableList()
- **GUI toolkits**: Adding scrollbars, borders to windows
- **Middleware**: Adding logging, authentication, caching

## Related Patterns

- **Adapter**: Changes interface, Decorator enhances responsibilities
- **Composite**: Decorator can be viewed as degenerate composite with only one component
- **Strategy**: Decorator changes skin, Strategy changes guts

## Real-World Example: Text Formatting

```java
interface Text {
    String getContent();
}

class PlainText implements Text {
    private String content;
    
    public PlainText(String content) {
        this.content = content;
    }
    
    @Override
    public String getContent() {
        return content;
    }
}

abstract class TextDecorator implements Text {
    protected Text text;
    
    public TextDecorator(Text text) {
        this.text = text;
    }
    
    @Override
    public String getContent() {
        return text.getContent();
    }
}

class BoldDecorator extends TextDecorator {
    public BoldDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String getContent() {
        return "<b>" + text.getContent() + "</b>";
    }
}

class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String getContent() {
        return "<i>" + text.getContent() + "</i>";
    }
}

class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String getContent() {
        return "<u>" + text.getContent() + "</u>";
    }
}

// Usage
public class TextFormattingDemo {
    public static void main(String[] args) {
        Text text = new PlainText("Hello World");
        System.out.println(text.getContent());
        
        text = new BoldDecorator(text);
        System.out.println(text.getContent());
        
        text = new ItalicDecorator(text);
        System.out.println(text.getContent());
        
        text = new UnderlineDecorator(text);
        System.out.println(text.getContent());
    }
}
```

## Real-World Example: Data Source

```python
class DataSource:
    def write_data(self, data):
        pass
    
    def read_data(self):
        pass

class FileDataSource(DataSource):
    def __init__(self, filename):
        self.filename = filename
    
    def write_data(self, data):
        print(f"Writing data to file: {data}")
    
    def read_data(self):
        return "File data"

class DataSourceDecorator(DataSource):
    def __init__(self, source):
        self._wrappee = source
    
    def write_data(self, data):
        self._wrappee.write_data(data)
    
    def read_data(self):
        return self._wrappee.read_data()

class EncryptionDecorator(DataSourceDecorator):
    def write_data(self, data):
        encrypted = self._encrypt(data)
        super().write_data(encrypted)
    
    def read_data(self):
        data = super().read_data()
        return self._decrypt(data)
    
    def _encrypt(self, data):
        return f"Encrypted({data})"
    
    def _decrypt(self, data):
        return f"Decrypted({data})"

class CompressionDecorator(DataSourceDecorator):
    def write_data(self, data):
        compressed = self._compress(data)
        super().write_data(compressed)
    
    def read_data(self):
        data = super().read_data()
        return self._decompress(data)
    
    def _compress(self, data):
        return f"Compressed({data})"
    
    def _decompress(self, data):
        return f"Decompressed({data})"

# Usage
source = FileDataSource("data.txt")
source = EncryptionDecorator(source)
source = CompressionDecorator(source)

source.write_data("Important data")
print(source.read_data())
```

## When to Use

- Add responsibilities to individual objects dynamically and transparently
- Responsibilities can be withdrawn
- Extension by subclassing is impractical (large number of independent extensions possible)
- Want to add functionality without modifying existing code

## Current World Examples & Famous Products

### 1. **Text Editors & Word Processors**
- **Microsoft Word**: Text formatting (bold, italic, underline, color, highlight)
- **Google Docs**: Layered text styling
- **Notion**: Block decorators (callouts, toggles, columns)
- **VS Code**: Syntax highlighting, linting decorators
- **Sublime Text**: Multiple cursor decorators

### 2. **Image Editing**
- **Photoshop**: Layer effects (drop shadow, glow, bevel, overlay)
- **Instagram**: Photo filters stacked on images
- **Snapchat**: Face filters and lenses
- **TikTok**: Video effects and filters
- **GIMP**: Layer decorators

### 3. **Web Browsers**
- **Chrome Extensions**: Ad blockers, password managers, grammar checkers
- **Firefox Add-ons**: Privacy tools, theme decorators
- **Safari Extensions**: Content blockers, reading mode
- **Edge**: Vertical tabs, collections decorators
- **Brave**: Built-in ad blocking and privacy decorators

### 4. **Streaming Services**
- **Netflix**: Subtitles, audio descriptions, playback speed decorators
- **YouTube**: Closed captions, quality settings, speed controls
- **Spotify**: Equalizer, crossfade, gapless playback
- **Twitch**: Chat overlays, emote decorators
- **Disney+**: GroupWatch, download decorators

### 5. **E-Commerce**
- **Amazon**: Gift wrapping, express shipping, insurance decorators
- **Shopify**: Product options (engraving, customization, gift messages)
- **Etsy**: Personalization options
- **eBay**: Listing upgrades (bold, highlight, featured)
- **Uber Eats**: Extra toppings, special instructions

### 6. **Coffee Shops & Food**
- **Starbucks**: Drink modifiers (extra shot, soy milk, whipped cream, flavor pumps)
- **Chipotle**: Burrito add-ons (guacamole, queso, extra protein)
- **Subway**: Sandwich toppings and extras
- **Pizza chains**: Toppings, crust types, sauce options
- **Smoothie King**: Enhancers and supplements

### 7. **Mobile Apps**
- **iOS**: App badges, notifications, widgets
- **Android**: Notification decorators, app shortcuts
- **WhatsApp**: Message reactions, formatting (bold, italic, strikethrough)
- **Telegram**: Message decorators (spoilers, code blocks)
- **Discord**: Message embeds, reactions

### 8. **Cloud Services**
- **AWS**: Security groups, encryption, monitoring decorators
- **Azure**: Resource tags, locks, policies
- **Google Cloud**: Labels, encryption, logging
- **Heroku**: Add-ons (databases, monitoring, caching)
- **DigitalOcean**: Droplet backups, monitoring, firewalls

### 9. **Development Frameworks**
- **Express.js**: Middleware decorators (authentication, logging, compression)
- **Spring**: Annotations (@Transactional, @Cacheable, @Async)
- **Django**: Decorators (@login_required, @cache_page)
- **Angular**: Decorators (@Component, @Injectable)
- **Python**: Function decorators (@staticmethod, @property, @lru_cache)

### 10. **Gaming**
- **Minecraft**: Enchantments on items
- **RPGs**: Equipment modifiers (gems, runes, enchantments)
- **Fortnite**: Skin wraps and back bling
- **League of Legends**: Champion skins and chromas
- **World of Warcraft**: Item enhancements and transmogrification

### 11. **Social Media**
- **Instagram**: Story stickers, filters, music, polls
- **Facebook**: Post reactions, comment decorators
- **Twitter/X**: Tweet formatting, media attachments
- **LinkedIn**: Post formatting, document attachments
- **Pinterest**: Pin descriptions, board covers

### 12. **Email Clients**
- **Gmail**: Labels, stars, importance markers, snooze
- **Outlook**: Categories, flags, follow-up reminders
- **Apple Mail**: VIP markers, flags
- **Thunderbird**: Tags, stars
- **ProtonMail**: Encryption, expiration decorators

### 13. **Project Management**
- **Jira**: Issue labels, flags, watchers
- **Trello**: Card labels, due dates, checklists, attachments
- **Asana**: Task tags, custom fields, dependencies
- **Monday.com**: Column decorators, automations
- **Notion**: Page properties, relations, rollups

### 14. **Transportation**
- **Uber**: Ride options (UberX + car seat, priority pickup, quiet mode)
- **Lyft**: Ride preferences (extra stops, preferred arrival time)
- **Airlines**: Seat selection, extra legroom, priority boarding, baggage
- **Hotels**: Room upgrades, late checkout, breakfast
- **Car rentals**: GPS, insurance, additional driver

### 15. **Security & Privacy**
- **VPN services**: Kill switch, split tunneling, ad blocking
- **Password managers**: 2FA, breach monitoring, secure sharing
- **Antivirus**: Real-time protection, firewall, VPN
- **Encryption tools**: Compression, password protection
- **Firewalls**: Intrusion detection, logging, alerts
