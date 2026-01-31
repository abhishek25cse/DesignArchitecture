# Chain of Responsibility Pattern

## Category
Behavioral Pattern

## Intent
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

## Motivation
Consider a context-sensitive help facility for a graphical user interface. The user can obtain help information on any part of the interface just by clicking on it. The help that's provided depends on the part of the interface that's selected and its context. If no specific help information exists for that part of the interface, then the help system should display a more general help message about the immediate context. The problem here is that the object that ultimately provides the help isn't known explicitly to the object that initiates the help request. What we need is a way to decouple the button that initiates the help request from the objects that might provide help information. The Chain of Responsibility pattern defines a chain of objects that have the chance to handle a request.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐
│   Handler    │◄────────┐
├──────────────┤         │
│+ handleReq() │         │
│+ setNext()   │         │
└──────────────┘         │
      △                  │
      │                  │
      ├──────────────────┼──────────────────┐
      │                  │                  │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ConcreteH1    │  │ConcreteH2    │  │ConcreteH3    │
├──────────────┤  ├──────────────┤  ├──────────────┤
│+ handleReq() │  │+ handleReq() │  │+ handleReq() │
└──────────────┘  └──────────────┘  └──────────────┘
```

## Participants

**Handler**
- Defines an interface for handling requests
- Implements the successor link (optional)

**ConcreteHandler**
- Handles requests it is responsible for
- Can access its successor
- If can handle request, does so; otherwise forwards to successor

**Client**
- Initiates the request to a ConcreteHandler object on the chain

## Consequences

**Benefits:**
- Reduced coupling
- Added flexibility in assigning responsibilities
- Receipt isn't guaranteed

**Drawbacks:**
- No guarantee of handling
- Can be hard to observe runtime characteristics

## Implementation

### Basic Implementation (Java)

```java
abstract class Handler {
    protected Handler nextHandler;
    
    public void setNext(Handler handler) {
        this.nextHandler = handler;
    }
    
    public abstract void handleRequest(String request);
}

class ConcreteHandler1 extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("Type1")) {
            System.out.println("ConcreteHandler1 handled request: " + request);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}

class ConcreteHandler2 extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("Type2")) {
            System.out.println("ConcreteHandler2 handled request: " + request);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}

class ConcreteHandler3 extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("Type3")) {
            System.out.println("ConcreteHandler3 handled request: " + request);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}

public class ChainDemo {
    public static void main(String[] args) {
        Handler h1 = new ConcreteHandler1();
        Handler h2 = new ConcreteHandler2();
        Handler h3 = new ConcreteHandler3();
        
        h1.setNext(h2);
        h2.setNext(h3);
        
        h1.handleRequest("Type1");
        h1.handleRequest("Type2");
        h1.handleRequest("Type3");
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Handler(ABC):
    def __init__(self):
        self._next_handler = None
    
    def set_next(self, handler):
        self._next_handler = handler
        return handler
    
    @abstractmethod
    def handle(self, request):
        if self._next_handler:
            return self._next_handler.handle(request)
        return None

class ConcreteHandler1(Handler):
    def handle(self, request):
        if request == "Type1":
            return f"ConcreteHandler1 handled {request}"
        else:
            return super().handle(request)

class ConcreteHandler2(Handler):
    def handle(self, request):
        if request == "Type2":
            return f"ConcreteHandler2 handled {request}"
        else:
            return super().handle(request)

class ConcreteHandler3(Handler):
    def handle(self, request):
        if request == "Type3":
            return f"ConcreteHandler3 handled {request}"
        else:
            return super().handle(request)

# Usage
h1 = ConcreteHandler1()
h2 = ConcreteHandler2()
h3 = ConcreteHandler3()

h1.set_next(h2).set_next(h3)

print(h1.handle("Type1"))
print(h1.handle("Type2"))
print(h1.handle("Type3"))
```

### JavaScript Implementation

```javascript
class Handler {
    setNext(handler) {
        this.nextHandler = handler;
        return handler;
    }
    
    handle(request) {
        if (this.nextHandler) {
            return this.nextHandler.handle(request);
        }
        return null;
    }
}

class ConcreteHandler1 extends Handler {
    handle(request) {
        if (request === "Type1") {
            return `ConcreteHandler1 handled ${request}`;
        }
        return super.handle(request);
    }
}

class ConcreteHandler2 extends Handler {
    handle(request) {
        if (request === "Type2") {
            return `ConcreteHandler2 handled ${request}`;
        }
        return super.handle(request);
    }
}

class ConcreteHandler3 extends Handler {
    handle(request) {
        if (request === "Type3") {
            return `ConcreteHandler3 handled ${request}`;
        }
        return super.handle(request);
    }
}

// Usage
const h1 = new ConcreteHandler1();
const h2 = new ConcreteHandler2();
const h3 = new ConcreteHandler3();

h1.setNext(h2).setNext(h3);

console.log(h1.handle("Type1"));
console.log(h1.handle("Type2"));
console.log(h1.handle("Type3"));
```

## Known Uses

- **Event handling**: GUI event propagation
- **Logging frameworks**: Different log levels
- **Servlet filters**: Request processing chain
- **Middleware**: Express.js middleware chain

## Related Patterns

- **Composite**: Often used together
- **Command**: Can use Chain of Responsibility to determine which command handles request

## Real-World Example: Support System

```java
abstract class SupportHandler {
    protected SupportHandler nextHandler;
    protected String level;
    
    public void setNext(SupportHandler handler) {
        this.nextHandler = handler;
    }
    
    public abstract void handleRequest(SupportTicket ticket);
}

class SupportTicket {
    private String issue;
    private int priority;
    
    public SupportTicket(String issue, int priority) {
        this.issue = issue;
        this.priority = priority;
    }
    
    public String getIssue() { return issue; }
    public int getPriority() { return priority; }
}

class Level1Support extends SupportHandler {
    public Level1Support() {
        this.level = "Level 1";
    }
    
    @Override
    public void handleRequest(SupportTicket ticket) {
        if (ticket.getPriority() <= 1) {
            System.out.println(level + " handled: " + ticket.getIssue());
        } else if (nextHandler != null) {
            nextHandler.handleRequest(ticket);
        }
    }
}

class Level2Support extends SupportHandler {
    public Level2Support() {
        this.level = "Level 2";
    }
    
    @Override
    public void handleRequest(SupportTicket ticket) {
        if (ticket.getPriority() <= 2) {
            System.out.println(level + " handled: " + ticket.getIssue());
        } else if (nextHandler != null) {
            nextHandler.handleRequest(ticket);
        }
    }
}

class Level3Support extends SupportHandler {
    public Level3Support() {
        this.level = "Level 3";
    }
    
    @Override
    public void handleRequest(SupportTicket ticket) {
        System.out.println(level + " handled: " + ticket.getIssue());
    }
}

// Usage
public class SupportDemo {
    public static void main(String[] args) {
        SupportHandler level1 = new Level1Support();
        SupportHandler level2 = new Level2Support();
        SupportHandler level3 = new Level3Support();
        
        level1.setNext(level2);
        level2.setNext(level3);
        
        level1.handleRequest(new SupportTicket("Password reset", 1));
        level1.handleRequest(new SupportTicket("Software bug", 2));
        level1.handleRequest(new SupportTicket("System crash", 3));
    }
}
```

## When to Use

- More than one object may handle a request
- Handler isn't known a priori
- Set of objects that can handle request should be specified dynamically
- Want to issue request to one of several objects without specifying receiver explicitly

## Current World Examples & Famous Products

### 1. **Customer Support Systems**
- **Zendesk**: Ticket escalation (L1 → L2 → L3 support)
- **Freshdesk**: Support tier routing
- **Intercom**: Message routing to appropriate team
- **Help Scout**: Conversation assignment chain
- **ServiceNow**: Incident escalation workflow

### 2. **Logging Frameworks**
- **Log4j**: Logger hierarchy (DEBUG → INFO → WARN → ERROR)
- **SLF4J**: Logging level chain
- **Winston**: Transport chain in Node.js
- **Serilog**: Sink pipeline
- **NLog**: Target chain

### 3. **Web Server Middleware**
- **Express.js**: Middleware chain (auth → validation → business logic)
- **ASP.NET**: HTTP pipeline middleware
- **Django**: Middleware stack
- **Spring**: Filter chain
- **Koa**: Middleware cascade

### 4. **Event Handling**
- **DOM Events**: Event bubbling (child → parent → document)
- **Android**: Touch event handling chain
- **iOS**: Responder chain
- **Windows**: Message routing
- **Qt**: Event propagation

### 5. **Approval Workflows**
- **Jira**: Approval chain for requests
- **SAP**: Purchase order approval hierarchy
- **Salesforce**: Approval processes
- **Microsoft Dynamics**: Workflow approval
- **Oracle**: Approval routing

### 6. **Security & Authentication**
- **Spring Security**: Filter security chain
- **Passport.js**: Strategy chain
- **OAuth**: Authorization chain
- **SAML**: Authentication chain
- **JWT**: Token validation chain

### 7. **Payment Processing**
- **Stripe**: Payment method fallback chain
- **PayPal**: Payment source priority
- **Square**: Payment processor chain
- **Braintree**: Payment method cascade
- **Adyen**: Payment routing chain

### 8. **Email Filtering**
- **Gmail**: Spam filter chain
- **Outlook**: Rule processing chain
- **SpamAssassin**: Filter chain
- **MailScanner**: Virus and spam checking chain
- **Postfix**: Mail filter chain

### 9. **Exception Handling**
- **Java**: Try-catch chain
- **Python**: Exception handler chain
- **JavaScript**: Error boundary chain
- **C++**: Exception propagation
- **C#**: Exception filter chain

### 10. **Network Protocols**
- **TCP/IP**: Protocol stack (Application → Transport → Network → Link)
- **HTTP**: Proxy chain
- **Firewall**: Rule chain processing
- **Load balancers**: Server selection chain
- **CDN**: Cache hierarchy

### 11. **Content Moderation**
- **Facebook**: Content review escalation
- **YouTube**: Video moderation chain
- **Reddit**: Moderator hierarchy
- **Discord**: Auto-mod → moderator → admin
- **Twitch**: Chat moderation chain

### 12. **Request Routing**
- **AWS API Gateway**: Request routing chain
- **Nginx**: Location matching chain
- **Apache**: .htaccess rule chain
- **Kong**: Plugin chain
- **Envoy**: Filter chain

### 13. **Validation Pipelines**
- **Form validation**: Field validators chain
- **Data validation**: Rule chain processing
- **Schema validation**: Validator chain
- **Input sanitization**: Filter chain
- **Business rule validation**: Rule engine chain

### 14. **Game Development**
- **Input handling**: Controller → game object chain
- **Collision detection**: Collision handler chain
- **AI decision making**: Behavior tree chain
- **Event systems**: Event handler chain
- **Command processing**: Command chain

### 15. **CI/CD Pipelines**
- **Jenkins**: Build step chain
- **GitLab CI**: Job dependency chain
- **GitHub Actions**: Workflow step chain
- **CircleCI**: Pipeline chain
- **Travis CI**: Build phase chain
