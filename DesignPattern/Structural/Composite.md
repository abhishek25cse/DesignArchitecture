# Composite Pattern

## Category
Structural Pattern

## Intent
Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

## Motivation
Graphics applications like drawing editors let users build complex diagrams out of simple components. The user can group components to form larger components, which in turn can be grouped to form still larger components. A simple implementation could define classes for graphical primitives such as Text and Lines plus other classes that act as containers for these primitives. But there's a problem with this approach: Code that uses these classes must treat primitive and container objects differently, even if most of the time the user treats them identically. Having to distinguish these objects makes the application more complex. The Composite pattern describes how to use recursive composition so that clients don't have to make this distinction.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐
│  Component   │
├──────────────┤
│ + operation()│
│ + add()      │
│ + remove()   │
│ + getChild() │
└──────────────┘
      △
      │
      ├────────────────────┬────────────────────┐
      │                    │                    │
┌──────────────┐    ┌──────────────┐          │
│     Leaf     │    │  Composite   │──────────┘
├──────────────┤    ├──────────────┤
│ + operation()│    │ + operation()│
└──────────────┘    │ + add()      │
                    │ + remove()   │
                    │ + getChild() │
                    └──────────────┘
```

## Participants

**Component**
- Declares the interface for objects in the composition
- Implements default behavior for the interface common to all classes
- Declares an interface for accessing and managing child components

**Leaf**
- Represents leaf objects in the composition (has no children)
- Defines behavior for primitive objects in the composition

**Composite**
- Defines behavior for components having children
- Stores child components
- Implements child-related operations in the Component interface

**Client**
- Manipulates objects in the composition through the Component interface

## Collaborations
- Clients use the Component class interface to interact with objects in the composite structure
- If recipient is a Leaf, request is handled directly
- If recipient is a Composite, it forwards requests to child components

## Consequences

**Benefits:**
- Defines class hierarchies of primitive and composite objects
- Makes client simple (clients can treat composite structures and individual objects uniformly)
- Makes it easier to add new kinds of components

**Drawbacks:**
- Can make design overly general
- Hard to restrict components of a composite

## Implementation

### Basic Implementation (Java)

```java
import java.util.ArrayList;
import java.util.List;

interface Component {
    void operation();
}

class Leaf implements Component {
    private String name;
    
    public Leaf(String name) {
        this.name = name;
    }
    
    @Override
    public void operation() {
        System.out.println("Leaf " + name + " operation");
    }
}

class Composite implements Component {
    private String name;
    private List<Component> children = new ArrayList<>();
    
    public Composite(String name) {
        this.name = name;
    }
    
    public void add(Component component) {
        children.add(component);
    }
    
    public void remove(Component component) {
        children.remove(component);
    }
    
    @Override
    public void operation() {
        System.out.println("Composite " + name + " operation");
        for (Component child : children) {
            child.operation();
        }
    }
}

public class CompositeDemo {
    public static void main(String[] args) {
        Composite root = new Composite("root");
        root.add(new Leaf("Leaf A"));
        root.add(new Leaf("Leaf B"));
        
        Composite comp = new Composite("Composite X");
        comp.add(new Leaf("Leaf XA"));
        comp.add(new Leaf("Leaf XB"));
        
        root.add(comp);
        root.add(new Leaf("Leaf C"));
        
        root.operation();
    }
}
```

### Python Implementation

```python
from abc import ABC, abstractmethod

class Component(ABC):
    @abstractmethod
    def operation(self):
        pass
    
    def add(self, component):
        pass
    
    def remove(self, component):
        pass
    
    def is_composite(self):
        return False

class Leaf(Component):
    def __init__(self, name):
        self.name = name
    
    def operation(self):
        return f"Leaf {self.name}"

class Composite(Component):
    def __init__(self, name):
        self.name = name
        self._children = []
    
    def add(self, component):
        self._children.append(component)
    
    def remove(self, component):
        self._children.remove(component)
    
    def is_composite(self):
        return True
    
    def operation(self):
        results = [f"Composite {self.name}("]
        for child in self._children:
            results.append(child.operation())
        results.append(")")
        return " ".join(results)

tree = Composite("root")
tree.add(Leaf("Leaf A"))
tree.add(Leaf("Leaf B"))

subtree = Composite("Subtree")
subtree.add(Leaf("Leaf XA"))
subtree.add(Leaf("Leaf XB"))

tree.add(subtree)
tree.add(Leaf("Leaf C"))

print(tree.operation())
```

### JavaScript Implementation

```javascript
class Component {
    operation() {
        throw new Error("Must implement operation");
    }
    
    add(component) {}
    remove(component) {}
    isComposite() {
        return false;
    }
}

class Leaf extends Component {
    constructor(name) {
        super();
        this.name = name;
    }
    
    operation() {
        return `Leaf ${this.name}`;
    }
}

class Composite extends Component {
    constructor(name) {
        super();
        this.name = name;
        this.children = [];
    }
    
    add(component) {
        this.children.push(component);
    }
    
    remove(component) {
        const index = this.children.indexOf(component);
        if (index !== -1) {
            this.children.splice(index, 1);
        }
    }
    
    isComposite() {
        return true;
    }
    
    operation() {
        const results = [`Composite ${this.name}(`];
        for (const child of this.children) {
            results.push(child.operation());
        }
        results.push(")");
        return results.join(" ");
    }
}

const tree = new Composite("root");
tree.add(new Leaf("Leaf A"));
tree.add(new Leaf("Leaf B"));

const subtree = new Composite("Subtree");
subtree.add(new Leaf("Leaf XA"));
subtree.add(new Leaf("Leaf XB"));

tree.add(subtree);
tree.add(new Leaf("Leaf C"));

console.log(tree.operation());
```

## Known Uses

- **GUI frameworks**: Containers and widgets
- **File systems**: Directories and files
- **Organization charts**: Departments and employees
- **Graphics editors**: Shapes and groups

## Related Patterns

- **Chain of Responsibility**: Often used with Composite
- **Decorator**: Often used with Composite
- **Iterator**: Can traverse composites
- **Visitor**: Can apply operation over composite

## Real-World Example: File System

```java
interface FileSystemComponent {
    void showDetails();
    long getSize();
}

class File implements FileSystemComponent {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void showDetails() {
        System.out.println("File: " + name + " (" + size + " bytes)");
    }
    
    @Override
    public long getSize() {
        return size;
    }
}

class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> components = new ArrayList<>();
    
    public Directory(String name) {
        this.name = name;
    }
    
    public void add(FileSystemComponent component) {
        components.add(component);
    }
    
    public void remove(FileSystemComponent component) {
        components.remove(component);
    }
    
    @Override
    public void showDetails() {
        System.out.println("Directory: " + name);
        for (FileSystemComponent component : components) {
            component.showDetails();
        }
    }
    
    @Override
    public long getSize() {
        long total = 0;
        for (FileSystemComponent component : components) {
            total += component.getSize();
        }
        return total;
    }
}
```

## When to Use

- Want to represent part-whole hierarchies of objects
- Want clients to ignore difference between compositions of objects and individual objects
- Structure can have any level of complexity and is dynamic

## Current World Examples & Famous Products

### 1. **File Systems**
- **Windows Explorer**: Folders containing files and other folders
- **macOS Finder**: Directory tree structure
- **Linux**: File system hierarchy (directories and files)
- **Google Drive**: Nested folders and files
- **Dropbox**: Folder and file organization

### 2. **Organization Charts**
- **Microsoft Visio**: Organizational hierarchy diagrams
- **Lucidchart**: Company structure charts
- **OrgChart**: Employee hierarchy visualization
- **BambooHR**: Organization structure management
- **Workday**: Corporate hierarchy representation

### 3. **UI Component Libraries**
- **React**: Component tree (components containing other components)
- **Angular**: Component hierarchy
- **Vue.js**: Nested component structure
- **SwiftUI**: View hierarchy
- **Android Views**: ViewGroup containing Views

### 4. **Graphics & Design**
- **Adobe Illustrator**: Groups of shapes and objects
- **Figma**: Frames containing nested elements
- **Sketch**: Artboards with nested layers
- **Photoshop**: Layer groups
- **Blender**: Object hierarchies and parenting

### 5. **Document Editors**
- **Microsoft Word**: Sections, paragraphs, sentences, words
- **Google Docs**: Document structure with nested elements
- **Notion**: Nested pages and blocks
- **Confluence**: Page hierarchy
- **Evernote**: Notebooks, stacks, and notes

### 6. **E-Commerce**
- **Amazon**: Product categories and subcategories
- **eBay**: Category tree structure
- **Shopify**: Collection hierarchy
- **WooCommerce**: Product category tree
- **Etsy**: Shop sections and listings

### 7. **Project Management**
- **Jira**: Epics, stories, subtasks
- **Asana**: Projects, sections, tasks, subtasks
- **Trello**: Boards, lists, cards, checklists
- **Monday.com**: Boards, groups, items, subitems
- **Microsoft Project**: Task hierarchy with dependencies

### 8. **Menu Systems**
- **Restaurant menus**: Categories, subcategories, items
- **Windows Start Menu**: Nested program groups
- **macOS Menu Bar**: Hierarchical menus
- **Website navigation**: Multi-level dropdown menus
- **Mobile app menus**: Nested navigation drawers

### 9. **Social Media**
- **Reddit**: Subreddits, posts, comments, nested replies
- **Facebook**: Groups, posts, comments, nested comments
- **Twitter/X**: Threads with nested replies
- **LinkedIn**: Company pages, departments, employees
- **Discord**: Servers, categories, channels

### 10. **Geographic Information**
- **Google Maps**: Countries, states, cities, streets
- **OpenStreetMap**: Geographic hierarchy
- **GPS navigation**: Route segments and waypoints
- **GIS systems**: Spatial data hierarchies
- **Uber**: Service areas and zones

### 11. **Music & Media**
- **Spotify**: Playlists, albums, tracks
- **iTunes**: Library, playlists, songs
- **YouTube**: Channels, playlists, videos
- **Netflix**: Categories, series, episodes
- **Plex**: Libraries, collections, media items

### 12. **Enterprise Software**
- **SAP**: Business process hierarchies
- **Salesforce**: Account hierarchies, opportunity structures
- **Oracle**: Database object hierarchies
- **ServiceNow**: Service catalog structure
- **Workday**: Organizational and financial hierarchies

### 13. **Gaming**
- **Unity**: GameObject hierarchy in scenes
- **Unreal Engine**: Actor hierarchy
- **Minecraft**: Block structures and buildings
- **The Sims**: Room and furniture hierarchies
- **Game UI**: Menu systems with nested elements

### 14. **Email & Communication**
- **Gmail**: Labels and nested labels
- **Outlook**: Folder hierarchy
- **Slack**: Workspaces, channels, threads
- **Microsoft Teams**: Teams, channels, conversations
- **Telegram**: Folders and chats

### 15. **Development Tools**
- **Git**: Repository, branches, commits, files
- **npm**: Package dependencies tree
- **Docker**: Container hierarchies
- **Kubernetes**: Cluster, namespaces, pods, containers
- **Visual Studio**: Solution, projects, folders, files
