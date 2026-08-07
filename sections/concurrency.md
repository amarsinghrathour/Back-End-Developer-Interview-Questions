### [[↑]](../README.md#toc) <a name='concurrency'>Questions about Concurrency:</a>

#### Why?
> Why do we need concurrency, anyway? Explain.

**Expert Answer:**

**The Short Answer:** 
Concurrency is required to maximize CPU utilization across multiple cores and to keep applications responsive by not blocking execution during slow I/O operations (like database queries or network requests).

**The Deep Dive:** 
We need concurrency for two main reasons:
1.  **Throughput (CPU Utilization):** Modern CPUs have hit a clock-speed wall and now scale by adding more physical cores. A single-threaded application uses exactly 1 core while the other 15 sit idle. Concurrency allows us to utilize the entire CPU.
2.  **Responsiveness (Latency):** In a web server, if a single thread blocks while waiting for a 2-second database query, the entire application freezes for everyone. Concurrency allows the program to handle other users' requests while waiting for that I/O to finish. Go excels at this via goroutines, making highly concurrent network services trivial to write.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive increases in application throughput and responsiveness.
* **Cons:** Introduces immense architectural complexity; introduces non-deterministic bugs (race conditions, deadlocks) that are notoriously difficult to reproduce and fix.

**Code Example:**
```go
func handleRequest(userID string) {
    // 1. Blocks for 1 second. 
    // In a single-threaded app, nobody else can use the server!
    data := slowDatabaseQuery(userID) 
    
    // 2. In Go, the net/http server spawns a goroutine for every request.
    // While this specific goroutine blocks, the Go scheduler instantly 
    // switches to a different goroutine to serve another user.
    fmt.Println(data)
}
```

#### Testing Concurrency
> Why is testing multithreaded/concurrent code so difficult?

**Expert Answer:**

**The Short Answer:** 
Concurrent code is inherently non-deterministic, meaning bugs like race conditions might only manifest 1 out of 10,000 times depending on the unpredictable microsecond timing of the OS thread scheduler.

**The Deep Dive:** 
Unit tests run sequentially and pass predictably. However, concurrent code depends on "thread interleaving." A race condition occurs only if Thread A reaches a specific line of code at the exact same microsecond as Thread B. Your tests might pass consistently on a fast MacBook, but fail randomly on a heavily loaded CI/CD server because the thread execution timing is different. Reproducing the bug requires perfectly recreating that timing, which is practically impossible. In Go, we rely heavily on the Data Race Detector (`go test -race`) to find these issues analytically rather than behaviorally.

**The Trade-offs (Pros/Cons):**
* **Pros (of using `-race`):** Finds hidden concurrency bugs automatically during CI testing.
* **Cons (of using `-race`):** The race detector slows down execution by 2-10x and consumes more memory, so it cannot be run in production.

**Code Example:**
```bash
# In Go, standard testing might pass a broken concurrent program:
go test ./...

# You MUST test with the race detector to catch non-deterministic data races:
go test -race ./...
# Output: WARNING: DATA RACE ...
```

#### Race Conditions
> What is a race condition? Code an example, using whatever language you like.

**Expert Answer:**

**The Short Answer:** 
A race condition occurs when two or more concurrent threads access shared data, and at least one modifies it, without proper synchronization (like a mutex lock), leading to corrupted data.

**The Deep Dive:** 
Most operations, like `counter++`, look like a single step but are actually three: Read the value, Increment the value, Write the value back. If Thread A and Thread B read the value `10` at the exact same time, they will both increment it to `11` and write it back. One increment is completely lost. To fix this, you must serialize access using a Mutex (so only one thread can Read-Modify-Write at a time) or use atomic hardware instructions.

**The Trade-offs (Pros/Cons):**
* **Pros (of fixing with Mutexes):** Guarantees data safety and correctness.
* **Cons:** Locks cause "contention." If 1,000 threads are waiting on a lock, your concurrent application degrades into a slow, sequential application.

**Code Example:**
```go
// BAD: A classic Race Condition
var counter int
func main() {
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // Read-Modify-Write is NOT atomic! Updates will be lost.
            counter++ 
        }()
    }
    wg.Wait()
    fmt.Println(counter) // Will rarely be 1000
}

// GOOD: Safe with Mutex
var mu sync.Mutex
// go func() { ... mu.Lock(); counter++; mu.Unlock(); ... }()
```

#### Deadlocks
> What is a deadlock? Would you be able to write some code that is affected by deadlocks?

**Expert Answer:**

**The Short Answer:** 
A deadlock is a complete system standstill that occurs when two or more concurrent processes are waiting for each other to release a resource (like a mutex lock).

**The Deep Dive:** 
Imagine Thread A locks Database 1 and needs Database 2 to finish. Thread B locks Database 2 and needs Database 1 to finish. Thread A waits for B, and B waits for A. Neither can proceed, and they will wait for eternity. Deadlocks are fatal errors. The most robust way to prevent deadlocks is to enforce a strict **Lock Ordering** policy across the entire codebase—always acquire locks in the exact same deterministic order.

**The Trade-offs (Pros/Cons):**
* **Pros (of Lock Ordering):** Mathematically guarantees freedom from deadlocks.
* **Cons:** Extremely difficult to enforce manually in a large, complex codebase without static analysis tools.

**Code Example:**
```go
// A fatal deadlock in Go
func main() {
    var mu1, mu2 sync.Mutex
    var wg sync.WaitGroup
    wg.Add(2)

    go func() { 
        defer wg.Done()
        mu1.Lock()
        time.Sleep(time.Millisecond) // Force context switch
        mu2.Lock() // Waiting for mu2 forever
        mu2.Unlock(); mu1.Unlock()
    }()

    go func() {
        defer wg.Done()
        mu2.Lock()
        time.Sleep(time.Millisecond) // Force context switch
        mu1.Lock() // Waiting for mu1 forever
        mu1.Unlock(); mu2.Unlock()
    }()
    
    // FATAL ERROR: all goroutines are asleep - deadlock!
    wg.Wait() 
}
```

#### Process Starvation
> What is process starvation? If you need, let's review its definition.

**Expert Answer:**

**The Short Answer:** 
Process starvation occurs when a concurrent thread is perpetually denied the resources (like CPU time or a lock) it needs to execute its work.

**The Deep Dive:** 
Unlike a deadlock (where the whole system freezes), in a starvation scenario, the system is actively working, but one specific thread is left behind permanently. This usually happens due to poor scheduling policies or heavy lock contention. For example, if a massive pool of highly active "reader" threads constantly acquire an RWMutex, a single "writer" thread might starve because it can never find a moment where *zero* readers hold the lock.

**The Trade-offs (Pros/Cons):**
* **Pros (of prioritizing certain threads):** Increases overall throughput by prioritizing fast operations over slow ones.
* **Cons:** Creates wildly unpredictable latency outliers for the starved threads (e.g., 99% of requests take 10ms, but 1% take 30 seconds).

**Code Example:**
```go
// In Go, the sync.RWMutex is designed to prevent writer starvation.
// If a writer calls Lock(), it blocks all NEW readers from acquiring RLock().
// This ensures the writer eventually gets a turn.
var mu sync.RWMutex

// Readers
go func() { mu.RLock(); defer mu.RUnlock() }()

// Writer (Will NOT starve in Go)
go func() { mu.Lock(); defer mu.Unlock() }() 
```

#### Free Algorithm
> What is a wait free algorithm?

**Expert Answer:**

**The Short Answer:** 
A wait-free algorithm guarantees that *every* concurrent thread will complete its operation in a bounded number of steps, regardless of what other threads are doing.

**The Deep Dive:** 
Wait-free algorithms are a strict, incredibly difficult subset of lock-free algorithms. 
In a simple lock-free system (using Compare-And-Swap loops), if heavy contention occurs, at least *one* thread makes progress, but others might have to retry their loops infinitely (starvation). 
In a **wait-free** system, there are no retry loops. Every thread finishes in predictable time, even if other threads are suspended by the OS. They rely strictly on hardware-level atomic instructions rather than software locks.

**The Trade-offs (Pros/Cons):**
* **Pros:** Absolute guarantee of no deadlocks and no starvation; predictable, real-time latency.
* **Cons:** Notoriously complex to design; usually restricted to highly specialized, low-level data structures (like single-producer/single-consumer ring buffers); often requires more memory overhead.

**Code Example:**
```go
// A lock-free (but NOT wait-free) algorithm.
// If multiple threads hit this, they might loop/retry many times.
func LockFreeAdd(addr *int32, delta int32) {
    for {
        old := atomic.LoadInt32(addr)
        // Compare-And-Swap (Hardware atomic)
        if atomic.CompareAndSwapInt32(addr, old, old+delta) {
            return // Success!
        }
        // Failed because another thread won. Retry.
    }
}

// A Wait-Free algorithm executes in exactly ONE step, no loops.
func WaitFreeAdd(addr *int32, delta int32) {
    atomic.AddInt32(addr, delta) // Hardware handles serialization
}
```
