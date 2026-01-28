# State Pattern

## Category
Behavioral Pattern

## Intent
Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

## Also Known As
Objects for States

## Motivation
Consider a class TCPConnection that represents a network connection. A TCPConnection object can be in one of several different states: Established, Listening, Closed. When a TCPConnection object receives requests from other objects, it responds differently depending on its current state. The State pattern describes how TCPConnection can exhibit different behavior in each state. The key idea in this pattern is to introduce an abstract class called TCPState to represent the states of the network connection. The TCPState class declares an interface common to all classes that represent different operational states. Subclasses of TCPState implement state-specific behavior.

## Structure

```
┌────────────┐
│  Context   │
├────────────┤         ┌──────────────┐
│            │────────►│    State     │
└────────────┘         ├──────────────┤
                       │ + handle()   │
                       └──────────────┘
                              △
                              │
                    ┌─────────┴─────────┐
                    │                   │
          ┌─────────────────┐  ┌─────────────────┐
          │  ConcreteStateA │  │  ConcreteStateB │
          ├─────────────────┤  ├─────────────────┤
          │ + handle()      │  │ + handle()      │
          └─────────────────┘  └─────────────────┘
```

## Participants

**Context**
- Defines the interface of interest to clients
- Maintains an instance of a ConcreteState subclass that defines the current state

**State**
- Defines an interface for encapsulating the behavior associated with a particular state of the Context

**ConcreteState subclasses**
- Each subclass implements a behavior associated with a state of the Context

## Consequences

**Benefits:**
- Localizes state-specific behavior and partitions behavior for different states
- Makes state transitions explicit
- State objects can be shared

**Drawbacks:**
- Increases number of classes
- Can make state transitions less explicit

## Implementation

### Basic Implementation (Java)

```java
interface State {
    void handle(Context context);
}

class Context {
    private State state;
    
    public Context(State state) {
        this.state = state;
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    public void request() {
        state.handle(this);
    }
}

class ConcreteStateA implements State {
    @Override
    public void handle(Context context) {
        System.out.println("ConcreteStateA handling request");
        context.setState(new ConcreteStateB());
    }
}

class ConcreteStateB implements State {
    @Override
    public void handle(Context context) {
        System.out.println("ConcreteStateB handling request");
        context.setState(new ConcreteStateA());
    }
}

public class StateDemo {
    public static void main(String[] args) {
        Context context = new Context(new ConcreteStateA());
        
        context.request();
        context.request();
        context.request();
        context.request();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class State(ABC):
    @abstractmethod
    def handle(self, context):
        pass

class Context:
    def __init__(self, state: State):
        self._state = state
    
    def transition_to(self, state: State):
        print(f"Context: Transition to {type(state).__name__}")
        self._state = state
    
    def request(self):
        self._state.handle(self)

class ConcreteStateA(State):
    def handle(self, context: Context):
        print("ConcreteStateA handling request")
        context.transition_to(ConcreteStateB())

class ConcreteStateB(State):
    def handle(self, context: Context):
        print("ConcreteStateB handling request")
        context.transition_to(ConcreteStateA())

context = Context(ConcreteStateA())
context.request()
context.request()
context.request()
```

### JavaScript Implementation

```javascript
class State {
    handle(context) {
        throw new Error("Must implement handle");
    }
}

class Context {
    constructor(state) {
        this.state = state;
    }
    
    transitionTo(state) {
        console.log(`Context: Transition to ${state.constructor.name}`);
        this.state = state;
    }
    
    request() {
        this.state.handle(this);
    }
}

class ConcreteStateA extends State {
    handle(context) {
        console.log("ConcreteStateA handling request");
        context.transitionTo(new ConcreteStateB());
    }
}

class ConcreteStateB extends State {
    handle(context) {
        console.log("ConcreteStateB handling request");
        context.transitionTo(new ConcreteStateA());
    }
}

const context = new Context(new ConcreteStateA());
context.request();
context.request();
context.request();
```

## Known Uses

- **TCP connection states**: Different behaviors in different connection states
- **Document workflow**: Draft, review, published states
- **Vending machines**: Different states based on coin insertion
- **Game character states**: Idle, walking, running, jumping

## Related Patterns

- **Flyweight**: Explains when and how State objects can be shared
- **Singleton**: State objects are often Singletons
- **Strategy**: Both patterns are based on composition

## Real-World Example: Vending Machine

```java
interface VendingMachineState {
    void insertCoin(VendingMachine machine);
    void selectProduct(VendingMachine machine);
    void dispense(VendingMachine machine);
}

class VendingMachine {
    private VendingMachineState state;
    private int count;
    
    public VendingMachine(int count) {
        this.count = count;
        this.state = new NoCoinState();
    }
    
    public void setState(VendingMachineState state) {
        this.state = state;
    }
    
    public void insertCoin() {
        state.insertCoin(this);
    }
    
    public void selectProduct() {
        state.selectProduct(this);
    }
    
    public void dispense() {
        state.dispense(this);
    }
    
    public int getCount() {
        return count;
    }
    
    public void releaseProduct() {
        if (count > 0) {
            count--;
        }
    }
}

class NoCoinState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin inserted");
        machine.setState(new HasCoinState());
    }
    
    @Override
    public void selectProduct(VendingMachine machine) {
        System.out.println("Please insert coin first");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Please insert coin first");
    }
}

class HasCoinState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin already inserted");
    }
    
    @Override
    public void selectProduct(VendingMachine machine) {
        System.out.println("Product selected");
        machine.setState(new DispensingState());
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Please select product first");
    }
}

class DispensingState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Please wait, dispensing product");
    }
    
    @Override
    public void selectProduct(VendingMachine machine) {
        System.out.println("Please wait, dispensing product");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Dispensing product");
        machine.releaseProduct();
        if (machine.getCount() > 0) {
            machine.setState(new NoCoinState());
        } else {
            System.out.println("Out of stock");
            machine.setState(new OutOfStockState());
        }
    }
}

class OutOfStockState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Out of stock");
    }
    
    @Override
    public void selectProduct(VendingMachine machine) {
        System.out.println("Out of stock");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Out of stock");
    }
}
```

## When to Use

- Object's behavior depends on its state
- Operations have large, multipart conditional statements that depend on object's state
- State-specific behavior should be defined independently
