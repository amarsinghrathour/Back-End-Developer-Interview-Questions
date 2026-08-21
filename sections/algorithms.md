### [[↑]](../README.md#toc) <a name='algorithms'>Questions about logic and algorithms:</a>

#### FIFO with LIFO
> Make a FIFO queue using only LIFO stacks. Then build a LIFO stack using only FIFO queues.

**Expert Answer:**

**The Short Answer:** 
You can simulate a Queue by transferring elements between two Stacks (inbox and outbox), and you can simulate a Stack using two Queues by cycling elements.

**The Deep Dive:** 
*   **Queue using Stacks:** You need two stacks: `inbox` and `outbox`. Enqueue operations always push to `inbox`. For a Dequeue operation, if `outbox` is empty, you pop *everything* from `inbox` and push it to `outbox`. This reverses the LIFO order into FIFO. You then pop from `outbox`.
*   **Stack using Queues:** You need two queues: `q1` and `q2`. To push an element, enqueue it to `q2`. Then, dequeue *everything* currently in `q1` and enqueue it to `q2`. Finally, swap the names of `q1` and `q2`. Now, `q1` holds the newest element at the absolute front.

**The Trade-offs (Pros/Cons):**
* **Pros:** Great for academic exercises and passing whiteboard interviews.
* **Cons:** Extremely inefficient in the real world. A native Queue operates in O(1) time, while these simulated structures degrade to O(N) time for their worst-case operations due to the constant transferring of elements.

**Code Example:**
```go
// Implementing a FIFO Queue using two LIFO Stacks (slices)
type MyQueue struct {
    inbox  []int
    outbox []int
}

func (q *MyQueue) Enqueue(val int) {
    q.inbox = append(q.inbox, val)
}

func (q *MyQueue) Dequeue() int {
    if len(q.outbox) == 0 {
        // Transfer all from inbox to outbox (reversing order)
        for i := len(q.inbox) - 1; i >= 0; i-- {
            q.outbox = append(q.outbox, q.inbox[i])
        }
        q.inbox = nil // Clear inbox
    }
    
    // Pop from outbox
    lastIdx := len(q.outbox) - 1
    val := q.outbox[lastIdx]
    q.outbox = q.outbox[:lastIdx]
    return val
}
```

#### Stack Overflow
> Write a snippet of code affected by a stack overflow.

**Expert Answer:**

**The Short Answer:** 
A stack overflow occurs when an infinite recursion lacks a base case, causing the program to exceed its allocated call stack memory.

**The Deep Dive:** 
Every time a function is called, the OS allocates a "stack frame" to hold local variables, arguments, and the return address. If a function calls itself infinitely, the program continuously allocates new stack frames until it exhausts the memory limit set by the operating system (or language runtime), triggering a fatal crash.

**The Trade-offs (Pros/Cons):**
* **Pros (of recursion):** Leads to very elegant, readable code for tree/graph traversals compared to iterative approaches.
* **Cons (of recursion):** Always runs the risk of a stack overflow if the depth of the data structure is unbounded.

**Code Example:**
```go
// A fatal stack overflow in Go
func InfiniteRecursion() {
    // Missing base case to stop the recursion!
    InfiniteRecursion()
}

func main() {
    InfiniteRecursion()
    // Output: runtime: goroutine stack exceeds 1000000000-byte limit
}
```

#### Tail Recursive n!
> Write a tail-recursive version of the factorial function.

**Expert Answer:**

**The Short Answer:** 
A tail-recursive function passes an accumulator argument so that the recursive call is the absolute last operation performed, theoretically allowing the compiler to optimize away the stack frame.

**The Deep Dive:** 
In standard recursion, the function must wait for the recursive call to return before it can multiply the result (e.g., `n * factorial(n-1)`). This requires keeping the current stack frame open. In a tail-recursive function, you pass the running total (accumulator) forward. The compiler (in languages like Elixir or Haskell) recognizes that no further work is needed after the return, and overwrites the current stack frame instead of creating a new one, preventing Stack Overflows entirely. (Note: The Go compiler currently does *not* support Tail Call Optimization, so this still consumes stack space in Go).

**The Trade-offs (Pros/Cons):**
* **Pros:** Allows infinite recursion depth without stack overflows in supported languages.
* **Cons:** Often slightly harder to read and reason about than standard recursion.

**Code Example:**
```go
// Tail Recursive Factorial in Go
// Note: Go does not optimize this, but the algorithm is correct.
func factorialTail(n int, accumulator int) int {
    if n == 0 {
        return accumulator // Base case: return the accumulated total
    }
    // The recursive call is the VERY LAST thing evaluated
    return factorialTail(n-1, n*accumulator) 
}

// Called via: factorialTail(5, 1)
```

#### REPL
> Using your preferred language, write a REPL that echoes your inputs. Evolve it to make it an RPN calculator.

**Expert Answer:**

**The Short Answer:** 
A REPL (Read-Eval-Print Loop) continuously reads user input, processes it, prints the result, and loops. We can evolve it into an RPN (Reverse Polish Notation) calculator using a Stack.

**The Deep Dive:** 
RPN evaluates mathematical expressions without parentheses by placing operators *after* their operands (e.g., `3 4 +` instead of `3 + 4`). To implement this, we scan input tokens one by one. If the token is a number, we push it to a stack. If the token is an operator, we pop the top two numbers from the stack, apply the operator, and push the result back onto the stack.

**The Trade-offs (Pros/Cons):**
* **Pros (of RPN):** Extremely easy to parse programmatically; completely eliminates the need for complex operator precedence (BODMAS) or parenthesis matching algorithms.
* **Cons:** Hard for humans to read and write without practice.

**Code Example:**
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    var stack []float64
    
    for {
        fmt.Print("RPN> ")
        if !scanner.Scan() { break } // EOF
        token := scanner.Text()
        
        // 1. If it's a number, push to stack
        if val, err := strconv.ParseFloat(token, 64); err == nil {
            stack = append(stack, val)
        } else if len(stack) >= 2 {
            // 2. If it's an operator, pop two, evaluate, push result
            b := stack[len(stack)-1] // Note: Top of stack is the second operand
            a := stack[len(stack)-2]
            stack = stack[:len(stack)-2] // Pop both
            
            switch token {
            case "+": stack = append(stack, a+b)
            case "*": stack = append(stack, a*b)
            case "-": stack = append(stack, a-b)
            case "/": stack = append(stack, a/b)
            }
        }
        fmt.Println("Stack:", stack)
    }
}
```

#### Defragger
> How would you design a "defragger" utility?

**Expert Answer:**

**The Short Answer:** 
A defragger reads the filesystem table to find files scattered across non-contiguous physical disk blocks, reads them into memory, and writes them back sequentially into a contiguous free space.

**The Deep Dive:** 
On spinning Hard Disk Drives (HDDs), reading sequential data is fast, but physically moving the read/write head to a different sector is incredibly slow. As files are deleted and created, large files become "fragmented" across gaps left by deleted files.
A defragger works by:
1. Scanning the Master File Table (MFT) to map all files and free space.
2. Identifying files split into multiple fragments.
3. Finding a contiguous block of free space large enough to hold the file.
4. Reading the fragmented file into memory and writing it sequentially to the new block.
5. Updating the MFT to point to the new location and marking the old fragments as free space.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastically speeds up read/write times on mechanical HDDs.
* **Cons:** Completely unnecessary and actively harmful to Solid State Drives (SSDs). Because SSDs have no moving parts, random access is just as fast as sequential access, and defragging merely wastes the limited write-cycles of the flash memory.

**Code Example:**
```go
// A conceptual mapping of a Defragger's logic
func Defrag(disk *Disk, mft *MasterFileTable) {
    for _, file := range mft.Files {
        if file.IsFragmented() {
            // Find a sequential block big enough for the whole file
            newLocation := disk.FindContiguousFreeSpace(file.Size)
            
            // Read scattered data, write it sequentially
            data := disk.ReadScattered(file.Sectors)
            disk.WriteSequential(newLocation, data)
            
            // Update pointers
            mft.UpdatePointer(file.ID, newLocation)
        }
    }
}
```

#### Mazes
> Write a program that builds random mazes.

**Expert Answer:**

**The Short Answer:** 
You can generate a perfect maze using a randomized Depth-First Search (Recursive Backtracking) to carve paths out of a solid grid of walls.

**The Deep Dive:** 
A "perfect maze" has exactly one path between any two cells and no loops. 
The algorithm:
1. Create a grid where every cell is surrounded by 4 walls.
2. Pick a random starting cell and push it to a Stack.
3. While the Stack is not empty:
    * Pop the current cell.
    * Find all its unvisited neighbors.
    * If there are unvisited neighbors:
        * Push the current cell back onto the stack.
        * Choose a random unvisited neighbor.
        * Knock down the wall between the current cell and the neighbor.
        * Mark the neighbor as visited and push it to the stack.
    * If a cell has no unvisited neighbors, the algorithm naturally backtracks (pops from the stack).

**The Trade-offs (Pros/Cons):**
* **Pros:** Generates aesthetically pleasing mazes with long, winding corridors.
* **Cons:** Because it goes very deep before backtracking, it can cause stack overflows on massive grids if implemented using pure recursion instead of an explicit Stack data structure.

**Code Example:**
```go
// Conceptual Randomized DFS for Maze Generation
func CarvePassagesFrom(cx, cy int, grid [][]Cell) {
    grid[cy][cx].Visited = true
    
    // Shuffle directions to ensure random carving
    directions := []Dir{North, South, East, West}
    rand.Shuffle(len(directions), func(i, j int) {
        directions[i], directions[j] = directions[j], directions[i]
    })
    
    for _, dir := range directions {
        nx, ny := cx+dir.dx, cy+dir.dy
        
        if IsValidCell(nx, ny, grid) && !grid[ny][nx].Visited {
            // Knock down the wall between current and neighbor
            grid[cy][cx].RemoveWall(dir)
            grid[ny][nx].RemoveWall(Opposite(dir))
            
            // Recurse
            CarvePassagesFrom(nx, ny, grid)
        }
    }
}
```

#### Memory Leaks
> Write a sample program that produces a memory leak.

**Expert Answer:**

**The Short Answer:** 
In garbage-collected languages, memory leaks occur when a developer unintentionally maintains a global or long-lived reference to objects that are no longer needed.

**The Deep Dive:** 
In languages like C, a leak is forgetting to call `free()`. In Go or Java, the Garbage Collector (GC) frees memory automatically, but *only* if the object is unreachable. The most common leak is a global cache (like a map) that grows infinitely because keys are inserted but never deleted. The GC sees the global map referencing the data and refuses to clean it up.

**The Trade-offs (Pros/Cons):**
* **Pros (of GC):** Eliminates 90% of memory management bugs (like Use-After-Free).
* **Cons:** Creates a false sense of security; logical memory leaks (holding references too long) still crash the server with OOM (Out Of Memory) errors.

**Code Example:**
```go
// A classic logical memory leak in Go
var cache = make(map[string][]byte) // Global reference

func ProcessRequest(id string) {
    // Allocate 10MB of data for this request
    data := make([]byte, 10*1024*1024) 
    
    // Storing it in the global map prevents the Garbage Collector
    // from EVER freeing this 10MB, even after the function returns!
    cache[id] = data 
    
    // Unless we explicitly `delete(cache, id)`, this server will OOM crash.
}
```

#### PRNG
> Generate a sequence of unique random numbers.

**Expert Answer:**

**The Short Answer:** 
To generate a unique sequence of random numbers from a known range, populate an array with the sequential numbers and apply the Fisher-Yates Shuffle algorithm.

**The Deep Dive:** 
A naive approach is to generate a random number, check if it's in a `map` or `Set`, and retry if it's a duplicate. This becomes exponentially slower as the set fills up, eventually acting like an infinite loop. 
The Fisher-Yates Shuffle solves this in perfect O(N) time. You create an array `[1, 2, 3, 4]`. You iterate backwards, picking a random index between 0 and your current position, and swap the two elements. This guarantees every permutation is equally likely and no duplicates are ever generated.

**The Trade-offs (Pros/Cons):**
* **Pros:** Guaranteed O(N) performance; impossible to generate duplicates.
* **Cons:** Requires O(N) memory. You cannot use this if asked to generate 10 billion unique numbers, as the array won't fit in RAM.

**Code Example:**
```go
// Generate 10,000 unique random numbers from 0 to 9999
func UniqueRandoms(n int) []int {
    nums := make([]int, n)
    for i := range nums { 
        nums[i] = i 
    }
    
    // Go's standard library implements Fisher-Yates natively!
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

**The Short Answer:** 
A basic Garbage Collector uses the "Mark-and-Sweep" algorithm: it traces all reachable objects starting from global roots (Mark) and then frees any memory that wasn't reached (Sweep).

**The Deep Dive:** 
1. **Mark Phase:** The GC pauses the program. It looks at the "roots" (active variables on the stack and global variables). It traverses every pointer it finds, recursively exploring the object graph. Every object visited is marked with a boolean flag: `is_reachable = true`.
2. **Sweep Phase:** The GC iterates linearly over every single object allocated in the heap. If `is_reachable` is false, the application can no longer access it, so the memory is freed. If it is true, the GC resets the flag to false to prepare for the next cycle.

**The Trade-offs (Pros/Cons):**
* **Pros:** Safely manages memory, preventing segmentation faults and dangling pointers.
* **Cons:** The "Stop The World" pause required during the Mark phase can cause unpredictable latency spikes in real-time applications (though modern Go minimizes these pauses to sub-millisecond).

**Code Example:**
```go
// Conceptual Mark and Sweep Algorithm
func GC_Run(roots []*Object, heap []*Object) {
    // 1. MARK
    for _, root := range roots {
        markReachable(root) // Recursively sets obj.reachable = true
    }
    
    // 2. SWEEP
    for i, obj := range heap {
        if !obj.reachable {
            freeMemory(obj) // Dead object
        } else {
            obj.reachable = false // Reset for next GC cycle
        }
    }
}
```

#### Queues
> Write a basic message broker, using whatever language you like.

**Expert Answer:**

**The Short Answer:** 
A simple message broker utilizes an in-memory map of channels (or queues) protected by a Read-Write Mutex to safely route published messages to multiple subscribers concurrently.

**The Deep Dive:** 
The broker pattern (Publish/Subscribe) decouples the sender from the receiver. We need a dictionary mapping a `topic` (string) to a list of subscribers. Because multiple goroutines will be subscribing and publishing at the exact same time, we must protect the map with a `sync.RWMutex` to prevent fatal concurrent map writes. 

**The Trade-offs (Pros/Cons):**
* **Pros:** Highly decoupled; extremely fast since it's entirely in-memory.
* **Cons:** Lacks persistence. If the server restarts, all undelivered messages are permanently lost (unlike Kafka or RabbitMQ, which write to disk).

**Code Example:**
```go
type Broker struct {
    topics map[string][]chan string
    mu     sync.RWMutex
}

// Subscribe returns a channel that will receive messages for the topic
func (b *Broker) Subscribe(topic string) <-chan string {
    ch := make(chan string, 10) // Buffered channel
    b.mu.Lock()
    b.topics[topic] = append(b.topics[topic], ch)
    b.mu.Unlock()
    return ch
}

// Publish broadcasts a message to all subscribers of the topic
func (b *Broker) Publish(topic, msg string) {
    b.mu.RLock()
    defer b.mu.RUnlock()
    
    for _, ch := range b.topics[topic] {
        select {
        case ch <- msg: // Non-blocking send
        default: 
            // If the subscriber's channel is full, drop the message 
            // to prevent the broker from deadlocking.
        }
    }
}
```

#### Simple Web Server
> Write a very basic web server. Draw a road map for features to be implemented in the future.

**Expert Answer:**

**The Short Answer:** 
A basic Go web server registers a handler function to a route and calls `ListenAndServe` on a port.

**The Deep Dive:** 
Go's `net/http` package is incredibly powerful, acting as a production-ready concurrent web server right out of the box (spawning a new goroutine for every incoming request automatically).
**Roadmap for a Production Server:**
1.  **Graceful Shutdown:** Intercept `SIGTERM` (Ctrl+C) to finish processing active requests before killing the server.
2.  **Routing:** Swap the default multiplexer for `chi` or `gorilla/mux` to support path parameters (e.g., `/users/{id}`).
3.  **Middleware:** Add standard middleware layers for Request Logging, Panic Recovery, and CORS.
4.  **Configuration:** Inject database connections and configurations via structs rather than relying on global variables.

**The Trade-offs (Pros/Cons):**
* **Pros (of standard library):** Zero third-party dependencies required for a highly performant web server.
* **Cons:** The standard multiplexer is relatively basic (prior to Go 1.22), lacking built-in regex routing and middleware chaining out of the box.

**Code Example:**
```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    // Define the route and handler
    http.HandleFunc("/api/hello", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, `{"message": "Hello, World!"}`)
    })

    log.Println("Server starting on :8080...")
    // Start the server (blocks forever)
    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatal(err)
    }
}
```

#### Sorting Huge Files
> How would you sort a 10GB file? How would your approach change with a 10TB one?

**Expert Answer:**

**The Short Answer:** 
A 10GB file can be loaded directly into modern RAM and sorted using standard algorithms. A 10TB file exceeds RAM and requires the **External Merge Sort** algorithm.

**The Deep Dive:** 
*   **10GB File:** Most modern backend servers have 16GB or 32GB of RAM. You can read the entire file into an array of strings in memory and run a standard O(N log N) algorithm like Quicksort (`sort.Slice` in Go).
*   **10TB File:** This exceeds RAM. You must sort it using disk space via External Merge Sort:
    1. Read 1GB chunks of the file into RAM, sort them, and write them back to disk as temporary files (yielding 10,000 sorted files).
    2. Open all 10,000 files simultaneously.
    3. Read the first line of each file into a Min-Heap data structure in RAM.
    4. Pop the absolute smallest element from the heap and write it to the final output file on disk.
    5. Read the next line from whichever file produced that smallest element and push it to the heap. Repeat until all files are consumed.

**The Trade-offs (Pros/Cons):**
* **Pros:** Allows sorting datasets of functionally infinite size.
* **Cons:** Disk I/O is extremely slow, making this process take hours or days compared to in-memory operations.

**Code Example:**
```go
// Conceptual snippet of the merging phase using a Min-Heap
func MergeSortedFiles(files []*os.File, output *os.File) {
    heap := InitMinHeap()
    
    // Push the very first line of every file into the heap
    for _, f := range files {
        heap.Push(ReadNextLine(f))
    }
    
    for heap.Len() > 0 {
        // Pop the globally smallest string
        smallest := heap.Pop()
        output.WriteString(smallest.Text + "\n")
        
        // Grab the next line from the file that just gave us the smallest string
        nextLine := ReadNextLine(smallest.SourceFile)
        if nextLine != nil {
            heap.Push(nextLine)
        }
    }
}
```

#### Duplicates
> How would you programmatically detect file duplicates?

**Expert Answer:**

**The Short Answer:** 
Filter files first by absolute file size, then by a partial hash of the first 4KB, and finally by a full SHA-256 hash to find true duplicates with minimal I/O overhead.

**The Deep Dive:** 
Hashing a 10GB file takes significant CPU and Disk I/O time. If you hash every file on a hard drive, it will take hours. We must optimize via a filtering pipeline:
1.  **Size (O(1)):** Group all files by exact byte size. If a file is the only one that is exactly `4,192,301` bytes, it cannot have a duplicate. Ignore it.
2.  **Partial Hash (O(1)):** For files that share the exact same size, calculate the MD5 hash of only the first 4KB of data. This catches files that happen to be the same size but are completely different formats (like an MP3 vs a PDF).
3.  **Full Hash (O(N)):** If two files share the exact same size AND the exact same partial hash, calculate the full SHA-256 hash of the entire file. If the full hashes match, you have guaranteed duplicates.

**The Trade-offs (Pros/Cons):**
* **Pros:** Eliminates 99% of heavy disk I/O compared to the naive approach of hashing everything.
* **Cons:** Hash collisions are technically possible (though statistically negligible with SHA-256), meaning a paranoid system might require a final byte-by-byte comparison.

**Code Example:**
```go
// The filtering pipeline
func FindDuplicates(files []File) {
    sizeMap := GroupBySize(files)
    
    for _, sameSizeFiles := range sizeMap {
        if len(sameSizeFiles) > 1 {
            // Only hash the first 4096 bytes!
            partialMap := GroupByPartialHash(sameSizeFiles, 4096)
            
            for _, samePartialFiles := range partialMap {
                if len(samePartialFiles) > 1 {
                    // Only do the expensive full hash as a last resort
                    FindTrueDuplicates(samePartialFiles) 
                }
            }
        }
    }
}
```


#### Rate Limiting (Token Bucket vs Leaky Bucket)
> Compare Token Bucket and Leaky Bucket algorithms for API rate limiting. When would you use which?

**Expert Answer:**

**The Short Answer:** 
Token Bucket allows bursts of traffic up to a maximum limit, while Leaky Bucket smooths traffic into a steady, constant outbound rate.

**The Deep Dive:** 
In Token Bucket (e.g., Stripe's API), tokens are added to a bucket at a fixed rate. If a user has a burst of requests, they can consume all tokens instantly, processing the burst quickly. In Leaky Bucket (e.g., NGINX default), requests pour in but leak out (are processed) at a strictly constant rate. If the bucket is full, new requests overflow and are dropped (HTTP 429). 
* Use Token Bucket when you want to absorb sudden spikes (e.g., a user logging in and loading 5 dashboard widgets simultaneously).
* Use Leaky Bucket when you must protect downstream legacy systems that will crash if they receive more than exactly X requests per second.

**The Trade-offs (Pros/Cons):**
* **Pros (Token):** More flexible and forgiving for end-users.
* **Cons (Leaky):** Can artificiality delay valid requests during minor spikes.

#### Geo-Spatial Hashing (Geohash)
> How do Geohashes work for spatial proximity searches (like Uber or Tinder)?

**Expert Answer:**

**The Short Answer:** 
Geohashing converts 2D latitude and longitude coordinates into a 1D string of characters, where matching prefixes indicate physical proximity.

**The Deep Dive:** 
Finding drivers near a user using raw latitude/longitude requires computing the Haversine formula against every driver in the database (O(N)), which is too slow. Geohashing divides the world into a grid. For example, the string "9q8yy" represents San Francisco. The longer the string, the smaller the grid (more precise). To find nearby drivers, you simply do a string prefix search (`WHERE geohash LIKE '9q8yy%'`), which leverages B-Tree indexes for O(log N) lookups.

**The Trade-offs (Pros/Cons):**
* **Pros:** Blazing fast indexing and retrieval of spatial data using standard databases (Redis/Postgres).
* **Cons:** Edge cases occur at the grid boundaries (two points can be 1 meter apart but have completely different Geohash prefixes).

#### Consistent Hashing & Data Hotspots
> Consistent hashing distributes data evenly, but what happens when a specific key (like a viral celebrity's profile) becomes a massive hotspot?

**Expert Answer:**

**The Short Answer:** 
Consistent hashing routes all traffic for a specific key to a single node. To mitigate a viral hotspot, you must salt the key or use a dedicated CDN/L1 cache.

**The Deep Dive:** 
Consistent hashing guarantees `hash("JustinBieberProfile")` always hits Node A. If that profile goes viral, Node A gets 100,000 RPS while Nodes B, C, and D sit idle. This is a "hot key" issue. The algorithm itself cannot fix this. You must solve it at the application layer by either:
1. Adding a short-lived local cache (L1 cache) in the memory of the web servers before it hits the distributed cache.
2. "Salting" the key: appending a random number (1-10) to the key so it distributes across 10 different nodes, though this requires the client to aggregate the data later.

**The Trade-offs (Pros/Cons):**
* **Pros:** Prevents single-node meltdowns during viral events.
* **Cons:** Salting drastically increases read complexity.

#### Bloom Filters in Distributed Systems
> Why are Bloom Filters so prevalent in distributed databases like Cassandra?

**Expert Answer:**

**The Short Answer:** 
Bloom Filters are highly memory-efficient probabilistic data structures used to quickly determine if a piece of data is *definitely not* present on a disk, saving expensive disk I/O reads.

**The Deep Dive:** 
In LSM-Tree databases like Cassandra, data is scattered across many immutable files (SSTables) on disk. When querying for a specific row, opening and reading every file would be disastrously slow. Instead, Cassandra keeps a small Bloom Filter in RAM for every SSTable. When queried, the Bloom Filter can return "Absolutely Not Here" (so the DB skips the file) or "Possibly Here" (so the DB reads the disk). This eliminates 99% of unnecessary disk I/O.

**The Trade-offs (Pros/Cons):**
* **Pros:** Astronomically faster reads; extremely small memory footprint.
* **Cons:** Cannot return a definite "Yes" (false positives occur); you cannot remove elements from a standard Bloom Filter.
