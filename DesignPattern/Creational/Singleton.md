# Singleton Pattern

## Category
Creational Pattern

## Intent
Ensure a class has only one instance and provide a global point of access to it.

## Motivation
Sometimes we need exactly one instance of a class. For example:
- A single database connection pool
- A configuration manager
- A logging service
- A print spooler

The Singleton pattern ensures that a class has only one instance and provides a way to access it globally. The class itself is responsible for keeping track of its sole instance.

## Structure

```
┌─────────────────────┐
│    Singleton        │
├─────────────────────┤
│ - instance: static  │
├─────────────────────┤
│ - Singleton()       │
│ + getInstance()     │
│ + operation()       │
└─────────────────────┘
```

## Participants

**Singleton**
- Defines an `getInstance()` operation that lets clients access its unique instance
- May be responsible for creating its own unique instance

## Collaborations
- Clients access a Singleton instance solely through the Singleton's `getInstance()` method

## Consequences

**Benefits:**
- Controlled access to sole instance
- Reduced namespace pollution
- Permits refinement of operations and representation
- Permits variable number of instances (can be extended to control number of instances)
- More flexible than class operations (static methods)

**Drawbacks:**
- Difficult to test (global state)
- Violates Single Responsibility Principle
- Can mask bad design (components know too much about each other)
- Requires special treatment in multithreaded environments
- Difficult to subclass

## Implementation

### Basic Implementation (Java)

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // Private constructor prevents instantiation
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public void operation() {
        // Business logic
    }
}
```

### Thread-Safe Implementation (Java)

```java
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### Eager Initialization (Java)

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
}
```

### Bill Pugh Singleton (Java)

```java
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

### Python Implementation

```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def operation(self):
        pass
```

### JavaScript Implementation

```javascript
class Singleton {
    constructor() {
        if (Singleton.instance) {
            return Singleton.instance;
        }
        Singleton.instance = this;
    }
    
    operation() {
        // Business logic
    }
}

// Or using closure
const Singleton = (function() {
    let instance;
    
    function createInstance() {
        return {
            operation: function() {
                // Business logic
            }
        };
    }
    
    return {
        getInstance: function() {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();
```

## Implementation Considerations

1. **Lazy vs Eager Initialization**: Decide whether to create instance at class load time or on first access
2. **Thread Safety**: Ensure thread-safe implementation in multithreaded environments
3. **Serialization**: Override `readResolve()` to prevent creating new instances during deserialization
4. **Reflection**: Prevent reflection attacks by throwing exception in constructor if instance exists
5. **Cloning**: Prevent cloning by not implementing Cloneable or throwing exception in `clone()`

## Known Uses

- **java.lang.Runtime**: Single runtime instance per JVM
- **java.awt.Desktop**: Desktop integration
- **Spring Framework**: Beans are singletons by default
- **Logger instances**: Log4j, SLF4J
- **Database connection pools**
- **Configuration managers**
- **Cache managers**

## Related Patterns

- **Abstract Factory**: Can be implemented as Singleton
- **Builder**: Can be implemented as Singleton
- **Prototype**: Can be implemented as Singleton
- **Facade**: Often implemented as Singleton

## Real-World Example

```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private Connection connection;
    
    private DatabaseConnection() {
        try {
            connection = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/mydb",
                "user",
                "password"
            );
        } catch (SQLException e) {
            throw new RuntimeException("Failed to create connection", e);
        }
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
    
    public Connection getConnection() {
        return connection;
    }
}
```

## Anti-Patterns to Avoid

1. **Overuse**: Not everything needs to be a Singleton
2. **Global State**: Can lead to hidden dependencies and tight coupling
3. **Testing Difficulties**: Hard to mock or replace in tests
4. **Concurrency Issues**: Improper thread-safe implementation

## Modern Alternatives

- **Dependency Injection**: Prefer DI containers to manage single instances
- **Monostate Pattern**: All instances share the same state
- **Service Locator**: Centralized registry for services

## Current World Examples & Famous Products

### 1. **Operating Systems**
- **Windows Task Manager**: Single instance ensures only one Task Manager runs at a time
- **macOS Dock**: Single instance managing all application shortcuts
- **Linux systemd**: Single init system managing all services

### 2. **Mobile Applications**
- **WhatsApp**: Single instance of message queue manager
- **Instagram**: Single instance of image cache manager
- **Spotify**: Single instance of audio player engine

### 3. **Web Browsers**
- **Google Chrome**: Single instance of download manager per profile
- **Firefox**: Single instance of bookmark manager
- **Safari**: Single instance of password manager

### 4. **Development Tools**
- **Visual Studio Code**: Single instance of extension manager
- **IntelliJ IDEA**: Single instance of project indexer
- **Git**: Single instance of repository lock manager

### 5. **Cloud Services**
- **AWS SDK**: Single instance of credential provider
- **Azure**: Single instance of authentication manager
- **Google Cloud**: Single instance of API client configuration

### 6. **Gaming**
- **Unity Engine**: Single instance of AudioListener
- **Unreal Engine**: Single instance of GameInstance
- **Steam**: Single instance of overlay manager

### 7. **Enterprise Software**
- **SAP**: Single instance of session manager
- **Salesforce**: Single instance of API rate limiter
- **Oracle Database**: Single instance of connection pool manager

### 8. **Popular Frameworks**
- **Spring Framework**: ApplicationContext (singleton scope by default)
- **Django**: Settings object (single configuration instance)
- **Express.js**: App instance in Node.js applications
- **React**: Redux store (single source of truth)
- **Angular**: Services with providedIn: 'root' (singleton across app)

### 9. **Real-World Business Applications**
- **E-commerce platforms**: Shopping cart manager, payment gateway connector
- **Banking systems**: Transaction logger, audit trail manager
- **Healthcare systems**: Patient record accessor, HIPAA compliance checker
- **Logistics**: GPS tracker manager, route optimizer

### 10. **IoT and Embedded Systems**
- **Smart Home Devices**: Device controller, network manager
- **Automotive**: Engine control unit (ECU) manager
- **Wearables**: Sensor data aggregator, battery monitor
