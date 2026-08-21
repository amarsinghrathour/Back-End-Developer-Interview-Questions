### [[↑]](../README.md#toc) <a name='design'>Questions about Software Design:</a>

#### Index 0
> Why do array indexes start with '0' in most languages?

**Expert Answer:**

**The Short Answer:** 
Array indexes start at 0 because the index represents the physical memory *offset* from the start of the array's contiguous memory block.

**The Deep Dive:** 
In foundational languages like C and assembly, an array is simply a pointer to a base memory address. When you access `array[index]`, the hardware calculates the exact memory location using the formula `base_address + (index * element_size)`. If indexes started at 1, the compiler would have to subtract 1 on every single array access (`base_address + ((index - 1) * element_size)`), which would introduce an unnecessary subtraction operation, wasting precious CPU cycles on early hardware.

**The Trade-offs (Pros/Cons):**
* **Pros:** Computationally efficient; elegant pointer arithmetic; avoids off-by-one errors in mathematical loop bounds.
* **Cons:** Less intuitive for humans outside of programming who naturally count starting from 1 (leading to languages like MATLAB and Lua using 1-based indexing).

**Code Example:**
```go
func printOffsets() {
    arr := [3]int{10, 20, 30}
    // Index 0 is exactly at the start of the array in memory
    fmt.Printf("Base Address: %p\n", &arr)
    fmt.Printf("Element 0: %p\n", &arr[0]) // Same as Base Address
}
```

#### TDD
> How do tests and TDD influence code design?

**Expert Answer:**

**The Short Answer:** 
Test-Driven Development (TDD) forces you to act as the first consumer of your own API, driving the code toward loose coupling and high cohesion.

**The Deep Dive:** 
When you write tests *after* writing code, you often end up writing tests specifically tailored to pass the rigid structure you just created. TDD reverses this. Because you must write the test before the implementation exists, you are forced to think entirely about the *interface* and the *usability* of the component. You quickly realize that if a function relies on a hardcoded global database connection, it is impossible to test. Thus, TDD naturally guides the developer into using Dependency Injection, breaking massive functions into smaller units, and designing clear interfaces.

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly modular, decoupled architecture; guarantees test coverage for all functional requirements; self-documenting code.
* **Cons:** Slower initial development speed; rigid adherence can lead to over-mocking and fragile tests if testing implementation details rather than behavior.

**Code Example:**
```go
// TDD forces us to inject dependencies rather than hardcode them
type Notifier interface {
    Send(msg string) error
}

// Highly testable design
type AlertService struct {
    client Notifier 
}

func (s *AlertService) AlertAdmin() {
    s.client.Send("System Down!")
}
```

#### DRY Violation
> Write a snippet of code violating the Don't Repeat Yourself (DRY) principle. Then, explain why it is a bad design, and fix it.

**Expert Answer:**

**The Short Answer:** 
DRY is violated when identical business logic is duplicated across multiple functions, causing severe maintenance issues when requirements change.

**The Deep Dive:** 
Copy-pasting logic is the fastest way to accrue technical debt. If a business rule changes (e.g., the legal age is raised from 18 to 21), a developer must hunt down every single instance of the hardcoded age check. If they miss even one, the system enters an inconsistent, buggy state. By extracting the logic into a single source of truth, you guarantee that a rule change is instantly and safely propagated across the entire application.

**The Trade-offs (Pros/Cons):**
* **Pros:** Reduces bugs; shrinks codebase size; creates a single source of truth for business rules.
* **Cons:** Over-abstracting code that *looks* similar but serves different domain purposes can lead to confusing, overly complex architectures (violating the Rule of Three).

**Code Example:**
```go
// BAD: Violating DRY
func HandleUserSignup(user User) {
    if user.Age < 18 { return } // Duplicated rule
}
func HandleUserUpdate(user User) {
    if user.Age < 18 { return } // Duplicated rule
}

// GOOD: Abstracting the rule
func isOfLegalAge(age int) bool {
    return age >= 18
}

func HandleUserSignup(user User) {
    if !isOfLegalAge(user.Age) { return }
}
```

#### Cohesion vs Coupling
> What's the difference between cohesion and coupling?

**Expert Answer:**

**The Short Answer:** 
Cohesion measures how strongly related the elements *inside* a module are, while coupling measures the degree of dependency *between* different modules.

**The Deep Dive:** 
The holy grail of software design is "High Cohesion, Loose Coupling." 
*   **Cohesion** looks inward. A highly cohesive package (e.g., `billing`) contains only logic related to invoicing and payments. Everything in the package belongs together. 
*   **Coupling** looks outward. If the `billing` package directly calls the `database` package's internal unexported functions, they are tightly coupled. A change in the database structure will break the billing logic. 

**The Trade-offs (Pros/Cons):**
* **Pros (High Cohesion/Loose Coupling):** Easy to replace components; changes in one system rarely break another; highly parallelizable development for large teams.
* **Cons:** Achieving loose coupling often requires extensive boilerplate (interfaces, DTOs, mappers) which can be overkill for small scripts.

**Code Example:**
```go
// BAD: Low cohesion, tight coupling
func ProcessOrder(db *sql.DB, ui *Terminal) {
    // Mixing database access and UI rendering
}

// GOOD: High cohesion, loose coupling via interface
type OrderSaver interface { Save(Order) }

func ProcessOrder(saver OrderSaver) {
    // Only cares about saving, doesn't care how it happens
}
```

#### Refactoring
> What is refactoring useful for?

**Expert Answer:**

**The Short Answer:** 
Refactoring is the systematic process of improving the internal structure of existing code without changing its external behavior.

**The Deep Dive:** 
As software evolves, new features are inevitably bolted onto existing systems in ways the original developers did not anticipate, leading to "code rot" and technical debt. Refactoring is the discipline of cleaning up this rot. It is not about adding new features; it is about paying down technical debt. It improves readability, reduces cognitive complexity, extracts massive functions into manageable pieces, and prepares the architecture to accept new features gracefully.

**The Trade-offs (Pros/Cons):**
* **Pros:** Keeps the codebase healthy and maintainable; drastically reduces onboarding time for new engineers; minimizes the chance of future bugs.
* **Cons:** Pauses feature delivery; extremely dangerous if the system lacks a robust automated test suite to catch regressions.

**Code Example:**
```go
// Before Refactoring: Magic numbers and inline logic
func getDiscount(price float64, userType int) float64 {
    if userType == 1 { return price * 0.90 }
    return price
}

// After Refactoring: Extracted constants and clear naming
const PremiumUser = 1
const PremiumDiscount = 0.90

func getDiscount(price float64, userType int) float64 {
    if userType == PremiumUser { 
        return price * PremiumDiscount 
    }
    return price
}
```

#### Code Comments
> Are comments in code useful? Some say they should be avoided as much as possible, and hopefully made unnecessary. Do you agree?

**Expert Answer:**

**The Short Answer:** 
Comments are essential for explaining *why* a piece of code exists (the business context), but should rarely be used to explain *what* the code is doing.

**The Deep Dive:** 
If you find yourself writing a comment to explain *what* a complex block of code does, it is a massive code smell indicating a failure in naming or design. The code should be refactored into a well-named function (self-documenting code). However, code cannot explain the external business context. Comments are absolutely vital for explaining non-obvious business rules, pointing out dirty hacks required to bypass bugs in external APIs, or documenting complex algorithmic tradeoffs. In Go, comments also serve a structural purpose, automatically generating documentation via `godoc`.

**The Trade-offs (Pros/Cons):**
* **Pros:** Explains edge cases, hacks, and domain context; generates standard documentation packages.
* **Cons:** Comments lie. When code changes, developers frequently forget to update the associated comments, leading to misleading and dangerous documentation rot.

**Code Example:**
```go
// BAD: Explaining 'what'
// Increment counter by 1
i++ 

// GOOD: Explaining 'why'
// We sleep for 100ms here because the legacy payment gateway 
// has a known race condition and will drop requests if sent instantly.
time.Sleep(100 * time.Millisecond)
```

#### Design vs Architecture
> What is the difference between design and architecture?

**Expert Answer:**

**The Short Answer:** 
Architecture encompasses the high-level, hard-to-change structural decisions of a system, while design focuses on the lower-level implementation details within those boundaries.

**The Deep Dive:** 
Architecture is macro. It dictates whether the system will be a monolith or microservices, whether it will use PostgreSQL or MongoDB, and how boundary contexts communicate (e.g., gRPC vs Kafka). These decisions are extremely expensive to reverse. Design is micro. It determines how a specific module is written—choosing between a Strategy or Factory pattern, defining struct layouts, and establishing interface boundaries. Good architecture allows bad design to be rewritten locally; bad architecture means the entire system fails regardless of how beautifully the local code is designed.

**The Trade-offs (Pros/Cons):**
* **Pros (of upfront architecture):** Prevents systemic scaling failures; establishes clear boundaries for autonomous teams.
* **Cons:** "Big Design Up Front" can lead to analysis paralysis and over-engineering before the actual business problem is fully understood.

**Code Example:**
```go
// Architecture Level (Choosing gRPC for inter-service communication)
conn, _ := grpc.Dial("user-service:50051")
client := pb.NewUserServiceClient(conn)

// Design Level (Applying a Factory pattern locally within a service)
func NewPaymentProcessor(gateway string) Processor {
    if gateway == "stripe" { return &Stripe{} }
    return &Paypal{}
}
```

#### Early Testing
> In TDD, why are tests written before code?

**Expert Answer:**

**The Short Answer:** 
Writing tests first forces the developer to define the API contract and thoroughly understand the requirements before writing implementation details.

**The Deep Dive:** 
If you write the code first, you inevitably fall into the trap of writing tests specifically tailored to pass the code you just wrote, completely masking edge cases. Writing the test first flips the paradigm: you define the exact behavior and error states the consumer expects. The test fails (because the code doesn't exist). Then, you write the absolute minimum amount of code required to make the test pass. This guarantees that no superfluous, untested "what-if" code is added to the codebase.

**The Trade-offs (Pros/Cons):**
* **Pros:** Guarantees 100% test coverage for functional requirements; forces simple, usable API designs; acts as executable documentation.
* **Cons:** Requires a steep learning curve and significant discipline; can feel unnatural and slow down early experimentation phases.

**Code Example:**
```go
// 1. Write the test first (it fails because Parse fails)
func TestParse(t *testing.T) {
    result := Parse("invalid")
    if result != nil { t.Fail() }
}

// 2. Write the minimal code to pass
func Parse(input string) *Data {
    return nil // Minimal passing code
}
```

#### Multiple Inheritance
> C++ supports multiple inheritance, and Java allows a class to implement multiple interfaces. What impact does using these facilities have on orthogonality? Is there a difference in impact between using multiple inheritance and multiple interfaces? Is there a difference between using delegation and using inheritance?

**Expert Answer:**

**The Short Answer:** 
Multiple inheritance ruins orthogonality by creating tangled state and ambiguity, whereas multiple interfaces preserve orthogonality by purely defining behavior contracts. 

**The Deep Dive:** 
*   **Multiple Inheritance:** Inheriting state and behavior from multiple classes leads to the infamous "Diamond Problem" (ambiguity when two parents share a common ancestor class). It tightly couples the child to the fragile internals of multiple parents.
*   **Multiple Interfaces:** Interfaces contain no state and no implementation. They simply promise behavior. A Go struct can implement dozens of interfaces seamlessly without tangling state.
*   **Delegation vs Inheritance:** Inheritance is a permanent, rigid "is-a" relationship. Delegation (Composition) is a dynamic "has-a" relationship. Go completely rejects inheritance in favor of composition, where a struct embeds other structs and delegates work to them, preserving strict orthogonality.

**The Trade-offs (Pros/Cons):**
* **Pros (of Interfaces/Composition):** Prevents deep taxonomic coupling; completely eliminates the Diamond Problem; highly modular.
* **Cons:** Delegation requires slightly more boilerplate code to wire components together compared to classical inheritance.

**Code Example:**
```go
// Go uses structural interfaces, allowing a struct to satisfy multiple
type Reader interface { Read() }
type Writer interface { Write() }

type File struct {}
func (f File) Read() {}
func (f File) Write() {}
// File seamlessly satisfies both Reader and Writer without inheriting state.
```

#### Domain Logic in Stored Procedures
> What are the pros and cons of holding domain logic in Stored Procedures?

**Expert Answer:**

**The Short Answer:** 
Stored Procedures offer high performance and centralized logic across applications, but they severely fragment business logic and create vendor lock-in.

**The Deep Dive:** 
In the 90s, shifting complex calculations into Stored Procedures was popular to avoid expensive network round-trips between the application and the database. However, as modern applications scaled, this became a severe anti-pattern. Domain logic belongs in the application layer. When business logic is hidden in SQL procedures, it becomes nearly impossible to unit test, version control in standard Git workflows, debug with standard tools, or migrate to a new database vendor.

**The Trade-offs (Pros/Cons):**
* **Pros:** Extremely fast execution for data-heavy operations; centralized logic if multiple legacy apps share the same database.
* **Cons:** Total vendor lock-in (e.g., tying yourself to Oracle PL/SQL); horrible developer experience for testing and version control; fragments domain logic across two entirely different environments.

**Code Example:**
```go
// BAD: Relying on Stored Procedure for domain logic
db.Exec("CALL CalculateUserTaxes(?)", userID) 

// GOOD: Domain logic in the application
user := repo.GetUser(userID)
taxes := taxCalculator.Calculate(user) // Easily unit-testable in Go
repo.SaveTaxes(userID, taxes)
```

#### OOP Took Over the World
> In your opinion, why has Object-Oriented Design dominated the market for so many years?

**Expert Answer:**

**The Short Answer:** 
OOP dominated because its concepts of nouns (Objects) and verbs (Methods) closely mapped to human mental models, making it highly intuitive for managing complex UI states.

**The Deep Dive:** 
During the rise of complex desktop software in the 90s (via Smalltalk, C++, and Java), developers needed a way to manage sprawling, highly stateful GUIs. Encapsulating state alongside behavior inside "Classes" proved incredibly effective at establishing strict code boundaries, which allowed massive enterprise teams to work without stepping on each other's toes. However, modern backend development (especially in Go and Rust) heavily favors Composition over Inheritance, recognizing that rigid class taxonomies often fail to accurately model evolving business domains.

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly intuitive modeling; strong encapsulation of state; massive ecosystem of established design patterns (GoF).
* **Cons:** Promotes rigid inheritance hierarchies; mutable state hidden inside objects makes concurrency (multithreading) notoriously difficult.

**Code Example:**
```go
// Modern languages like Go reject classical OOP inheritance.
// Instead of a rigid class hierarchy (Animal -> Dog), Go relies on behavior:
type Barker interface {
    Bark()
}

// Dog is just a struct, not a subclass.
type Dog struct{}
func (d Dog) Bark() { fmt.Println("Woof!") }
```

#### Bad Design
> What would you do to understand if your code has a bad design?

**Expert Answer:**

**The Short Answer:** 
Code has a bad design if it is rigid (hard to change), fragile (breaks easily), immobile (hard to reuse), and difficult to unit test.

**The Deep Dive:** 
You can smell bad design through everyday developer friction. 
1.  **Rigidity:** When a simple feature request requires touching 15 different files.
2.  **Fragility:** When fixing a bug in the billing module inexplicably breaks the user login module.
3.  **Immobility:** When you want to reuse a helpful utility function in another project, but realize it's hopelessly entangled with global state or specific database configurations.
4.  **Testability:** The ultimate litmus test. If writing a simple unit test requires 200 lines of setup code and complex mocking frameworks, the architecture is fundamentally flawed.

**The Trade-offs (Pros/Cons):**
* **Pros (of fixing bad design):** Code becomes a joy to work with; shipping velocity increases predictably over time.
* **Cons:** Fixing systemic bad design requires massive refactoring efforts that halt new feature development.

**Code Example:**
```go
// A sign of bad design: Impossible to test easily
func CheckStatus() string {
    // Hidden dependency! Must have a real Postgres database running to test this.
    db := sql.Open("postgres", "...") 
    // ...
}

// Good design: Trivial to test via dependency injection
func CheckStatus(db DBInterface) string {
    // Can pass a mock DB in tests!
}
```


#### API Gateway vs Service Mesh
> When should you use an API Gateway, and when should you use a Service Mesh like Istio/Envoy?

**Expert Answer:**

**The Short Answer:** 
An API Gateway manages North-South traffic (from the internet into your cluster), while a Service Mesh manages East-West traffic (internal service-to-service communication).

**The Deep Dive:** 
An API Gateway (like Kong or AWS API Gateway) is the front door. It handles rate limiting, user authentication (JWT validation), and SSL termination for external clients (Mobile/Web). 
A Service Mesh is deployed as a "sidecar" proxy alongside every internal microservice. It handles internal complexities that you don't want to code into every app: mutual TLS (mTLS) between services, internal retries, circuit breaking, and distributed tracing. You usually need both: the Gateway handles the messy outside world, and the Mesh secures the internal network.

**The Trade-offs (Pros/Cons):**
* **Pros:** Complete separation of infrastructure concerns (networking/security) from business logic.
* **Cons:** Service Meshes are notoriously difficult to configure and add a slight latency overhead to every internal network hop.

#### Hexagonal Architecture (Ports & Adapters)
> Why are modern backend systems moving toward Hexagonal (Clean) Architecture?

**Expert Answer:**

**The Short Answer:** 
It isolates the core business logic from external frameworks, databases, and UI, making the application infinitely more testable and adaptable.

**The Deep Dive:** 
In traditional layered architecture, the business logic often depends directly on the database layer (e.g., calling SQL directly). If you change databases, the business logic breaks. Hexagonal Architecture inverses this. The core Domain has no dependencies. It defines "Ports" (interfaces). The database is just an "Adapter" that plugs into the port. 
This means you can test 100% of your business logic in milliseconds by plugging in an In-Memory Adapter, without ever booting up a real database or HTTP server.

**The Trade-offs (Pros/Cons):**
* **Pros:** Blazing fast unit tests; prevents vendor lock-in.
* **Cons:** Requires writing significant boilerplate code (interfaces and mappers) for even simple CRUD operations.

#### Event Sourcing
> What is Event Sourcing, and why is it used in financial or auditing systems?

**Expert Answer:**

**The Short Answer:** 
Instead of storing the current state of an entity, Event Sourcing stores every single state-changing event in an immutable, append-only log. The current state is calculated by replaying the events.

**The Deep Dive:** 
If a user's bank balance is $100, traditional databases just store `balance: 100`. If there's a bug, you have no idea how it got to $100. In Event Sourcing, the database stores: `AccountCreated($0) -> Deposited($150) -> Withdrew($50)`. 
To get the balance, you sum the events. If a bug occurs, you have a perfect audit trail. You can also "time travel" by replaying the events up to a specific date to see the exact system state at that moment in time.

**The Trade-offs (Pros/Cons):**
* **Pros:** Perfect auditability; naturally supports CQRS and event-driven architectures.
* **Cons:** Querying the current state requires replaying events, which is slow (mitigated by "Snapshots"); schema evolution of historical events is painful.

#### Strangler Fig Pattern
> How do you safely migrate a legacy monolith to microservices using the Strangler Fig Pattern?

**Expert Answer:**

**The Short Answer:** 
You place a proxy in front of the monolith, build new features as microservices, and slowly route traffic to the new services until the monolith dies.

**The Deep Dive:** 
"Big Bang" rewrites (shutting down development for a year to rewrite everything) almost always fail. The Strangler Pattern mitigates this risk. 
1. Put an API Gateway in front of the monolith. All traffic goes to the monolith.
2. Build the new `BillingService`.
3. Configure the API Gateway to route `/billing` traffic to the new service, and everything else to the monolith.
4. Repeat for every domain. Over time, the new services "strangle" the monolith until it handles zero traffic and can be safely deleted.

**The Trade-offs (Pros/Cons):**
* **Pros:** Incremental, low-risk migration; allows delivering business value continuously during the rewrite.
* **Cons:** For a long time, you must maintain both the legacy and the new system, often requiring messy data synchronization between the old and new databases.
