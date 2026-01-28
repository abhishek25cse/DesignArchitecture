# Strategy Pattern

## Category
Behavioral Pattern

## Intent
Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

## Also Known As
Policy

## Motivation
Many algorithms exist for breaking a stream of text into lines. Hard-wiring all such algorithms into the classes that require them isn't desirable for several reasons: clients that need linebreaking get more complex if they include the linebreaking code, different algorithms will be appropriate at different times, and it's difficult to add new algorithms and vary existing ones when linebreaking is an integral part of a client. We can avoid these problems by defining classes that encapsulate different linebreaking algorithms. An algorithm that's encapsulated in this way is called a strategy.

## Structure

```
┌────────────┐
│  Context   │
├────────────┤         ┌──────────────┐
│            │────────►│  Strategy    │
└────────────┘         ├──────────────┤
                       │+ algorithm() │
                       └──────────────┘
                              △
                              │
                    ┌─────────┴─────────┐
                    │                   │
          ┌─────────────────┐  ┌─────────────────┐
          │ConcreteStrategyA│  │ConcreteStrategyB│
          ├─────────────────┤  ├─────────────────┤
          │ + algorithm()   │  │ + algorithm()   │
          └─────────────────┘  └─────────────────┘
```

## Participants

**Strategy**
- Declares an interface common to all supported algorithms

**ConcreteStrategy**
- Implements the algorithm using the Strategy interface

**Context**
- Is configured with a ConcreteStrategy object
- Maintains a reference to a Strategy object
- May define an interface that lets Strategy access its data

## Consequences

**Benefits:**
- Families of related algorithms
- Alternative to subclassing
- Eliminates conditional statements
- Choice of implementations

**Drawbacks:**
- Clients must be aware of different Strategies
- Communication overhead between Strategy and Context
- Increased number of objects

## Implementation

### Basic Implementation (Java)

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;
    private String cvv;
    
    public CreditCardStrategy(String cardNumber, String cvv) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

class PayPalStrategy implements PaymentStrategy {
    private String email;
    
    public PayPalStrategy(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

public class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        cart.setPaymentStrategy(new CreditCardStrategy("1234-5678-9012-3456", "123"));
        cart.checkout(100);
        
        cart.setPaymentStrategy(new PayPalStrategy("user@example.com"));
        cart.checkout(200);
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Strategy(ABC):
    @abstractmethod
    def do_algorithm(self, data):
        pass

class ConcreteStrategyA(Strategy):
    def do_algorithm(self, data):
        return sorted(data)

class ConcreteStrategyB(Strategy):
    def do_algorithm(self, data):
        return sorted(data, reverse=True)

class Context:
    def __init__(self, strategy: Strategy):
        self._strategy = strategy
    
    @property
    def strategy(self):
        return self._strategy
    
    @strategy.setter
    def strategy(self, strategy: Strategy):
        self._strategy = strategy
    
    def do_business_logic(self):
        result = self._strategy.do_algorithm(["a", "b", "c", "d", "e"])
        print(",".join(result))

context = Context(ConcreteStrategyA())
print("Client: Strategy A")
context.do_business_logic()

print("\nClient: Strategy B")
context.strategy = ConcreteStrategyB()
context.do_business_logic()
```

### JavaScript Implementation

```javascript
class Strategy {
    doAlgorithm(data) {
        throw new Error("Must implement doAlgorithm");
    }
}

class ConcreteStrategyA extends Strategy {
    doAlgorithm(data) {
        return data.sort();
    }
}

class ConcreteStrategyB extends Strategy {
    doAlgorithm(data) {
        return data.sort().reverse();
    }
}

class Context {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    setStrategy(strategy) {
        this.strategy = strategy;
    }
    
    doBusinessLogic() {
        const result = this.strategy.doAlgorithm(["a", "b", "c", "d", "e"]);
        console.log(result.join(","));
    }
}

const context = new Context(new ConcreteStrategyA());
console.log("Client: Strategy A");
context.doBusinessLogic();

console.log("\nClient: Strategy B");
context.setStrategy(new ConcreteStrategyB());
context.doBusinessLogic();
```

## Known Uses

- **Java Comparator**: Different sorting strategies
- **Compression algorithms**: Different compression strategies
- **Validation strategies**: Different validation rules
- **Routing algorithms**: Different path-finding strategies

## Related Patterns

- **Flyweight**: Strategy objects often make good flyweights
- **State**: Both patterns are based on composition

## Real-World Example: Sorting

```java
interface SortStrategy {
    void sort(int[] array);
}

class BubbleSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using bubble sort");
        int n = array.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (array[j] > array[j + 1]) {
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
}

class QuickSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using quick sort");
        quickSort(array, 0, array.length - 1);
    }
    
    private void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }
    
    private int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = (low - 1);
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;
        return i + 1;
    }
}

class SortContext {
    private SortStrategy strategy;
    
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sortArray(int[] array) {
        strategy.sort(array);
    }
}
```

## When to Use

- Many related classes differ only in their behavior
- Need different variants of an algorithm
- Algorithm uses data that clients shouldn't know about
- Class defines many behaviors as multiple conditional statements
