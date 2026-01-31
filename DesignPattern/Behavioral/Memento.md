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

## Current World Examples & Famous Products

### 1. **Text Editors & IDEs**
- **Microsoft Word**: Document version history, undo/redo
- **Google Docs**: Revision history with restore
- **VS Code**: File state snapshots, undo stack
- **IntelliJ IDEA**: Local history, undo operations
- **Sublime Text**: Edit history management

### 2. **Graphics & Design Software**
- **Photoshop**: History panel, snapshots
- **Illustrator**: Undo states
- **Figma**: Version history
- **Sketch**: Document versions
- **GIMP**: Undo history

### 3. **Database Systems**
- **MySQL**: Transaction savepoints
- **PostgreSQL**: Point-in-time recovery
- **Oracle**: Flashback technology
- **SQL Server**: Database snapshots
- **MongoDB**: Snapshot isolation

### 4. **Version Control Systems**
- **Git**: Commit history, stash
- **SVN**: Repository snapshots
- **Mercurial**: Changeset history
- **Perforce**: Shelving changes
- **GitHub**: Branch snapshots

### 5. **Gaming**
- **Save games**: Game state snapshots
- **Emulators**: Save states
- **Dark Souls**: Bonfire checkpoints
- **Civilization**: Turn-based saves
- **RPGs**: Quick save/load

### 6. **Virtual Machines**
- **VMware**: VM snapshots
- **VirtualBox**: Machine state snapshots
- **Hyper-V**: Checkpoints
- **Parallels**: Snapshots
- **QEMU**: VM state saving

### 7. **Mobile Apps**
- **iOS**: App state preservation
- **Android**: Activity state bundles
- **React Native**: State persistence
- **Flutter**: State restoration
- **Xamarin**: Page state management

### 8. **Web Browsers**
- **Chrome**: Tab state restoration
- **Firefox**: Session restore
- **Safari**: Browsing state recovery
- **Edge**: Tab recovery
- **Opera**: Session management

### 9. **Cloud Storage**
- **Dropbox**: File version history
- **Google Drive**: Version management
- **OneDrive**: File versioning
- **Box**: Version control
- **iCloud**: Document versions

### 10. **Backup Systems**
- **Time Machine**: System snapshots
- **Windows Backup**: System restore points
- **Acronis**: Image backups
- **Veeam**: VM snapshots
- **Carbonite**: File versioning

### 11. **E-Commerce Shopping**
- **Amazon**: Saved carts
- **Shopping cart persistence**: Session state
- **Wishlist**: Saved item states
- **Checkout recovery**: Form state preservation
- **Order drafts**: Incomplete order states

### 12. **Form Builders**
- **Google Forms**: Draft saving
- **Typeform**: Response drafts
- **JotForm**: Form state preservation
- **SurveyMonkey**: Survey progress saving
- **Microsoft Forms**: Draft responses

### 13. **Project Management**
- **Jira**: Issue history
- **Trello**: Card history
- **Asana**: Task revision history
- **Monday.com**: Item change log
- **Notion**: Page version history

### 14. **Financial Software**
- **QuickBooks**: Transaction history
- **Mint**: Budget snapshots
- **TurboTax**: Return versions
- **Excel**: Workbook versions
- **Banking apps**: Transaction history

### 15. **Configuration Management**
- **Kubernetes**: ConfigMap versions
- **Terraform**: State files
- **Ansible**: Playbook history
- **Chef**: Cookbook versions
- **Puppet**: Configuration snapshots
