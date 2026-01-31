# Command Pattern

## Category
Behavioral Pattern

## Intent
Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

## Also Known As
Action, Transaction

## Motivation
Sometimes it's necessary to issue requests to objects without knowing anything about the operation being requested or the receiver of the request. The Command pattern lets toolkit objects make requests of unspecified application objects by turning the request itself into an object. This object can be stored and passed around like other objects. The key to this pattern is an abstract Command class, which declares an interface for executing operations. In the simplest form this interface includes an abstract Execute operation. Concrete Command subclasses specify a receiver-action pair by storing the receiver as an instance variable and by implementing Execute to invoke the request. The receiver has the knowledge required to carry out the request.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      │ creates
      ▼
┌──────────────┐
│   Command    │◄────────┌──────────────┐
├──────────────┤         │   Invoker    │
│ + execute()  │         ├──────────────┤
└──────────────┘         │              │
      △                  └──────────────┘
      │
      │
┌──────────────┐
│ConcreteCommand│──────►┌──────────────┐
├──────────────┤        │   Receiver   │
│ + execute()  │        ├──────────────┤
└──────────────┘        │ + action()   │
                        └──────────────┘
```

## Participants

**Command**
- Declares an interface for executing an operation

**ConcreteCommand**
- Defines a binding between a Receiver object and an action
- Implements Execute by invoking the corresponding operation(s) on Receiver

**Client**
- Creates a ConcreteCommand object and sets its receiver

**Invoker**
- Asks the command to carry out the request

**Receiver**
- Knows how to perform the operations associated with carrying out a request

## Consequences

**Benefits:**
- Decouples object that invokes operation from one that knows how to perform it
- Commands are first-class objects (can be manipulated and extended)
- Can assemble commands into composite command
- Easy to add new Commands

**Drawbacks:**
- Can result in many small command classes

## Implementation

### Basic Implementation (Java)

```java
interface Command {
    void execute();
    void undo();
}

class Light {
    public void on() {
        System.out.println("Light is ON");
    }
    
    public void off() {
        System.out.println("Light is OFF");
    }
}

class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.on();
    }
    
    @Override
    public void undo() {
        light.off();
    }
}

class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.off();
    }
    
    @Override
    public void undo() {
        light.on();
    }
}

class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
    
    public void pressUndo() {
        command.undo();
    }
}

public class CommandDemo {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOn = new LightOnCommand(light);
        Command lightOff = new LightOffCommand(light);
        
        RemoteControl remote = new RemoteControl();
        
        remote.setCommand(lightOn);
        remote.pressButton();
        remote.pressUndo();
        
        remote.setCommand(lightOff);
        remote.pressButton();
        remote.pressUndo();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class Light:
    def on(self):
        print("Light is ON")
    
    def off(self):
        print("Light is OFF")

class LightOnCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.on()
    
    def undo(self):
        self.light.off()

class LightOffCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.off()
    
    def undo(self):
        self.light.on()

class RemoteControl:
    def __init__(self):
        self.command = None
    
    def set_command(self, command):
        self.command = command
    
    def press_button(self):
        self.command.execute()
    
    def press_undo(self):
        self.command.undo()

light = Light()
light_on = LightOnCommand(light)
light_off = LightOffCommand(light)

remote = RemoteControl()

remote.set_command(light_on)
remote.press_button()
remote.press_undo()

remote.set_command(light_off)
remote.press_button()
remote.press_undo()
```

### JavaScript Implementation

```javascript
class Command {
    execute() {
        throw new Error("Must implement execute");
    }
    
    undo() {
        throw new Error("Must implement undo");
    }
}

class Light {
    on() {
        console.log("Light is ON");
    }
    
    off() {
        console.log("Light is OFF");
    }
}

class LightOnCommand extends Command {
    constructor(light) {
        super();
        this.light = light;
    }
    
    execute() {
        this.light.on();
    }
    
    undo() {
        this.light.off();
    }
}

class LightOffCommand extends Command {
    constructor(light) {
        super();
        this.light = light;
    }
    
    execute() {
        this.light.off();
    }
    
    undo() {
        this.light.on();
    }
}

class RemoteControl {
    setCommand(command) {
        this.command = command;
    }
    
    pressButton() {
        this.command.execute();
    }
    
    pressUndo() {
        this.command.undo();
    }
}

const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.setCommand(lightOn);
remote.pressButton();
remote.pressUndo();

remote.setCommand(lightOff);
remote.pressButton();
remote.pressUndo();
```

## Known Uses

- **GUI buttons and menu items**: Actions triggered by UI elements
- **Macro recording**: Recording and playback of commands
- **Undo/Redo functionality**: Text editors, graphics programs
- **Transaction systems**: Database transactions
- **Job queues**: Scheduling and executing tasks

## Related Patterns

- **Composite**: Can implement MacroCommands
- **Memento**: Can keep state for undo
- **Prototype**: Commands can be cloned

## Real-World Example: Text Editor

```java
interface Command {
    void execute();
    void undo();
}

class TextEditor {
    private StringBuilder text = new StringBuilder();
    
    public void write(String words) {
        text.append(words);
    }
    
    public void deleteLast(int length) {
        int start = text.length() - length;
        if (start >= 0) {
            text.delete(start, text.length());
        }
    }
    
    public String getText() {
        return text.toString();
    }
}

class WriteCommand implements Command {
    private TextEditor editor;
    private String text;
    
    public WriteCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.text = text;
    }
    
    @Override
    public void execute() {
        editor.write(text);
    }
    
    @Override
    public void undo() {
        editor.deleteLast(text.length());
    }
}

class CommandHistory {
    private Stack<Command> history = new Stack<>();
    
    public void push(Command command) {
        history.push(command);
    }
    
    public Command pop() {
        return history.pop();
    }
    
    public boolean isEmpty() {
        return history.isEmpty();
    }
}
```

## When to Use

- Parameterize objects by an action to perform
- Specify, queue, and execute requests at different times
- Support undo operations
- Support logging changes
- Structure system around high-level operations built on primitive operations

## Current World Examples & Famous Products

### 1. **Text Editors & IDEs**
- **Microsoft Word**: Undo/redo operations for text editing
- **Google Docs**: Command history for collaborative editing
- **VS Code**: Edit operations, refactoring commands
- **Sublime Text**: Multiple cursor commands
- **IntelliJ IDEA**: Refactoring and code transformation commands

### 2. **Graphics & Design Software**
- **Photoshop**: Layer operations, filter applications (undo/redo)
- **Illustrator**: Shape transformations, path operations
- **Figma**: Design changes with undo/redo
- **Sketch**: Layer modifications
- **GIMP**: Image manipulation commands

### 3. **Smart Home Systems**
- **Amazon Alexa**: Voice commands ("Turn on lights", "Set temperature")
- **Google Home**: Device control commands
- **Apple HomeKit**: Scene commands, automation
- **SmartThings**: Routine commands
- **IFTTT**: Applet commands

### 4. **Task Schedulers & Automation**
- **Cron**: Scheduled command execution
- **Windows Task Scheduler**: Automated task commands
- **Jenkins**: Build job commands
- **Airflow**: DAG task commands
- **Zapier**: Workflow action commands

### 5. **Version Control Systems**
- **Git**: Commit, revert, cherry-pick commands
- **SVN**: Checkout, commit, revert operations
- **Mercurial**: Changeset operations
- **Perforce**: Submit, revert commands
- **GitHub Actions**: Workflow commands

### 6. **Database Systems**
- **SQL**: Transaction commands (BEGIN, COMMIT, ROLLBACK)
- **MongoDB**: Database operation commands
- **Redis**: Command queue execution
- **PostgreSQL**: Prepared statements
- **MySQL**: Stored procedures

### 7. **Game Development**
- **Unity**: Player action commands, input system
- **Unreal Engine**: Blueprint commands
- **Game replays**: Recorded command playback
- **RTS games**: Unit commands (move, attack, build)
- **Fighting games**: Combo commands

### 8. **Remote Controls & IoT**
- **TV remotes**: Button press commands
- **Universal remotes**: Macro commands
- **Smart thermostats**: Temperature adjustment commands
- **Robot vacuums**: Cleaning commands
- **Drone controllers**: Flight commands

### 9. **E-Commerce & Shopping**
- **Amazon**: Order placement, cancellation commands
- **Shopping carts**: Add, remove, update quantity commands
- **Shopify**: Inventory management commands
- **Payment processing**: Transaction commands
- **Refund systems**: Return processing commands

### 10. **Email Clients**
- **Gmail**: Email actions (archive, delete, mark as read)
- **Outlook**: Message operations, rules
- **Apple Mail**: Mailbox operations
- **Thunderbird**: Filter commands
- **ProtonMail**: Encryption commands

### 11. **Media Players**
- **VLC**: Playback commands (play, pause, skip)
- **Spotify**: Queue management commands
- **YouTube**: Video control commands
- **Netflix**: Playback control
- **iTunes**: Library management commands

### 12. **Build Systems**
- **Make**: Build target commands
- **Gradle**: Task execution commands
- **Maven**: Goal execution commands
- **npm scripts**: Package commands
- **Webpack**: Build configuration commands

### 13. **Cloud Services**
- **AWS CLI**: Service operation commands
- **Azure CLI**: Resource management commands
- **Google Cloud SDK**: gcloud commands
- **Kubernetes**: kubectl commands
- **Docker**: Container management commands

### 14. **Messaging Apps**
- **Slack**: Slash commands, bot commands
- **Discord**: Bot commands, moderation actions
- **Telegram**: Bot commands
- **WhatsApp**: Message actions
- **Microsoft Teams**: Bot framework commands

### 15. **Operating Systems**
- **Windows**: System commands, PowerShell cmdlets
- **macOS**: Terminal commands, Automator actions
- **Linux**: Shell commands, systemd commands
- **Android**: Intent commands
- **iOS**: Shortcuts app commands
