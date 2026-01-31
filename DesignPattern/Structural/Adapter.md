# Adapter Pattern

## Category
Structural Pattern

## Intent
Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

## Also Known As
Wrapper

## Motivation
Sometimes a toolkit class that's designed for reuse isn't reusable only because its interface doesn't match the domain-specific interface an application requires. Consider a drawing editor that lets users draw and arrange graphical elements. The drawing editor's key abstraction is the graphical object, which has an editable shape and can draw itself. The interface for graphical objects is defined by an abstract class called Shape. The editor defines a subclass of Shape for each kind of graphical object: LineShape for lines, PolygonShape for polygons, and so forth. But a TextShape subclass that can display and edit text is considerably more difficult to implement, since even basic text editing involves complicated screen update and buffer management. Meanwhile, an off-the-shelf user interface toolkit might already provide a sophisticated TextView class for displaying and editing text. Ideally we'd like to reuse TextView to implement TextShape, but the toolkit wasn't designed with Shape classes in mind. So we can't use TextView and Shape objects interchangeably. We could change the TextView class so that it conforms to the Shape interface, but that isn't an option unless we have the toolkit's source code. Even if we did, it wouldn't make sense to change TextView; the toolkit shouldn't have to adopt domain-specific interfaces just to make one application work. Instead, we could define TextShape so that it adapts the TextView interface to Shape's. We can do this in one of two ways: (1) by inheriting Shape's interface and TextView's implementation or (2) by composing a TextView instance within a TextShape and implementing TextShape in terms of TextView's interface. These two approaches correspond to the class and object versions of the Adapter pattern.

## Structure

### Class Adapter (using multiple inheritance)

```
┌────────────┐
│   Client   │
├────────────┤
│            │───────►┌──────────────┐
└────────────┘        │    Target    │
                      ├──────────────┤
                      │ + request()  │
                      └──────────────┘
                             △
                             │
                      ┌──────┴───────┐
                      │   Adapter    │
                      ├──────────────┤
                      │ + request()  │
                      └──────────────┘
                             △
                             │
                      ┌──────────────┐
                      │   Adaptee    │
                      ├──────────────┤
                      │+ specificReq()│
                      └──────────────┘
```

### Object Adapter (using composition)

```
┌────────────┐
│   Client   │
├────────────┤
│            │───────►┌──────────────┐
└────────────┘        │    Target    │
                      ├──────────────┤
                      │ + request()  │
                      └──────────────┘
                             △
                             │
                      ┌──────┴───────┐
                      │   Adapter    │──────►┌──────────────┐
                      ├──────────────┤       │   Adaptee    │
                      │ + request()  │       ├──────────────┤
                      └──────────────┘       │+ specificReq()│
                                             └──────────────┘
```

## Participants

**Target**
- Defines the domain-specific interface that Client uses

**Client**
- Collaborates with objects conforming to the Target interface

**Adaptee**
- Defines an existing interface that needs adapting

**Adapter**
- Adapts the interface of Adaptee to the Target interface

## Collaborations
- Clients call operations on an Adapter instance. In turn, the adapter calls Adaptee operations that carry out the request

## Consequences

**Class Adapter:**
- Adapts Adaptee to Target by committing to a concrete Adaptee class
- Lets Adapter override some of Adaptee's behavior
- Introduces only one object, no additional pointer indirection needed

**Object Adapter:**
- Lets a single Adapter work with many Adaptees
- Makes it harder to override Adaptee behavior

**Benefits:**
- Single Responsibility Principle: Separate interface conversion from business logic
- Open/Closed Principle: Introduce new adapters without breaking existing code

**Drawbacks:**
- Overall complexity increases (new interfaces and classes)

## Implementation

### Object Adapter (Java)

```java
// Target interface
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee
class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
    
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

// Adapter
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedMediaPlayer;
    
    public MediaAdapter(String audioType) {
        advancedMediaPlayer = new AdvancedMediaPlayer();
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedMediaPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMediaPlayer.playMp4(fileName);
        }
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        } else if (audioType.equalsIgnoreCase("vlc") || 
                   audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media type: " + audioType);
        }
    }
}

// Usage
public class AdapterDemo {
    public static void main(String[] args) {
        AudioPlayer audioPlayer = new AudioPlayer();
        
        audioPlayer.play("mp3", "song.mp3");
        audioPlayer.play("mp4", "video.mp4");
        audioPlayer.play("vlc", "movie.vlc");
        audioPlayer.play("avi", "film.avi");
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Target(ABC):
    @abstractmethod
    def request(self):
        pass

class Adaptee:
    def specific_request(self):
        return "Adaptee's specific request"

class Adapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee
    
    def request(self):
        return f"Adapter: (TRANSLATED) {self._adaptee.specific_request()}"

# Usage
def client_code(target: Target):
    print(target.request())

adaptee = Adaptee()
print(f"Adaptee: {adaptee.specific_request()}")

adapter = Adapter(adaptee)
client_code(adapter)
```

### JavaScript Implementation

```javascript
class Target {
    request() {
        return "Target: Default behavior";
    }
}

class Adaptee {
    specificRequest() {
        return "Adaptee's specific request";
    }
}

class Adapter extends Target {
    constructor(adaptee) {
        super();
        this.adaptee = adaptee;
    }
    
    request() {
        return `Adapter: (TRANSLATED) ${this.adaptee.specificRequest()}`;
    }
}

// Usage
function clientCode(target) {
    console.log(target.request());
}

const adaptee = new Adaptee();
console.log(`Adaptee: ${adaptee.specificRequest()}`);

const adapter = new Adapter(adaptee);
clientCode(adapter);
```

## Implementation Considerations

1. **How much adapting does Adapter do?**: Ranges from simple interface conversion to supporting entirely different set of operations

2. **Pluggable adapters**: Make class reusable by building interface adaptation into the class

3. **Using two-way adapters**: Provide transparency by adapting in both directions

## Known Uses

- **Java Collections**: Arrays.asList() adapts arrays to List interface
- **InputStreamReader**: Adapts InputStream to Reader
- **JDBC-ODBC Bridge**: Adapts JDBC to ODBC
- **Legacy system integration**: Adapting old interfaces to new systems

## Related Patterns

- **Bridge**: Similar structure but different intent (Bridge separates interface from implementation)
- **Decorator**: Enhances object without changing interface
- **Proxy**: Provides same interface, doesn't change it

## Real-World Example: Payment Gateway

```java
// Target interface
interface PaymentProcessor {
    void processPayment(double amount);
}

// Adaptee - Third-party payment service
class StripePayment {
    public void makePayment(double dollars) {
        System.out.println("Processing $" + dollars + " via Stripe");
    }
}

class PayPalPayment {
    public void sendPayment(double amount) {
        System.out.println("Processing $" + amount + " via PayPal");
    }
}

// Adapters
class StripeAdapter implements PaymentProcessor {
    private StripePayment stripePayment;
    
    public StripeAdapter(StripePayment stripePayment) {
        this.stripePayment = stripePayment;
    }
    
    @Override
    public void processPayment(double amount) {
        stripePayment.makePayment(amount);
    }
}

class PayPalAdapter implements PaymentProcessor {
    private PayPalPayment payPalPayment;
    
    public PayPalAdapter(PayPalPayment payPalPayment) {
        this.payPalPayment = payPalPayment;
    }
    
    @Override
    public void processPayment(double amount) {
        payPalPayment.sendPayment(amount);
    }
}

// Client
class ShoppingCart {
    private PaymentProcessor paymentProcessor;
    
    public void setPaymentProcessor(PaymentProcessor processor) {
        this.paymentProcessor = processor;
    }
    
    public void checkout(double amount) {
        paymentProcessor.processPayment(amount);
    }
}

// Usage
public class PaymentDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        // Use Stripe
        cart.setPaymentProcessor(new StripeAdapter(new StripePayment()));
        cart.checkout(100.0);
        
        // Use PayPal
        cart.setPaymentProcessor(new PayPalAdapter(new PayPalPayment()));
        cart.checkout(200.0);
    }
}
```

## Real-World Example: Temperature Sensor

```python
class FahrenheitSensor:
    def get_temperature(self):
        return 98.6

class CelsiusInterface:
    def get_celsius_temperature(self):
        pass

class TemperatureAdapter(CelsiusInterface):
    def __init__(self, fahrenheit_sensor):
        self.fahrenheit_sensor = fahrenheit_sensor
    
    def get_celsius_temperature(self):
        fahrenheit = self.fahrenheit_sensor.get_temperature()
        celsius = (fahrenheit - 32) * 5/9
        return round(celsius, 2)

# Usage
sensor = FahrenheitSensor()
adapter = TemperatureAdapter(sensor)

print(f"Temperature in Fahrenheit: {sensor.get_temperature()}°F")
print(f"Temperature in Celsius: {adapter.get_celsius_temperature()}°C")
```

## When to Use

- Want to use an existing class with incompatible interface
- Want to create a reusable class that cooperates with unrelated or unforeseen classes
- Need to use several existing subclasses but impractical to adapt their interface by subclassing every one (object adapter)
- Legacy code integration
- Third-party library integration

## Current World Examples & Famous Products

### 1. **Payment Gateways**
- **Stripe**: Adapter for various payment methods (cards, wallets, bank transfers)
- **PayPal**: Adapts different payment sources to unified interface
- **Square**: Adapts in-person and online payment methods
- **Braintree**: Payment method adapter for multiple providers
- **Adyen**: Multi-currency and payment method adapter

### 2. **Cloud Storage Services**
- **Dropbox**: Adapts local file system to cloud storage
- **Google Drive**: File system adapter for cloud storage
- **OneDrive**: Windows file system integration adapter
- **Box**: Enterprise storage adapter
- **AWS S3**: Object storage adapter for various protocols

### 3. **Database Connectivity**
- **JDBC**: Adapts different database vendors to common interface
- **ODBC**: Universal database adapter
- **Hibernate**: ORM adapter for various databases
- **Entity Framework**: .NET database adapter
- **Sequelize**: Node.js ORM adapter for multiple databases

### 4. **Social Media Integration**
- **Zapier**: Adapts 5000+ apps to work together
- **IFTTT**: Service adapter for automation
- **Buffer**: Social media posting adapter (Facebook, Twitter, LinkedIn, Instagram)
- **Hootsuite**: Multi-platform social media adapter
- **Sprout Social**: Unified social media management adapter

### 5. **Authentication Services**
- **Auth0**: Adapts multiple authentication providers (Google, Facebook, SAML, LDAP)
- **Okta**: Identity provider adapter
- **Firebase Authentication**: Multi-provider auth adapter
- **AWS Cognito**: User pool adapter for various identity providers
- **Azure AD**: Enterprise authentication adapter

### 6. **Messaging & Communication**
- **Twilio**: SMS/voice adapter for multiple carriers
- **SendGrid**: Email delivery adapter
- **Slack integrations**: Adapts external services to Slack
- **Microsoft Teams connectors**: Third-party service adapters
- **Discord bots**: External service adapters

### 7. **Media & Content**
- **FFmpeg**: Media format adapter (converts between video/audio formats)
- **ImageMagick**: Image format adapter
- **Pandoc**: Document format converter/adapter
- **Handbrake**: Video transcoding adapter
- **CloudConvert**: Universal file format adapter

### 8. **Mobile Development**
- **React Native**: JavaScript to native mobile adapter
- **Cordova/PhoneGap**: Web to mobile app adapter
- **Ionic**: Web framework to mobile adapter
- **Flutter**: Dart to native adapter
- **Xamarin**: C# to native mobile adapter

### 9. **API Integration**
- **GraphQL**: REST API adapter to GraphQL
- **gRPC**: Protocol buffer adapter
- **Postman**: API testing adapter
- **Swagger/OpenAPI**: API documentation adapter
- **Kong**: API gateway adapter

### 10. **Hardware & IoT**
- **Arduino**: Hardware abstraction adapter
- **Raspberry Pi GPIO**: Hardware interface adapter
- **Home Assistant**: Smart home device adapter (supports 2000+ integrations)
- **SmartThings**: IoT device adapter
- **MQTT**: IoT messaging protocol adapter

### 11. **E-Commerce**
- **Shopify Apps**: Third-party service adapters
- **WooCommerce**: WordPress to e-commerce adapter
- **Magento extensions**: Service integration adapters
- **BigCommerce**: Multi-channel selling adapter
- **PrestaShop**: Module adapters

### 12. **Development Tools**
- **Babel**: Modern JavaScript to older version adapter
- **TypeScript**: TypeScript to JavaScript adapter
- **Webpack**: Module bundler adapter
- **Docker**: Application to container adapter
- **WSL (Windows Subsystem for Linux)**: Linux to Windows adapter

### 13. **Enterprise Software**
- **SAP**: Legacy system adapters
- **Salesforce**: External system integration adapters
- **Oracle Fusion**: Enterprise application adapters
- **Microsoft Dynamics**: Business application adapters
- **ServiceNow**: IT service management adapters

### 14. **Gaming**
- **Steam**: Game platform adapter
- **Epic Games Store**: Game launcher adapter
- **Xbox Game Pass**: Multi-platform gaming adapter
- **PlayStation Remote Play**: Console to mobile adapter
- **GeForce NOW**: Cloud gaming adapter

### 15. **Operating Systems**
- **Wine**: Windows applications on Linux adapter
- **Rosetta 2**: x86 to ARM adapter (Apple Silicon)
- **Windows Compatibility Mode**: Legacy app adapter
- **Android Runtime**: Android apps on Chrome OS
- **Proton**: Windows games on Linux adapter
