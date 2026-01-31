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

## Current World Examples & Famous Products

### 1. **Social Media Feeds**
- **Facebook**: News feed pagination
- **Instagram**: Photo feed scrolling
- **Twitter/X**: Timeline iteration
- **LinkedIn**: Feed browsing
- **TikTok**: Video feed swiping

### 2. **E-Commerce Product Listings**
- **Amazon**: Search results pagination
- **eBay**: Listing browsing
- **Shopify**: Product collection iteration
- **Etsy**: Shop item browsing
- **AliExpress**: Product catalog navigation

### 3. **Music & Media Streaming**
- **Spotify**: Playlist iteration, album browsing
- **YouTube**: Video playlist playback
- **Netflix**: Content carousel browsing
- **Apple Music**: Library iteration
- **Pandora**: Station track iteration

### 4. **Email Clients**
- **Gmail**: Email list iteration, thread browsing
- **Outlook**: Inbox navigation
- **Apple Mail**: Message list scrolling
- **Thunderbird**: Folder message iteration
- **ProtonMail**: Email browsing

### 5. **File Managers**
- **Windows Explorer**: Directory iteration
- **macOS Finder**: File browsing
- **Linux file managers**: Directory traversal
- **Google Drive**: Folder navigation
- **Dropbox**: File list iteration

### 6. **Database Query Results**
- **MySQL**: ResultSet iteration
- **PostgreSQL**: Cursor-based iteration
- **MongoDB**: Cursor iteration
- **Redis**: Key scanning
- **Elasticsearch**: Search result pagination

### 7. **Programming Language Collections**
- **Java**: Iterator interface, for-each loops
- **Python**: Iterator protocol, generators
- **JavaScript**: Iterable protocol, for...of
- **C++**: STL iterators
- **C#**: IEnumerable, foreach

### 8. **Photo & Video Apps**
- **Instagram**: Story browsing
- **Snapchat**: Snap iteration
- **Google Photos**: Album browsing
- **Flickr**: Photo stream iteration
- **VSCO**: Feed scrolling

### 9. **News & Content Aggregators**
- **Reddit**: Post browsing, comment iteration
- **Flipboard**: Article browsing
- **Feedly**: RSS feed iteration
- **Apple News**: Story browsing
- **Google News**: News feed scrolling

### 10. **Messaging Apps**
- **WhatsApp**: Chat list iteration, message scrolling
- **Telegram**: Channel message browsing
- **Slack**: Channel history iteration
- **Discord**: Message history scrolling
- **Microsoft Teams**: Chat iteration

### 11. **Map Applications**
- **Google Maps**: Search results iteration
- **Apple Maps**: Place list browsing
- **Waze**: Route alternatives iteration
- **OpenStreetMap**: POI browsing
- **Mapbox**: Feature iteration

### 12. **Project Management Tools**
- **Jira**: Issue list iteration, sprint browsing
- **Trello**: Card iteration across lists
- **Asana**: Task list browsing
- **Monday.com**: Board item iteration
- **Notion**: Database record iteration

### 13. **Cloud Storage & Backup**
- **AWS S3**: Object listing iteration
- **Azure Blob Storage**: Container iteration
- **Google Cloud Storage**: Bucket browsing
- **Backblaze**: File version iteration
- **Wasabi**: Object iteration

### 14. **Gaming**
- **Steam**: Game library iteration
- **PlayStation Store**: Game catalog browsing
- **Xbox Store**: Content browsing
- **Epic Games**: Library iteration
- **GOG**: Game collection browsing

### 15. **Development Tools**
- **Git**: Commit history iteration
- **GitHub**: Repository browsing, PR iteration
- **npm**: Package dependency iteration
- **Docker**: Container list iteration
- **Kubernetes**: Pod list iteration
