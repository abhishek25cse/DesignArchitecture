# Flyweight Pattern

## Category
Structural Pattern

## Intent
Use sharing to support large numbers of fine-grained objects efficiently.

## Motivation
Some applications could benefit from using objects throughout their design, but a naive implementation would be prohibitively expensive. For example, most document editor implementations have text formatting and editing facilities that are modularized to some extent. Object-oriented document editors typically use objects to represent embedded elements like tables and figures. However, they usually stop short of using an object for each character in the document, even though doing so would promote flexibility at the finest levels in the application. Characters and embedded elements could then be treated uniformly with respect to how they are drawn and formatted. The application could be extended to support new character sets without disturbing other functionality. The application's object structure could literally mimic the document's physical structure. The Flyweight pattern describes how to share objects to allow their use at fine granularities without prohibitive cost.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────────┐
│FlyweightFactory  │
├──────────────────┤         ┌──────────────┐
│ + getFlyweight() │────────►│  Flyweight   │
└──────────────────┘         ├──────────────┤
                             │ + operation()│
                             └──────────────┘
                                    △
                                    │
                          ┌─────────┴─────────┐
                          │                   │
                  ┌───────────────┐  ┌────────────────┐
                  │ConcreteFlyweight│ │UnsharedFlyweight│
                  ├───────────────┤  ├────────────────┤
                  │ + operation() │  │ + operation()  │
                  └───────────────┘  └────────────────┘
```

## Participants

**Flyweight**
- Declares an interface through which flyweights can receive and act on extrinsic state

**ConcreteFlyweight**
- Implements the Flyweight interface and adds storage for intrinsic state
- Must be sharable

**UnsharedConcreteFlyweight**
- Not all Flyweight subclasses need to be shared

**FlyweightFactory**
- Creates and manages flyweight objects
- Ensures that flyweights are shared properly

**Client**
- Maintains a reference to flyweight(s)
- Computes or stores the extrinsic state of flyweight(s)

## Collaborations
- State that a flyweight needs to function must be characterized as either intrinsic or extrinsic
- Intrinsic state is stored in the ConcreteFlyweight object
- Extrinsic state is stored or computed by Client objects

## Consequences

**Benefits:**
- Reduces number of objects
- Reduces memory footprint
- May reduce computation costs

**Drawbacks:**
- Introduces run-time costs associated with transferring, finding, and computing extrinsic state
- Makes code more complex

## Implementation

### Basic Implementation (Java)

```java
import java.util.HashMap;
import java.util.Map;

interface Shape {
    void draw(int x, int y);
}

class Circle implements Shape {
    private String color;
    private int radius;
    
    public Circle(String color) {
        this.color = color;
        this.radius = 10;
    }
    
    @Override
    public void draw(int x, int y) {
        System.out.println("Drawing " + color + " circle at (" + x + ", " + y + ")");
    }
}

class ShapeFactory {
    private static final Map<String, Shape> circleMap = new HashMap<>();
    
    public static Shape getCircle(String color) {
        Circle circle = (Circle) circleMap.get(color);
        
        if (circle == null) {
            circle = new Circle(color);
            circleMap.put(color, circle);
            System.out.println("Creating circle of color: " + color);
        }
        return circle;
    }
}

public class FlyweightDemo {
    private static final String[] colors = {"Red", "Green", "Blue", "White", "Black"};
    
    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            Circle circle = (Circle) ShapeFactory.getCircle(getRandomColor());
            circle.draw(getRandomX(), getRandomY());
        }
    }
    
    private static String getRandomColor() {
        return colors[(int) (Math.random() * colors.length)];
    }
    
    private static int getRandomX() {
        return (int) (Math.random() * 100);
    }
    
    private static int getRandomY() {
        return (int) (Math.random() * 100);
    }
}
```

### Python Implementation

```python
class Flyweight:
    def __init__(self, shared_state):
        self._shared_state = shared_state
    
    def operation(self, unique_state):
        s = str(self._shared_state)
        u = str(unique_state)
        print(f"Flyweight: Shared ({s}) and unique ({u}) state")

class FlyweightFactory:
    _flyweights = {}
    
    def __init__(self, initial_flyweights):
        for state in initial_flyweights:
            self._flyweights[self._get_key(state)] = Flyweight(state)
    
    def _get_key(self, state):
        return "_".join(sorted(state))
    
    def get_flyweight(self, shared_state):
        key = self._get_key(shared_state)
        
        if key not in self._flyweights:
            print(f"FlyweightFactory: Creating new flyweight")
            self._flyweights[key] = Flyweight(shared_state)
        else:
            print(f"FlyweightFactory: Reusing existing flyweight")
        
        return self._flyweights[key]
    
    def list_flyweights(self):
        count = len(self._flyweights)
        print(f"FlyweightFactory: I have {count} flyweights:")
        for key in self._flyweights.keys():
            print(key)

factory = FlyweightFactory([
    ["Chevrolet", "Camaro2018", "pink"],
    ["Mercedes Benz", "C300", "black"],
    ["Mercedes Benz", "C500", "red"],
])

factory.list_flyweights()

flyweight = factory.get_flyweight(["Chevrolet", "Camaro2018", "pink"])
flyweight.operation(["CL234IR", "James Doe"])

flyweight = factory.get_flyweight(["BMW", "M5", "red"])
flyweight.operation(["CL234IR", "James Doe"])

factory.list_flyweights()
```

### JavaScript Implementation

```javascript
class Flyweight {
    constructor(sharedState) {
        this.sharedState = sharedState;
    }
    
    operation(uniqueState) {
        const s = JSON.stringify(this.sharedState);
        const u = JSON.stringify(uniqueState);
        console.log(`Flyweight: Shared (${s}) and unique (${u}) state`);
    }
}

class FlyweightFactory {
    constructor(initialFlyweights) {
        this.flyweights = {};
        for (const state of initialFlyweights) {
            this.flyweights[this.getKey(state)] = new Flyweight(state);
        }
    }
    
    getKey(state) {
        return state.sort().join("_");
    }
    
    getFlyweight(sharedState) {
        const key = this.getKey(sharedState);
        
        if (!(key in this.flyweights)) {
            console.log("FlyweightFactory: Creating new flyweight");
            this.flyweights[key] = new Flyweight(sharedState);
        } else {
            console.log("FlyweightFactory: Reusing existing flyweight");
        }
        
        return this.flyweights[key];
    }
    
    listFlyweights() {
        const count = Object.keys(this.flyweights).length;
        console.log(`FlyweightFactory: I have ${count} flyweights:`);
        for (const key in this.flyweights) {
            console.log(key);
        }
    }
}

const factory = new FlyweightFactory([
    ["Chevrolet", "Camaro2018", "pink"],
    ["Mercedes Benz", "C300", "black"],
    ["Mercedes Benz", "C500", "red"],
]);

factory.listFlyweights();

let flyweight = factory.getFlyweight(["Chevrolet", "Camaro2018", "pink"]);
flyweight.operation(["CL234IR", "James Doe"]);

flyweight = factory.getFlyweight(["BMW", "M5", "red"]);
flyweight.operation(["CL234IR", "James Doe"]);

factory.listFlyweights();
```

## Known Uses

- **Java String Pool**: Strings are shared
- **Integer caching**: Java caches Integer objects from -128 to 127
- **Game development**: Sharing textures, sprites
- **Text editors**: Sharing character objects

## Related Patterns

- **Composite**: Often combined with Flyweight to implement shared leaf nodes
- **State and Strategy**: Often implemented as Flyweights

## Real-World Example: Text Editor

```java
class Character {
    private char symbol;
    private String font;
    private int size;
    
    public Character(char symbol, String font, int size) {
        this.symbol = symbol;
        this.font = font;
        this.size = size;
    }
    
    public void display(int row, int column) {
        System.out.println("Character '" + symbol + "' at (" + row + ", " + column + 
                         ") with font " + font + " size " + size);
    }
}

class CharacterFactory {
    private Map<String, Character> characters = new HashMap<>();
    
    public Character getCharacter(char symbol, String font, int size) {
        String key = symbol + font + size;
        
        if (!characters.containsKey(key)) {
            characters.put(key, new Character(symbol, font, size));
            System.out.println("Creating new character: " + symbol);
        }
        
        return characters.get(key);
    }
    
    public int getTotalCharacters() {
        return characters.size();
    }
}
```

## When to Use

- Application uses large number of objects
- Storage costs are high because of sheer quantity of objects
- Most object state can be made extrinsic
- Many groups of objects may be replaced by relatively few shared objects
- Application doesn't depend on object identity

## Current World Examples & Famous Products

### 1. **Text Editors & Word Processors**
- **Microsoft Word**: Character objects shared across document
- **Google Docs**: Glyph sharing for text rendering
- **VS Code**: Token sharing for syntax highlighting
- **Sublime Text**: Shared character representations
- **Notepad++**: Text buffer optimization

### 2. **Web Browsers**
- **Chrome**: String interning, shared CSS styles
- **Firefox**: Shared font glyphs
- **Safari**: WebKit string deduplication
- **Edge**: Shared DOM node styles
- **Opera**: Resource caching and sharing

### 3. **Gaming**
- **Minecraft**: Block type sharing (millions of blocks, few types)
- **Fortnite**: Shared textures and models
- **World of Warcraft**: NPC model sharing
- **Unity**: Prefab instantiation with shared data
- **Unreal Engine**: Static mesh instancing

### 4. **Graphics & Design**
- **Photoshop**: Brush tip sharing
- **Illustrator**: Symbol libraries (shared graphics)
- **Figma**: Component instances sharing properties
- **Sketch**: Symbol overrides with shared base
- **Blender**: Linked duplicates

### 5. **Database Systems**
- **Oracle**: Shared pool for SQL statements
- **MySQL**: Query cache sharing
- **PostgreSQL**: Shared buffer cache
- **MongoDB**: Connection pooling
- **Redis**: String interning

### 6. **Java & JVM**
- **String Pool**: Shared string literals
- **Integer Cache**: Cached Integer objects (-128 to 127)
- **Enum**: Single instance per enum constant
- **Class metadata**: Shared across instances
- **Bytecode**: Shared method bytecode

### 7. **Mobile Apps**
- **Instagram**: Image thumbnail caching
- **Facebook**: News feed item recycling
- **Twitter/X**: Tweet template sharing
- **WhatsApp**: Message bubble reuse
- **Spotify**: Album art caching

### 8. **Map Applications**
- **Google Maps**: Tile caching and reuse
- **Apple Maps**: Map tile sharing
- **Waze**: Icon and marker sharing
- **OpenStreetMap**: Tile server caching
- **Mapbox**: Vector tile sharing

### 9. **E-Commerce**
- **Amazon**: Product image caching
- **eBay**: Listing template sharing
- **Shopify**: Theme asset sharing
- **Etsy**: Product photo optimization
- **AliExpress**: Image CDN caching

### 10. **Social Media**
- **Instagram**: Filter presets sharing
- **TikTok**: Effect templates
- **Snapchat**: Lens sharing
- **Pinterest**: Pin image caching
- **LinkedIn**: Profile template sharing

### 11. **Cloud Services**
- **AWS Lambda**: Container reuse
- **Docker**: Layer sharing between images
- **Kubernetes**: ConfigMap and Secret sharing
- **Azure Functions**: Execution context sharing
- **Google Cloud Run**: Container instance pooling

### 12. **Office Suites**
- **Microsoft Office**: Font glyph sharing
- **Google Workspace**: Template sharing
- **LibreOffice**: Style template sharing
- **Apple iWork**: Theme element sharing
- **WPS Office**: Resource sharing

### 13. **Streaming Services**
- **Netflix**: Thumbnail caching
- **YouTube**: Video chunk caching
- **Spotify**: Audio chunk sharing
- **Twitch**: Emote caching
- **Disney+**: Asset preloading

### 14. **Development Tools**
- **Git**: Object deduplication
- **npm**: Package caching
- **Docker**: Image layer caching
- **Webpack**: Module caching
- **Maven**: Dependency caching

### 15. **Operating Systems**
- **Windows**: DLL sharing between processes
- **Linux**: Shared libraries (.so files)
- **macOS**: Framework sharing
- **Android**: Shared system resources
- **iOS**: Shared frameworks and libraries
