### [[↑]](../README.md#toc) <a name='architecture'>Questions about Software Architecture:</a>

#### No Cache
> When is a cache not useful or even dangerous?

**Expert Answer:**

**The Short Answer:** 
A cache is dangerous when consistency is strictly required (like banking transactions) and useless when data has extremely high cardinality with zero repeat access.

**The Deep Dive:** 
Caching improves read latency but sacrifices consistency. If you cache a user's bank balance and the cache falls out of sync with the database, a user might overdraw their account. In these highly transactional scenarios, reading from the authoritative source is mandatory. Conversely, caching is useless for logging unique API request metrics; if every request is unique, your cache hit rate is 0%, meaning you waste memory and add network latency managing evictions for data that will never be requested again.

**The Trade-offs (Pros/Cons):**
* **Pros (of avoiding caches):** Guarantees strong consistency; reduces architectural complexity; avoids cache-invalidation bugs (one of the hardest problems in computer science).
* **Cons:** The database must absorb 100% of the read load, requiring significantly more hardware scaling.

**Code Example:**
```go
// BAD: Caching highly transactional data
func GetBalance(userID string) float64 {
    // If the cache is stale, the user gets free money
    if balance, ok := cache.Get(userID); ok { return balance }
    return db.GetBalance(userID)
}

// GOOD: Bypassing cache for transactions
func GetBalanceStrongConsistency(userID string) float64 {
    // Always read from the source of truth
    return db.GetBalance(userID)
}
```

#### Event-Driven Architecture
> Why does Event-Driven Architecture improve scalability?

**Expert Answer:**

**The Short Answer:** 
Event-Driven Architecture (EDA) decouples producers from consumers, allowing asynchronous processing and independent scaling of system components.

**The Deep Dive:** 
In a traditional synchronous system, if a user uploads a video, the HTTP server blocks until the video is encoded, starving the server of resources. In EDA, the HTTP handler simply drops a `VideoUploaded` event onto a message broker (like Kafka or RabbitMQ) and immediately returns a `202 Accepted` to the user. The web servers remain completely unblocked. Background workers pull events from the queue at their own pace.

**The Trade-offs (Pros/Cons):**
* **Pros:** Massive scalability; resilience (if workers crash, messages remain in the queue); decoupled deployments.
* **Cons:** Eventual consistency makes UI updates tricky; incredibly difficult to debug tracing across asynchronous boundaries.

**Code Example:**
```go
// Synchronous (Blocks the web server)
func UploadHandler(w http.ResponseWriter, r *http.Request) {
    EncodeVideo(r.Body) // Blocks for 10 minutes!
    w.WriteHeader(200)
}

// Event-Driven (Scalable)
func EDAUploadHandler(w http.ResponseWriter, r *http.Request) {
    kafka.Publish("video.uploaded", videoID) // Takes 1ms
    w.WriteHeader(http.StatusAccepted) // Frees up the web server instantly
}
```

#### Readability
> What makes code readable?

**Expert Answer:**

**The Short Answer:** 
Readable code optimizes for the engineer reading it 6 months later by using expressive names, early returns, and prioritizing clarity over cleverness.

**The Deep Dive:** 
Code is read far more often than it is written. Readability means minimizing cognitive load. In Go, this is culturally enforced. It means using short variable names for tight scopes (like `i` in a loop) but highly descriptive names for global scopes. It means avoiding deeply nested `if/else` statements in favor of Guard Clauses. Most importantly, "Clear is better than clever." A readable implementation using a simple loop is infinitely superior to a "clever" one-liner using bitwise hacks that nobody on the team can comprehend.

**The Trade-offs (Pros/Cons):**
* **Pros:** Drastically reduces onboarding time; prevents bugs during refactoring; simplifies code reviews.
* **Cons:** Sometimes "readable" code is slightly more verbose or marginally less performant than highly optimized, unreadable alternatives.

**Code Example:**
```go
// BAD: Unreadable (Nested, poor names)
func proc(d []int) {
    if d != nil {
        if len(d) > 0 {
            // logic
        }
    }
}

// GOOD: Readable (Guard clauses, expressive names)
func ProcessData(data []int) {
    if len(data) == 0 {
        return // Early return
    }
    // logic
}
```

#### Emergent and Evolutionary
> What is the difference between emergent design and evolutionary architecture?

**Expert Answer:**

**The Short Answer:** 
Emergent Design happens at the micro/code level as patterns naturally arise during development, while Evolutionary Architecture happens at the macro/system level to allow long-term structural changes.

**The Deep Dive:** 
*   **Emergent Design:** Instead of architecting every class upfront, you write the simplest code possible. As duplication or complexities arise, you refactor into reusable components. TDD heavily drives this.
*   **Evolutionary Architecture:** You design the macro system (databases, queues, deployment) so it can adapt over years without massive rewrites. Utilizing bounded contexts, microservices, and continuous delivery allows the architecture itself to evolve safely.

**The Trade-offs (Pros/Cons):**
* **Pros:** Prevents analysis paralysis and "Big Design Up Front"; ensures you only build abstractions when you actually need them.
* **Cons:** Requires rigorous automated testing; without discipline, "emergent design" quickly devolves into "spaghetti architecture."

**Code Example:**
```go
// Emergent Design in Go:
// Don't define a massive interface upfront.
// Start with a concrete struct.
type LocalUploader struct{} 

// Later, when you need an S3 uploader, extract the interface:
type Uploader interface {
    Upload(file []byte) error
}
```

#### Scale-Out, Scale-Up
> Scale out vs scale up: how are they different? When to apply one, when the other?

**Expert Answer:**

**The Short Answer:** 
Scaling up (vertical) means buying a bigger, more powerful server; scaling out (horizontal) means adding more cheap servers to share the load.

**The Deep Dive:** 
Scaling up is simple: if your database is slow, buy a machine with more RAM and CPU. There are zero architectural changes required. However, you eventually hit a hard physical limit. Scaling out is theoretically infinite, but architecturally complex. To scale out, your web servers must be stateless (managed by a load balancer). Scaling out a relational database is notoriously difficult (requiring sharding or complex active-active replication).

**The Trade-offs (Pros/Cons):**
* **Pros of Scale-Up:** Zero architectural changes; maintains strong consistency.
* **Pros of Scale-Out:** Infinite scalability; fault tolerance (if one node dies, others take over).

**Code Example:**
```go
// To Scale Out, state MUST NOT be kept in the application memory.

// BAD: Prevents Scale-Out
var userSessions = make(map[string]User)

// GOOD: Allows Scale-Out (State is externalized)
func GetSession(sessionID string) User {
    return redisCache.Get(sessionID) 
}
```

#### Failures User Sessions
> How to deal with failover and user sessions?

**Expert Answer:**

**The Short Answer:** 
User sessions must be decoupled from application memory and stored externally so that any server can process any request.

**The Deep Dive:** 
If a user authenticates and the session is stored in the RAM of `Server A`, the user is "sticky" to that server. If `Server A` crashes, the session is wiped, and the user is forcefully logged out. To survive failovers, the application must be stateless. The Session ID is sent to the client as a signed cookie. When the load balancer routes the next request to `Server B`, `Server B` reads the cookie and fetches the session data from a highly available, external key-value store like an isolated Redis cluster.

**The Trade-offs (Pros/Cons):**
* **Pros:** True high availability; allows horizontal scaling of web tiers; zero downtime deployments without affecting active users.
* **Cons:** Adds latency to every request (requires a network hop to Redis); introduces a new point of failure (the session cache).

**Code Example:**
```go
// Handling sessions in a scalable Go web app
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        cookie, _ := r.Cookie("session_id")
        
        // Fetch from external Redis cluster, NOT local memory
        sessionData, err := redisClient.Get(cookie.Value).Result()
        if err != nil {
            http.Error(w, "Unauthorized", 401)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

#### CQRS
> What is CQRS (Command Query Responsibility Segregation)? How is it different from the oldest Command-Query Separation Principle?

**Expert Answer:**

**The Short Answer:** 
CQS is a code-level principle separating read/write methods, while CQRS is an architectural pattern physically separating read and write data stores.

**The Deep Dive:** 
*   **CQS (Code):** A function should either change state (Command) or return state (Query), but never both.
*   **CQRS (Architecture):** In high-traffic systems, the optimal database for writing (e.g., event sourcing) is rarely the optimal database for reading. CQRS separates them. You might write commands to Kafka, which asynchronously projects the data into an Elasticsearch cluster optimized purely for blazing-fast read queries.

**The Trade-offs (Pros/Cons):**
* **Pros:** Independent scaling of reads vs writes; allows optimizing storage schemas perfectly for their specific access patterns.
* **Cons:** High complexity; guarantees Eventual Consistency (the read database lags behind the write database), which complicates UI logic.

**Code Example:**
```go
// CQS (Code Level)
func SaveUser(u User) {} // Command (no return)
func GetUser(id string) User {} // Query (no state change)

// CQRS (Architecture Level)
type CommandHandler struct {
    kafkaWriter KafkaProducer // Writes to event log
}

type QueryHandler struct {
    elasticReader ElasticClient // Reads from optimized search index
}
```

#### n-tier
> Would you discuss the pros and cons of such an approach?

**Expert Answer:**

**The Short Answer:** 
N-tier architecture physically separates an application into logical layers (Presentation, Business, Data Access), usually deployed on separate servers.

**The Deep Dive:** 
By separating the layers, changes to the database schema don't bleed into the UI. It enforces strict Separation of Concerns. Security is enhanced because the presentation layer cannot access the database directly; it must route through the business layer. However, for modern applications, strict N-tier often leads to "anemic domain models," where layers are just dumb pipes passing DTOs back and forth.

**The Trade-offs (Pros/Cons):**
* **Pros:** High security isolation; allows independent scaling of tiers; clear organizational boundaries for teams.
* **Cons:** Over-engineering for simple apps; introduces severe network latency (multiple hops to fulfill one request); creates deployment complexity.

**Code Example:**
```go
// N-Tier boundaries represented in code
// 1. Presentation Tier
func ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // 2. Business Tier
    result := businessLayer.ProcessAction() 
    w.Write(result)
}

// 3. Data Tier
func (b *BusinessLayer) ProcessAction() []byte {
    return b.dataLayer.FetchData() 
}
```

#### Scalability
> How would you design a software system for scalability?

**Expert Answer:**

**The Short Answer:** 
Design for stateless compute, asynchronous background processing, aggressive caching, and read-replicas for databases.

**The Deep Dive:** 
1.  **Stateless Compute:** Go web servers must hold no state so they can be horizontally scaled infinitely behind an AWS ALB.
2.  **Asynchronous I/O:** Move heavy processing (email sending, image processing) off the HTTP request path and onto message queues (RabbitMQ/SQS) processed by background workers.
3.  **Caching:** Cache static assets at the Edge (CDN) and heavy DB queries in memory (Redis/Memcached).
4.  **Database Scaling:** Use Read-Replicas for read-heavy workloads. If writes become the bottleneck, shard the database.

**The Trade-offs (Pros/Cons):**
* **Pros:** System can handle viral traffic spikes without crashing; high availability.
* **Cons:** Drastically increased infrastructure costs; highly complex deployments; tracing bugs across decoupled distributed systems is difficult.

**Code Example:**
```go
// Scalability rule #1: Move slow work to a queue
func HandleSignup(w http.ResponseWriter, r *http.Request) {
    db.SaveUser(r)
    
    // Do NOT send the email synchronously!
    // mailer.SendWelcomeEmail() 
    
    // Send to a queue to be processed by a scalable worker pool
    queue.Publish("email_queue", r.Email) 
    w.WriteHeader(200)
}
```

#### C10K
> It may be interesting to discuss the strategies you know to deal with that problem. Would you like to try?

**Expert Answer:**

**The Short Answer:** 
The C10K problem (handling 10,000 concurrent connections) was solved by replacing heavy OS threads with asynchronous I/O and lightweight userspace scheduling.

**The Deep Dive:** 
In the 90s, handling 10k connections was impossible because web servers spawned a heavy OS thread (costing 1MB+ of RAM) for every connection. Modern solutions rely on event loops and non-blocking I/O (`epoll` on Linux). 
Go solved this elegantly at the language level. You write synchronous-looking code, spawning a `goroutine` for every connection. Goroutines only take ~2KB of memory. Under the hood, Go's runtime multiplexes tens of thousands of goroutines onto just a few OS threads using `epoll`, completely hiding the complexity of asynchronous I/O from the developer.

**The Trade-offs (Pros/Cons):**
* **Pros:** Enables massive concurrency (WebSockets, chat servers) on minimal hardware.
* **Cons:** Event-loop models (like Node.js) can block the entire server if a CPU-heavy task runs. (Go's scheduler mitigates this via preemption).

**Code Example:**
```go
// Go natively solves C10K. This loop can easily handle 
// tens of thousands of concurrent connections.
listener, _ := net.Listen("tcp", ":8080")
for {
    conn, _ := listener.Accept()
    // Spawns a lightweight 2KB goroutine, NOT a heavy OS thread
    go handleConnection(conn) 
}
```

#### CGI
> Can you discuss why CGI became obsolete, and was instead replaced with other architectural approaches?

**Expert Answer:**

**The Short Answer:** 
CGI became obsolete because it spawned a completely new, heavy OS process for every single HTTP request, causing servers to run out of memory under load.

**The Deep Dive:** 
In the early web, the Common Gateway Interface (CGI) executed a script (like Perl) by forking a new process. This is incredibly slow and resource-intensive. If 1,000 users hit the site, the server spawned 1,000 processes and crashed. It was replaced by FastCGI (which pooled processes) and modules like `mod_php`. Today, languages like Go compile a hyper-efficient HTTP server directly into the binary, utilizing multiplexed threads instead of heavy processes, offering orders of magnitude better performance.

**The Trade-offs (Pros/Cons):**
* **Pros (of CGI):** Language agnostic; absolute isolation (if one script crashed, it didn't crash the server).
* **Cons:** Abysmal performance; completely unscalable for modern web traffic.

**Code Example:**
```go
// Go replaces the need for CGI completely by embedding a 
// highly concurrent HTTP server directly in the binary.
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, World!")) // Served via goroutines, not processes
    })
    http.ListenAndServe(":8080", nil)
}
```

#### Vendor Lock-in
> How would you defend the design of your systems against vendor Lock-in?

**Expert Answer:**

**The Short Answer:** 
Defend against vendor lock-in by using the Hexagonal Architecture (Ports and Adapters) to abstract away cloud-specific SDKs behind interfaces.

**The Deep Dive:** 
Write your core business logic in pure code with zero third-party dependencies. If you need to save data, define a `Repository` interface. Write an "Adapter" that implements that interface using AWS DynamoDB. The business logic only talks to the interface. If AWS prices skyrocket, you simply write a new Adapter for Google Cloud Spanner. The core business logic never changes because it is blissfully unaware of the cloud provider.

**The Trade-offs (Pros/Cons):**
* **Pros:** Freedom to migrate providers; makes unit testing trivial (mocking the interface instead of the cloud provider).
* **Cons:** "Lowest Common Denominator" effect. Abstracting too heavily prevents you from utilizing the unique, powerful features of a specific vendor.

**Code Example:**
```go
// Port (Interface owned by your domain)
type Storage interface {
    Save(data string) error
}

// Adapter (Vendor specific, kept at the edges of the app)
type S3Adapter struct { /* AWS deps */ }
func (s *S3Adapter) Save(data string) error { return s3.PutObject(...) }

// Business Logic (Pure, Vendor Agnostic)
func Process(s Storage) {
    s.Save("Important Data") // Works with S3, GCS, or Mocks!
}
```

#### CPUs
> What's new in CPUs since the 80s, and how does it affect programming?

**Expert Answer:**

**The Short Answer:** 
CPUs stopped gaining clock speed and instead scaled out into multiple cores and deep cache hierarchies (L1/L2/L3), revolutionizing how we write performant code.

**The Deep Dive:** 
1.  **Multi-core:** To utilize hardware, programmers must write concurrent code (hence the rise of Go and its goroutines).
2.  **Cache Lines:** Memory access is astronomically slow compared to modern CPU cycles. CPUs now pre-fetch memory into blazing-fast local caches. 
*Impact:* This birthed **Data-Oriented Design**. Iterating over contiguous memory (an array of structs) is drastically faster than iterating over an array of pointers to structs, because following random pointers causes "cache misses" forcing the CPU to wait for slow main memory.

**The Trade-offs (Pros/Cons):**
* **Pros (of DOD/Contiguous Memory):** Massive performance gains; maximizes CPU cache utilization.
* **Cons:** Focusing on memory layout can conflict with traditional Object-Oriented paradigms which favor pointer-heavy object graphs.

**Code Example:**
```go
// BAD for CPU Cache: Array of pointers (Heap fragmentation)
type Player struct { ID int }
var players []*Player 

// GOOD for CPU Cache: Contiguous slice of values.
// The CPU pre-fetches the entire block into L1 cache, making iteration blazingly fast.
var players []Player
```

#### DDOS
> How could a denial of service arise not maliciously but due to a design or architectural problem?

**Expert Answer:**

**The Short Answer:** 
Self-inflicted DoS typically arises from the "Thundering Herd" problem or missing rate limits on expensive internal endpoints.

**The Deep Dive:** 
If an expensive database query (e.g., aggregating global statistics) is cached for 10 minutes, the server hums along happily. But the second the cache expires, if 1,000 concurrent requests hit the server, all 1,000 requests miss the cache simultaneously and query the database. The database CPU spikes to 100% and crashes the entire system. This is a purely architectural DoS.

**The Trade-offs (Pros/Cons):**
* **Pros (of fixing it):** Bulletproof system resilience under viral load.
* **Cons:** Requires implementing cache-stampede protection (singleflight) or asynchronous cache warming, which adds complexity.

**Code Example:**
```go
import "golang.org/x/sync/singleflight"
var g singleflight.Group

func GetExpensiveData() (string, error) {
    // If 1,000 requests arrive, only ONE executes the internal function.
    // The other 999 wait and share the exact same result.
    v, err, _ := g.Do("cache_key", func() (interface{}, error) {
        return db.ExpensiveQuery()
    })
    return v.(string), err
}
```
