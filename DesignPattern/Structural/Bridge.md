# Bridge Pattern

## Category
Structural Pattern

## Intent
Decouple an abstraction from its implementation so that the two can vary independently.

## Also Known As
Handle/Body

## Motivation
When an abstraction can have one of several possible implementations, the usual way to accommodate them is to use inheritance. An abstract class defines the interface to the abstraction, and concrete subclasses implement it in different ways. But this approach isn't always flexible enough. Inheritance binds an implementation to the abstraction permanently, which makes it difficult to modify, extend, and reuse abstractions and implementations independently. Consider the implementation of a portable Window abstraction in a user interface toolkit. This abstraction should enable us to write applications that work on both X Window System and IBM's Presentation Manager (PM), for example. Using inheritance, we could define an abstract class Window and subclasses XWindow and PMWindow that implement the Window interface for the different platforms. But this approach has two drawbacks: (1) It's inconvenient to extend the Window abstraction to cover different kinds of windows or new platforms. (2) It makes client code platform-dependent. The Bridge pattern addresses these problems by putting the Window abstraction and its implementation in separate class hierarchies.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────────┐
│   Abstraction    │
├──────────────────┤         ┌──────────────────┐
│ + operation()    │────────►│  Implementor     │
└──────────────────┘         ├──────────────────┤
      △                      │ + operationImpl()│
      │                      └──────────────────┘
      │                             △
┌──────────────────┐                │
│RefinedAbstraction│         ┌──────┴──────┐
├──────────────────┤         │             │
│ + operation()    │  ┌──────────────┐ ┌──────────────┐
└──────────────────┘  │ConcreteImplA │ │ConcreteImplB │
                      ├──────────────┤ ├──────────────┤
                      │+ operationImpl│ │+ operationImpl│
                      └──────────────┘ └──────────────┘
```

## Participants

**Abstraction**
- Defines the abstraction's interface
- Maintains a reference to an object of type Implementor

**RefinedAbstraction**
- Extends the interface defined by Abstraction

**Implementor**
- Defines the interface for implementation classes

**ConcreteImplementor**
- Implements the Implementor interface and defines its concrete implementation

## Collaborations
- Abstraction forwards client requests to its Implementor object

## Consequences

**Benefits:**
- Decouples interface and implementation
- Improved extensibility
- Hiding implementation details from clients

**Drawbacks:**
- Increased complexity

## Implementation

### Basic Implementation (Java)

```java
interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int volume);
}

class TV implements Device {
    private int volume = 0;
    
    @Override
    public void turnOn() {
        System.out.println("TV: Turned on");
    }
    
    @Override
    public void turnOff() {
        System.out.println("TV: Turned off");
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("TV: Volume set to " + volume);
    }
}

class Radio implements Device {
    private int volume = 0;
    
    @Override
    public void turnOn() {
        System.out.println("Radio: Turned on");
    }
    
    @Override
    public void turnOff() {
        System.out.println("Radio: Turned off");
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("Radio: Volume set to " + volume);
    }
}

abstract class RemoteControl {
    protected Device device;
    
    public RemoteControl(Device device) {
        this.device = device;
    }
    
    public void togglePower() {
        System.out.println("RemoteControl: Toggle power");
    }
    
    public void volumeUp() {
        System.out.println("RemoteControl: Volume up");
    }
    
    public void volumeDown() {
        System.out.println("RemoteControl: Volume down");
    }
}

class BasicRemote extends RemoteControl {
    public BasicRemote(Device device) {
        super(device);
    }
    
    @Override
    public void togglePower() {
        System.out.println("BasicRemote: Toggle power");
        device.turnOn();
    }
}

class AdvancedRemote extends RemoteControl {
    public AdvancedRemote(Device device) {
        super(device);
    }
    
    public void mute() {
        System.out.println("AdvancedRemote: Mute");
        device.setVolume(0);
    }
}

public class BridgeDemo {
    public static void main(String[] args) {
        Device tv = new TV();
        RemoteControl remote = new BasicRemote(tv);
        remote.togglePower();
        
        Device radio = new Radio();
        AdvancedRemote advancedRemote = new AdvancedRemote(radio);
        advancedRemote.togglePower();
        advancedRemote.mute();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Implementor(ABC):
    @abstractmethod
    def operation_impl(self):
        pass

class ConcreteImplementorA(Implementor):
    def operation_impl(self):
        return "ConcreteImplementorA operation"

class ConcreteImplementorB(Implementor):
    def operation_impl(self):
        return "ConcreteImplementorB operation"

class Abstraction:
    def __init__(self, implementor: Implementor):
        self._implementor = implementor
    
    def operation(self):
        return f"Abstraction: {self._implementor.operation_impl()}"

class RefinedAbstraction(Abstraction):
    def operation(self):
        return f"RefinedAbstraction: {self._implementor.operation_impl()}"

impl_a = ConcreteImplementorA()
abstraction = Abstraction(impl_a)
print(abstraction.operation())

impl_b = ConcreteImplementorB()
refined = RefinedAbstraction(impl_b)
print(refined.operation())
```

### JavaScript Implementation

```javascript
class Implementor {
    operationImpl() {
        throw new Error("Must implement operationImpl");
    }
}

class ConcreteImplementorA extends Implementor {
    operationImpl() {
        return "ConcreteImplementorA operation";
    }
}

class ConcreteImplementorB extends Implementor {
    operationImpl() {
        return "ConcreteImplementorB operation";
    }
}

class Abstraction {
    constructor(implementor) {
        this.implementor = implementor;
    }
    
    operation() {
        return `Abstraction: ${this.implementor.operationImpl()}`;
    }
}

class RefinedAbstraction extends Abstraction {
    operation() {
        return `RefinedAbstraction: ${this.implementor.operationImpl()}`;
    }
}

const implA = new ConcreteImplementorA();
const abstraction = new Abstraction(implA);
console.log(abstraction.operation());

const implB = new ConcreteImplementorB();
const refined = new RefinedAbstraction(implB);
console.log(refined.operation());
```

## Known Uses

- **JDBC**: Separates database API from driver implementation
- **GUI frameworks**: Separating window abstractions from platform implementations
- **Graphics libraries**: Separating shapes from rendering

## Related Patterns

- **Abstract Factory**: Can create and configure a particular Bridge
- **Adapter**: Makes unrelated classes work together

## Real-World Example: Drawing API

```java
interface DrawingAPI {
    void drawCircle(double x, double y, double radius);
}

class DrawingAPI1 implements DrawingAPI {
    @Override
    public void drawCircle(double x, double y, double radius) {
        System.out.printf("API1: Circle at (%.2f, %.2f) radius %.2f%n", x, y, radius);
    }
}

class DrawingAPI2 implements DrawingAPI {
    @Override
    public void drawCircle(double x, double y, double radius) {
        System.out.printf("API2: Circle at (%.2f, %.2f) radius %.2f%n", x, y, radius);
    }
}

abstract class Shape {
    protected DrawingAPI drawingAPI;
    
    protected Shape(DrawingAPI drawingAPI) {
        this.drawingAPI = drawingAPI;
    }
    
    public abstract void draw();
    public abstract void resizeByPercentage(double pct);
}

class CircleShape extends Shape {
    private double x, y, radius;
    
    public CircleShape(double x, double y, double radius, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }
    
    @Override
    public void resizeByPercentage(double pct) {
        radius *= (1 + pct / 100);
    }
}
```

## When to Use

- Want to avoid permanent binding between abstraction and implementation
- Both abstractions and implementations should be extensible by subclassing
- Changes in implementation should have no impact on clients
- Want to share implementation among multiple objects

## Current World Examples & Famous Products

### 1. **Remote Controls & Devices**
- **Universal remotes**: Control different TV brands (Samsung, LG, Sony)
- **Smart home apps**: Control various device brands through one interface
- **Logitech Harmony**: Universal remote for multiple devices
- **Apple TV Remote**: Controls different streaming services
- **Roku Remote**: Works with various content providers

### 2. **Graphics & Rendering**
- **Unity**: Rendering abstraction for DirectX, OpenGL, Vulkan, Metal
- **Unreal Engine**: Platform-independent rendering
- **Qt**: GUI framework for Windows, macOS, Linux
- **Java Swing**: Platform-independent UI
- **Electron**: Web technologies on different OS platforms

### 3. **Database Drivers**
- **JDBC**: Database abstraction for MySQL, PostgreSQL, Oracle
- **ADO.NET**: .NET data access for various databases
- **SQLAlchemy**: Python ORM for multiple databases
- **Hibernate**: Java ORM with multiple database support
- **Prisma**: TypeScript ORM for various databases

### 4. **Payment Processing**
- **Stripe**: Payment abstraction for cards, wallets, bank transfers
- **PayPal**: Multiple payment method abstraction
- **Square**: Various payment type support
- **Braintree**: Payment gateway abstraction
- **Adyen**: Multi-payment method platform

### 5. **Cloud Storage**
- **Rclone**: Unified interface for 40+ cloud storage providers
- **MultCloud**: Manage multiple cloud services
- **CloudMounter**: Mount various cloud storages as local drives
- **Air Explorer**: Multi-cloud file manager
- **Cyberduck**: FTP/cloud storage client

### 6. **Messaging Platforms**
- **Pidgin**: Multi-protocol instant messaging (supports 15+ protocols)
- **Trillian**: Universal messaging client
- **Franz**: Messaging app aggregator
- **Rambox**: Multi-service messaging
- **Station**: Unified workspace for apps

### 7. **Media Players**
- **VLC**: Plays various formats using different codecs
- **Kodi**: Media center for different platforms
- **Plex**: Media server abstraction
- **MPC-HC**: Multiple renderer support
- **mpv**: Cross-platform media player

### 8. **Game Controllers**
- **Steam Input**: Controller abstraction for various game controllers
- **DS4Windows**: PlayStation controller on Windows
- **Xpadder**: Keyboard/mouse emulation for controllers
- **JoyToKey**: Controller to keyboard mapping
- **AntiMicro**: Gamepad to keyboard/mouse

### 9. **Development Frameworks**
- **React Native**: JavaScript abstraction for iOS/Android
- **Flutter**: Dart abstraction for multiple platforms
- **Xamarin**: C# abstraction for mobile platforms
- **.NET Core**: Cross-platform .NET
- **Cordova**: Web to native mobile bridge

### 10. **Notification Systems**
- **Pushover**: Notification abstraction for multiple devices
- **OneSignal**: Push notification for web, mobile, email
- **Firebase Cloud Messaging**: Multi-platform notifications
- **Airship**: Omnichannel messaging
- **Twilio**: Multi-channel communication

### 11. **Authentication**
- **Auth0**: Authentication abstraction for various providers
- **OAuth**: Authorization abstraction
- **SAML**: SSO abstraction
- **OpenID Connect**: Identity layer abstraction
- **Passport.js**: Authentication middleware abstraction

### 12. **Printing**
- **CUPS**: Unix printing abstraction for various printers
- **Windows Print Spooler**: Printer driver abstraction
- **Google Cloud Print**: Cloud printing abstraction
- **AirPrint**: iOS printing abstraction
- **Universal Print**: Microsoft cloud printing

### 13. **Video Conferencing**
- **Zoom**: Multiple device and platform support
- **Microsoft Teams**: Cross-platform communication
- **Google Meet**: Browser and app abstraction
- **Webex**: Multi-device conferencing
- **Jitsi**: Open-source video bridge

### 14. **IoT Platforms**
- **AWS IoT**: Device abstraction for various protocols
- **Azure IoT Hub**: Multi-protocol device support
- **Google Cloud IoT**: Device management abstraction
- **ThingSpeak**: IoT data platform
- **Particle**: IoT device abstraction

### 15. **Virtualization**
- **VMware**: Hardware abstraction for guest OS
- **VirtualBox**: Cross-platform virtualization
- **QEMU**: Machine emulator abstraction
- **Hyper-V**: Windows virtualization platform
- **KVM**: Linux kernel virtualization
