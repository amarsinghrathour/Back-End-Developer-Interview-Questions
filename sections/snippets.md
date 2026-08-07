### [[↑]](../README.md#toc) <a name='snippets'>Questions about snippets of code:</a>

#### Beware the Closure
> What's the output of this Go function (assuming Go 1.21 or older)?
> ```go
> func hookupevents() {
>     var funcs []func()
>     for i := 0; i < 3; i++ {
>         funcs = append(funcs, func() {
>             fmt.Println(i)
>         })
>     }
>     for _, f := range funcs {
>         f()
>     }
> }
> ```

**Expert Answer:**
The output is `3` printed three times.
In Go (prior to 1.22), the `for` loop variable `i` was instantiated only once per loop. The closure inside the loop captures a *reference* to the single variable `i`, not its value at that iteration. By the time the functions are actually executed, the loop has finished, and the final value of `i` is 3. (Note: This classic bug was finally fixed in Go 1.22!).

#### Runtime Type Preservation
> What's the output of this Go snippet and why?
> ```go
> var li []int
> var lf []float32
> if reflect.TypeOf(li) == reflect.TypeOf(lf) {
>     fmt.Println("Equal")
> } else {
>     fmt.Println("Not Equal")
> }
> ```

**Expert Answer:**
The output is `"Not Equal"`.
Go strictly preserves exact type information at runtime. When using reflection (`reflect.TypeOf`), `[]int` and `[]float32` are distinctly different types. 

*(Note: This is a classic interview question to contrast Go with JVM languages. In Java, Generics are implemented via **Type Erasure**. At runtime, generic type information is erased, meaning the Java equivalent of this snippet would actually evaluate to "Equal"! Go avoids this by using monomorphization to preserve exact types at runtime.)*

#### Memory Leak
> Can you spot the memory leak?
> ```go
> type Stack struct {
>     elements []interface{}
> }
> 
> func (s *Stack) Push(e interface{}) {
>     s.elements = append(s.elements, e)
> }
> 
> func (s *Stack) Pop() interface{} {
>     if len(s.elements) == 0 {
>         return nil
>     }
>     lastIndex := len(s.elements) - 1
>     e := s.elements[lastIndex]
>     s.elements = s.elements[:lastIndex]
>     return e
> }
> ```

**Expert Answer:**
Yes, in the `Pop()` method:
```go
func (s *Stack) Pop() interface{} {
    // ...
    e := s.elements[lastIndex]
    s.elements = s.elements[:lastIndex]
    return e
}
```
When an element is popped, the slice is re-sliced to `[:lastIndex]`. However, the physical backing array of `elements` still holds a pointer to the popped object at `lastIndex`. The Garbage Collector cannot free that object because the backing array is still pointing to it!
*Fix:* After fetching the element, explicitly nil out the array index before reslicing:
```go
e := s.elements[lastIndex]
s.elements[lastIndex] = nil // Free the reference!
s.elements = s.elements[:lastIndex]
return e
```

#### Kill the witch
> Can you get rid of this switch and make this snippet more object oriented?
> ```go
> type Formatter struct {
>     service Service
> }
> 
> func (f *Formatter) doTheJob(theInput string) string {
>     response := f.service.askForPermission()
>     switch response {
>     case "FAIL":
>         return "error"
>     case "OK":
>         return fmt.Sprintf("%s%s", theInput, theInput)
>     default:
>         return ""
>     }
> }
> ```

**Expert Answer:**
Use **Polymorphism** and the Strategy/Command pattern. Instead of returning raw Strings ("FAIL", "OK") that force the caller to use a switch statement, the `Service` should return an interface (e.g., `PermissionResponse`) that implements a `Format()` method.

```go
type PermissionResponse interface {
    Format(input string) string
}

type FailResponse struct{}
func (f FailResponse) Format(input string) string { return "error" }

type OKResponse struct{}
func (o OKResponse) Format(input string) string { return input + input }

// In the Formatter:
func (f *Formatter) DoTheJob(input string) string {
    response := f.service.AskForPermission() // returns PermissionResponse
    return response.Format(input) // No switch needed!
}
```

#### Kill the if
> Can you get rid of these ifs and make this snippet of code more object oriented?
> ```go
> type TheService struct {
>     fileHandler   FileHandler
>     fooRepository FooRepository
> }
> 
> func (s *TheService) Execute(file string) string {
>     rewrittenUrl := s.fileHandler.getXmlFileFromFileName(file)
>     executionId := s.fileHandler.getExecutionIdFromFileName(file)
> 
>     if executionId == "" || rewrittenUrl == "" {
>         return ""
>     }
> 
>     knownFoo := s.fooRepository.getFooByXmlFileName(rewrittenUrl)
> 
>     if knownFoo == nil {
>         return ""
>     }
> 
>     return knownFoo.DoThat(file)
> }
> ```

**Expert Answer:**
To completely remove the `if` statements, we must push the responsibility down into the dependencies using the **Null Object Pattern**. Instead of `TheService` constantly checking for empty strings or `nil` values, the `fooRepository` should guarantee it *always* returns an object that implements the `Foo` interface, even if the item is not found or the inputs are invalid.

Here is the refactored, object-oriented Go code:

```go
// 1. Define the interface
type Foo interface {
    DoThat(file string) string
}

// 2. Define a "Null Object" that safely does nothing
type NullFoo struct{}
func (n NullFoo) DoThat(file string) string { return "" }

// 3. Define the real implementation
type ValidFoo struct { /* ... */ }
func (v ValidFoo) DoThat(file string) string { return "Actual logic" }

// 4. Refactor TheService to blindly trust its dependencies
func (s *TheService) Execute(file string) string {
    rewrittenUrl := s.fileHandler.GetXmlFileFromFileName(file)
    executionId := s.fileHandler.GetExecutionIdFromFileName(file)
    
    // The repository is now responsible for validating the inputs.
    // If executionId or rewrittenUrl are invalid, or if the item isn't found,
    // the repository returns a `NullFoo` instead of `nil`.
    knownFoo := s.fooRepository.GetFoo(rewrittenUrl, executionId) 
    
    // Zero `if` statements! We safely call the method via polymorphism.
    return knownFoo.DoThat(file)
}
```

#### Kill the if-chain
> How would you refactor this code?
> ```go
> func ExecuteAll() error {
>     var err error
> 
>     if err = Operation1(); err == nil {
>         if err = Operation2(); err == nil {
>             if err = Operation3(); err == nil {
>                 if err = Operation4(); err == nil {
>                 } else {
>                     err = errors.New("OPERATION4FAILED")
>                 }
>             } else {
>                 err = errors.New("OPERATION3FAILED")
>             }
>         } else {
>             err = errors.New("OPERATION2FAILED")
>         }
>     } else {
>         err = errors.New("OPERATION1FAILED")
>     }
> 
>     return err
> }
> ```

**Expert Answer:**
This is the classic "Arrow Anti-Pattern." The solution is **Early Returns (Guard Clauses)**. In Go, this is standard practice because it keeps the "happy path" aligned to the left edge of the screen, making it vastly more readable.

```go
func ExecuteOperations() error {
    if err := Operation1(); err != nil {
        return OPERATION1FAILED
    }
    if err := Operation2(); err != nil {
        return OPERATION2FAILED
    }
    if err := Operation3(); err != nil {
        return OPERATION3FAILED
    }
    if err := Operation4(); err != nil {
        return OPERATION4FAILED
    }
    return nil // S_OK
}
```

*Read more in the original resources: [kill-the-if-chain.md](../snippets/kill-the-if-chain.md)*
