### [[↑]](../README.md#toc) <a name='concurrency'>Questions about Concurrency:</a>

#### Why?
> Why do we need concurrency, anyway? Explain.

**Expert Answer:**
We need concurrency for two main reasons:
1.  **Performance (Throughput & CPU Utilization):** Modern CPUs have hit a clock-speed wall and scale by adding more cores. A single-threaded application uses only 1 core while the other 15 sit idle. Concurrency allows us to utilize the entire CPU.
2.  **Responsiveness (Latency):** In a web server or UI, if a single thread is blocked waiting for a slow database query, the entire application freezes. Concurrency allows the program to handle other requests (or keep the UI responsive) while waiting for I/O. Go excels at this via goroutines, making highly concurrent network services trivial to write.

#### Testing Concurrency
> Why is testing multithreaded/concurrent code so difficult?

**Expert Answer:**
Because concurrency is inherently non-deterministic. A race condition might only manifest 1 in 10,000 executions depending on the exact microsecond timing of the operating system's thread scheduler. Unit tests might pass consistently on a developer's fast machine but fail randomly on a heavily loaded CI server. Reproducing bugs requires exactly recreating the timing of thread interleaving, which is practically impossible. 
In Go, we mitigate this heavily by using the built-in Data Race Detector (`go test -race`).

#### Race Conditions
> What is a race condition? Code an example, using whatever language you like.

**Expert Answer:**
A race condition occurs when two or more threads/goroutines access shared data concurrently, and at least one of them modifies it, without proper synchronization.

*Go Example:*
```go
var counter int

func Increment() {
    // Read -> Modify -> Write (Not atomic)
    counter++ 
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            Increment() // RACE CONDITION!
        }()
    }
    wg.Wait()
    // counter will rarely be 1000 due to lost updates
}
```
*Fix:* Use `sync.Mutex` to lock around `counter++`, or use `atomic.AddInt64`.

#### Deadlocks
> What is a deadlock? Would you be able to write some code that is affected by deadlocks?

**Expert Answer:**
A deadlock occurs when two or more concurrent processes are waiting for each other to release a resource (like a mutex), resulting in a complete standstill where neither can proceed.

*Go Example:*
```go
func main() {
    var mu1, mu2 sync.Mutex
    var wg sync.WaitGroup
    wg.Add(2)

    go func() { // Goroutine A
        defer wg.Done()
        mu1.Lock()
        time.Sleep(time.Millisecond) // Ensure B gets mu2
        mu2.Lock() // Waits forever for B to release mu2
        mu2.Unlock()
        mu1.Unlock()
    }()

    go func() { // Goroutine B
        defer wg.Done()
        mu2.Lock()
        time.Sleep(time.Millisecond) // Ensure A gets mu1
        mu1.Lock() // Waits forever for A to release mu1
        mu1.Unlock()
        mu2.Unlock()
    }()
    
    wg.Wait() // FATAL ERROR: all goroutines are asleep - deadlock!
}
```
*Fix:* Always acquire locks in the exact same deterministic order across all goroutines.

#### Process Starvation
> What is process starvation? If you need, let's review its definition.

**Expert Answer:**
Process starvation happens when a thread or process is perpetually denied the resources (like CPU time or a lock) it needs to execute its work. This usually happens because the scheduler prioritizes other threads (e.g., highly active goroutines continuously grabbing a lock before a lower-priority or sleeping goroutine gets a chance).
*Contrast with Deadlock:* In a deadlock, nobody makes progress. In starvation, the system as a whole is making progress, but one specific thread is left behind permanently.

#### Free Algorithm
> What is a wait free algorithm?

**Expert Answer:**
Wait-free algorithms are a strict subset of lock-free algorithms. 
In a lock-free system, at least *one* thread is guaranteed to make progress. In a **wait-free** system, *every* thread is guaranteed to complete its operation in a bounded number of steps, regardless of what other threads are doing or if they are suspended. 
They rely heavily on hardware-level atomic instructions (like Compare-And-Swap) rather than mutexes. Writing wait-free code is notoriously complex and usually reserved for highly specialized, low-level data structures (e.g., specific ring buffers).
