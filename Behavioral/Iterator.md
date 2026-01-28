# Iterator Pattern

## Category
Behavioral Pattern

## Intent
Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

## Also Known As
Cursor

## Motivation
An aggregate object such as a list should give you a way to access its elements without exposing its internal structure. Moreover, you might want to traverse the list in different ways, depending on what you want to accomplish. But you probably don't want to bloat the List interface with operations for different traversals, even if you could anticipate the ones you will need. You might also need to have more than one traversal pending on the same list. The Iterator pattern lets you do all this. The key idea is to take the responsibility for access and traversal out of the list object and put it into an iterator object. The Iterator class defines an interface for accessing the list's elements. An iterator object is responsible for keeping track of the current element; that is, it knows which elements have been traversed already.

## Structure

```
┌────────────┐
│   Client   │
└────────────┘
      │
      ▼
┌──────────────┐         ┌──────────────┐
│  Aggregate   │         │   Iterator   │
├──────────────┤         ├──────────────┤
│+ createIter()│         │ + first()    │
└──────────────┘         │ + next()     │
      △                  │ + isDone()   │
      │                  │ + currentItem│
      │                  └──────────────┘
┌──────────────┐                △
│ConcreteAggr  │                │
├──────────────┤         ┌──────────────┐
│+ createIter()│────────►│ConcreteIter  │
└──────────────┘         ├──────────────┤
                         │ + first()    │
                         │ + next()     │
                         │ + isDone()   │
                         │ + currentItem│
                         └──────────────┘
```

## Participants

**Iterator**
- Defines an interface for accessing and traversing elements

**ConcreteIterator**
- Implements the Iterator interface
- Keeps track of the current position in the traversal

**Aggregate**
- Defines an interface for creating an Iterator object

**ConcreteAggregate**
- Implements the Iterator creation interface to return an instance of the proper ConcreteIterator

## Consequences

**Benefits:**
- Supports variations in traversal of an aggregate
- Simplifies the Aggregate interface
- More than one traversal can be pending on an aggregate

**Drawbacks:**
- Additional classes and complexity

## Implementation

### Basic Implementation (Java)

```java
interface Iterator<T> {
    boolean hasNext();
    T next();
}

interface Container<T> {
    Iterator<T> createIterator();
}

class BookCollection implements Container<String> {
    private String[] books;
    private int index = 0;
    
    public BookCollection(int size) {
        books = new String[size];
    }
    
    public void addBook(String book) {
        if (index < books.length) {
            books[index++] = book;
        }
    }
    
    @Override
    public Iterator<String> createIterator() {
        return new BookIterator();
    }
    
    private class BookIterator implements Iterator<String> {
        private int currentIndex = 0;
        
        @Override
        public boolean hasNext() {
            return currentIndex < books.length && books[currentIndex] != null;
        }
        
        @Override
        public String next() {
            return books[currentIndex++];
        }
    }
}

public class IteratorDemo {
    public static void main(String[] args) {
        BookCollection books = new BookCollection(5);
        books.addBook("Design Patterns");
        books.addBook("Clean Code");
        books.addBook("Refactoring");
        
        Iterator<String> iterator = books.createIterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

### Python Implementation

```python
from collections.abc import Iterable, Iterator

class BookCollection(Iterable):
    def __init__(self):
        self._books = []
    
    def add_book(self, book):
        self._books.append(book)
    
    def __iter__(self):
        return BookIterator(self._books)

class BookIterator(Iterator):
    def __init__(self, books):
        self._books = books
        self._index = 0
    
    def __next__(self):
        try:
            book = self._books[self._index]
            self._index += 1
            return book
        except IndexError:
            raise StopIteration

# Usage
books = BookCollection()
books.add_book("Design Patterns")
books.add_book("Clean Code")
books.add_book("Refactoring")

for book in books:
    print(book)
```

### JavaScript Implementation

```javascript
class BookCollection {
    constructor() {
        this.books = [];
    }
    
    addBook(book) {
        this.books.push(book);
    }
    
    [Symbol.iterator]() {
        let index = 0;
        const books = this.books;
        
        return {
            next() {
                if (index < books.length) {
                    return { value: books[index++], done: false };
                } else {
                    return { done: true };
                }
            }
        };
    }
}

// Usage
const books = new BookCollection();
books.addBook("Design Patterns");
books.addBook("Clean Code");
books.addBook("Refactoring");

for (const book of books) {
    console.log(book);
}
```

## Known Uses

- **Java Collections Framework**: Iterator interface
- **C++ STL**: Iterator concept
- **Python**: Iterator protocol
- **JavaScript**: Iterable protocol

## Related Patterns

- **Composite**: Iterators are often applied to recursive structures
- **Factory Method**: Polymorphic iterators rely on factory methods
- **Memento**: Can be used with Iterator to capture iteration state

## Real-World Example: Social Network

```java
interface ProfileIterator {
    boolean hasNext();
    Profile next();
}

interface SocialNetwork {
    ProfileIterator createFriendsIterator(String profileId);
    ProfileIterator createCoworkersIterator(String profileId);
}

class Profile {
    private String id;
    private String email;
    
    public Profile(String id, String email) {
        this.id = id;
        this.email = email;
    }
    
    public String getId() { return id; }
    public String getEmail() { return email; }
}

class Facebook implements SocialNetwork {
    private Map<String, Profile> profiles;
    
    public Facebook(Map<String, Profile> profiles) {
        this.profiles = profiles;
    }
    
    @Override
    public ProfileIterator createFriendsIterator(String profileId) {
        return new FacebookIterator(this, "friends", profileId);
    }
    
    @Override
    public ProfileIterator createCoworkersIterator(String profileId) {
        return new FacebookIterator(this, "coworkers", profileId);
    }
    
    private class FacebookIterator implements ProfileIterator {
        private Facebook facebook;
        private String type;
        private String profileId;
        private int currentPosition = 0;
        private List<String> profileIds = new ArrayList<>();
        
        public FacebookIterator(Facebook facebook, String type, String profileId) {
            this.facebook = facebook;
            this.type = type;
            this.profileId = profileId;
            lazyLoad();
        }
        
        private void lazyLoad() {
            // Simulate loading friend/coworker IDs
            profileIds.add("friend1");
            profileIds.add("friend2");
        }
        
        @Override
        public boolean hasNext() {
            return currentPosition < profileIds.size();
        }
        
        @Override
        public Profile next() {
            if (!hasNext()) {
                return null;
            }
            String id = profileIds.get(currentPosition);
            currentPosition++;
            return facebook.profiles.get(id);
        }
    }
}
```

## When to Use

- Access aggregate object's contents without exposing internal representation
- Support multiple traversals of aggregate objects
- Provide uniform interface for traversing different aggregate structures
