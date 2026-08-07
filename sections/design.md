### [[↑]](../README.md#toc) <a name='design'>Questions about Code Design:</a>

#### High Cohesion, Loose Coupling
> It is often said that one of the most important goals in Object-Oriented Design (and code design in general) is to have High Cohesion and Loose Coupling. What does it mean? Why is it that important and how is it achieved?

**Expert Answer:**
**High Cohesion** means that all elements within a module or struct belong strictly together. They share the same responsibility. **Loose Coupling** means that a module has very little knowledge of other modules; it interacts with them through simple, stable interfaces.
It is important because highly cohesive code is easy to understand and maintain, while loosely coupled code is easy to test and change without causing cascading failures. In Go, you achieve this by defining narrow `interfaces` (loose coupling) and grouping related functions into small packages (high cohesion).

#### Index 0
> Why do array indexes start with '0' in most languages?

**Expert Answer:**
It originates from C and assembly language, where an array name acts as a pointer to the start of a continuous block of memory. The index represents the *offset* from the base address. So, the first element has an offset of 0 (`address + 0`), the second has an offset of 1 (`address + 1 * size`), and so on. Starting at 1 would require a subtraction (`address + (index - 1) * size`) on every array access, which was computationally expensive on early hardware.

#### TDD
> How do tests and TDD influence code design?

**Expert Answer:**
Test-Driven Development (TDD) acts as a powerful design tool rather than just a verification method. Because you write the test before the implementation, you are forced to think about the *interface* and *usability* of the code from the perspective of a consumer. 
It naturally leads to:
1. **Loose Coupling:** You cannot easily unit test a function tightly coupled to a database; TDD forces you to inject dependencies (e.g., passing a database interface).
2. **Smaller Functions/Structs:** Testing large, complex functions is hard. TDD encourages breaking them down into highly cohesive, focused units.

#### DRY Violation
> Write a snippet of code violating the Don't Repeat Yourself (DRY) principle. Then, explain why it is a bad design, and fix it.

**Expert Answer:**
*Violation:*
```go
func HandleUserSignup(user User) {
    if user.Age < 18 {
        log.Println("Validation failed: User too young")
        return
    }
    // ... logic
}

func HandleUserUpdate(user User) {
    if user.Age < 18 {
        log.Println("Validation failed: User too young")
        return
    }
    // ... logic
}
```
This is bad because if the legal age changes, we have to update multiple places.

*Fix:*
```go
func isUserOfAge(user User) bool {
    if user.Age < 18 {
        log.Println("Validation failed: User too young")
        return false
    }
    return true
}

func HandleUserSignup(user User) {
    if !isUserOfAge(user) { return }
}
```

#### Cohesion vs Coupling
> What's the difference between cohesion and coupling?

**Expert Answer:**
*   **Cohesion** looks *inward*. It measures how strongly related the functions and data within a single module/package are. (e.g., A `billing` package containing only invoicing logic has high cohesion).
*   **Coupling** looks *outward*. It measures how dependent one module is on another. (e.g., If the `billing` package directly calls the `database` package, they are tightly coupled).

#### Refactoring
> What is refactoring useful for?

**Expert Answer:**
Refactoring is the process of restructuring existing computer code without changing its external behavior. It is critical for managing "Technical Debt". Over time, as features are added, code rots and loses its clean design. Refactoring is useful for:
1. Improving code readability and maintainability.
2. Reducing complexity and improving architecture.
3. Making it easier to find bugs.
4. Preparing the code structure to accommodate a new feature seamlessly.

#### Code Comments
> Are comments in code useful? Some say they should be avoided as much as possible, and hopefully made unnecessary. Do you agree?

**Expert Answer:**
Code should be largely self-documenting (expressive variable names, short functions). However, comments are absolutely essential for explaining *why* something is done, not *what* is being done. 
I agree that comments explaining "what" (e.g., `// loop over users`) indicate a failure in naming or design. But comments explaining non-obvious business rules, hacks required for external APIs, or complex algorithmic choices are invaluable. In Go, comments are also structurally important as they generate standard `godoc` documentation for packages and exported types.

#### Design vs Architecture
> What is the difference between design and architecture?

**Expert Answer:**
*   **Architecture** refers to the high-level structural choices of a system that are costly to change once implemented. This involves choosing microservices vs monolith, database choices, messaging queues, and how bounded contexts communicate.
*   **Design** refers to the lower-level implementation details within those architectural boundaries. This includes choosing algorithms, applying Go interfaces, struct layout, and applying design patterns (like Strategy or Factory). Architecture is macro; design is micro.

#### Early Testing
> In TDD, why are tests written before code?

**Expert Answer:**
Writing the test first forces the developer to define the contract and behavior of the component before getting bogged down in implementation details. It clarifies the requirements. If you write the code first, you often end up writing tests specifically tailored to pass the code you just wrote, masking edge cases. Furthermore, writing tests first guarantees 100% test coverage for all functional requirements by definition.

#### Multiple Inheritance
> C++ supports multiple inheritance, and Java allows a class to implement multiple interfaces. What impact does using these facilities have on orthogonality? Is there a difference in impact between using multiple inheritance and multiple interfaces? Is there a difference between using delegation and using inheritance?

**Expert Answer:**
*   **Multiple Inheritance (C++)** ruins orthogonality. It leads to the "Diamond Problem" (ambiguity when inheriting from two classes that share a base) and creates deep, tangled dependencies.
*   **Multiple Interfaces (Java/Go)** preserves orthogonality. Interfaces only promise *behavior*, not state or implementation. A struct in Go can satisfy dozens of single-method interfaces (like `io.Reader` and `io.Writer`) cleanly.
*   **Delegation vs Inheritance:** Inheritance is an "is-a" relationship, permanently binding the subclass to the superclass. Delegation (Composition in Go) is a "has-a" relationship, where a struct forwards work to an embedded component. Delegation preserves orthogonality and allows changing behaviors at runtime.

#### Domain Logic in Stored Procedures
> What are the pros and cons of holding domain logic in Stored Procedures?

**Expert Answer:**
*   **Pros:** Execution speed (avoids network trips between the app and DB), centralized logic if multiple applications (e.g., Go app and legacy PHP app) use the same database.
*   **Cons:** Business logic becomes fragmented. Stored procedures are notoriously difficult to unit test, version control (compared to app code), debug, and refactor. They tie your application permanently to a specific database vendor (e.g., Postgres PL/pgSQL). In modern development, domain logic belongs in the application layer.

#### OOP Took Over the World
> In your opinion, why has Object-Oriented Design dominated the market for so many years?

**Expert Answer:**
OOD dominated because it closely mapped to human mental models of the physical world (nouns as Objects, verbs as Methods). It provided a compelling way to structure complex GUI applications in the 90s (like Smalltalk and Java UIs). Additionally, the encapsulation of state and behavior made managing large teams of developers easier, as code boundaries were strongly enforced by classes. However, modern languages like Go emphasize a hybrid approach, taking the best of OOD (interfaces, encapsulation) while discarding the baggage (inheritance).

#### Bad Design
> What would you do to understand if your code has a bad design?

**Expert Answer:**
Code smells that indicate bad design include:
1.  **Rigidity:** A small change in one place requires cascading changes across the codebase.
2.  **Fragility:** The code breaks in unexpected places when a modification is made.
3.  **Immobility:** You cannot reuse a component in another project because it is deeply entangled with its current environment.
4.  **Testing friction:** If writing a unit test requires 100 lines of setup and mocking, the design is highly coupled and flawed.
