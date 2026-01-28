# Observer Pattern

## Category
Behavioral Pattern

## Intent
Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

## Also Known As
Dependents, Publish-Subscribe

## Motivation
A common side-effect of partitioning a system into a collection of cooperating classes is the need to maintain consistency between related objects. The Observer pattern describes how to establish these relationships. The key objects in this pattern are subject and observer. A subject may have any number of dependent observers. All observers are notified whenever the subject undergoes a change in state. In response, each observer will query the subject to synchronize its state with the subject's state.

## Participants

**Subject**
- Knows its observers
- Provides an interface for attaching and detaching Observer objects

**Observer**
- Defines an updating interface for objects that should be notified of changes in a subject

**ConcreteSubject**
- Stores state of interest to ConcreteObserver objects
- Sends a notification to its observers when its state changes

**ConcreteObserver**
- Maintains a reference to a ConcreteSubject object
- Stores state that should stay consistent with the subject's
- Implements the Observer updating interface

## Consequences

**Benefits:**
- Abstract coupling between Subject and Observer
- Support for broadcast communication
- Unexpected updates can be problematic

**Drawbacks:**
- Can cause memory leaks if observers are not properly detached
- Update overhead when many observers exist

## Implementation

### Basic Implementation (Java)

```java
import java.util.ArrayList;
import java.util.List;

interface Observer {
    void update(String message);
}

interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String state;
    
    public void setState(String state) {
        this.state = state;
        notifyObservers();
    }
    
    public String getState() {
        return state;
    }
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(state);
        }
    }
}

class ConcreteObserver implements Observer {
    private String name;
    
    public ConcreteObserver(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.println(name + " received update: " + message);
    }
}

public class ObserverDemo {
    public static void main(String[] args) {
        ConcreteSubject subject = new ConcreteSubject();
        
        Observer observer1 = new ConcreteObserver("Observer 1");
        Observer observer2 = new ConcreteObserver("Observer 2");
        
        subject.attach(observer1);
        subject.attach(observer2);
        
        subject.setState("State changed to A");
        subject.setState("State changed to B");
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, subject):
        pass

class Subject:
    def __init__(self):
        self._observers = []
        self._state = None
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update(self)
    
    @property
    def state(self):
        return self._state
    
    @state.setter
    def state(self, value):
        self._state = value
        self.notify()

class ConcreteObserverA(Observer):
    def update(self, subject):
        print(f"ConcreteObserverA: Reacted to state {subject.state}")

class ConcreteObserverB(Observer):
    def update(self, subject):
        print(f"ConcreteObserverB: Reacted to state {subject.state}")

subject = Subject()
observer_a = ConcreteObserverA()
observer_b = ConcreteObserverB()

subject.attach(observer_a)
subject.attach(observer_b)

subject.state = "State 1"
subject.state = "State 2"
```

### JavaScript Implementation

```javascript
class Subject {
    constructor() {
        this.observers = [];
    }
    
    attach(observer) {
        this.observers.push(observer);
    }
    
    detach(observer) {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
            this.observers.splice(index, 1);
        }
    }
    
    notify(data) {
        this.observers.forEach(observer => observer.update(data));
    }
}

class Observer {
    update(data) {
        throw new Error("Must implement update");
    }
}

class ConcreteObserverA extends Observer {
    update(data) {
        console.log(`ConcreteObserverA received: ${data}`);
    }
}

class ConcreteObserverB extends Observer {
    update(data) {
        console.log(`ConcreteObserverB received: ${data}`);
    }
}

const subject = new Subject();
const observerA = new ConcreteObserverA();
const observerB = new ConcreteObserverB();

subject.attach(observerA);
subject.attach(observerB);

subject.notify("Event 1");
subject.notify("Event 2");
```

## Known Uses

- **Event handling systems**: GUI frameworks
- **MVC architecture**: Model notifies views
- **Reactive programming**: RxJS, React state management
- **Message queues**: Pub/sub systems

## Related Patterns

- **Mediator**: Encapsulates complex update semantics
- **Singleton**: Subject is often a Singleton

## Real-World Example: Weather Station

```java
interface WeatherObserver {
    void update(float temperature, float humidity, float pressure);
}

class WeatherData {
    private List<WeatherObserver> observers;
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherData() {
        observers = new ArrayList<>();
    }
    
    public void registerObserver(WeatherObserver observer) {
        observers.add(observer);
    }
    
    public void removeObserver(WeatherObserver observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers() {
        for (WeatherObserver observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObservers();
    }
}

class CurrentConditionsDisplay implements WeatherObserver {
    @Override
    public void update(float temperature, float humidity, float pressure) {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}

class StatisticsDisplay implements WeatherObserver {
    private float maxTemp = 0.0f;
    private float minTemp = 200;
    private float tempSum = 0.0f;
    private int numReadings;
    
    @Override
    public void update(float temperature, float humidity, float pressure) {
        tempSum += temperature;
        numReadings++;
        
        if (temperature > maxTemp) {
            maxTemp = temperature;
        }
        
        if (temperature < minTemp) {
            minTemp = temperature;
        }
        
        System.out.println("Avg/Max/Min temperature = " + (tempSum / numReadings) + "/" + maxTemp + "/" + minTemp);
    }
}
```

## When to Use

- When an abstraction has two aspects, one dependent on the other
- When a change to one object requires changing others
- When an object should be able to notify other objects without assumptions about who those objects are
