# Complete Design & Architecture Patterns Cheat Sheet

## Table of Contents
1. [Creational Patterns](#creational-patterns)
2. [Structural Patterns](#structural-patterns)
3. [Behavioral Patterns](#behavioral-patterns)
4. [Architectural Patterns](#architectural-patterns)
5. [Concurrency Patterns](#concurrency-patterns)
6. [Real-World Use Cases](#real-world-use-cases)
7. [Pattern Comparison Tables](#pattern-comparison-tables)

---

## Creational Patterns

### 1. Singleton Pattern

**Intent:** Ensure a class has only one instance and provide a global point of access to it.

| Aspect | Details |
|--------|---------|
| **Implementation Types** | Eager initialization, Lazy initialization, Thread-safe, Bill Pugh, Enum |
| **Thread Safety** | Use double-checked locking, synchronized methods, or Bill Pugh approach |
| **Common Pitfalls** | Reflection attacks, Serialization issues, Cloning problems |
| **Testing Challenges** | Difficult to mock, Global state issues, Hidden dependencies |
| **Memory Impact** | Single instance throughout application lifetime |
| **Initialization Cost** | Can be expensive if eager, delayed if lazy |

**Real-World Examples:**
- Database connection pool manager
- Logger (Log4j, SLF4J)
- Configuration manager
- Device drivers
- Cache manager
- Thread pool
- Window managers in GUI

**Code Indicators:**
```
✓ private constructor
✓ static getInstance() method
✓ static instance variable
✗ Multiple instances possible
```

---

### 2. Factory Method Pattern

**Intent:** Define an interface for creating objects, but let subclasses decide which class to instantiate.

| Aspect | Details |
|--------|---------|
| **Decoupling Level** | High - clients don't know concrete classes |
| **Extensibility** | Easy to add new product types |
| **Complexity** | Moderate - requires parallel class hierarchy |
| **Polymorphism** | Uses subclassing for object creation |
| **Parallel Hierarchies** | Creator hierarchy + Product hierarchy |
| **Common Variants** | Parameterized factory, Simple factory |

**Real-World Examples:**
- Document creation in word processors (PDFDocument, WordDocument)
- GUI frameworks (WindowsButton, MacButton)
- Database connections (MySQLConnection, PostgreSQLConnection)
- Logistics systems (TruckDelivery, ShipDelivery)
- Payment processors (CreditCardPayment, PayPalPayment)

**Use Case Matrix:**

| Scenario | Factory Method | Abstract Factory | Builder |
|----------|---------------|------------------|---------|
| Single product family | ✓ | ✗ | ✗ |
| Multiple product families | ✗ | ✓ | ✗ |
| Complex construction | ✗ | ✗ | ✓ |
| Runtime type selection | ✓ | ✓ | ✗ |

---

### 3. Abstract Factory Pattern

**Intent:** Provide an interface for creating families of related objects without specifying concrete classes.

| Aspect | Details |
|--------|---------|
| **Product Families** | Creates multiple related products |
| **Consistency** | Ensures products from same family are used together |
| **Scalability** | Difficult to add new products, easy to add families |
| **Coupling** | Very loose - clients don't know concrete products |
| **Factory Hierarchy** | Often has multiple concrete factories |
| **Common Use** | Cross-platform applications, UI toolkits |

**Real-World Examples:**
- UI libraries (Windows UI, Mac UI, Linux UI themes)
- Database access layers (SQL Server, Oracle, MySQL factories)
- Game engines (2D renderer, 3D renderer with related assets)
- Report generators (PDF, Excel, HTML output formats)
- E-commerce platforms (Different payment, shipping, inventory systems by region)

**Product Family Example:**

| Factory Type | Button | Checkbox | Textbox |
|-------------|--------|----------|---------|
| Windows | WindowsButton | WindowsCheckbox | WindowsTextbox |
| Mac | MacButton | MacCheckbox | MacTextbox |
| Linux | LinuxButton | LinuxCheckbox | LinuxTextbox |

---

### 4. Builder Pattern

**Intent:** Separate construction of complex object from its representation.

| Aspect | Details |
|--------|---------|
| **Construction Steps** | Multi-step construction process |
| **Immutability** | Often produces immutable objects |
| **Fluent Interface** | Commonly uses method chaining |
| **Director** | Optional class for standard construction sequences |
| **Validation** | Can validate during or after building |
| **Use Frequency** | Very common in modern codebases |

**Real-World Examples:**
- StringBuilder, StringBuffer
- SQL query builders
- HTTP request builders
- Configuration builders
- Email composers
- Meal builders (fast food ordering)
- Resume/CV builders
- Test data builders

**Telescoping Constructor Problem:**
```
Before Builder:
Person(name, age, phone, address, email, city, state, zip, country)

After Builder:
Person.builder()
    .name("John")
    .age(30)
    .phone("123-456")
    .build()
```

---

### 5. Prototype Pattern

**Intent:** Create new objects by copying existing ones (prototypes).

| Aspect | Details |
|--------|---------|
| **Cloning Type** | Shallow copy vs Deep copy |
| **Performance** | Faster than creating from scratch |
| **Registry** | Often maintains prototype registry |
| **Circular References** | Challenging with deep copy |
| **Use Cases** | When instantiation is expensive |
| **Language Support** | Java Cloneable, JavaScript prototypes |

**Real-World Examples:**
- Document templates
- Game character cloning (with different attributes)
- Database record duplication
- Configuration presets
- Cell division simulation
- Graphic editor shapes
- Form templates

**Deep vs Shallow Copy:**

| Scenario | Shallow Copy | Deep Copy |
|----------|-------------|-----------|
| Simple objects | ✓ Sufficient | ✗ Overkill |
| Nested objects | ✗ Shares references | ✓ Independent copies |
| Performance | ✓ Fast | ✗ Slower |
| Memory | ✓ Less usage | ✗ More usage |

---

## Structural Patterns

### 6. Adapter Pattern

**Intent:** Convert interface of a class into another interface clients expect.

| Aspect | Details |
|--------|---------|
| **Adaptation Type** | Class adapter (inheritance), Object adapter (composition) |
| **Direction** | One-way or two-way adapter |
| **Interface Conversion** | Translates between incompatible interfaces |
| **Legacy Integration** | Primary use for legacy system integration |
| **Flexibility** | Object adapter more flexible |
| **Common Scenario** | Third-party library integration |

**Real-World Examples:**
- Power adapters (110V to 220V)
- Card readers (SD to USB)
- Media players (play multiple formats)
- Database drivers (JDBC adapters)
- XML to JSON converters
- Legacy API wrappers
- Third-party library integration (Stripe, PayPal adapters)

**Adapter Types Comparison:**

| Feature | Class Adapter | Object Adapter |
|---------|--------------|----------------|
| Implementation | Inheritance | Composition |
| Flexibility | Less flexible | More flexible |
| Multiple adaptees | Not possible | Possible |
| Override behavior | Easy | Through adaptee |
| Language support | Requires multiple inheritance | Works everywhere |

---

### 7. Bridge Pattern

**Intent:** Decouple abstraction from implementation so both can vary independently.

| Aspect | Details |
|--------|---------|
| **Hierarchies** | Two separate hierarchies |
| **Flexibility** | Both abstraction and implementation extensible |
| **Runtime Binding** | Implementation can be changed at runtime |
| **Platform Independence** | Excellent for cross-platform code |
| **Complexity** | Higher initial complexity |
| **vs Adapter** | Designed upfront vs retrofitted |

**Real-World Examples:**
- Remote controls for different devices (TV, DVD, Sound System)
- Graphics rendering (Windows, Linux, Mac implementations)
- Database drivers (abstraction: Database, implementation: MySQL, PostgreSQL)
- Message senders (Email, SMS with different providers)
- Drawing shapes with different rendering APIs (OpenGL, DirectX)
- Vehicle types with different engines

**Bridge Structure:**

```
Abstraction          Implementation
    |                      |
    |                      |
Shape                 Renderer
├── Circle            ├── VectorRenderer
└── Rectangle         └── RasterRenderer
```

---

### 8. Composite Pattern

**Intent:** Compose objects into tree structures to represent part-whole hierarchies.

| Aspect | Details |
|--------|---------|
| **Structure** | Tree-like structure |
| **Uniformity** | Treats individual and composite objects uniformly |
| **Recursion** | Operations often recursive |
| **Type Safety** | Can have type safety issues |
| **Common Operations** | Add, remove, get child, operation |
| **Complexity** | Can become complex with many node types |

**Real-World Examples:**
- File system (files and directories)
- Organization hierarchy
- GUI components (containers and widgets)
- Menu systems
- XML/HTML DOM
- Graphics scene graphs
- Expression trees (mathematical expressions)

**File System Example:**

| Component | Type | Children? | Example |
|-----------|------|-----------|---------|
| File | Leaf | No | document.txt |
| Directory | Composite | Yes | /home/user/ |
| Shortcut | Leaf/Composite | Depends | link to folder |

**Operations:**

```
Component
├── size()
├── print()
└── search()

Composite implements all + manage children
Leaf implements all (no children)
```

---

### 9. Decorator Pattern

**Intent:** Attach additional responsibilities to object dynamically.

| Aspect | Details |
|--------|---------|
| **Wrapping** | Wraps objects recursively |
| **Transparency** | Same interface as wrapped object |
| **Flexibility** | Add/remove responsibilities at runtime |
| **Order** | Order of decorators can matter |
| **vs Inheritance** | More flexible than static inheritance |
| **Common Issue** | Identity problems (decorated != original) |

**Real-World Examples:**
- Java I/O streams (BufferedReader, FileReader)
- UI scrollbars, borders
- Pizza toppings
- Coffee condiments
- Text formatting (bold, italic, underline)
- Encryption/compression layers
- Caching layers

**Decorator Stacking:**

```
Component
└── ConcreteComponent (Coffee: $2)
    └── Decorator1 (Milk: +$0.50)
        └── Decorator2 (Sugar: +$0.25)
            └── Decorator3 (Whipped Cream: +$0.75)
            
Total: $3.50
```

**Common Combinations:**

| Base | Decorator 1 | Decorator 2 | Decorator 3 | Result |
|------|------------|------------|------------|--------|
| Coffee | Milk | Sugar | Caramel | Caramel Latte |
| Text | Bold | Italic | Underline | Formatted Text |
| Stream | Buffer | Compress | Encrypt | Secure Buffered Stream |

---

### 10. Facade Pattern

**Intent:** Provide unified interface to set of interfaces in subsystem.

| Aspect | Details |
|--------|---------|
| **Simplification** | Simplifies complex subsystem |
| **Layering** | Often used for layering |
| **Direct Access** | Doesn't prevent direct subsystem access |
| **Loose Coupling** | Reduces dependencies |
| **Additional Layer** | Adds indirection layer |
| **God Object Risk** | Can become too large |

**Real-World Examples:**
- Home theater system controller
- Computer startup (CPU, Memory, HDD coordination)
- E-commerce checkout process
- Hotel booking system
- Car ignition system
- Compiler (lexer, parser, code generator facade)
- Framework APIs

**Complex Subsystem Example:**

| Without Facade | With Facade |
|----------------|-------------|
| AudioSystem.on() | HomeTheater.watchMovie() |
| AudioSystem.setVolume(5) | (handles all) |
| Projector.on() | |
| Projector.wideScreen() | |
| DVDPlayer.on() | |
| DVDPlayer.play(movie) | |
| Lights.dim(10%) | |

---

### 11. Flyweight Pattern

**Intent:** Use sharing to support large numbers of fine-grained objects efficiently.

| Aspect | Details |
|--------|---------|
| **State Division** | Intrinsic (shared) vs Extrinsic (unique) |
| **Memory Optimization** | Significant memory savings |
| **Factory** | Usually uses factory for management |
| **Immutability** | Intrinsic state should be immutable |
| **Identification** | Objects identified by intrinsic state |
| **Common Use** | Text editors, game engines |

**Real-World Examples:**
- Text editor character formatting
- Game forests (tree types shared, positions unique)
- Web browser rendering (font instances)
- String interning (Java String pool)
- Connection pools
- Object pools in games
- Icon caching

**State Separation:**

| Object Type | Intrinsic State (Shared) | Extrinsic State (Unique) |
|-------------|-------------------------|-------------------------|
| Tree | TreeType, texture, color | Position (x, y) |
| Character | Font, size, style | Position in document |
| Particle | Image, color | Position, velocity |
| Chess Piece | Color, type, sprite | Board position |

**Memory Impact:**

```
Without Flyweight:
1,000,000 trees × 50KB = 50GB

With Flyweight:
10 tree types × 50KB = 500KB
1,000,000 positions × 16 bytes = 16MB
Total: ~16.5MB (99.97% reduction)
```

---

### 12. Proxy Pattern

**Intent:** Provide surrogate or placeholder for another object to control access.

| Aspect | Details |
|--------|---------|
| **Proxy Types** | Virtual, Remote, Protection, Smart reference, Cache |
| **Transparency** | Can be transparent to client |
| **Lazy Loading** | Delays expensive operations |
| **Access Control** | Controls who can access |
| **Forwarding** | Forwards requests to real subject |
| **Common Use** | Security, caching, lazy loading |

**Real-World Examples:**
- Virtual proxy: Image loading
- Remote proxy: RPC, web services
- Protection proxy: Access control systems
- Smart reference: Reference counting
- Caching proxy: Database query results
- Logging proxy: Audit trails
- Credit card (proxy for bank account)

**Proxy Types Comparison:**

| Proxy Type | Purpose | Example | When Created |
|------------|---------|---------|--------------|
| Virtual | Lazy initialization | Large image loading | On demand |
| Remote | Network communication | Web service client | At startup |
| Protection | Access control | Admin-only methods | At startup |
| Cache | Performance | Database results | At startup |
| Smart Reference | Memory management | Ref counting | At startup |
| Logging | Auditing | Method call logging | At startup |

---

## Behavioral Patterns

### 13. Chain of Responsibility Pattern

**Intent:** Avoid coupling sender to receiver by giving multiple objects chance to handle request.

| Aspect | Details |
|--------|---------|
| **Chain Structure** | Linked list of handlers |
| **Flexibility** | Dynamic chain configuration |
| **Coupling** | Loose coupling sender-receiver |
| **Handling** | Any handler can process or pass on |
| **Termination** | Request may go unhandled |
| **Order Matters** | Handler order affects behavior |

**Real-World Examples:**
- Event handling in GUI (event bubbling)
- Logging frameworks (different log levels)
- Servlet filters in web applications
- Exception handling
- Approval workflows (manager → director → VP → CEO)
- Tech support escalation
- ATM money dispensing

**Chain Configuration:**

| Request Type | Handler 1 | Handler 2 | Handler 3 | Handler 4 |
|--------------|-----------|-----------|-----------|-----------|
| Purchase < $1000 | Manager ✓ | - | - | - |
| Purchase < $5000 | Manager ✗ | Director ✓ | - | - |
| Purchase < $25000 | Manager ✗ | Director ✗ | VP ✓ | - |
| Purchase > $25000 | Manager ✗ | Director ✗ | VP ✗ | CEO ✓ |

---

### 14. Command Pattern

**Intent:** Encapsulate request as object, allowing parameterization and queuing.

| Aspect | Details |
|--------|---------|
| **Encapsulation** | Request as first-class object |
| **Decoupling** | Invoker decoupled from receiver |
| **Undo/Redo** | Natural fit for undo operations |
| **Queuing** | Commands can be queued |
| **Logging** | Command history for audit |
| **Macro Commands** | Composite commands possible |

**Real-World Examples:**
- Text editor operations (cut, copy, paste)
- GUI buttons and menu items
- Database transactions
- Task schedulers
- Remote control buttons
- Macro recording
- Job queues
- Wizard steps

**Command Components:**

| Component | Responsibility | Example |
|-----------|---------------|---------|
| Command | Interface | execute(), undo() |
| ConcreteCommand | Implements command | PasteCommand |
| Receiver | Performs action | Document |
| Invoker | Triggers command | Button, MenuItem |
| Client | Creates command | Application |

**Undo/Redo Stack:**

```
Action Stack:
[Type "Hello"] → [Delete "o"] → [Type "p"] → [Undo] → [Undo] → [Redo]

State progression:
"" → "Hello" → "Hell" → "Hellp" → "Hell" → "Hello" → "Hell"
```

---

### 15. Iterator Pattern

**Intent:** Provide way to access elements sequentially without exposing representation.

| Aspect | Details |
|--------|---------|
| **Traversal** | Sequential access abstraction |
| **Encapsulation** | Hides internal structure |
| **Multiple Iterators** | Can have concurrent iterations |
| **Iterator Types** | Forward, bidirectional, random access |
| **Language Support** | Built-in in most modern languages |
| **Modification** | Concurrent modification issues |

**Real-World Examples:**
- Java Iterator, C++ iterators
- Database result sets
- File system traversal
- Social media feed scrolling
- Playlist navigation
- Menu navigation
- Search results pagination

**Iterator Types:**

| Type | Operations | Use Case | Example |
|------|-----------|----------|---------|
| Forward | next(), hasNext() | Simple traversal | Singly linked list |
| Bidirectional | next(), previous() | Back and forth | Doubly linked list |
| Random Access | get(index) | Direct access | ArrayList |
| External | Client controls | Explicit iteration | Java Iterator |
| Internal | Iterator controls | Callback-based | forEach |

---

### 16. Mediator Pattern

**Intent:** Define object that encapsulates how set of objects interact.

| Aspect | Details |
|--------|---------|
| **Centralization** | Centralizes complex communications |
| **Loose Coupling** | Colleagues don't reference each other |
| **Reusability** | Easier to reuse colleagues |
| **God Object Risk** | Mediator can become too complex |
| **Single Point** | Single point for interaction logic |
| **Common Use** | Dialog boxes, chat rooms |

**Real-World Examples:**
- Air traffic control
- Chat room
- Dialog boxes (form validation)
- Model-View-Controller (Controller is mediator)
- Auction system
- Social media platform
- Smart home hub

**Without vs With Mediator:**

| Scenario | Without Mediator | With Mediator |
|----------|-----------------|---------------|
| Dependencies | n × (n-1) | n × 1 |
| Colleagues (n=5) | 20 connections | 5 connections |
| Add new colleague | Update all colleagues | Update mediator only |
| Change interaction | Modify multiple classes | Modify mediator |

**Chat Room Example:**

```
Without Mediator:
User1 ←→ User2 ←→ User3 ←→ User4
(Every user knows every other user)

With Mediator:
User1 → ChatRoom ← User2
         ↑   ↓
       User3 User4
(Users only know ChatRoom)
```

---

### 17. Memento Pattern

**Intent:** Capture and externalize object's internal state for later restoration.

| Aspect | Details |
|--------|---------|
| **Encapsulation** | Preserves encapsulation boundaries |
| **Roles** | Originator, Memento, Caretaker |
| **State Capture** | Snapshot of internal state |
| **Undo Support** | Natural for undo/redo |
| **Memory** | Can be memory intensive |
| **Serialization** | Often uses serialization |

**Real-World Examples:**
- Text editor undo/redo
- Game save states
- Database transactions (rollback)
- Browser history (back button)
- Version control systems
- Photoshop history
- Form draft saving

**Pattern Roles:**

| Role | Responsibility | Access to State | Example |
|------|---------------|----------------|---------|
| Originator | Creates and uses memento | Full | Document |
| Memento | Stores state | None (opaque) | DocumentState |
| Caretaker | Manages mementos | None | UndoManager |

**State Management:**

```
Operation Timeline:
State 0 → Action 1 → State 1 → Action 2 → State 2 → Undo → State 1

Memento Stack:
[State 0] → [State 1] → [State 2]
                         ↓ Undo
                      [State 1]
```

---

### 18. Observer Pattern

**Intent:** Define one-to-many dependency so when one object changes, all dependents notified.

| Aspect | Details |
|--------|---------|
| **Also Known As** | Pub-Sub, Event-Listener |
| **Notification** | Automatic notification mechanism |
| **Coupling** | Loose coupling between subject and observers |
| **Push vs Pull** | Data can be pushed or pulled |
| **Order** | Notification order not guaranteed |
| **Memory Leaks** | Common if observers not removed |

**Real-World Examples:**
- Event handling systems
- Model-View-Controller (Model → Views)
- Newsletter subscriptions
- Social media notifications
- Stock price updates
- RSS feeds
- Real-time dashboards
- Spreadsheet cell updates

**Push vs Pull Model:**

| Aspect | Push Model | Pull Model |
|--------|-----------|-----------|
| Data transfer | Subject sends data | Observer requests data |
| Efficiency | Less efficient (unnecessary data) | More efficient |
| Coupling | Higher (subject knows what to send) | Lower |
| Flexibility | Less flexible | More flexible |
| Example | `update(data)` | `update()` then `subject.getData()` |

**Common Issues:**

| Issue | Cause | Solution |
|-------|-------|----------|
| Memory Leak | Observers not unregistered | Weak references, explicit cleanup |
| Update Storms | Cascading updates | Batch notifications |
| Order Dependency | Observers expect order | Explicit priority system |
| Unexpected Updates | Hidden dependencies | Clear documentation |

---

### 19. State Pattern

**Intent:** Allow object to alter behavior when internal state changes.

| Aspect | Details |
|--------|---------|
| **State Encapsulation** | Each state is separate class |
| **Transitions** | Explicit state transitions |
| **Context** | Maintains current state |
| **vs Conditional** | Eliminates large switch/if statements |
| **Extensibility** | Easy to add new states |
| **Common Use** | Workflow engines, game AI |

**Real-World Examples:**
- Vending machine states
- TCP connection states
- Document approval workflow
- Media player (playing, paused, stopped)
- Order processing (pending, shipped, delivered)
- Traffic light
- Turnstile/Gate

**State Transition Example:**

| Current State | Event | Next State | Action |
|--------------|-------|------------|--------|
| Draft | Submit | Moderation | Send to moderator |
| Moderation | Approve | Published | Publish |
| Moderation | Reject | Draft | Notify author |
| Published | Unpublish | Draft | Remove from site |

**Vending Machine States:**

```
States: NoQuarter, HasQuarter, Sold, SoldOut

Transitions:
NoQuarter --insert quarter--> HasQuarter
HasQuarter --eject quarter--> NoQuarter
HasQuarter --turn crank--> Sold
Sold --dispense--> NoQuarter (if items left)
Sold --dispense--> SoldOut (if no items)
```

---

### 20. Strategy Pattern

**Intent:** Define family of algorithms, encapsulate each, make them interchangeable.

| Aspect | Details |
|--------|---------|
| **Algorithm Family** | Related algorithms grouped |
| **Runtime Selection** | Choose algorithm at runtime |
| **Encapsulation** | Algorithms encapsulated in classes |
| **Composition** | Uses composition over inheritance |
| **Context** | Maintains reference to strategy |
| **Common Use** | Sorting, compression, validation |

**Real-World Examples:**
- Payment methods (CreditCard, PayPal, Bitcoin)
- Sorting algorithms (QuickSort, MergeSort, BubbleSort)
- Compression algorithms (ZIP, RAR, TAR)
- Route calculation (shortest, fastest, scenic)
- Pricing strategies (regular, holiday, clearance)
- Authentication methods (OAuth, JWT, Basic)

**Strategy Selection Matrix:**

| Context | Strategy Options | Selection Criteria |
|---------|-----------------|-------------------|
| Navigation | Driving, Walking, Transit | User preference, distance |
| Payment | Card, PayPal, Crypto | User choice, region |
| Shipping | Standard, Express, Overnight | Speed requirement, cost |
| Compression | None, Fast, Maximum | Performance vs size tradeoff |
| Validation | Strict, Lenient, Custom | Business rules |

**Benefits Over Conditionals:**

```
Before (Conditionals):
if (paymentType == "credit") {
    // credit card logic
} else if (paymentType == "paypal") {
    // paypal logic
} else if (paymentType == "crypto") {
    // crypto logic
}

After (Strategy):
paymentStrategy.pay(amount);
// Strategy selected at runtime, no conditionals
```

---

### 21. Template Method Pattern

**Intent:** Define skeleton of algorithm, deferring some steps to subclasses.

| Aspect | Details |
|--------|---------|
| **Hollywood Principle** | "Don't call us, we'll call you" |
| **Inheritance** | Uses inheritance, not composition |
| **Hook Methods** | Optional methods subclasses can override |
| **Invariant Steps** | Fixed steps in template method |
| **Code Reuse** | Promotes code reuse |
| **Inversion of Control** | Framework calls application code |

**Real-World Examples:**
- Game AI (initialize, start, play, end)
- Data mining algorithms
- Beverage preparation (tea, coffee)
- Home building process
- Banking operations
- Test frameworks (setup, test, teardown)
- Web framework lifecycle

**Template Structure:**

| Method Type | Override? | Example | Purpose |
|-------------|----------|---------|---------|
| Template Method | No | buildHouse() | Defines algorithm |
| Abstract Operation | Yes | buildWalls() | Must implement |
| Hook Operation | Optional | addGarage() | Can implement |
| Common Operation | No | buyLand() | Used by all |

**Beverage Example:**

```
Template Method: prepareRecipe()
1. boilWater() [common]
2. brew() [abstract - coffee: brewCoffee, tea: steepTea]
3. pourInCup() [common]
4. addCondiments() [abstract - coffee: addSugarAndMilk, tea: addLemon]
5. addExtras() [hook - optional]
```

---

### 22. Visitor Pattern

**Intent:** Represent operation on elements of object structure, defining new operations without changing elements.

| Aspect | Details |
|--------|---------|
| **Double Dispatch** | Uses double dispatch technique |
| **Separation** | Separates algorithm from object structure |
| **Extensibility** | Easy to add operations, hard to add elements |
| **Element Access** | May break encapsulation |
| **Accumulation** | Can accumulate state during traversal |
| **Complexity** | One of most complex patterns |

**Real-World Examples:**
- Compiler (parse tree operations)
- Tax calculation for different items
- Export to different formats
- Rendering graphics objects
- Shopping cart pricing
- Insurance claim processing
- Code analyzers/linters

**Visitor vs Other Patterns:**

| Aspect | Visitor | Strategy | Command |
|--------|---------|----------|---------|
| Purpose | Operations on structure | Algorithm selection | Encapsulate request |
| Elements | Multiple types | Single context | Single receiver |
| Extension | Easy add operations | Easy add strategies | Easy add commands |
| Coupling | Higher | Lower | Lower |

**Shopping Cart Example:**

| Item Type | Visitor: PriceCalculator | Visitor: TaxCalculator | Visitor: WeightCalculator |
|-----------|------------------------|----------------------|------------------------|
| Book | Price × quantity | Price × tax rate | Weight × quantity |
| Fruit | Price × weight | (Price × weight) × tax | Weight |
| Electronics | Price + shipping | Price × higher tax | Weight + packaging |

---

## Architectural Patterns

### 23. Layered Architecture (N-Tier)

**Intent:** Organize system into layers with specific responsibilities.

| Aspect | Details |
|--------|---------|
| **Layer Types** | Presentation, Business Logic, Data Access, Database |
| **Communication** | Each layer talks to adjacent layers |
| **Strict Layering** | Can only call layer directly below |
| **Relaxed Layering** | Can call any lower layer |
| **Deployment** | Can be distributed or monolithic |
| **Scalability** | Vertical scaling easier than horizontal |

**Common Layer Configurations:**

| Configuration | Layers | Use Case |
|--------------|--------|----------|
| 3-Tier | Presentation, Logic, Data | Traditional enterprise apps |
| 4-Tier | Presentation, Application, Domain, Infrastructure | Domain-driven design |
| 5-Tier | Presentation, Service, Business, Data Access, Database | Service-oriented apps |

**Real-World Examples:**
- Enterprise web applications
- Banking systems
- E-commerce platforms
- ERP systems
- CRM systems
- Content management systems

**Layer Responsibilities:**

| Layer | Responsibility | Technologies | Testability |
|-------|---------------|--------------|-------------|
| Presentation | UI, input validation | HTML, CSS, React, Angular | Integration tests |
| Business Logic | Business rules, workflows | Java, C#, Python | Unit tests |
| Data Access | CRUD operations | ORM (Hibernate, EF) | Unit tests |
| Database | Data persistence | MySQL, PostgreSQL, MongoDB | Integration tests |

---

### 24. Model-View-Controller (MVC)

**Intent:** Separate application into Model (data), View (UI), Controller (input logic).

| Aspect | Details |
|--------|---------|
| **Separation** | Clear separation of concerns |
| **Components** | Model, View, Controller |
| **Data Flow** | Controller → Model → View |
| **Observation** | View observes model |
| **Testing** | Easy to test components separately |
| **Variants** | MVP, MVVM, MVA |

**Real-World Examples:**
- Ruby on Rails applications
- Spring MVC (Java)
- ASP.NET MVC
- Django (Python)
- Express.js (Node.js)
- iOS applications
- Android applications

**MVC Variants Comparison:**

| Pattern | View Awareness | Controller Role | Best For |
|---------|---------------|-----------------|----------|
| MVC | Knows Model | Handles input | Web apps |
| MVP | Passive | Presenter mediates | Desktop apps |
| MVVM | Data binding | Minimal controller | Rich clients |
| MVA | Actions as controllers | Lightweight | Modern web |

**Component Interactions:**

```
User Input → Controller
            ↓
Controller → Model (updates)
            ↓
Model → View (notifies)
            ↓
View → User (renders)
```

**Responsibilities:**

| Component | Responsibilities | Examples |
|-----------|-----------------|----------|
| Model | Business logic, data, state | User, Product, Order classes |
| View | Display, UI rendering | HTML templates, JSP, React components |
| Controller | Input handling, flow control | UserController, OrderController |

---

### 25. Model-View-ViewModel (MVVM)

**Intent:** Facilitate UI development separation through data binding.

| Aspect | Details |
|--------|---------|
| **Data Binding** | Two-way binding between View and ViewModel |
| **ViewModel** | Abstraction of View |
| **Testability** | Excellent - can test without UI |
| **Frameworks** | WPF, Angular, Vue, Knockout |
| **State Management** | ViewModel maintains state |
| **Learning Curve** | Steeper than MVC |

**Real-World Examples:**
- WPF applications
- Angular applications
- Vue.js applications
- Xamarin mobile apps
- Knockout.js apps
- Blazor applications

**MVVM vs MVC:**

| Aspect | MVVM | MVC |
|--------|------|-----|
| View-Model relation | Data binding | Manual updates |
| Controller presence | No explicit controller | Explicit controller |
| Testability | Very high | High |
| Code in View | Minimal | Can be substantial |
| Best for | Desktop/SPA | Server-rendered web |
| Learning curve | Steeper | Gentler |

**Data Binding Example:**

```
View (HTML):
<input [(ngModel)]="user.name">
<div>{{user.name}}</div>

ViewModel (TypeScript):
user = { name: "" };
// Automatic sync - changes in either direction propagate
```

---

### 26. Microservices Architecture

**Intent:** Structure application as collection of loosely coupled, independently deployable services.

| Aspect | Details |
|--------|---------|
| **Service Size** | Small, focused services |
| **Communication** | REST, gRPC, message queues |
| **Data** | Database per service |
| **Deployment** | Independent deployment |
| **Scalability** | Scale services independently |
| **Complexity** | High operational complexity |

**Real-World Examples:**
- Netflix
- Amazon
- Uber
- Spotify
- Twitter
- Airbnb

**Microservices vs Monolith:**

| Aspect | Microservices | Monolith |
|--------|--------------|----------|
| Deployment | Independent | All-or-nothing |
| Scaling | Service-level | Application-level |
| Technology | Polyglot | Uniform |
| Team structure | Small, autonomous | Larger, coordinated |
| Testing | Complex integration | Simpler |
| Initial complexity | Higher | Lower |
| Long-term flexibility | Higher | Lower |
| Data consistency | Eventual | Immediate |

**Service Decomposition Strategies:**

| Strategy | Description | Example |
|----------|-------------|---------|
| Business capability | By business function | User Service, Payment Service, Inventory |
| Subdomain | By DDD subdomain | Order Management, Catalog, Shipping |
| Transaction | By transaction type | Read Service, Write Service |
| Team ownership | By team boundaries | Team A services, Team B services |

**Communication Patterns:**

| Pattern | Sync/Async | Use Case | Example |
|---------|-----------|----------|---------|
| REST API | Sync | Simple CRUD | HTTP GET /users |
| Message Queue | Async | Event-driven | RabbitMQ, Kafka |
| gRPC | Sync | High performance | Service-to-service |
| Event Bus | Async | Broadcasting | Order placed event |

---

### 27. Event-Driven Architecture

**Intent:** Produce, detect, consume, and react to events in loosely coupled manner.

| Aspect | Details |
|--------|---------|
| **Event Types** | Simple events, Event streams, Complex events |
| **Components** | Event producers, consumers, channels |
| **Communication** | Asynchronous |
| **Coupling** | Very loose |
| **Scalability** | Excellent |
| **Consistency** | Eventual consistency |

**Real-World Examples:**
- Real-time analytics platforms
- IoT systems
- Stock trading platforms
- Social media feeds
- Gaming platforms
- Monitoring systems
- E-commerce order processing

**Event Types:**

| Type | Description | Example | Processing |
|------|-------------|---------|------------|
| Simple Event | Single occurrence | Button clicked | Immediate |
| Event Stream | Continuous events | Sensor readings | Windowed |
| Complex Event | Pattern of events | Fraud detection | Pattern matching |
| Domain Event | Business event | Order placed | Business logic |

**Event Processing Styles:**

| Style | Description | Use Case | Latency |
|-------|-------------|----------|---------|
| Simple Processing | One event → one action | User login | Very low |
| Stream Processing | Continuous processing | Real-time analytics | Low |
| Complex Processing | Pattern detection | Fraud detection | Medium |
| Batch Processing | Accumulated events | Nightly reports | High |

**Event Flow Example:**

```
Producer: Order Service
Event: OrderPlaced(orderId, userId, amount)

Consumers:
- Inventory Service → Reserve items
- Payment Service → Process payment
- Notification Service → Email confirmation
- Analytics Service → Update statistics
- Shipping Service → Prepare shipment
```

---

### 28. Hexagonal Architecture (Ports and Adapters)

**Intent:** Create application with loosely coupled, testable components.

| Aspect | Details |
|--------|---------|
| **Core** | Business logic in center |
| **Ports** | Interfaces defining entry/exit points |
| **Adapters** | Implementations of ports |
| **Dependency Direction** | Always toward center |
| **Testing** | Very easy to test |
| **Also Called** | Onion Architecture, Clean Architecture |

**Real-World Examples:**
- Enterprise applications with complex domains
- Applications requiring multiple UIs
- Systems needing extensive testing
- Long-lived applications
- Domain-driven design implementations

**Architecture Layers:**

| Layer | Contains | Dependencies | Examples |
|-------|----------|--------------|----------|
| Domain | Business logic, entities | None | User, Order, Policy |
| Application | Use cases | Domain only | CreateOrder, ProcessPayment |
| Infrastructure | Technical details | Application | Database, HTTP, Email |
| Presentation | UI, APIs | Application | REST API, Web UI, CLI |

**Ports vs Adapters:**

| Concept | Type | Purpose | Example |
|---------|------|---------|---------|
| Port | Interface | Defines contract | PaymentPort |
| Adapter | Implementation | Connects to external | StripeAdapter |
| Primary Port | Inbound | Entry to app | REST Controller |
| Secondary Port | Outbound | Exit from app | Database Repository |

**Benefits Matrix:**

| Concern | Traditional | Hexagonal |
|---------|------------|-----------|
| Testability | Framework-dependent | Framework-independent |
| Technology change | Ripple effects | Isolated to adapters |
| Business logic | Mixed with tech | Pure, isolated |
| Multiple UIs | Difficult | Easy |

---

### 29. Service-Oriented Architecture (SOA)

**Intent:** Structure application as collection of services with well-defined interfaces.

| Aspect | Details |
|--------|---------|
| **Service Granularity** | Coarser than microservices |
| **Communication** | SOAP, REST, ESB |
| **Reusability** | High service reusability |
| **ESB** | Often uses Enterprise Service Bus |
| **Governance** | Strong governance model |
| **Standards** | Heavy on standards (WS-*) |

**Real-World Examples:**
- Banking enterprise systems
- Insurance platforms
- Telecommunications systems
- Government systems
- Healthcare information systems
- Supply chain management

**SOA vs Microservices:**

| Aspect | SOA | Microservices |
|--------|-----|---------------|
| Service size | Larger | Smaller |
| Data sharing | Shared databases common | Database per service |
| Communication | ESB, SOAP | REST, lightweight |
| Governance | Centralized | Decentralized |
| Deployment | Coordinated | Independent |
| Reusability focus | High | Moderate |
| Complexity | Higher | Lower per service |

**SOA Components:**

| Component | Purpose | Example |
|-----------|---------|---------|
| Service | Business functionality | Customer Service, Order Service |
| ESB | Message routing, transformation | Mule ESB, WSO2 |
| Service Registry | Service discovery | UDDI, Eureka |
| Orchestration | Workflow management | BPEL, Apache Camel |
| Security | Authentication, authorization | WS-Security, OAuth |

---

### 30. CQRS (Command Query Responsibility Segregation)

**Intent:** Separate read and write operations into different models.

| Aspect | Details |
|--------|---------|
| **Separation** | Reads and writes use different models |
| **Scalability** | Scale reads and writes independently |
| **Optimization** | Optimize each model separately |
| **Complexity** | Increased complexity |
| **Consistency** | Eventual consistency common |
| **Pair With** | Often used with Event Sourcing |

**Real-World Examples:**
- E-commerce platforms (product catalog)
- Social media feeds
- Financial trading systems
- Analytics dashboards
- Collaborative editing tools
- Booking systems

**Command vs Query:**

| Aspect | Command (Write) | Query (Read) |
|--------|----------------|--------------|
| Purpose | Change state | Return data |
| Return value | Void or status | Data |
| Side effects | Yes | No |
| Validation | Complex | Simple |
| Model | Normalized | Denormalized |
| Optimization | Writes | Reads |

**Use Case Analysis:**

| Scenario | Traditional | CQRS | Benefit |
|----------|------------|------|---------|
| E-commerce product view | Same model | Separate read model | Faster queries |
| Blog comments | Same model | Event-sourced | Full history |
| User dashboard | Same model | Materialized views | Performance |
| Reporting | Same model | Read replicas | No write impact |

**Implementation Patterns:**

```
Simple CQRS:
Write Model → Database ← Read Model (same DB)

Advanced CQRS:
Write Model → Write DB
            ↓ events
         Read Model → Read DB (optimized)

CQRS + Event Sourcing:
Commands → Events → Event Store
                   ↓ projections
                 Read Models
```

---

### 31. Event Sourcing

**Intent:** Store all changes as sequence of events rather than current state.

| Aspect | Details |
|--------|---------|
| **Storage** | Events, not state |
| **Immutability** | Events are immutable |
| **Replay** | Can replay events to rebuild state |
| **Audit Trail** | Complete history |
| **Temporal Queries** | Query state at any point in time |
| **Complexity** | High implementation complexity |

**Real-World Examples:**
- Version control systems (Git)
- Banking transactions
- Audit systems
- Collaborative editing (Google Docs)
- Blockchain
- Gaming state management
- Workflow engines

**Event Sourcing vs Traditional:**

| Aspect | Traditional | Event Sourcing |
|--------|------------|----------------|
| Storage | Current state | Event log |
| History | Lost or separate | Built-in |
| Debugging | Difficult | Easy (replay) |
| Audit | Extra effort | Free |
| Complexity | Lower | Higher |
| Queries | Fast | Requires projection |
| Data loss | Possible | Very difficult |

**Event Types:**

| Event Type | Example | Purpose |
|-----------|---------|---------|
| Domain Event | OrderPlaced | Business event |
| System Event | EmailSent | Technical event |
| Integration Event | PaymentReceived | External system |
| Snapshot Event | AccountSnapshot | Performance optimization |

**Event Schema Evolution:**

| Strategy | Description | Complexity | Use When |
|----------|-------------|------------|----------|
| Upcasting | Transform old events on read | Low | Few changes |
| Versioning | Multiple event versions | Medium | Moderate changes |
| Copy-Transform | Create new event store | High | Major changes |
| Weak Schema | Flexible event structure | Low | Rapid evolution |

**State Reconstruction:**

```
Events:
1. AccountCreated(id: 123, balance: 0)
2. MoneyDeposited(id: 123, amount: 100)
3. MoneyWithdrawn(id: 123, amount: 30)
4. MoneyDeposited(id: 123, amount: 50)

Current State (replay):
Account(id: 123, balance: 120)

Historical State (replay to event 2):
Account(id: 123, balance: 100)
```

---

### 32. Client-Server Architecture

**Intent:** Separate system into client (requests) and server (provides services).

| Aspect | Details |
|--------|---------|
| **Separation** | Clear client-server boundary |
| **Communication** | Network-based |
| **Server Types** | Stateless or stateful |
| **Scalability** | Vertical and horizontal |
| **Centralization** | Server centralizes logic/data |
| **Common Protocol** | HTTP, TCP, WebSocket |

**Real-World Examples:**
- Web applications
- Email systems (SMTP, POP3, IMAP)
- File servers (FTP)
- Database systems
- Mobile apps with backend
- Gaming servers

**Client-Server Variations:**

| Type | Description | Example | Pros | Cons |
|------|-------------|---------|------|------|
| Thin Client | Logic on server | Web browser | Easy updates | Network dependent |
| Thick Client | Logic on client | Desktop app | Rich experience | Hard to update |
| Two-Tier | Client → Database | Desktop DB app | Simple | Not scalable |
| Three-Tier | Client → App → DB | Modern web | Scalable | More complex |
| N-Tier | Multiple servers | Enterprise | Very scalable | Very complex |

**Stateless vs Stateful:**

| Aspect | Stateless Server | Stateful Server |
|--------|-----------------|-----------------|
| Session info | Client stores | Server stores |
| Scalability | Excellent | Limited |
| Reliability | High | Lower |
| Example | REST API | WebSocket chat |
| Load balancing | Easy | Complex |
| Memory usage | Low | High |

---

### 33. Peer-to-Peer Architecture

**Intent:** Distributed architecture where peers are equally privileged.

| Aspect | Details |
|--------|---------|
| **Equality** | All nodes equal |
| **Decentralization** | No central authority |
| **Discovery** | Peer discovery mechanism needed |
| **Resilience** | High fault tolerance |
| **Scalability** | Grows with nodes |
| **Common Use** | File sharing, blockchain |

**Real-World Examples:**
- BitTorrent
- Bitcoin/Blockchain
- Skype (early versions)
- IPFS (InterPlanetary File System)
- Gnutella
- WebRTC applications

**P2P Types:**

| Type | Description | Discovery | Example |
|------|-------------|-----------|---------|
| Unstructured | Random connections | Flooding | Gnutella |
| Structured | Organized topology | DHT | Chord, Kademlia |
| Hybrid | Some central coordination | Central index | Early Napster, Skype |
| Supernodes | Some powerful nodes | Supernodes | BitTorrent |

**P2P vs Client-Server:**

| Aspect | P2P | Client-Server |
|--------|-----|---------------|
| Single point of failure | No | Yes |
| Scalability | Excellent | Limited |
| Control | Distributed | Centralized |
| Security | Challenging | Easier |
| Implementation | Complex | Simpler |
| Bandwidth | Distributed | Centralized |
| Consistency | Eventual | Immediate |

---

### 34. Serverless Architecture

**Intent:** Build applications without managing servers, using FaaS.

| Aspect | Details |
|--------|---------|
| **Execution Model** | Functions triggered by events |
| **Scaling** | Automatic |
| **Pricing** | Pay per execution |
| **State** | Stateless functions |
| **Provider** | AWS Lambda, Azure Functions, Google Cloud Functions |
| **Cold Start** | Initial latency issue |

**Real-World Examples:**
- AWS Lambda functions
- Azure Functions
- Google Cloud Functions
- Image processing pipelines
- Chatbot backends
- API backends
- Scheduled tasks
- Data transformation

**Serverless Components:**

| Component | Purpose | Example |
|-----------|---------|---------|
| FaaS | Function execution | AWS Lambda |
| API Gateway | HTTP routing | API Gateway |
| Event Sources | Trigger functions | S3, DynamoDB |
| Storage | Persist data | S3, DynamoDB |
| Monitoring | Observe functions | CloudWatch |

**Cost Comparison:**

| Scenario | Traditional Server | Serverless |
|----------|-------------------|------------|
| Always running | Fixed cost | Much higher |
| Sporadic traffic | Fixed cost | Much lower |
| Predictable load | Fixed cost | Similar |
| Variable load | Fixed (overprovisioned) | Pay per use |
| Development time | Longer | Shorter |

**Limitations:**

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| Cold start | 100ms-5s latency | Keep warm, provisioned concurrency |
| Timeout | Max 15 min (AWS) | Chain functions, step functions |
| Memory | Max 10GB (AWS) | Optimize or use containers |
| Stateless | No local state | External storage |
| Vendor lock-in | Hard to migrate | Multi-cloud abstraction |

---

## Concurrency Patterns

### 35. Active Object Pattern

**Intent:** Decouple method execution from invocation for concurrency.

| Aspect | Details |
|--------|---------|
| **Thread Model** | Each object has own thread |
| **Queue** | Method requests queued |
| **Asynchrony** | Asynchronous method execution |
| **Futures** | Returns future/promise |
| **Synchronization** | Simplified |
| **Use Case** | Background processing, message passing |

**Real-World Examples:**
- GUI event handlers
- Actor model implementations (Akka)
- Message queue processors
- Background job processors
- Asynchronous I/O operations

**Components:**

| Component | Role | Example |
|-----------|------|---------|
| Proxy | Client interface | Submit job |
| Scheduler | Manages queue | Thread pool |
| Servant | Actual implementation | Business logic |
| Future | Result placeholder | CompletableFuture |
| Activation Queue | Request queue | LinkedBlockingQueue |

---

### 36. Monitor Object Pattern

**Intent:** Synchronize concurrent method execution to ensure thread safety.

| Aspect | Details |
|--------|---------|
| **Synchronization** | Method-level locking |
| **Condition Variables** | For coordination |
| **Thread Safety** | Automatic |
| **Performance** | Can be bottleneck |
| **Granularity** | Object-level locking |
| **Language Support** | Java synchronized, C# lock |

**Real-World Examples:**
- Java synchronized methods
- Thread-safe collections
- Connection pools
- Resource managers
- Shared caches

**Pattern vs Alternatives:**

| Pattern | Granularity | Complexity | Performance |
|---------|-------------|------------|-------------|
| Monitor Object | Object-level | Low | Medium |
| Fine-grained locking | Field-level | High | High |
| Lock-free | None | Very high | Very high |
| Active Object | Method-level | Medium | Medium |

---

### 37. Reactor Pattern

**Intent:** Handle service requests delivered concurrently via event demultiplexing.

| Aspect | Details |
|--------|---------|
| **Event Loop** | Single-threaded event loop |
| **I/O Model** | Non-blocking I/O |
| **Handlers** | Event handlers registered |
| **Scalability** | Excellent for I/O-bound |
| **Complexity** | Simpler than multi-threading |
| **Examples** | Node.js, Netty, Redis |

**Real-World Examples:**
- Node.js runtime
- Netty framework
- Redis server
- NGINX
- Event-driven servers
- High-concurrency web servers

**Reactor vs Thread-per-Connection:**

| Aspect | Reactor | Thread-per-Connection |
|--------|---------|---------------------|
| Threads | One or few | Many |
| Context switching | Minimal | High |
| Memory | Low | High (stack per thread) |
| Scalability | High (I/O-bound) | Limited |
| Complexity | Event-driven logic | Simpler logic |
| CPU-bound tasks | Poor | Good |

---

### 38. Thread Pool Pattern

**Intent:** Manage pool of worker threads to execute tasks efficiently.

| Aspect | Details |
|--------|---------|
| **Reuse** | Threads reused for multiple tasks |
| **Queue** | Task queue for pending work |
| **Size** | Fixed, dynamic, or cached pool |
| **Benefits** | Reduced creation overhead |
| **Common Use** | Server applications |
| **Implementations** | ExecutorService, ThreadPoolExecutor |

**Real-World Examples:**
- Web servers (thread per request)
- Database connection pools
- Background task processors
- Parallel processing frameworks
- Application servers

**Pool Types:**

| Type | Size | Use Case | Example |
|------|------|----------|---------|
| Fixed | Constant | Predictable load | newFixedThreadPool(10) |
| Cached | Dynamic | Variable load | newCachedThreadPool() |
| Single | 1 | Sequential tasks | newSingleThreadExecutor() |
| Scheduled | Variable | Periodic tasks | newScheduledThreadPool() |
| Work Stealing | CPU cores | Parallel tasks | ForkJoinPool |

**Configuration Considerations:**

| Factor | Too Few Threads | Too Many Threads | Optimal |
|--------|----------------|------------------|---------|
| CPU utilization | Underutilized | Context switching overhead | CPU cores + 1 |
| I/O tasks | Waiting | Memory overhead | Higher (50-100+) |
| Response time | High | Variable | Balanced |
| Throughput | Low | Can decrease | Depends on workload |

---

## Real-World Use Cases

### E-Commerce Platform

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| Product Catalog | Repository, Factory Method | Data access, product creation |
| Shopping Cart | Memento, Command | Undo operations, actions |
| Payment Processing | Strategy, Adapter | Multiple payment methods, gateways |
| Order Management | State, Observer | Order workflow, notifications |
| Inventory | Singleton, Observer | Single inventory, stock updates |
| Recommendation | Strategy, Decorator | Different algorithms, enhanced recommendations |
| Architecture | Microservices, CQRS | Scalability, read/write optimization |

### Banking System

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| Transaction Processing | Command, Memento | Transaction encapsulation, rollback |
| Account Management | Composite, Proxy | Account hierarchies, access control |
| Interest Calculation | Strategy, Template Method | Different rates, calculation process |
| Audit Trail | Observer, Event Sourcing | Logging, complete history |
| Authentication | Chain of Responsibility | Multiple auth methods |
| Reports | Builder, Visitor | Complex report building, operations |
| Architecture | Layered, SOA | Enterprise standards, service reuse |

### Social Media Platform

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| News Feed | Observer, Iterator | Post updates, feed scrolling |
| Post Creation | Builder, Factory Method | Complex posts, different types |
| Notifications | Observer, Mediator | User notifications, interaction coordination |
| Media Upload | Proxy, Facade | Lazy loading, complex upload process |
| User Profiles | Flyweight, Prototype | Memory optimization, profile templates |
| Friend Suggestions | Strategy, Decorator | Different algorithms, enhanced suggestions |
| Architecture | Microservices, Event-Driven | Scale, real-time updates |

### Content Management System

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| Content Types | Abstract Factory, Composite | Different content families, hierarchies |
| Publishing Workflow | State, Chain of Responsibility | Publication states, approval chain |
| Versioning | Memento, Command | Content history, undo/redo |
| Templates | Template Method, Builder | Page structure, complex layouts |
| Plugins | Decorator, Strategy | Add functionality, different behaviors |
| Media Library | Flyweight, Proxy | Memory optimization, lazy loading |
| Architecture | Layered, MVC | Traditional CMS structure |

### Game Development

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| Game Objects | Prototype, Object Pool | Clone entities, reuse objects |
| Game States | State, Command | Game flow, player actions |
| AI Behavior | Strategy, State | Different AI types, behavior states |
| Graphics | Flyweight, Facade | Share textures, simplify rendering |
| Input Handling | Command, Chain of Responsibility | Undo, input processing |
| Audio | Singleton, Observer | Audio manager, sound events |
| Architecture | Event-Driven, ECS | Performance, flexibility |

### IoT Platform

| Component | Patterns Used | Reason |
|-----------|--------------|--------|
| Device Communication | Adapter, Bridge | Different protocols, abstraction |
| Data Collection | Observer, Reactor | Device events, high concurrency |
| Data Processing | Strategy, Chain of Responsibility | Different processors, processing pipeline |
| Device Management | Proxy, Facade | Remote access, simplify complexity |
| Rules Engine | Interpreter, Command | Rule execution, actions |
| Time Series Data | Event Sourcing, CQRS | Complete history, optimized queries |
| Architecture | Microservices, Event-Driven | Scale, real-time processing |

---

## Pattern Comparison Tables

### Creational Patterns Comparison

| Pattern | Complexity | Flexibility | Use Case | Instance Control |
|---------|-----------|-------------|----------|------------------|
| Singleton | Low | Low | Single instance | Very strict |
| Factory Method | Medium | Medium | Subclass decides type | No control |
| Abstract Factory | High | High | Product families | No control |
| Builder | Medium | High | Complex construction | No control |
| Prototype | Low | Medium | Clone existing | No control |

### Structural Patterns Comparison

| Pattern | Purpose | Complexity | Runtime Flexibility | Main Benefit |
|---------|---------|------------|---------------------|--------------|
| Adapter | Interface conversion | Low | Low | Legacy integration |
| Bridge | Separate abstraction/implementation | High | High | Independent variation |
| Composite | Tree structures | Medium | Medium | Uniform treatment |
| Decorator | Add responsibilities | Medium | High | Dynamic extension |
| Facade | Simplify interface | Low | Low | Ease of use |
| Flyweight | Share objects | High | Low | Memory savings |
| Proxy | Control access | Low | Medium | Access control |

### Behavioral Patterns Comparison

| Pattern | Communication | Complexity | Coupling | Main Use |
|---------|--------------|------------|----------|----------|
| Chain of Responsibility | Request passing | Medium | Loose | Request handling |
| Command | Encapsulate request | Low | Loose | Undo/redo |
| Iterator | Sequential access | Low | Loose | Traversal |
| Mediator | Centralized | High | Loose | Complex interactions |
| Memento | State capture | Low | Loose | Undo |
| Observer | One-to-many notify | Medium | Loose | Event handling |
| State | State-based behavior | Medium | Medium | Workflow |
| Strategy | Algorithm selection | Low | Loose | Algorithm variants |
| Template Method | Algorithm skeleton | Low | Tight | Code reuse |
| Visitor | Operations on structure | High | Medium | Add operations |

### Architectural Patterns Comparison

| Pattern | Scalability | Complexity | Best For | Deployment |
|---------|------------|------------|----------|------------|
| Layered | Medium | Low | Traditional apps | Monolith |
| MVC | Medium | Medium | Web apps | Monolith |
| MVVM | Medium | Medium | Rich clients | Monolith |
| Microservices | Very High | Very High | Large systems | Distributed |
| Event-Driven | Very High | High | Real-time systems | Distributed |
| Hexagonal | Medium | Medium | Testable apps | Flexible |
| SOA | High | High | Enterprise | Distributed |
| CQRS | Very High | High | Read-heavy apps | Flexible |
| Serverless | Auto | Medium | Variable load | Cloud |

### Pattern Selection Decision Tree

```
Need to create objects?
├─ Single instance? → Singleton
├─ Complex construction? → Builder
├─ Clone existing? → Prototype
├─ Family of objects? → Abstract Factory
└─ Defer to subclass? → Factory Method

Need to compose structures?
├─ Convert interface? → Adapter
├─ Tree structure? → Composite
├─ Add responsibilities? → Decorator
├─ Simplify complex system? → Facade
├─ Share objects? → Flyweight
├─ Control access? → Proxy
└─ Separate abstraction? → Bridge

Need behavior/algorithms?
├─ Sequential access? → Iterator
├─ Encapsulate request? → Command
├─ One-to-many notification? → Observer
├─ State-based behavior? → State
├─ Algorithm family? → Strategy
├─ Request chain? → Chain of Responsibility
├─ Mediate complex interactions? → Mediator
├─ Capture state? → Memento
├─ Algorithm skeleton? → Template Method
└─ Operations on structure? → Visitor

Need architecture?
├─ Traditional enterprise? → Layered
├─ Web application? → MVC/MVVM
├─ Massive scale? → Microservices
├─ Real-time events? → Event-Driven
├─ Read/write separation? → CQRS
├─ Complete audit? → Event Sourcing
├─ High testability? → Hexagonal
└─ Variable load? → Serverless
```

---

## Anti-Patterns to Avoid

### Common Anti-Patterns

| Anti-Pattern | Description | Impact | Solution |
|--------------|-------------|--------|----------|
| God Object | One class does everything | Unmaintainable | Single Responsibility Principle |
| Spaghetti Code | Tangled, unstructured code | Hard to modify | Refactor, use patterns |
| Golden Hammer | Same solution for every problem | Suboptimal solutions | Learn multiple patterns |
| Lava Flow | Dead code nobody removes | Confusion, bloat | Regular cleanup |
| Copy-Paste Programming | Code duplication | Maintenance nightmare | DRY principle |
| Premature Optimization | Optimize too early | Wasted effort | Profile first |
| Big Ball of Mud | No clear architecture | Technical debt | Incremental refactoring |
| Cargo Cult Programming | Copy without understanding | Bugs, complexity | Understand before using |

---

## Best Practices

### Pattern Usage Guidelines

| Guideline | Description |
|-----------|-------------|
| **Understand the problem first** | Don't force patterns where they don't fit |
| **Start simple** | Add patterns as complexity demands |
| **Consider trade-offs** | Every pattern has costs |
| **Know when not to use** | Overuse adds unnecessary complexity |
| **Combine patterns** | Patterns work together |
| **Follow SOLID** | Patterns support SOLID principles |
| **Think long-term** | Consider maintainability |
| **Team consensus** | Ensure team understands patterns used |

### Pattern Implementation Checklist

- [ ] Problem clearly identified
- [ ] Pattern appropriate for problem
- [ ] Team familiar with pattern
- [ ] Trade-offs acceptable
- [ ] Implementation matches intent
- [ ] Code remains readable
- [ ] Tests cover pattern implementation
- [ ] Documentation explains rationale
- [ ] Performance impact acceptable
- [ ] Maintenance cost acceptable

---

*This cheat sheet is a living document. Patterns should be applied judiciously based on actual needs, not because they exist. The best code is often the simplest code that solves the problem effectively.*