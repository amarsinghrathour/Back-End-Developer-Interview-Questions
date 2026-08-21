### [[↑]](../README.md#toc) <a name='patterns'>Questions about Design Patterns:</a>

#### Globals Are Evil
> Why are global and static objects evil? Can you show it with a code example?

**Expert Answer:**

**The Short Answer:** 
Global variables introduce hidden, shared state across an application, leading to tight coupling and unpredictable side effects.

**The Deep Dive:** 
When multiple parts of a system rely on and mutate the same global state, it becomes impossible to reason about the flow of data. It destroys thread safety in concurrent applications, as multiple threads might mutate the state simultaneously causing race conditions. Furthermore, it makes code incredibly difficult to unit test because the state persists between tests and affects parallel test execution.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extremely convenient for rapid prototyping; easy access from anywhere without passing parameters.
* **Cons:** Breaks encapsulation; makes unit testing a nightmare; inherently thread-unsafe unless heavily locked.

**Code Example:**
```go
package database

import "database/sql"

// BAD: Global state
var DBConnection *sql.DB 

func SaveUser(user User) {
    // Tight coupling to global state makes this impossible to test in isolation
    DBConnection.Exec("INSERT...") 
}

// GOOD: Dependency Injection
type UserRepository struct {
    db *sql.DB
}

func (repo *UserRepository) SaveUser(user User) {
    repo.db.Exec("INSERT...") 
}
```

#### Inversion of Control
> Tell me about Inversion of Control and how it improves the design of code.

**Expert Answer:**

**The Short Answer:** 
Inversion of Control (IoC) is a design principle where a generic framework controls the execution flow, calling into custom-written components, rather than custom code controlling the flow.

**The Deep Dive:** 
IoC flips the traditional programming model. Instead of your business logic dictating when to read from a network or database, a framework handles the lifecycle and triggers your logic via callbacks or interfaces. The most common manifestation of this is Dependency Injection (DI). In Go, this is heavily implemented using `interfaces`. Components do not instantiate their dependencies; they declare an interface they need, and the "control" of what concrete type is provided is inverted to the caller.

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly decoupled systems; trivial to swap implementations (e.g., swapping a real DB for a mock in tests); separates configuration from use.
* **Cons:** Can introduce cognitive overhead (harder to follow the execution path); can lead to "interface soup" if overused.

**Code Example:**
```go
// The component declares what it needs (IoC via Dependency Injection)
type PaymentProcessor interface {
    Process(amount float64) error
}

type CheckoutService struct {
    processor PaymentProcessor // Dependency is injected
}

func (c *CheckoutService) Checkout(amount float64) {
    c.processor.Process(amount) // Agnostic to the actual implementation
}
```

*Read more in the original resources: [inversion-of-control.md](../design-patterns/inversion-of-control.md)*

#### Law of Demeter
> The Law of Demeter (the Principle of Least Knowledge) states that each unit should have only limited knowledge about other units and it should only talk to its immediate friends (sometimes stated as "don't talk to strangers").
> Would you write code violating this principle, show why it is a bad design and then fix it?

**Expert Answer:**

**The Short Answer:** 
The Law of Demeter ensures objects only interact with their direct dependencies, preventing brittle code that relies on the internal structure of other objects.

**The Deep Dive:** 
Often summarized as "Tell, Don't Ask," this principle prevents long chains of method calls (e.g., `a.getB().getC().doSomething()`). When an object reaches deep into another object's internal structure, it becomes tightly coupled to that structure. If the internal structure changes, the caller breaks. By adhering to the law, objects expose high-level behaviors rather than their internal composition.

**The Trade-offs (Pros/Cons):**
* **Pros:** High encapsulation; loosely coupled components; highly resilient to refactoring.
* **Cons:** Can lead to a proliferation of "wrapper" methods that simply delegate calls to internal components.

**Code Example:**
```go
// BAD: Violating Demeter (reaching into internal structure)
customer.Wallet.DeductMoney(50.0)

// GOOD: Following Demeter (Tell, Don't Ask)
// Customer encapsulates its own wallet logic
customer.Charge(50.0)
```

*Read more in the original resources: [law-of-demeter.md](../design-patterns/law-of-demeter.md)*

#### Active-Record
> Active-Record is the design pattern that promotes objects to include functions such as Insert, Update, and Delete, and properties that correspond to the columns in some underlying database table. In your opinion and experience, which are the limits and pitfalls of the this pattern?

**Expert Answer:**

**The Short Answer:** 
Active-Record is an ORM pattern where domain models map exactly to database tables and are responsible for their own persistence logic.

**The Deep Dive:** 
In Active-Record, a class is a direct representation of a database row. The class itself contains the SQL execution methods (like `.Save()` or `.Delete()`). While this is highly intuitive and allows for incredibly fast development in frameworks like Ruby on Rails, it notoriously violates the Single Responsibility Principle. Your domain logic becomes hopelessly entangled with your database schema and persistence mechanism.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extremely fast prototyping; low cognitive load for simple CRUD apps; intuitive developer experience.
* **Cons:** Violates SRP; makes domain logic almost impossible to unit test without a real database; fails completely when domain models diverge from the database schema.

**Code Example:**
```go
// Active Record (Mock implementation)
type User struct {
    ID   int
    Name string
}

// The struct is responsible for saving itself
func (u *User) Save() error {
    return globalDB.Exec("INSERT INTO users...", u.Name)
}
```

*Read more in the original resources: [active-record.md](../design-patterns/active-record.md)*

#### Data-Mapper
> Data-Mapper is a design pattern that promotes the use of a layer of Mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself. On the contrary, in Active-Record objects directly incorporate operations for persisting themselves to a database, and properties corresponding to the underlying database tables. Do you have an opinion on those patterns? When would you use one instead of the other?

**Expert Answer:**

**The Short Answer:** 
Data-Mapper separates in-memory domain objects from the database schema by introducing an intermediary persistence layer (the mapper).

**The Deep Dive:** 
Unlike Active-Record, Data-Mapper enforces strict separation of concerns. The domain objects (structs) have zero knowledge of the database, SQL, or persistence logic. They are "Plain Old Go Objects." The Mapper handles moving data back and forth. This is essential for Domain-Driven Design (DDD), where the shape of the business logic rarely maps 1:1 with relational database normalization.

**The Trade-offs (Pros/Cons):**
* **Pros:** Perfect separation of concerns; highly testable domain logic; database schema can evolve independently of domain models.
* **Cons:** High boilerplate and overhead; overkill for simple CRUD applications.

**Code Example:**
```go
// Data Mapper approach
type User struct { // Pure domain object
    ID   int
    Name string
}

// The Mapper (Repository) handles persistence
type UserMapper struct {
    db *sql.DB
}

func (m *UserMapper) Save(u *User) error {
    return m.db.Exec("INSERT INTO users...", u.Name)
}
```

#### Billion Dollar Mistake
> Tony Hoare who invented the null reference once said "I call it my billion-dollar mistake".
> Would you discuss the techniques to avoid it, such as the Null Object Pattern introduced by the GOF book, or Option types?

**Expert Answer:**

**The Short Answer:** 
The "Billion Dollar Mistake" refers to null pointers, which cause catastrophic runtime crashes; they can be mitigated using Option types, Null Objects, or value semantics.

**The Deep Dive:** 
Null references break type safety because a caller assumes they are receiving a valid object, but instead receive a time-bomb that crashes the program upon dereferencing. To avoid this, modern languages use Option/Maybe types (forcing the caller to explicitly handle the `None` case at compile time). In Go, which lacks native Option types, the idiom is to use multiple return values (`value, err`) or the Null Object Pattern (returning an object that implements an interface but safely does nothing).

**The Trade-offs (Pros/Cons):**
* **Pros (of avoiding nulls):** Complete elimination of Null Pointer Exceptions (panics); self-documenting code.
* **Cons:** Verbose error checking in languages without elegant Option type matching (like Go's repetitive `if err != nil`).

**Code Example:**
```go
// Null Object Pattern in Go
type Logger interface {
    Log(msg string)
}

// Instead of passing a `nil` Logger which causes panics:
type NoopLogger struct{}
func (n NoopLogger) Log(msg string) {} // Safely does nothing

func Process(l Logger) {
    l.Log("Processing...") // Guaranteed to be safe
}
```

#### Inheritance vs Composition
> Many state that, in Object-Oriented Programming, composition is often a better option than inheritance. What's you opinion?

**Expert Answer:**

**The Short Answer:** 
Composition favors building complex objects by assembling smaller, interchangeable parts (has-a) rather than inheriting behavior through rigid class hierarchies (is-a).

**The Deep Dive:** 
Classical inheritance creates deep, fragile hierarchies. If a superclass changes, all subclasses are impacted (the Fragile Base Class problem). It also forces a rigid taxonomy that rarely maps perfectly to the real world. Composition, on the other hand, allows you to dynamically assemble behaviors. Go's entire design philosophy is built around this: Go completely omits classical inheritance and instead relies entirely on struct embedding and interfaces.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extreme flexibility; prevents deep taxonomic coupling; code is easier to unit test via mock components.
* **Cons:** Can lead to more boilerplate since you must explicitly wire components together; lacks the convenience of overriding a single inherited method.

**Code Example:**
```go
// Go uses Struct Embedding for Composition
type Engine struct{}
func (e *Engine) Start() {}

type Car struct {
    Engine // Car "has-a" Engine, but exposes its methods natively
}

func main() {
    c := Car{}
    c.Start() // Promoted method from embedded struct
}
```

#### Anti-Corruption Layer
> What is an Anti-corruption Layer?

**Expert Answer:**

**The Short Answer:** 
An Anti-Corruption Layer (ACL) is a translation layer that isolates a clean domain model from a messy legacy or external system.

**The Deep Dive:** 
In Domain-Driven Design, when your modern system must integrate with a legacy system that has a terrible data model or confusing API, you build an ACL. Instead of letting the legacy terminology and data structures "corrupt" your new clean architecture, the ACL acts as an adapter. It translates legacy objects into your modern domain objects, ensuring that the rest of your application remains pure and unaware of the legacy system's quirks.

**The Trade-offs (Pros/Cons):**
* **Pros:** Protects domain purity; allows legacy systems to be swapped or refactored later without touching core business logic.
* **Cons:** Adds an extra layer of latency, translation logic, and maintenance overhead.

**Code Example:**
```go
// The clean domain object
type User struct {
    FullName string
}

// The ACL translates from the messy legacy system
func FetchUserFromLegacyACL(id int) User {
    // Legacy system returns weird fields: "F_NAME_XYZ", "L_NAME_XYZ"
    legacyData := LegacyAPI.Get(id) 
    
    // Translate to clean domain model
    return User{
        FullName: legacyData.F_NAME_XYZ + " " + legacyData.L_NAME_XYZ,
    }
}
```

#### Singleton
> Singleton is a design pattern that restricts the instantiation of a class to one single object. Writing a Thread-Safe Singleton class is not so obvious. Would you try?

**Expert Answer:**

**The Short Answer:** 
A Singleton ensures only one instance of an object exists globally; making it thread-safe requires synchronization primitives to prevent race conditions during initialization.

**The Deep Dive:** 
In highly concurrent environments, multiple threads might attempt to initialize the Singleton at the exact same moment. If not properly locked, this can result in multiple instances being created, completely breaking the pattern. While traditional languages use complex Double-Checked Locking, Go provides an incredibly elegant and robust solution out-of-the-box using the `sync.Once` primitive.

**The Trade-offs (Pros/Cons):**
* **Pros:** Guarantees a single point of truth (e.g., a shared configuration object or connection pool); lazy initialization saves resources.
* **Cons:** It is essentially global state in disguise; severely complicates unit testing; hides dependencies.

**Code Example:**
```go
import "sync"

type singleton struct {}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    // sync.Once guarantees thread-safe, exactly-once execution
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

#### Data Abstraction
> The ability to change implementation without affecting clients is called Data Abstraction. Produce an example violating this property, then fix it.

**Expert Answer:**

**The Short Answer:** 
Data Abstraction hides the internal representation of an object, exposing only behavior through methods to prevent clients from breaking when internals change.

**The Deep Dive:** 
When an object's internal state is fully public, clients couple themselves directly to that specific data structure. If you later realize you need to change how that data is stored (e.g., calculating a value on-the-fly instead of storing it), you will break every client that depended on the raw fields. By encapsulating state and exposing getters/setters (or behavior), the implementation can evolve independently.

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly resilient to change; protects data integrity; allows adding side-effects (like logging) to property access.
* **Cons:** Boilerplate getter/setter methods; slight performance overhead of a function call vs direct memory access.

**Code Example:**
```go
// BAD: Violating Abstraction
type Rectangle struct {
    Width  float64 // Public field
}

// GOOD: Data Abstraction
type Rectangle struct {
    width  float64 // Private field
}

// Expose behavior, not data
func (r *Rectangle) SetWidth(w float64) { r.width = w }
func (r *Rectangle) Width() float64     { return r.width }
```

#### Don't Repeat Yourself
> Write a snippet of code violating the Don't Repeat Yourself (DRY) principle. Then, fix it.

**Expert Answer:**

**The Short Answer:** 
DRY is the principle of reducing repetition of software patterns by abstracting common logic into reusable methods.

**The Deep Dive:** 
When logic is copy-pasted across a codebase, it creates a maintenance nightmare. If a bug is found or a business rule changes, developers must hunt down every copy of the code and update it. Failing to update even one copy introduces inconsistencies. By extracting the logic into a single source of truth, changes propagate safely across the entire system.

**The Trade-offs (Pros/Cons):**
* **Pros:** Single source of truth; reduces bugs; makes codebase smaller and easier to maintain.
* **Cons:** Overzealous DRYing can lead to premature abstraction, where two pieces of code that *look* similar but change for different reasons are forced together (violating the Rule of Three).

**Code Example:**
```go
// BAD: Violating DRY
func PrintInvoice(items []Item) {
    total := 0.0
    for _, item := range items { total += item.Price }
    fmt.Printf("Total: %f\n", total * 1.20) // Tax logic duplicated
}

func ProcessRefund(items []Item) {
    total := 0.0
    for _, item := range items { total += item.Price }
    fmt.Printf("Refund: %f\n", total * 1.20) // Tax logic duplicated
}

// GOOD: Abstracting the logic
func calcTotalWithTax(items []Item) float64 {
    total := 0.0
    for _, item := range items { total += item.Price }
    return total * 1.20
}
```

*Read more in the original resources: [dont-repeat-yourself.md](../design-patterns/dont-repeat-yourself.md)*

#### Dependency Hell
> How would you deal with Dependency Hell?

**Expert Answer:**

**The Short Answer:** 
Dependency Hell occurs when software packages require conflicting, mutually exclusive versions of shared dependencies, making the project impossible to build.

**The Deep Dive:** 
This typically happens in complex dependency graphs (A requires C v1.0, B requires C v2.0). To resolve it, modern ecosystems enforce strict Semantic Versioning. Go solves this elegantly via **Go Modules** and Minimal Version Selection (MVS), which guarantees reproducible builds by always preferring the oldest allowed version that satisfies all requirements.

**The Trade-offs (Pros/Cons):**
* **Pros (of Dependency Management systems):** Reproducible builds; isolated environments; automatic resolution.
* **Cons:** Can bloat binary sizes if multiple versions of the same library are compiled; requires strict adherence to SemVer by library authors.

**Code Example:**
```go
// In Go, go.mod handles dependencies. 
// If true isolation is required to prevent upstream breakage, 
// vendoring pulls all dependencies directly into your repo:

// bash:
// go mod vendor
```

#### Goto Is Evil
> Is goto evil? What's your opinion on the use of goto?

**Expert Answer:**

**The Short Answer:** 
`goto` is generally considered "evil" because it leads to unstructured "spaghetti code," but it has highly specific, valid use cases in systems programming.

**The Deep Dive:** 
Dijkstra famously wrote "Go To Statement Considered Harmful." Using `goto` arbitrarily makes control flow impossible to trace mentally, leading to catastrophic bugs. However, languages like C and Go retain it for specific patterns. In Go, it is occasionally used in highly performant standard library code to cleanly break out of deeply nested loops or jump to a unified error-cleanup block, avoiding repetitive conditional checks.

**The Trade-offs (Pros/Cons):**
* **Pros:** Can marginally improve performance and simplify cleanup logic in very low-level systems code.
* **Cons:** Rapidly devolves into unmaintainable spaghetti code; bypasses structured programming concepts (loops, functions).

**Code Example:**
```go
// Valid use case for goto: breaking deeply nested loops
func SearchMatrix(matrix [][]int, target int) bool {
    for i := range matrix {
        for j := range matrix[i] {
            if matrix[i][j] == target {
                goto Found
            }
        }
    }
    return false
    
Found:
    fmt.Println("Found it!")
    return true
}
```

#### Robustness Principle
> The robustness principle is a general design guideline for software that recommends "be conservative in what you send, be liberal in what you accept". Would you like to discuss the rationale of this principle?

**Expert Answer:**

**The Short Answer:** 
Postel's Law ensures distributed systems remain resilient by strictly adhering to specifications when sending data, but gracefully tolerating slight malformations when receiving it.

**The Deep Dive:** 
Originally formulated for TCP/IP, this principle allowed the early internet to function despite different vendors building slightly incompatible implementations. Being liberal in what you accept (e.g., ignoring unknown JSON fields rather than crashing) provides graceful degradation. Being conservative in what you send ensures you don't burden consumers with parsing errors.

**The Trade-offs (Pros/Cons):**
* **Pros:** High system resilience; easier integration with third-party APIs; backwards compatibility.
* **Cons:** In modern security contexts, being "liberal" is a massive anti-pattern. Liberal parsers frequently lead to security exploits like HTTP Request Smuggling. Modern APIs favor strict boundary validation.

**Code Example:**
```go
// Go's JSON decoder is liberal by default: it simply ignores unknown fields.
type Payload struct {
    ID int `json:"id"`
}

// Even if the JSON contains {"id": 1, "hacker_data": "drop tables"}, 
// Go will safely ignore "hacker_data" unless explicitly configured otherwise.
json.Unmarshal(data, &payload) 
```

#### Separation of Concerns
> Separation of Concerns is a design principle for separating a computer program into distinct areas, each one addressing a separate concern. Would you discuss this topic?

**Expert Answer:**

**The Short Answer:** 
Separation of Concerns (SoC) prevents monolithic spaghetti code by isolating different aspects of an application (like UI, business logic, and database access) into independent layers.

**The Deep Dive:** 
If presentation logic (HTTP routing), business rules (tax calculation), and persistence (SQL queries) are mixed into a single massive function, changing the database schema risks breaking the HTTP routing. By separating these into layers (e.g., Hexagonal Architecture or N-Tier), teams can work in parallel, and implementations can be swapped seamlessly.

**The Trade-offs (Pros/Cons):**
* **Pros:** Code becomes independently testable; vastly improves maintainability; prevents side-effect bugs across layers.
* **Cons:** Increases initial cognitive load; requires writing interfaces and mapping layers which adds boilerplate code.

**Code Example:**
```go
// BAD: Mixed Concerns
func HandleCheckout(w http.ResponseWriter, r *http.Request) {
    db.Exec("INSERT...") // DB logic mixed with HTTP logic
}

// GOOD: Separated Concerns
// 1. HTTP Layer
func HandleCheckout(w http.ResponseWriter, r *http.Request) {
    service.Checkout(r.Context(), payload)
}

// 2. Business Logic Layer
func (s *Service) Checkout(ctx context.Context, p Payload) {
    s.repo.Save(p)
}
```


#### The Outbox Pattern
> How do you guarantee that a database update and a message queue publish both succeed or both fail?

**Expert Answer:**

**The Short Answer:** 
By using the Outbox Pattern: you save the database update and the event message to a local "Outbox" table in a single atomic transaction, then use a background worker to publish the event to the queue.

**The Deep Dive:** 
If a user creates an order, you must update the `Orders` table and publish an `OrderCreated` event to Kafka. If the database succeeds but Kafka is down, the system is permanently corrupted. Distributed Transactions (2PC) are too slow. 
Instead, you create an `Outbox` table in the *same* database. You open a single SQL transaction, insert the Order into the `Orders` table, and insert the Kafka message into the `Outbox` table. Commit. Because it's one database, it's perfectly atomic. A separate process (like Debezium) constantly polls the `Outbox` table and pushes the rows to Kafka, guaranteeing at-least-once delivery.

**The Trade-offs (Pros/Cons):**
* **Pros:** Bulletproof data consistency across microservices without the overhead of 2-Phase Commit.
* **Cons:** Requires consumers of the message queue to be strictly idempotent, because the background worker might crash and publish the exact same message twice.

#### CQRS (Command Query Responsibility Segregation)
> Why separate read operations from write operations into entirely different models?

**Expert Answer:**

**The Short Answer:** 
Because a data schema optimized for heavy, transactional writes (3rd Normal Form) is completely different from a schema optimized for lightning-fast reads (denormalized JSON).

**The Deep Dive:** 
In a complex system (like an e-commerce dashboard), writing an order requires complex validation and normalized tables to prevent anomalies. However, rendering the dashboard requires joining 15 tables, which is painfully slow. CQRS completely separates them. The "Command" side writes to a normalized SQL database. It then fires an event. The "Query" side listens to the event and builds a pre-calculated, flat JSON document in ElasticSearch or Redis. When the UI asks for the dashboard, it just fetches the single JSON blob in O(1) time.

**The Trade-offs (Pros/Cons):**
* **Pros:** Unbelievable read performance; allows scaling reads and writes completely independently.
* **Cons:** Introduces eventual consistency (the user might write an order, refresh the page, and not see it for 2 seconds); massively increases architectural complexity.

#### Saga Pattern (Choreography vs Orchestration)
> How do you manage a transaction that spans multiple microservices without using a distributed lock?

**Expert Answer:**

**The Short Answer:** 
Using a Saga, which breaks the distributed transaction into local transactions, relying on compensating actions (rollbacks) if a later step fails.

**The Deep Dive:** 
Booking a vacation requires flights, hotels, and a rental car (3 different microservices). If the car fails, you must cancel the flight and hotel. 
* **Choreography:** The Flight service emits an event, the Hotel service hears it and books, emitting another event. There is no central controller. (Good for simple workflows, hard to track).
* **Orchestration:** A central "Saga Execution Coordinator" (like AWS Step Functions) explicitly tells the Flight service to book. If successful, it tells the Hotel service. If the Car service fails, the Orchestrator explicitly sends a "Cancel" command back to the Flight and Hotel services.

**The Trade-offs (Pros/Cons):**
* **Pros:** Maintains data consistency across domains without locking databases.
* **Cons:** "Compensating transactions" are incredibly difficult to write correctly, especially if the compensating action *also* fails (requiring manual human intervention).

#### Backend-For-Frontend (BFF) Pattern
> Why should a Mobile app and a Web app communicate with different API gateways?

**Expert Answer:**

**The Short Answer:** 
Because mobile screens and desktop screens require vastly different payloads; a BFF creates a custom, optimized API layer for each specific user interface.

**The Deep Dive:** 
A generalized API (`GET /product/123`) returns a massive JSON blob with 50 fields. The desktop web app uses all 50. The mobile app only has screen space for 5 fields. Sending 45 useless fields over a 3G mobile network drains the battery and destroys latency. 
Instead of bloating the core microservices with UI-specific logic, you create a "Mobile BFF" (a lightweight NodeJS server). The Mobile App calls the BFF. The BFF calls the internal microservices in parallel, trims the JSON down to exactly the 5 fields needed, and sends it back to the phone.

**The Trade-offs (Pros/Cons):**
* **Pros:** Optimized payload size and battery life for mobile devices; frontend teams can own their BFFs.
* **Cons:** Code duplication across different BFFs; creates a new moving part to deploy and monitor.
