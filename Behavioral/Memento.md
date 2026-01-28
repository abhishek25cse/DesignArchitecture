# Memento Pattern

## Category
Behavioral Pattern

## Intent
Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

## Also Known As
Token

## Motivation
Sometimes it's necessary to record the internal state of an object. This is required when implementing checkpoints and undo mechanisms that let users back out of tentative operations or recover from errors. You must save state information somewhere so that you can restore objects to their previous states. But objects normally encapsulate some or all of their state, making it inaccessible to other objects and impossible to save externally. Exposing this state would violate encapsulation, which can compromise the application's reliability and extensibility. The Memento pattern can be used to accomplish this without exposing the object's internal structure.

## Participants

**Memento**
- Stores internal state of the Originator object
- Protects against access by objects other than the originator

**Originator**
- Creates a memento containing a snapshot of its current internal state
- Uses the memento to restore its internal state

**Caretaker**
- Responsible for the memento's safekeeping
- Never operates on or examines the contents of a memento

## Consequences

**Benefits:**
- Preserves encapsulation boundaries
- Simplifies Originator

**Drawbacks:**
- Using mementos might be expensive
- Hidden costs in caring for mementos

## Implementation

### Basic Implementation (Java)

```java
class Memento {
    private String state;
    
    public Memento(String state) {
        this.state = state;
    }
    
    public String getState() {
        return state;
    }
}

class Originator {
    private String state;
    
    public void setState(String state) {
        System.out.println("Setting state to: " + state);
        this.state = state;
    }
    
    public String getState() {
        return state;
    }
    
    public Memento saveToMemento() {
        System.out.println("Saving to Memento");
        return new Memento(state);
    }
    
    public void restoreFromMemento(Memento memento) {
        state = memento.getState();
        System.out.println("Restored from Memento: " + state);
    }
}

class Caretaker {
    private List<Memento> mementos = new ArrayList<>();
    
    public void addMemento(Memento memento) {
        mementos.add(memento);
    }
    
    public Memento getMemento(int index) {
        return mementos.get(index);
    }
}

public class MementoDemo {
    public static void main(String[] args) {
        Originator originator = new Originator();
        Caretaker caretaker = new Caretaker();
        
        originator.setState("State1");
        caretaker.addMemento(originator.saveToMemento());
        
        originator.setState("State2");
        caretaker.addMemento(originator.saveToMemento());
        
        originator.setState("State3");
        
        originator.restoreFromMemento(caretaker.getMemento(0));
        originator.restoreFromMemento(caretaker.getMemento(1));
    }
}
```

### Python Implementation

```python
class Memento:
    def __init__(self, state):
        self._state = state
    
    def get_state(self):
        return self._state

class Originator:
    def __init__(self):
        self._state = None
    
    def set_state(self, state):
        print(f"Setting state to: {state}")
        self._state = state
    
    def save_to_memento(self):
        print("Saving to Memento")
        return Memento(self._state)
    
    def restore_from_memento(self, memento):
        self._state = memento.get_state()
        print(f"Restored from Memento: {self._state}")

class Caretaker:
    def __init__(self):
        self._mementos = []
    
    def add_memento(self, memento):
        self._mementos.append(memento)
    
    def get_memento(self, index):
        return self._mementos[index]

# Usage
originator = Originator()
caretaker = Caretaker()

originator.set_state("State1")
caretaker.add_memento(originator.save_to_memento())

originator.set_state("State2")
caretaker.add_memento(originator.save_to_memento())

originator.set_state("State3")

originator.restore_from_memento(caretaker.get_memento(0))
originator.restore_from_memento(caretaker.get_memento(1))
```

## When to Use

- Snapshot of object's state must be saved for later restoration
- Direct interface to obtain state would expose implementation details and break encapsulation
