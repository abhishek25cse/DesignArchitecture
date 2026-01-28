# Proxy Pattern

## Category
Structural Pattern

## Intent
Provide a surrogate or placeholder for another object to control access to it.

## Also Known As
Surrogate

## Motivation
One reason for controlling access to an object is to defer the full cost of its creation and initialization until we actually need to use it. Consider a document editor that can embed graphical objects in a document. Some graphical objects, like large raster images, can be expensive to create. But opening a document should be fast, so we should avoid creating all the expensive objects at once when the document is opened. The solution is to use another object, an image proxy, that acts as a stand-in for the real image. The proxy acts just like the image and takes care of instantiating it when it's required.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐
│   Subject    │
├──────────────┤
│ + request()  │
└──────────────┘
      △
      │
      ├────────────────────┬────────────────────┐
      │                    │                    │
┌──────────────┐    ┌──────────────┐          │
│  RealSubject │    │    Proxy     │──────────┘
├──────────────┤    ├──────────────┤
│ + request()  │    │ + request()  │
└──────────────┘    └──────────────┘
```

## Participants

**Subject**
- Defines the common interface for RealSubject and Proxy

**RealSubject**
- Defines the real object that the proxy represents

**Proxy**
- Maintains a reference to the real subject
- Provides an interface identical to Subject
- Controls access to the real subject

## Types of Proxies

1. **Virtual Proxy**: Creates expensive objects on demand
2. **Protection Proxy**: Controls access to the original object
3. **Remote Proxy**: Represents object in different address space
4. **Smart Proxy**: Performs additional actions when object is accessed

## Consequences

**Benefits:**
- Controls access to object
- Can add additional functionality
- Lazy initialization
- Access control
- Logging and monitoring

**Drawbacks:**
- Response time might increase
- Additional complexity

## Implementation

### Virtual Proxy (Java)

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image: " + filename);
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;
    
    public ProxyImage(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}

public class ProxyDemo {
    public static void main(String[] args) {
        Image image = new ProxyImage("photo.jpg");
        
        image.display();
        System.out.println();
        image.display();
    }
}
```

### Protection Proxy (Python)

```python
from abc import ABC, abstractmethod

class Subject(ABC):
    @abstractmethod
    def request(self):
        pass

class RealSubject(Subject):
    def request(self):
        return "RealSubject: Handling request"

class ProtectionProxy(Subject):
    def __init__(self, real_subject: RealSubject, user_role: str):
        self._real_subject = real_subject
        self._user_role = user_role
    
    def request(self):
        if self._check_access():
            result = self._real_subject.request()
            self._log_access()
            return result
        else:
            return "ProtectionProxy: Access denied"
    
    def _check_access(self):
        print("ProtectionProxy: Checking access")
        return self._user_role == "admin"
    
    def _log_access(self):
        print("ProtectionProxy: Logging access")

admin = ProtectionProxy(RealSubject(), "admin")
print(admin.request())

user = ProtectionProxy(RealSubject(), "user")
print(user.request())
```

### JavaScript Implementation

```javascript
class Subject {
    request() {
        throw new Error("Must implement request");
    }
}

class RealSubject extends Subject {
    request() {
        return "RealSubject: Handling request";
    }
}

class Proxy extends Subject {
    constructor(realSubject) {
        super();
        this.realSubject = realSubject;
    }
    
    request() {
        if (this.checkAccess()) {
            const result = this.realSubject.request();
            this.logAccess();
            return result;
        }
        return "Proxy: Access denied";
    }
    
    checkAccess() {
        console.log("Proxy: Checking access");
        return true;
    }
    
    logAccess() {
        console.log("Proxy: Logging access");
    }
}

const realSubject = new RealSubject();
const proxy = new Proxy(realSubject);
console.log(proxy.request());
```

## Known Uses

- **Java RMI**: Remote proxies
- **Hibernate**: Lazy loading proxies
- **Spring AOP**: Method interceptors
- **ES6 Proxy**: Built-in proxy support

## Related Patterns

- **Adapter**: Changes interface, Proxy provides same interface
- **Decorator**: Adds responsibilities, Proxy controls access

## Real-World Examples

### Caching Proxy

```java
interface DataService {
    String getData(String key);
}

class RealDataService implements DataService {
    @Override
    public String getData(String key) {
        System.out.println("Fetching data from database for key: " + key);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Data for " + key;
    }
}

class CachingProxy implements DataService {
    private RealDataService realService;
    private Map<String, String> cache;
    
    public CachingProxy() {
        this.realService = new RealDataService();
        this.cache = new HashMap<>();
    }
    
    @Override
    public String getData(String key) {
        if (cache.containsKey(key)) {
            System.out.println("Returning cached data for key: " + key);
            return cache.get(key);
        }
        
        String data = realService.getData(key);
        cache.put(key, data);
        return data;
    }
}
```

## When to Use

- Need lazy initialization
- Need access control
- Need logging or monitoring
- Need remote object representation
- Need smart reference management
