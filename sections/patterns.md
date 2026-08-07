### [[↑]](../README.md#toc) <a name='patterns'>Questions about Design Patterns:</a>

#### Globals Are Evil
> Why are global and static objects evil? Can you show it with a code example?

**Expert Answer:**
Global variables introduce hidden, shared state across an application. This leads to tight coupling, unpredictable side effects, and makes code incredibly difficult to unit test since the state persists between tests and affects parallel test execution. It also destroys thread safety in concurrent applications.

*Bad Example:*
```go
package database

import "database/sql"

// Global state
var DBConnection *sql.DB 

func SaveUser(user User) {
    // Tight coupling to global state
    DBConnection.Exec("INSERT...") 
}
```
*Better approach:* Inject dependencies via structs (Dependency Injection) so the lifecycle is controlled and mockable.

#### Inversion of Control
> Tell me about Inversion of Control and how it improves the design of code.

**Expert Answer:**
Inversion of Control (IoC) is a design principle where the custom-written portions of a program receive the flow of control from a generic framework, rather than the custom code controlling the flow. A common form of IoC is Dependency Injection (DI).
In Go, this is heavily implemented using `interfaces`. It improves design by decoupling the execution of a task from implementation. Components do not instantiate their dependencies; they simply declare an interface they need. This makes swapping implementations trivial (e.g., swapping a real database for an in-memory mock), and vastly simplifies testing.

*Read more in the original resources: [inversion-of-control.md](../design-patterns/inversion-of-control.md)*

#### Law of Demeter
> The Law of Demeter (the Principle of Least Knowledge) states that each unit should have only limited knowledge about other units and it should only talk to its immediate friends (sometimes stated as "don't talk to strangers").
> Would you write code violating this principle, show why it is a bad design and then fix it?

**Expert Answer:**
Violating the Law of Demeter creates brittle code that breaks when the internal structure of dependencies changes.

*Violation:*
```go
// We have to know that Customer has a Wallet, and Wallet has Money.
customer.Wallet.DeductMoney(50.0)
```
This is bad because the caller is intimately tied to the internal structure of `Customer`. If we change `Wallet` to a `CreditAccount`, the caller breaks.

*Fix:*
```go
// Tell, Don't Ask. 
customer.Charge(50.0)
```
Now, `Customer` encapsulates the payment logic. The caller doesn't need to know about wallets.

*Read more in the original resources: [law-of-demeter.md](../design-patterns/law-of-demeter.md)*

#### Active-Record
> Active-Record is the design pattern that promotes objects to include functions such as Insert, Update, and Delete, and properties that correspond to the columns in some underlying database table. In your opinion and experience, which are the limits and pitfalls of the this pattern?

**Expert Answer:**
Active-Record tightly couples your business domain logic directly to your persistence layer and database schema. 
*   **Violates Single Responsibility Principle:** Structs are responsible for both business rules and saving themselves.
*   **Testing:** It is notoriously difficult to unit test domain logic without spinning up a real database or mocking heavy ORM layers (like GORM).
*   **Complexity:** It works wonderfully for simple CRUD applications, but as business logic becomes complex (involving multiple entities and transactions), Active-Record becomes a massive bottleneck.

*Read more in the original resources: [active-record.md](../design-patterns/active-record.md)*

#### Data-Mapper
> Data-Mapper is a design pattern that promotes the use of a layer of Mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself. On the contrary, in Active-Record objects directly incorporate operations for persisting themselves to a database, and properties corresponding to the underlying database tables. Do you have an opinion on those patterns? When would you use one instead of the other?

**Expert Answer:**
Data-Mapper provides strict separation of concerns. Your domain objects (`Structs` in Go) are plain types unaware of the database.
*   **When to use Active-Record:** Prototyping, simple CRUD applications, and microservices where the domain logic is trivial.
*   **When to use Data-Mapper:** Enterprise applications, complex business domains requiring Domain-Driven Design (DDD), and systems where you need to rigorously unit test domain logic in isolation. It prevents database schemas from dictating domain object structures. Go's standard library and tools like `sqlx` often lean towards a Data-Mapper-like approach via manual scanning or mapping.

#### Billion Dollar Mistake
> Tony Hoare who invented the null reference once said "I call it my billion-dollar mistake".
> Would you discuss the techniques to avoid it, such as the Null Object Pattern introduced by the GOF book, or Option types?

**Expert Answer:**
Null references (nil pointers in Go) lead to runtime panics that crash applications unexpectedly. Techniques to avoid them include:
1.  **Values over Pointers:** In Go, default to passing by value unless you explicitly need mutation or performance gains. Zero values (`""`, `0`, `false`) are safer than `nil` pointers.
2.  **Multiple Return Values:** Go handles the absence of a value naturally using multiple return values (e.g., `value, ok := map[key]`).
3.  **Null Object Pattern:** Returning an object that implements the expected interface but does "nothing" (e.g., a `noopLogger` instead of `nil`). This prevents `if logger != nil` checks scattered throughout the codebase.

#### Inheritance vs Composition
> Many state that, in Object-Oriented Programming, composition is often a better option than inheritance. What's you opinion?

**Expert Answer:**
I strongly agree, and Go's entire design philosophy is built around this! Go completely omits classical inheritance (is-a relationship) because it creates rigid, deep hierarchies that are difficult to change.
Instead, Go uses Composition (via struct embedding) and Interfaces. Composition allows you to assemble behaviors dynamically. It promotes smaller, focused, and easily testable components. Go proved that you don't need inheritance to build robust, reusable software.

#### Anti-Corruption Layer
> What is an Anti-corruption Layer?

**Expert Answer:**
In Domain-Driven Design, an Anti-Corruption Layer (ACL) is a translation layer placed between a new, clean domain model and a legacy/external system. It prevents the external system's poorly designed or incompatible data structures from "corrupting" the new system's clean domain model. The ACL uses patterns like Facades and Adapters to map data seamlessly between the two bounded contexts.

#### Singleton
> Singleton is a design pattern that restricts the instantiation of a class to one single object. Writing a Thread-Safe Singleton class is not so obvious. Would you try?

**Expert Answer:**
In Go, writing a thread-safe singleton is incredibly elegant and robust using the `sync.Once` primitive from the standard library.

*Thread-Safe Singleton in Go:*
```go
package singleton

import "sync"

type singleton struct {}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

#### Data Abstraction
> The ability to change implementation without affecting clients is called Data Abstraction. Produce an example violating this property, then fix it.

**Expert Answer:**
*Violation:*
```go
package geometry

type Rectangle struct {
    // Internal representation is exported and exposed.
    Width  float64
    Height float64
}
```
If we later want to calculate area on the fly or change the internal representation, all clients break.

*Fix:*
```go
package geometry

type Rectangle struct {
    // Unexported fields
    width  float64
    height float64
}

// Exported methods
func (r *Rectangle) SetWidth(w float64) { r.width = w }
func (r *Rectangle) Width() float64     { return r.width }
// Now we can change the internal state storage without breaking clients.
```

#### Don't Repeat Yourself
> Write a snippet of code violating the Don't Repeat Yourself (DRY) principle. Then, fix it.

**Expert Answer:**
*Violation:*
```go
func PrintInvoice(items []Item) {
    var total float64
    for _, item := range items {
        total += item.Price
    }
    tax := total * 0.20
    fmt.Printf("Total: %f\n", total+tax)
}

func ProcessRefund(items []Item) {
    var total float64
    for _, item := range items {
        total += item.Price
    }
    tax := total * 0.20
    fmt.Printf("Refund: %f\n", total+tax)
}
```

*Fix (Extracting the common logic):*
```go
func calculateTotalWithTax(items []Item) float64 {
    var total float64
    for _, item := range items {
        total += item.Price
    }
    return total + (total * 0.20)
}

func PrintInvoice(items []Item) {
    fmt.Printf("Total: %f\n", calculateTotalWithTax(items))
}

func ProcessRefund(items []Item) {
    fmt.Printf("Refund: %f\n", calculateTotalWithTax(items))
}
```

*Read more in the original resources: [dont-repeat-yourself.md](../design-patterns/dont-repeat-yourself.md)*

#### Dependency Hell
> How would you deal with Dependency Hell?

**Expert Answer:**
Dependency hell occurs when packages require conflicting versions of shared dependencies. 
In Go, this is elegantly handled by **Go Modules** (`go.mod`).
1.  **Semantic Versioning:** Go modules enforce SemVer strictly.
2.  **Minimal Version Selection (MVS):** Go's algorithm for selecting versions prefers the *oldest* allowed version that satisfies all requirements, ensuring reproducible builds.
3.  **Vendoring:** If extreme isolation is needed, `go mod vendor` pulls all dependencies directly into the repository.
4.  **Minimize Dependencies:** In Go, the standard library is extremely powerful; the community heavily favors relying on the stdlib over importing massive third-party libraries for trivial tasks.

#### Goto Is Evil
> Is goto evil? What's your opinion on the use of goto?

**Expert Answer:**
In high-level application programming, `goto` is generally considered "evil" because it creates spaghetti code and makes control flow impossible to trace.
However, Go *does* include a `goto` statement. It is sometimes used in systems programming or low-level performance-critical code for cleanly breaking out of deeply nested loops or jumping to a unified error-handling/cleanup block at the end of a function (similar to C). Outside of these rare, specific use cases, it should be heavily avoided in favor of Go's `defer` and standard control flow.

#### Robustness Principle
> The robustness principle is a general design guideline for software that recommends "be conservative in what you send, be liberal in what you accept". Would you like to discuss the rationale of this principle?

**Expert Answer:**
Postel's Law is designed to make distributed systems and network protocols resilient. Being conservative in what you send ensures you strictly follow specifications, making life easier for consumers. Being liberal in what you accept means graceful degradation—ignoring unknown fields in JSON rather than crashing (which Go's `encoding/json` does by default).
*Caveat:* In modern secure systems, being *too* liberal is an anti-pattern. Parsing ambiguities caused by liberal parsers often lead to severe security vulnerabilities (e.g., HTTP Request Smuggling). Today, strict validation at the boundaries is often preferred over extreme liberality.

#### Separation of Concerns
> Separation of Concerns is a design principle for separating a computer program into distinct areas, each one addressing a separate concern. Would you discuss this topic?

**Expert Answer:**
SoC is the bedrock of maintainable software architecture. If presentation logic (HTTP handlers), business rules (tax calculation), and persistence (SQL queries) are mixed in one function, changes to the HTTP routing risk breaking database logic.
By separating these concerns (e.g., using layered architectures, Hexagonal Architecture/Ports and Adapters), teams can work in parallel, code becomes independently unit-testable (using interfaces to mock persistence), and implementations can be swapped without touching the core business logic.
