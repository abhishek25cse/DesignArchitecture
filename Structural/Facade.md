# Facade Pattern

## Category
Structural Pattern

## Intent
Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

## Motivation
Structuring a system into subsystems helps reduce complexity. A common design goal is to minimize the communication and dependencies between subsystems. One way to achieve this goal is to introduce a facade object that provides a single, simplified interface to the more general facilities of a subsystem. Consider a programming environment that gives applications access to its compiler subsystem. This subsystem contains classes such as Scanner, Parser, ProgramNode, BytecodeStream, and ProgramNodeBuilder that implement the compiler. Some specialized applications might need to access these classes directly. But most clients of a compiler generally don't care about details like parsing and code generation; they merely want to compile some code. For them, the powerful but low-level interfaces in the compiler subsystem only complicate their task. To provide a higher-level interface that can shield clients from these classes, the compiler subsystem also includes a Compiler class. This class defines a unified interface to the compiler's functionality. The Compiler class acts as a facade: It offers clients a single, simple interface to the compiler subsystem.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐
│    Facade    │
├──────────────┤
│ + operation()│───────┐
└──────────────┘       │
                       │
      ┌────────────────┼────────────────┐
      │                │                │
      ▼                ▼                ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│SubsystemA│    │SubsystemB│    │SubsystemC│
├──────────┤    ├──────────┤    ├──────────┤
│          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘
```

## Participants

**Facade**
- Knows which subsystem classes are responsible for a request
- Delegates client requests to appropriate subsystem objects

**Subsystem classes**
- Implement subsystem functionality
- Handle work assigned by the Facade object
- Have no knowledge of the facade

## Collaborations
- Clients communicate with the subsystem by sending requests to Facade, which forwards them to the appropriate subsystem object(s)
- Clients that use the facade don't have to access its subsystem objects directly

## Consequences

**Benefits:**
- Shields clients from subsystem components
- Promotes weak coupling between subsystem and clients
- Doesn't prevent applications from using subsystem classes if needed
- Simplifies porting to other platforms

**Drawbacks:**
- Can become a god object coupled to all classes of an app

## Implementation

### Basic Implementation (Java)

```java
// Subsystem classes
class CPU {
    public void freeze() {
        System.out.println("CPU: Freezing processor");
    }
    
    public void jump(long position) {
        System.out.println("CPU: Jumping to position " + position);
    }
    
    public void execute() {
        System.out.println("CPU: Executing instructions");
    }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory: Loading data at position " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive: Reading " + size + " bytes from sector " + lba);
        return new byte[size];
    }
}

// Facade
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        System.out.println("Starting computer...");
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
        System.out.println("Computer started successfully!");
    }
}

// Client
public class FacadeDemo {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();
    }
}
```

### Python Implementation

```python
class CPU:
    def freeze(self):
        print("CPU: Freezing processor")
    
    def jump(self, position):
        print(f"CPU: Jumping to position {position}")
    
    def execute(self):
        print("CPU: Executing instructions")

class Memory:
    def load(self, position, data):
        print(f"Memory: Loading data at position {position}")

class HardDrive:
    def read(self, lba, size):
        print(f"HardDrive: Reading {size} bytes from sector {lba}")
        return bytes(size)

class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()
    
    def start(self):
        print("Starting computer...")
        self.cpu.freeze()
        self.memory.load(0, self.hard_drive.read(0, 1024))
        self.cpu.jump(0)
        self.cpu.execute()
        print("Computer started successfully!")

# Usage
computer = ComputerFacade()
computer.start()
```

### JavaScript Implementation

```javascript
class CPU {
    freeze() {
        console.log("CPU: Freezing processor");
    }
    
    jump(position) {
        console.log(`CPU: Jumping to position ${position}`);
    }
    
    execute() {
        console.log("CPU: Executing instructions");
    }
}

class Memory {
    load(position, data) {
        console.log(`Memory: Loading data at position ${position}`);
    }
}

class HardDrive {
    read(lba, size) {
        console.log(`HardDrive: Reading ${size} bytes from sector ${lba}`);
        return new Uint8Array(size);
    }
}

class ComputerFacade {
    constructor() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    start() {
        console.log("Starting computer...");
        this.cpu.freeze();
        this.memory.load(0, this.hardDrive.read(0, 1024));
        this.cpu.jump(0);
        this.cpu.execute();
        console.log("Computer started successfully!");
    }
}

// Usage
const computer = new ComputerFacade();
computer.start();
```

## Implementation Considerations

1. **Reducing client-subsystem coupling**: Make Facade an abstract class with concrete subclasses for different implementations

2. **Public versus private subsystem classes**: Subsystem is analogous to class, facade to public interface

## Known Uses

- **JDBC**: Provides facade to database operations
- **SLF4J**: Facade for various logging frameworks
- **Spring Framework**: Many facade classes for complex operations
- **Compiler interfaces**: Simplified compilation APIs

## Related Patterns

- **Abstract Factory**: Can be used with Facade to provide interface for creating subsystem objects
- **Mediator**: Similar to Facade in that it abstracts functionality of existing classes. Mediator's purpose is to abstract arbitrary communication between colleague objects, often centralizing functionality that doesn't belong in any one of them
- **Singleton**: Facade objects are often Singletons

## Real-World Example: Home Theater

```java
// Subsystem classes
class Amplifier {
    public void on() {
        System.out.println("Amplifier on");
    }
    
    public void setVolume(int level) {
        System.out.println("Amplifier volume set to " + level);
    }
    
    public void off() {
        System.out.println("Amplifier off");
    }
}

class DVDPlayer {
    public void on() {
        System.out.println("DVD Player on");
    }
    
    public void play(String movie) {
        System.out.println("Playing movie: " + movie);
    }
    
    public void stop() {
        System.out.println("DVD Player stopped");
    }
    
    public void off() {
        System.out.println("DVD Player off");
    }
}

class Projector {
    public void on() {
        System.out.println("Projector on");
    }
    
    public void wideScreenMode() {
        System.out.println("Projector in widescreen mode");
    }
    
    public void off() {
        System.out.println("Projector off");
    }
}

class Lights {
    public void dim(int level) {
        System.out.println("Lights dimmed to " + level + "%");
    }
    
    public void on() {
        System.out.println("Lights on");
    }
}

// Facade
class HomeTheaterFacade {
    private Amplifier amp;
    private DVDPlayer dvd;
    private Projector projector;
    private Lights lights;
    
    public HomeTheaterFacade(Amplifier amp, DVDPlayer dvd, 
                            Projector projector, Lights lights) {
        this.amp = amp;
        this.dvd = dvd;
        this.projector = projector;
        this.lights = lights;
    }
    
    public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...");
        lights.dim(10);
        projector.on();
        projector.wideScreenMode();
        amp.on();
        amp.setVolume(5);
        dvd.on();
        dvd.play(movie);
    }
    
    public void endMovie() {
        System.out.println("Shutting down movie theater...");
        lights.on();
        projector.off();
        amp.off();
        dvd.stop();
        dvd.off();
    }
}

// Usage
public class HomeTheaterDemo {
    public static void main(String[] args) {
        Amplifier amp = new Amplifier();
        DVDPlayer dvd = new DVDPlayer();
        Projector projector = new Projector();
        Lights lights = new Lights();
        
        HomeTheaterFacade homeTheater = 
            new HomeTheaterFacade(amp, dvd, projector, lights);
        
        homeTheater.watchMovie("Inception");
        System.out.println();
        homeTheater.endMovie();
    }
}
```

## Real-World Example: Banking System

```python
class Account:
    def __init__(self, account_number):
        self.account_number = account_number
        self.balance = 0
    
    def deposit(self, amount):
        self.balance += amount
        print(f"Deposited ${amount} to account {self.account_number}")
    
    def withdraw(self, amount):
        if self.balance >= amount:
            self.balance -= amount
            print(f"Withdrew ${amount} from account {self.account_number}")
            return True
        return False
    
    def get_balance(self):
        return self.balance

class SecurityService:
    def authenticate(self, account_number, pin):
        print(f"Authenticating account {account_number}")
        return True

class TransactionService:
    def log_transaction(self, account_number, transaction_type, amount):
        print(f"Logging {transaction_type} of ${amount} for account {account_number}")

class NotificationService:
    def send_notification(self, account_number, message):
        print(f"Sending notification to account {account_number}: {message}")

class BankingFacade:
    def __init__(self):
        self.accounts = {}
        self.security = SecurityService()
        self.transaction = TransactionService()
        self.notification = NotificationService()
    
    def create_account(self, account_number):
        self.accounts[account_number] = Account(account_number)
        print(f"Account {account_number} created")
    
    def deposit(self, account_number, pin, amount):
        if self.security.authenticate(account_number, pin):
            account = self.accounts.get(account_number)
            if account:
                account.deposit(amount)
                self.transaction.log_transaction(account_number, "DEPOSIT", amount)
                self.notification.send_notification(
                    account_number, 
                    f"Deposit of ${amount} successful"
                )
    
    def withdraw(self, account_number, pin, amount):
        if self.security.authenticate(account_number, pin):
            account = self.accounts.get(account_number)
            if account and account.withdraw(amount):
                self.transaction.log_transaction(account_number, "WITHDRAWAL", amount)
                self.notification.send_notification(
                    account_number, 
                    f"Withdrawal of ${amount} successful"
                )

# Usage
bank = BankingFacade()
bank.create_account("12345")
bank.deposit("12345", "1234", 1000)
bank.withdraw("12345", "1234", 500)
```

## When to Use

- Want to provide simple interface to complex subsystem
- Many dependencies between clients and implementation classes of abstraction
- Want to layer subsystems (use facade to define entry point to each subsystem level)
- Need to decouple subsystem from clients and other subsystems
