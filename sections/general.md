### [[↑]](../README.md#toc) <a name='general'>General Questions:</a>

#### Why FP?
> Why does functional programming matter? When should a functional programming language be used?

**Expert Answer:**
Functional programming matters because it eliminates shared mutable state. By ensuring functions are pure (no side effects) and data is immutable, you completely eliminate entire classes of concurrency bugs (like race conditions and deadlocks). FP should be used in highly concurrent systems, data transformation pipelines (like MapReduce), and complex domain logic where testability and determinism are paramount. While Go is not a purely functional language, adopting FP patterns (like higher-order functions and minimizing global state) makes Go code vastly more robust.

#### Browsers
> How do companies like Microsoft, Google, Opera and Mozilla profit from their browsers?

**Expert Answer:**
Primarily through default search engine deals. Google pays Mozilla (Firefox) hundreds of millions of dollars a year to be the default search engine, driving ad revenue back to Google. Google Chrome exists to ensure users stay in the Google ecosystem and view Google ads. Browsers also gather immense amounts of telemetry and user behavioral data, which fuels targeted advertising platforms.

#### TCP Sockets
> Why does opening a TCP socket have a large overhead?

**Expert Answer:**
Opening a TCP socket requires a "3-way handshake" (SYN, SYN-ACK, ACK) before any application data can be sent. This takes a minimum of 1.5 Round Trip Times (RTT). If you add TLS for HTTPS, you add another 1-2 RTTs for cryptographic key exchange. If a server is across the globe (e.g., 100ms ping), opening a socket can take 300-500ms before a single byte of HTTP data is transferred. This is why Connection Pooling (which Go's `net/http` client does automatically by default) is critical for backend performance.

#### Encapsulation
> What is encapsulation important for?

**Expert Answer:**
Encapsulation hides the internal state and implementation details of an object/struct from the outside world, exposing only a safe, controlled API (methods). It is important because it prevents external code from putting the object into an invalid state. In Go, encapsulation is achieved simply using capitalization: `type user struct` (unexported) vs `type User struct` (exported), and unexported fields `u.passwordHash`.

#### Real-time systems
> What is a real-time system and how is it different from an ordinary system?

**Expert Answer:**
A real-time system guarantees a response within a strict, predictable time constraint (deadline). 
*   **Hard real-time:** Missing a deadline causes total system failure (e.g., pacemaker, anti-lock brakes).
*   **Soft real-time:** Missing a deadline degrades performance but isn't fatal (e.g., video streaming).
Ordinary systems (like standard web servers) prioritize average throughput over worst-case latency. Real-time systems prioritize worst-case latency guarantees above all else.

#### Real-time and memory allocation
> What's the relationship between real-time languages and heap memory allocation?

**Expert Answer:**
Heap memory allocation requires Garbage Collection (GC) to clean up unused objects. Traditional GC algorithms (like Java's "Stop The World") halt all application execution to sweep memory. A 500ms GC pause in a hard real-time system (like a missile guidance system) is catastrophic. Therefore, true hard real-time systems use languages like C, C++, or Rust (no GC) or strictly pre-allocate all memory on startup to avoid dynamic heap allocation entirely. Go's GC is optimized for incredibly low latency (sub-millisecond pauses), making it excellent for soft real-time (like trading or gaming), but still unfit for hard real-time.

#### Immutability
> Immutability is the practice of setting values once, at the moment of their creation, and never changing them. How can immutability help write safer code?

**Expert Answer:**
If an object cannot change after it is created, you never have to worry if another thread/goroutine is modifying it while you read it. You don't need mutexes or locks to read immutable data. It completely neutralizes race conditions and makes reasoning about code flow trivial, since a variable passed to a function is guaranteed not to be modified behind your back.

#### Mutable vs Immutable
> What are the pros and cons of mutable and immutable values.

**Expert Answer:**
*   **Mutable:**
    *   *Pros:* Highly memory and CPU efficient (modifying an array in-place is fast).
    *   *Cons:* Dangerous in concurrent environments; requires complex locking (mutexes); harder to track state changes over time.
*   **Immutable:**
    *   *Pros:* Inherently thread-safe; pure functions; easy to undo/time-travel (e.g., Redux).
    *   *Cons:* Memory intensive. Changing one element in an immutable array requires copying the entire array, causing heavy Garbage Collection pressure.

#### Object-Relational Impedance Mismatch
> What's the Object-Relational impedance mismatch?

**Expert Answer:**
It is the fundamental philosophical clash between Object-Oriented Programming (OOP) and Relational Databases (RDBMS). 
OOP relies on encapsulation, deep inheritance hierarchies, and pointer navigation (graphs). SQL relies on flat tables, foreign keys, and set theory. Attempting to force an object graph into a flat table requires complex, leaky abstractions (ORMs like Hibernate or GORM) that often result in atrocious SQL queries (the N+1 problem) when the graph becomes deep. Go avoids this by favoring simple flat structs and Data Mappers (like `sqlx`) over heavy active-record ORMs.

#### Sizing a Cache
> Which principles would you apply to define the size of a cache?

**Expert Answer:**
1.  **Available RAM:** The hard physical limit. 
2.  **Working Set Size:** The cache only needs to be large enough to hold the *frequently accessed* data (the 20% of data accessed 80% of the time), not the entire database.
3.  **Eviction Policy:** Using LRU (Least Recently Used) or LFU (Least Frequently Used) ensures that once the size limit is hit, the least valuable data is dropped.
4.  **TTL (Time to Live):** Data that naturally expires limits unbounded cache growth.

#### TCP and HTTP
> What's the difference between TCP and HTTP?

**Expert Answer:**
*   **TCP (Transport Layer - Layer 4):** Ensures that a stream of bytes sent from computer A arrives at computer B perfectly intact and in order. It knows nothing about what those bytes mean.
*   **HTTP (Application Layer - Layer 7):** Sits *on top of* TCP. It dictates the *format* and *meaning* of those bytes (e.g., "This byte stream is a GET request for /index.html with these Headers"). TCP is the delivery truck; HTTP is the letter inside the truck.

#### Client-Side vs Server-Side
> What are the tradeoffs of client-side rendering vs. server-side rendering?

**Expert Answer:**
*   **Server-Side Rendering (SSR):** The server (e.g., a Go backend using `html/template`) generates full HTML and sends it to the browser.
    *   *Pros:* Blazing fast initial load, perfect for SEO, works on slow mobile devices.
    *   *Cons:* Full page reloads feel clunky; heavy server CPU load.
*   **Client-Side Rendering (CSR):** The server sends a blank HTML page and a massive JavaScript bundle (React/Vue). The JS renders the UI in the browser and fetches data via API.
    *   *Pros:* Highly dynamic, app-like feel; offloads rendering CPU to the user's device.
    *   *Cons:* Slow initial load (white screen while JS downloads); terrible SEO (unless pre-rendered); heavy memory usage on cheap phones.

#### Reliable and non-reliable channels
> How could you develop a reliable communication protocol based on a non-reliable one?

**Expert Answer:**
This is exactly how TCP is built on top of the unreliable IP protocol! You implement:
1.  **Sequence Numbers:** Tag every packet with a number so the receiver can reorder them if they arrive out of sequence.
2.  **Acknowledgments (ACKs):** The receiver sends a message back saying "I received packet #5".
3.  **Timeouts and Retries:** If the sender doesn't receive an ACK for packet #5 within 200ms, it assumes the packet was lost and resends it.
4.  **Checksums:** Validate the payload wasn't corrupted in transit.
