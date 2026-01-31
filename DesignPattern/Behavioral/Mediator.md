# Mediator Pattern

## Category
Behavioral Pattern

## Intent
Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.

## Motivation
Object-oriented design encourages the distribution of behavior among objects. Such distribution can result in an object structure with many connections between objects; in the worst case, every object ends up knowing about every other. Though partitioning a system into many objects generally enhances reusability, proliferating interconnections tend to reduce it again. Lots of interconnections make it less likely that an object can work without the support of others—the system acts as though it were monolithic. Moreover, it can be difficult to change the system's behavior in any significant way, since behavior is distributed among many objects. As a result, you may be forced to define many subclasses to customize the system's behavior. The Mediator pattern addresses these problems by introducing a mediator object to manage the interactions between a set of objects.

## Structure

```
┌──────────────┐
│   Mediator   │
├──────────────┤
│+ notify()    │
└──────────────┘
       △
       │
┌──────────────┐
│ConcreteMediator│──────┐
├──────────────┤       │
│+ notify()    │       │
└──────────────┘       │
       │               │
       │               │
       ▼               ▼
┌──────────────┐  ┌──────────────┐
│ Colleague1   │  │ Colleague2   │
├──────────────┤  ├──────────────┤
│              │  │              │
└──────────────┘  └──────────────┘
```

## Participants

**Mediator**
- Defines an interface for communicating with Colleague objects

**ConcreteMediator**
- Implements cooperative behavior by coordinating Colleague objects
- Knows and maintains its colleagues

**Colleague classes**
- Each Colleague class knows its Mediator object
- Each colleague communicates with its mediator whenever it would have otherwise communicated with another colleague

## Consequences

**Benefits:**
- Limits subclassing
- Decouples colleagues
- Simplifies object protocols
- Abstracts how objects cooperate
- Centralizes control

**Drawbacks:**
- Mediator can become complex
- Can become a god object

## Implementation

### Basic Implementation (Java)

```java
interface Mediator {
    void notify(Component sender, String event);
}

abstract class Component {
    protected Mediator mediator;
    
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }
}

class Button extends Component {
    public void click() {
        System.out.println("Button clicked");
        mediator.notify(this, "click");
    }
}

class TextBox extends Component {
    private String text = "";
    
    public void setText(String text) {
        this.text = text;
        System.out.println("TextBox text set to: " + text);
        mediator.notify(this, "textChanged");
    }
    
    public String getText() {
        return text;
    }
}

class Checkbox extends Component {
    private boolean checked = false;
    
    public void setChecked(boolean checked) {
        this.checked = checked;
        System.out.println("Checkbox checked: " + checked);
        mediator.notify(this, "checkChanged");
    }
    
    public boolean isChecked() {
        return checked;
    }
}

class DialogMediator implements Mediator {
    private Button button;
    private TextBox textBox;
    private Checkbox checkbox;
    
    public DialogMediator(Button button, TextBox textBox, Checkbox checkbox) {
        this.button = button;
        this.textBox = textBox;
        this.checkbox = checkbox;
        
        button.setMediator(this);
        textBox.setMediator(this);
        checkbox.setMediator(this);
    }
    
    @Override
    public void notify(Component sender, String event) {
        if (sender == button && event.equals("click")) {
            System.out.println("Mediator: Button clicked, processing...");
            if (checkbox.isChecked()) {
                System.out.println("Mediator: Checkbox is checked, submitting: " + textBox.getText());
            }
        } else if (sender == textBox && event.equals("textChanged")) {
            System.out.println("Mediator: Text changed");
        } else if (sender == checkbox && event.equals("checkChanged")) {
            System.out.println("Mediator: Checkbox state changed");
        }
    }
}

public class MediatorDemo {
    public static void main(String[] args) {
        Button button = new Button();
        TextBox textBox = new TextBox();
        Checkbox checkbox = new Checkbox();
        
        DialogMediator mediator = new DialogMediator(button, textBox, checkbox);
        
        textBox.setText("Hello World");
        checkbox.setChecked(true);
        button.click();
    }
}
```

### Python Implementation

```python
from abc import ABC

class Mediator(ABC):
    def notify(self, sender, event):
        pass

class Component:
    def __init__(self, mediator=None):
        self._mediator = mediator
    
    @property
    def mediator(self):
        return self._mediator
    
    @mediator.setter
    def mediator(self, mediator):
        self._mediator = mediator

class Button(Component):
    def click(self):
        print("Button clicked")
        self.mediator.notify(self, "click")

class TextBox(Component):
    def __init__(self, mediator=None):
        super().__init__(mediator)
        self._text = ""
    
    def set_text(self, text):
        self._text = text
        print(f"TextBox text set to: {text}")
        self.mediator.notify(self, "textChanged")
    
    @property
    def text(self):
        return self._text

class DialogMediator(Mediator):
    def __init__(self, button, textbox):
        self.button = button
        self.textbox = textbox
        
        button.mediator = self
        textbox.mediator = self
    
    def notify(self, sender, event):
        if sender == self.button and event == "click":
            print(f"Mediator: Button clicked, text is: {self.textbox.text}")
        elif sender == self.textbox and event == "textChanged":
            print("Mediator: Text changed")

# Usage
button = Button()
textbox = TextBox()
mediator = DialogMediator(button, textbox)

textbox.set_text("Hello World")
button.click()
```

### JavaScript Implementation

```javascript
class Mediator {
    notify(sender, event) {
        throw new Error("Must implement notify");
    }
}

class Component {
    constructor(mediator = null) {
        this.mediator = mediator;
    }
    
    setMediator(mediator) {
        this.mediator = mediator;
    }
}

class Button extends Component {
    click() {
        console.log("Button clicked");
        this.mediator.notify(this, "click");
    }
}

class TextBox extends Component {
    constructor(mediator = null) {
        super(mediator);
        this.text = "";
    }
    
    setText(text) {
        this.text = text;
        console.log(`TextBox text set to: ${text}`);
        this.mediator.notify(this, "textChanged");
    }
}

class DialogMediator extends Mediator {
    constructor(button, textbox) {
        super();
        this.button = button;
        this.textbox = textbox;
        
        button.setMediator(this);
        textbox.setMediator(this);
    }
    
    notify(sender, event) {
        if (sender === this.button && event === "click") {
            console.log(`Mediator: Button clicked, text is: ${this.textbox.text}`);
        } else if (sender === this.textbox && event === "textChanged") {
            console.log("Mediator: Text changed");
        }
    }
}

// Usage
const button = new Button();
const textbox = new TextBox();
const mediator = new DialogMediator(button, textbox);

textbox.setText("Hello World");
button.click();
```

## Known Uses

- **GUI frameworks**: Dialog boxes, form validation
- **Air traffic control**: Coordinating aircraft
- **Chat rooms**: Coordinating messages between users
- **MVC frameworks**: Controller acts as mediator

## Related Patterns

- **Facade**: Abstracts subsystem, Mediator abstracts colleague interactions
- **Observer**: Colleagues can communicate using Observer pattern

## Real-World Example: Chat Room

```java
interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}

abstract class User {
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message);
}

class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }
    
    @Override
    public void send(String message) {
        System.out.println(name + " sending: " + message);
        mediator.sendMessage(message, this);
    }
    
    @Override
    public void receive(String message) {
        System.out.println(name + " received: " + message);
    }
}

class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();
    
    @Override
    public void addUser(User user) {
        users.add(user);
    }
    
    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message);
            }
        }
    }
}

public class ChatDemo {
    public static void main(String[] args) {
        ChatMediator chatRoom = new ChatRoom();
        
        User user1 = new ChatUser(chatRoom, "Alice");
        User user2 = new ChatUser(chatRoom, "Bob");
        User user3 = new ChatUser(chatRoom, "Charlie");
        
        chatRoom.addUser(user1);
        chatRoom.addUser(user2);
        chatRoom.addUser(user3);
        
        user1.send("Hello everyone!");
    }
}
```

## When to Use

- Set of objects communicate in well-defined but complex ways
- Reusing an object is difficult because it refers to and communicates with many other objects
- Behavior distributed between several classes should be customizable without lots of subclassing

## Current World Examples & Famous Products

### 1. **Chat Applications**
- **Slack**: Channel mediator for user messages
- **Discord**: Server mediator for channel communication
- **Microsoft Teams**: Team mediator for collaboration
- **WhatsApp**: Group chat mediator
- **Telegram**: Group and channel mediator

### 2. **Air Traffic Control**
- **Flight management systems**: Coordinating aircraft communication
- **Airport tower**: Mediating runway assignments
- **Air traffic control**: Managing flight paths
- **Ground control**: Coordinating ground movements
- **Approach control**: Managing landing sequences

### 3. **Stock Trading Platforms**
- **NASDAQ**: Order matching mediator
- **NYSE**: Trade execution mediator
- **Robinhood**: Order routing mediator
- **E*TRADE**: Trade coordination
- **Interactive Brokers**: Multi-market mediator

### 4. **Smart Home Hubs**
- **Amazon Alexa**: Device coordination mediator
- **Google Home**: Smart home device mediator
- **Apple HomeKit**: Accessory communication mediator
- **SmartThings Hub**: IoT device mediator
- **Home Assistant**: Device integration mediator

### 5. **Game Multiplayer Systems**
- **Game servers**: Player interaction mediator
- **Matchmaking systems**: Player pairing mediator
- **Lobby systems**: Team formation mediator
- **Voice chat**: Communication mediator
- **Dedicated servers**: Game state mediator

### 6. **UI Frameworks**
- **React**: Component communication via props/context
- **Angular**: Service mediator for components
- **Vue.js**: Event bus mediator
- **Redux**: State management mediator
- **MobX**: Observable state mediator

### 7. **Message Brokers**
- **RabbitMQ**: Message routing mediator
- **Apache Kafka**: Event streaming mediator
- **Redis Pub/Sub**: Message distribution mediator
- **AWS SQS**: Queue mediator
- **Azure Service Bus**: Message mediator

### 8. **Workflow Orchestration**
- **Apache Airflow**: Task coordination mediator
- **Kubernetes**: Container orchestration mediator
- **AWS Step Functions**: Workflow mediator
- **Temporal**: Workflow execution mediator
- **Camunda**: Process orchestration mediator

### 9. **Social Media Platforms**
- **Facebook**: News feed mediator
- **Twitter/X**: Timeline mediator
- **Instagram**: Feed algorithm mediator
- **LinkedIn**: Connection mediator
- **TikTok**: For You page mediator

### 10. **E-Commerce Marketplaces**
- **Amazon**: Buyer-seller mediator
- **eBay**: Auction mediator
- **Etsy**: Shop-customer mediator
- **Alibaba**: B2B transaction mediator
- **Shopify**: Store-customer mediator

### 11. **Video Conferencing**
- **Zoom**: Meeting mediator
- **Microsoft Teams**: Call mediator
- **Google Meet**: Conference mediator
- **Webex**: Session mediator
- **Jitsi**: Video bridge mediator

### 12. **API Gateways**
- **Kong**: Service communication mediator
- **AWS API Gateway**: Request routing mediator
- **Azure API Management**: API mediator
- **Apigee**: API orchestration mediator
- **Tyk**: API coordination mediator

### 13. **Event-Driven Systems**
- **AWS EventBridge**: Event routing mediator
- **Azure Event Grid**: Event distribution mediator
- **Google Cloud Pub/Sub**: Message mediator
- **Apache Pulsar**: Event streaming mediator
- **NATS**: Messaging system mediator

### 14. **Collaboration Tools**
- **Notion**: Workspace collaboration mediator
- **Confluence**: Team documentation mediator
- **Miro**: Whiteboard collaboration mediator
- **Figma**: Design collaboration mediator
- **Google Workspace**: Document collaboration mediator

### 15. **MVC Frameworks**
- **Spring MVC**: Controller as mediator
- **Django**: View as mediator
- **Ruby on Rails**: Controller mediator
- **ASP.NET MVC**: Controller coordination
- **Laravel**: Controller mediator
