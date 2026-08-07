### [[↑]](../README.md#toc) <a name='languages'>Questions about Languages:</a>

#### 3 worst defects
> Tell me the 3 worst defects of your preferred language

**Expert Answer:**
Focusing on Go (Golang):
1. **Error Handling Verbosity:** The `if err != nil` pattern, while explicit, leads to highly repetitive and verbose code that can obscure the actual business logic.
2. **Lack of Enums:** Go lacks true sum types or enums (using `iota` and constants is a weak substitute that lacks compile-time exhaustiveness checking).
3. **Nil Interface Confusion:** A `nil` pointer inside an interface wrapper makes the interface itself non-`nil`. This is a massive gotcha for newcomers checking `if val == nil`.

#### Functional Programming
> Why is there a rising interest on Functional Programming?

**Expert Answer:**
Functional Programming (FP) avoids shared mutable state and side effects. As CPU clock speeds stagnated and multi-core processing became the norm, writing concurrent, thread-safe code in OOP became notoriously difficult (deadlocks, race conditions). FP's immutability makes concurrent programming drastically safer and more predictable. It also makes code highly testable since functions are deterministic (given the same input, always return the same output).

#### Closures
> What is a closure, and what is useful for? What's in common between closures and classes?

**Expert Answer:**
A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.
Like classes (objects), closures combine data (state) with behavior (functions). You can use a closure to encapsulate state without needing to define a full struct/class.

*Go Example:*
```go
func Counter() func() int {
    count := 0 // State encapsulated by the closure
    return func() int {
        count++
        return count
    }
}
```

#### Generics
> What are generics useful for?

**Expert Answer:**
Generics allow you to write functions and data structures that operate on any data type while still maintaining compile-time type safety. Before Go 1.18, you either had to write the same function for `int`, `float64`, etc., or use `interface{}` which bypassed compile-time safety and required runtime assertions.

*Go Example:*
```go
func Min[T constraints.Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}
```

#### High-Order Functions
> What are higher-order functions? What are they useful for? Write one, in your preferred language.

**Expert Answer:**
A higher-order function is a function that does at least one of the following: takes one or more functions as arguments, or returns a function as its result. They are essential for abstractions like `map`, `filter`, or middleware.

*Go Example:*
```go
// Middleware is a higher-order function
func LoggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("Request received")
        next(w, r)
    }
}
```

#### Loops and Recursion
> Write a loop, then transform it into a recursive function, using only immutable structures (i.e. avoid using variables). Discuss.

**Expert Answer:**
*Loop (Mutable):*
```go
func Sum(nums []int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
```

*Recursive (Immutable):*
```go
func SumRecursive(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    return nums[0] + SumRecursive(nums[1:])
}
```
*Discussion:* The recursive version has no mutable state (`total`), making it mathematically pure. However, in Go, recursion lacks tail-call optimization, meaning deep recursion will cause a stack overflow. Loops are heavily preferred in Go.

#### Functions as First-Class Citizens
> What does it mean when a language treats functions as first-class citizens? Why is it important?

**Expert Answer:**
It means functions can be assigned to variables, passed as arguments to other functions, and returned from functions, just like an `int` or a `string`. 
It is crucial because it enables higher-order functions, callbacks, closures, and highly flexible API designs (like functional options pattern in Go) that dramatically reduce boilerplate.

#### Anonymous Functions
> Show me an example where an anonymous function can be useful.

**Expert Answer:**
Anonymous functions (lambdas) are extremely useful for `defer` statements and short-lived asynchronous routines in Go.

*Go Example:*
```go
func Process() {
    // Anonymous function used in a goroutine
    go func() {
        log.Println("Running in background")
    }()
    
    // Anonymous function used to capture a variable for defer
    mu.Lock()
    defer func() {
        mu.Unlock()
        log.Println("Unlocked")
    }()
}
```

#### Static and Dynamic typing
> There are a lot of different type systems. Let's talk about static and dynamic type systems, and about strong and weak ones. You surely have an opinion and a preference about this topic. Would you like to share them, and discuss why and when would you promote one particular type system for developing an enterprise software?

**Expert Answer:**
*   **Static vs Dynamic:** Static (Go, Java) checks types at compile-time. Dynamic (Python, JS) checks types at runtime.
*   **Strong vs Weak:** Strong (Go, Python) strictly enforces types (no implicit coercion between e.g., int and string). Weak (JavaScript, C) allows implicit conversions.

For enterprise software, **Strongly, Statically Typed** languages (like Go) are highly preferred. The compiler catches entire classes of bugs before the code even runs, making massive refactoring safe. Dynamic languages allow for faster initial prototyping, but maintaining large dynamic codebases often requires massive test suites just to ensure type correctness.

#### Namespaces
> What are namespaces useful for? Invent an alternative.

**Expert Answer:**
Namespaces (or `packages` in Go) group related code and prevent naming collisions. Without them, you couldn't have a `math.Max` and a `custom.Max`.
*Alternative:* A tag-based resolution system where functions are tagged (e.g., `#math`, `#core`), and the compiler infers which function you mean based on the types of arguments passed or the context of the caller.

#### Language Interoperability
> Talk about interoperability between Java and C# (in alternative, choose 2 other arbitrary languages)

**Expert Answer:**
Let's talk about Go and C via `cgo`.
Go has built-in interoperability with C. You can write C code directly in Go files using comments right above `import "C"`.
*   **Pros:** Allows Go to leverage decades of highly optimized C libraries (like SQLite or image processing).
*   **Cons:** It breaks Go's blazing-fast cross-compilation, adds significant overhead to function calls across the boundary, and introduces memory safety risks (segfaults) back into the Go runtime.

#### Hate of Java
> Why do many software engineers not like Java?

**Expert Answer:**
Java suffers from a reputation of immense boilerplate (Getters/Setters, massive class names like `AbstractSingletonProxyFactoryBean`), heavy memory footprints (due to the JVM), and slow startup times. While modern Java has improved immensely, the lingering corporate legacy of verbose, over-architected Object-Oriented patterns (like extreme factory patterns and deep inheritance) leaves a bad taste compared to the lean, pragmatic approach of languages like Go.

#### Good and Bad Languages
> What makes a good language good and a bad language bad?

**Expert Answer:**
A "good" language gets out of the developer's way. It has a readable syntax, a strong standard library, fast compilation/execution, and excellent tooling (formatters, linters, package managers). Go is great because of its simplicity and native concurrency.
A "bad" language has unpredictable behavior (like JS type coercion `[] + {}`), fragmented ecosystems, slow tooling, or an overly complex specification that requires developers to hold too many rules in their head.

#### Referential Transparency
> Write two functions, one referentially transparent and the other one referentially opaque. Discuss.

**Expert Answer:**
*Referentially Transparent (Pure):*
```go
func Add(a, b int) int {
    return a + b // Always returns same output for same inputs
}
```

*Referentially Opaque (Impure):*
```go
var counter int
func IncrementAndAdd(a, b int) int {
    counter++ // Side effect!
    return a + b + counter 
}
```
Referential transparency means you can replace a function call with its resulting value without changing the program's behavior. It makes code trivially testable and parallelizable.

#### Stack and Heap
> What is a stack and what is a heap? What's a stack overflow?

**Expert Answer:**
*   **Stack:** Fast, contiguous memory allocated automatically for function execution (local variables). It grows and shrinks as functions are called and return.
*   **Heap:** Slower, dynamic memory used for objects that must outlive the function that created them. In Go, the Garbage Collector manages this.
*   **Stack Overflow:** Occurs when a program uses more stack memory than is available, usually caused by infinitely deep recursive function calls.

#### Pattern Matching
> Some languages, especially the ones that promote a functional approach, allow a technique called pattern matching. Do you know it? How is pattern matching different from switch clauses?

**Expert Answer:**
Pattern matching (found in Rust, Scala, Erlang) is like a `switch` statement on steroids. While a `switch` usually evaluates a single value against constants, pattern matching can deconstruct complex data types, match on shapes of data, and bind variables in a single expression. It guarantees exhaustiveness at compile-time. Go lacks true pattern matching, relying on `switch` and type assertions (`switch v := i.(type)`).

#### Exceptions
> Why do some languages have no exceptions by design? What are the pros and cons?

**Expert Answer:**
Go explicitly omits exceptions in favor of returning `error` as a standard value.
*   **Pros:** Control flow is explicitly visible. You know exactly which functions can fail, and the handling is done right at the call site rather than jumping invisibly up the call stack. It prevents hidden crash loops.
*   **Cons:** It leads to extreme verbosity (`if err != nil`). Error handling can drown out the actual business logic of the function.

#### Variant and Contravariant Inheritance
> If `Cat` is an `Animal`, is `TakeCare<Cat>` a `TakeCare<Animal>`?

**Expert Answer:**
This deals with generic variance. In Go (since 1.18 generics), types are invariant. A `[]Cat` is *not* a `[]Animal`. You cannot pass a slice of Cats to a function expecting a slice of Animals. This is a deliberate design choice to prevent runtime type errors (e.g., if it were allowed, you could insert a `Dog` into the `[]Animal`, which would corrupt the underlying `[]Cat`).

#### Constructors and Interfaces
> In Java, C# and many other languages, why are constructors not part of the interface?

**Expert Answer:**
Interfaces define *behavior* (what an object can do once it exists). Constructors define *instantiation* (how an object is created). Since interfaces are meant to abstract away implementation details, dictating exactly how an underlying struct/class must be initialized violates that abstraction. In Go, we use factory functions (e.g., `NewUser()`) which can return interfaces, completely decoupling creation from behavior.

#### Node.js
> In the last years there has been a lot of hype around Node.js. What's your opinion on using a language that was initially conceived to run in the browser in the backend?

**Expert Answer:**
Node.js successfully proved that asynchronous, non-blocking I/O is highly scalable for web servers. Using the same language (JS) across the stack lowered the barrier to entry. However, JavaScript's single-threaded nature makes it terrible for CPU-bound tasks, and its dynamic typing (without TS) causes scaling pain in large teams. Go provides a much better backend experience: native concurrency (goroutines) across multiple cores, strict static typing, and blazing fast performance, without the "callback hell" or heavy memory footprint of Node.js.

#### Java and time-traveling
> Pretend you have a time machine and pretend that you have the opportunity to go to a particular point in time during Go's history, and talk with some of the architects. What would you try to convince them of?

**Expert Answer:**
I would travel to the early days of Go and heavily advocate for **Enums / Sum Types**. The current standard of using `iota` for enums is fundamentally flawed because the compiler cannot check for exhaustiveness in `switch` statements, and out-of-bounds integers can easily be cast to the "enum" type. A true algebraic data type system would have eliminated countless bugs without complicating the language.

#### Eliminate Null
> Imagine you want to remove the possibility to have null references in your preferred language: how would you achieve this goal? What consequences would this have?

**Expert Answer:**
In Go, `nil` is heavily used for pointers and interfaces. To eliminate it, we would need to introduce the `Option[T]` pattern into the standard library (now possible with Generics) and force all pointers to be initialized to a valid memory address. 
*   **Consequence:** The language would become vastly safer (no nil pointer dereferences). However, it would become more verbose, as you'd constantly be unwrapping `Option` types, and allocating default objects for everything could slightly impact memory performance.
