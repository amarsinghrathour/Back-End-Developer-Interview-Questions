### [[↑]](../README.md#toc) <a name='algorithms'>Questions about logic and algorithms:</a>

#### FIFO with LIFO
> Make a FIFO queue using only LIFO stacks. Then build a LIFO stack using only FIFO queues.

**Expert Answer:**
*   **Queue using Stacks:** Use two stacks (`inbox` and `outbox`). Push elements to `inbox`. When a dequeue is requested, if `outbox` is empty, pop everything from `inbox` and push to `outbox` (reversing the order). Then pop from `outbox`.
*   **Stack using Queues:** Use two queues (`q1`, `q2`). To push `x`, enqueue it to `q2`. Then dequeue everything from `q1` and enqueue to `q2`. Swap the names of `q1` and `q2`. Now `q1` has the newest element at the front.

#### Stack Overflow
> Write a snippet of code affected by a stack overflow.

**Expert Answer:**
Infinite recursion without a base case.
```go
func InfiniteRecursion() {
    InfiniteRecursion()
}
```

#### Tail Recursive n!
> Write a tail-recursive version of the factorial function.

**Expert Answer:**
Go does *not* optimize tail calls, but the theoretical implementation passes an accumulator.
```go
func factorialTail(n int, accumulator int) int {
    if n == 0 {
        return accumulator
    }
    return factorialTail(n-1, n*accumulator)
}
// Call with: factorialTail(5, 1)
```

#### REPL
> Using your preferred language, write a REPL that echoes your inputs. Evolve it to make it an RPN calculator.

**Expert Answer:**
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    var stack []float64
    for {
        fmt.Print("> ")
        if !scanner.Scan() { break }
        token := scanner.Text()
        
        if val, err := strconv.ParseFloat(token, 64); err == nil {
            stack = append(stack, val)
        } else if len(stack) >= 2 {
            a, b := stack[len(stack)-2], stack[len(stack)-1]
            stack = stack[:len(stack)-2]
            switch token {
            case "+": stack = append(stack, a+b)
            case "*": stack = append(stack, a*b)
            // ... - and /
            }
        }
        fmt.Println(stack)
    }
}
```

#### Defragger
> How would you design a "defragger" utility?

**Expert Answer:**
At a high level:
1. Scan the filesystem's FAT/MFT to identify all files and their physical block locations on the disk.
2. Identify fragmented files (files split across non-contiguous blocks).
3. Find a continuous block of free space large enough to hold the fragmented file.
4. Read the fragmented file into memory (or a temporary contiguous disk area).
5. Write the file sequentially into the new contiguous free space.
6. Update the filesystem table to point to the new location and mark the old blocks as free.

#### Mazes
> Write a program that builds random mazes.

**Expert Answer:**
The easiest approach is **Recursive Backtracking** (a randomized Depth First Search).
1. Create a grid of cells surrounded by walls.
2. Choose a random starting cell and mark it as visited.
3. While the current cell has unvisited neighbors:
    a. Randomly choose one unvisited neighbor.
    b. Knock down the wall between the current cell and the chosen neighbor.
    c. Push the current cell to a stack.
    d. Make the chosen neighbor the current cell and mark it visited.
4. If there are no unvisited neighbors, pop a cell from the stack and make it the current cell. Repeat until the stack is empty.

#### Memory Leaks
> Write a sample program that produces a memory leak.

**Expert Answer:**
In a garbage-collected language like Go, leaks happen when you unintentionally hold onto references.
```go
var cache = make(map[string][]byte) // Global map

func ProcessRequest(id string) {
    // Allocate 10MB per request
    data := make([]byte, 10*1024*1024) 
    cache[id] = data 
    // We never delete from the global cache, causing a massive memory leak.
}
```

#### PRNG
> Generate a sequence of unique random numbers.

**Expert Answer:**
If the range is known and relatively small (e.g., 1 to 10,000), use a **Fisher-Yates Shuffle**.
```go
func UniqueRandoms(n int) []int {
    nums := make([]int, n)
    for i := range nums { nums[i] = i }
    
    rand.Seed(time.Now().UnixNano())
    rand.Shuffle(n, func(i, j int) {
        nums[i], nums[j] = nums[j], nums[i]
    })
    return nums
}
```

#### Garbage Collecting
> Write a simple garbage collection system.

**Expert Answer:**
**Mark-and-Sweep Algorithm:**
1. **Mark Phase:** Start from the "roots" (global variables and stack variables). Traverse every object they point to (recursively traversing pointers). Mark every visited object with a boolean flag `is_reachable = true`.
2. **Sweep Phase:** Iterate over every single allocated object in the heap. If `is_reachable` is false, free the memory. If it is true, reset the flag to false for the next GC cycle.

#### Queues
> Write a basic message broker, using whatever language you like.

**Expert Answer:**
```go
type Broker struct {
    topics map[string][]chan string
    mu     sync.RWMutex
}

func (b *Broker) Subscribe(topic string) <-chan string {
    ch := make(chan string, 10)
    b.mu.Lock()
    b.topics[topic] = append(b.topics[topic], ch)
    b.mu.Unlock()
    return ch
}

func (b *Broker) Publish(topic, msg string) {
    b.mu.RLock()
    defer b.mu.RUnlock()
    for _, ch := range b.topics[topic] {
        select {
        case ch <- msg: // Non-blocking send
        default: // Drop if subscriber is full
        }
    }
}
```

#### Simple Web Server
> Write a very basic web server. Draw a road map for features to be implemented in the future.

**Expert Answer:**
```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, World!"))
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
*Roadmap:*
1. Add graceful shutdown (capturing SIGTERM).
2. Add structured logging and metrics (Prometheus).
3. Add a router (e.g., `chi`) for path variables.
4. Add middleware for authentication (JWT) and rate-limiting.

#### Sorting Huge Files
> How would you sort a 10GB file? How would your approach change with a 10TB one?

**Expert Answer:**
*   **10GB File:** If you have 16GB of RAM, load it entirely into memory and run a standard Quicksort (`sort.Slice` in Go).
*   **10TB File:** You must use **External Merge Sort**.
    1. Read 1GB chunks of the file into RAM.
    2. Sort the 1GB chunk in RAM.
    3. Write the sorted chunk to a temporary file on disk (resulting in 10,000 sorted files).
    4. Open all 10,000 files. Create a Min-Heap. Read the first line of each file into the heap.
    5. Pop the smallest element from the heap and write it to the final output file.
    6. Read the next line from the file that produced the smallest element and push it to the heap. Repeat until done.

#### Duplicates
> How would you programmatically detect file duplicates?

**Expert Answer:**
To avoid reading identical gigabyte files multiple times:
1. Group all files by exact File Size (O(1) operation). Ignore files with a unique size.
2. For files with the same size, calculate the MD5 hash of the first 4KB of each file. Group by this partial hash.
3. For files that share a partial hash, calculate the full SHA-256 hash. If full hashes match, the files are duplicates.
